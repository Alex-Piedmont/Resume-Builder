# Generating .docx Resumes with Node.js

This reference covers the technical approach for generating ATS-friendly resume documents using the `docx` npm library.

## ATS Compatibility Principles

Applicant Tracking Systems parse .docx files by reading the underlying XML. To ensure reliable parsing:

1. **Use Word heading styles** — ATS systems identify sections (Experience, Education) by looking for `Heading1`, `Heading2`, etc. in the document styles. Plain bold/colored text is invisible to them.
2. **Never use tables for layout** — Tables confuse ATS parsers. Use tab stops for multi-column alignment (title + date on same line).
3. **No text boxes, columns, or images** — Stick to a single-column flow of paragraphs.
4. **Use standard section names** — "Professional Experience", "Education", "Core Competencies" are universally recognized.

## Setup

```bash
npm install docx
```

## Architecture

The resume generator is a single Node.js script that:
1. Defines document-level styles (headings, body, bullets)
2. Defines formatting constants (fonts, colors, sizes, margins)
3. Provides helper functions for repeated patterns (section headers, experience entries, bullets)
4. Builds a `Document` object with all content
5. Writes the buffer to a .docx file

## Formatting Constants

```javascript
const { Document, Packer, Paragraph, TextRun, TabStopPosition, TabStopType,
        AlignmentType, LevelFormat, ExternalHyperlink,
        BorderStyle, HeadingLevel, convertInchesToTwip } = require('docx');
const fs = require('fs');

// Typography
const FONT = "Calibri";
const SZ_NAME = 26;      // 13pt (half-points)
const SZ_CONTACT = 17;   // 8.5pt
const SZ_SECTION = 20;   // 10pt
const SZ_BODY = 19;      // 9.5pt
const SZ_SMALL = 17;     // 8.5pt

// Colors (hex without #)
const C1 = "1F3864";     // Navy — name, section headers, titles
const C2 = "2E75B6";     // Medium blue — underlines, hyperlinks
const CB = "333333";     // Dark gray — body text
const CL = "666666";     // Light gray — dates, locations, contact info

// Page geometry (all values in twips: 1 inch = 1440 twips)
const PW = 12240;        // 8.5" page width
const PH = 15840;        // 11" page height
const ML = 720, MR = 720;  // 0.5" left/right margins
const MT = 460, MB = 400;  // ~0.32" / ~0.28" top/bottom margins
const CW = PW - ML - MR;   // Content width
```

### Notes on Units

- **Font sizes** in docx-js are in **half-points**. So `size: 19` = 9.5pt.
- **Spacing** values (before/after on paragraphs) are in **twips**. 1pt = 20 twips.
- **Page dimensions and margins** are also in twips.

## Document Styles

Define heading styles so ATS systems can identify resume structure. These styles live in the `Document` constructor:

```javascript
const doc = new Document({
    styles: {
        paragraphStyles: [
            {
                id: "Heading2",
                name: "Heading 2",
                basedOn: "Normal",
                next: "Normal",
                run: {
                    font: FONT,
                    size: SZ_SECTION,
                    bold: true,
                    color: C1,
                    characterSpacing: 50,
                    allCaps: true,
                },
                paragraph: {
                    spacing: { before: 80, after: 15 },
                    border: { bottom: { style: BorderStyle.SINGLE, size: 1, color: C2 } },
                },
            },
            {
                id: "Heading3",
                name: "Heading 3",
                basedOn: "Normal",
                next: "Normal",
                run: {
                    font: FONT,
                    size: SZ_BODY,
                    bold: true,
                    color: C1,
                },
                paragraph: {
                    spacing: { before: 0, after: 0 },
                },
            },
        ],
    },
    // ... numbering, sections (see below)
});
```

**Why this matters:** When an ATS encounters `<w:pStyle w:val="Heading2"/>` in the XML, it knows that paragraph is a section title. Without a heading style, section headers are just bold-colored text — visually identical to a human reader, but invisible to automated parsing.

## Helper Functions

### Section Header

Creates a Heading 2 paragraph. ATS systems will recognize this as a section delimiter:

```javascript
function sh(text) {
    return new Paragraph({
        heading: HeadingLevel.HEADING_2,
        children: [new TextRun({ text: text })]
    });
}
```

The style definition on Heading2 handles the uppercase, letter-spacing, color, and bottom border. The `sh()` function just needs to set the heading level and provide the text.

### Experience Entry Header

Uses tab stops instead of tables. A right-aligned tab stop pushes the date to the right margin. Two paragraphs: one for title + date, one for company + location.

```javascript
function eh(title, company, location, dates) {
    return [
        // Row 1: Title (left) + Dates (right-aligned via tab stop)
        new Paragraph({
            heading: HeadingLevel.HEADING_3,
            tabStops: [{ type: TabStopType.RIGHT, position: CW }],
            children: [
                new TextRun({ text: title }),
                new TextRun({ text: "\t" }),
                new TextRun({ text: dates, font: FONT, size: SZ_SMALL, color: CL, bold: false }),
            ]
        }),
        // Row 2: Company + Location
        new Paragraph({
            spacing: { before: 0, after: 10 },
            children: [
                new TextRun({ text: company, font: FONT, size: SZ_BODY, italics: true, color: CB }),
                new TextRun({ text: location ? `  |  ${location}` : "", font: FONT, size: SZ_SMALL, color: CL }),
            ]
        }),
    ];
}
```

**Important:** `eh()` returns an **array** of two paragraphs. Use the spread operator (`...eh(...)`) when adding to the section's `children` array.

**Why Heading 3 for job titles:** ATS systems use heading hierarchy to associate bullets with roles. A bullet following a Heading 3 is understood to belong to that role. Without the heading style, the ATS may lump all bullets together or fail to identify individual positions.

### Bullet Point

```javascript
function b(text) {
    return new Paragraph({
        numbering: { reference: "bul", level: 0 },
        spacing: { before: 5, after: 5 },
        children: [new TextRun({ text: text, font: FONT, size: SZ_BODY, color: CB })]
    });
}
```

## Document Assembly

```javascript
const doc = new Document({
    styles: {
        // ... (styles block from above)
    },
    numbering: { config: [{
        reference: "bul",
        levels: [{
            level: 0,
            format: LevelFormat.BULLET,
            text: "\u2022",
            alignment: AlignmentType.LEFT,
            style: {
                paragraph: { indent: { left: 340, hanging: 170 } },
                run: { font: FONT, size: SZ_BODY }
            }
        }]
    }] },
    sections: [{
        properties: {
            page: {
                size: { width: PW, height: PH },
                margin: { top: MT, bottom: MB, left: ML, right: MR }
            }
        },
        children: [
            // NAME (centered, large, letter-spaced)
            new Paragraph({
                alignment: AlignmentType.CENTER,
                spacing: { after: 15 },
                children: [new TextRun({
                    text: "FULL NAME",
                    font: FONT, size: SZ_NAME, bold: true, color: C1,
                    characterSpacing: 80
                })]
            }),

            // CONTACT LINE (centered, gray)
            new Paragraph({
                alignment: AlignmentType.CENTER,
                spacing: { after: 20 },
                children: [
                    new TextRun({
                        text: "City, ST  |  555-123-4567  |  email@example.com  |  ",
                        font: FONT, size: SZ_CONTACT, color: CL
                    }),
                    new ExternalHyperlink({
                        link: "https://linkedin.com/in/example",
                        children: [new TextRun({
                            text: "LinkedIn",
                            font: FONT, size: SZ_CONTACT, color: C2, underline: {}
                        })]
                    }),
                ]
            }),

            // PROFESSIONAL SUMMARY
            sh("Professional Summary"),
            new Paragraph({
                spacing: { before: 15, after: 25 },
                children: [new TextRun({
                    text: "3-4 sentence impact summary here...",
                    font: FONT, size: SZ_BODY, color: CB
                })]
            }),

            // PROFESSIONAL EXPERIENCE
            sh("Professional Experience"),
            ...eh("Job Title", "Company Name", "City, ST", "2023 \u2013 Present"),
            b("Impact-first bullet point here..."),
            b("Another bullet point..."),

            // EDUCATION
            sh("Education"),
            // (see Education section below)

            // CORE COMPETENCIES
            sh("Core Competencies"),
            // (see Core Competencies section below)
        ]
    }]
});

// Write to file
Packer.toBuffer(doc).then(buf => {
    const path = "output/Resume.docx";
    fs.writeFileSync(path, buf);
    console.log(`Written: ${path} (${buf.length} bytes)`);
});
```

## Education Entries

Each education entry uses tab stops for alignment — no tables:

```javascript
function eduEntry(degree, school, year) {
    return new Paragraph({
        tabStops: [{ type: TabStopType.RIGHT, position: CW }],
        spacing: { before: 5, after: 5 },
        children: [
            new TextRun({ text: degree, font: FONT, size: SZ_BODY, bold: true, color: C1 }),
            new TextRun({ text: "  \u2013  ", font: FONT, size: SZ_BODY, color: CB }),
            new TextRun({ text: school, font: FONT, size: SZ_BODY, italics: true, color: CB }),
            new TextRun({ text: "\t" }),
            new TextRun({ text: year, font: FONT, size: SZ_SMALL, color: CL }),
        ]
    });
}

// Usage:
eduEntry("MBA, Finance", "University of Texas", "2024"),
eduEntry("B.S., Computer Science", "Stanford University", "2018"),
```

## Core Competencies

Use grouped categories with bold labels:

```javascript
new Paragraph({
    spacing: { before: 20, after: 10 },
    children: [
        new TextRun({ text: "Strategy: ", font: FONT, size: SZ_SMALL, bold: true, color: C1 }),
        new TextRun({
            text: "Corporate Strategy  |  Market Intelligence  |  Financial Modeling",
            font: FONT, size: SZ_SMALL, color: CB
        }),
    ]
})
```

## Page Fit Estimation

When content is tight, estimate vertical space to predict page overflow. Key values:

- Line height ≈ font_size_halfpts × 10 × 1.15 (twips)
- Available height = PH - MT - MB = 14,980 twips with current margins
- A 2-line bullet at SZ_BODY ≈ 447 twips (including 5+5 spacing)
- A 3-line bullet ≈ 665 twips
- Section header ≈ 325 twips (80 before + text + 15 after)
- Experience entry header ≈ 452 twips (title row + company row + 10 after)

If the estimate shows overflow, tighten in this order:
1. Reduce bullet count on older roles
2. Trim wordier bullets by a few words to eliminate a wrapped line
3. Reduce margins (minimum ~0.5" left/right, ~0.25" top/bottom)
4. Reduce spacing values on bullets, headers, experience entries
5. Use SZ_SMALL for Core Competencies section
