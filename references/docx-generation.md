# Generating .docx Resumes with Node.js

This reference covers the technical approach for generating professional resume documents using the `docx` npm library.

## Setup

```bash
npm install docx
```

## Architecture

The resume generator is a single Node.js script that:
1. Defines formatting constants (fonts, colors, sizes, margins)
2. Provides helper functions for repeated patterns (section headers, experience entries, bullets)
3. Builds a `Document` object with all content
4. Writes the buffer to a .docx file

## Formatting Constants

```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
        AlignmentType, LevelFormat, ExternalHyperlink,
        BorderStyle, WidthType } = require('docx');
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

// Invisible table borders (for layout tables)
const NB = { style: BorderStyle.NONE, size: 0, color: "FFFFFF" };
const NBS = { top: NB, bottom: NB, left: NB, right: NB };
```

### Notes on Units

- **Font sizes** in docx-js are in **half-points**. So `size: 19` = 9.5pt.
- **Spacing** values (before/after on paragraphs) are in **twips**. 1pt = 20 twips.
- **Page dimensions and margins** are also in twips.

## Helper Functions

### Section Header

Creates an uppercase, letter-spaced heading with a blue underline rule:

```javascript
function sh(text) {
    return new Paragraph({
        spacing: { before: 80, after: 15 },
        border: { bottom: { style: BorderStyle.SINGLE, size: 1, color: C2 } },
        children: [new TextRun({
            text: text.toUpperCase(),
            font: FONT, size: SZ_SECTION, bold: true, color: C1,
            characterSpacing: 50
        })]
    });
}
```

### Experience Entry Header

A two-row, two-column table with no visible borders. Row 1: title (left) + dates (right). Row 2: company + location (left).

```javascript
function eh(title, company, location, dates) {
    const cL = Math.round(CW * 0.72), cR = CW - cL;
    return new Table({
        width: { size: CW, type: WidthType.DXA },
        columnWidths: [cL, cR],
        rows: [
            new TableRow({ children: [
                new TableCell({ borders: NBS, width: { size: cL, type: WidthType.DXA },
                    children: [new Paragraph({ spacing: { before: 0, after: 0 }, children: [
                        new TextRun({ text: title, font: FONT, size: SZ_BODY, bold: true, color: C1 }),
                    ]})] }),
                new TableCell({ borders: NBS, width: { size: cR, type: WidthType.DXA },
                    children: [new Paragraph({ alignment: AlignmentType.RIGHT, spacing: { before: 0, after: 0 }, children: [
                        new TextRun({ text: dates, font: FONT, size: SZ_SMALL, color: CL }),
                    ]})] }),
            ]}),
            new TableRow({ children: [
                new TableCell({ borders: NBS, width: { size: cL, type: WidthType.DXA },
                    children: [new Paragraph({ spacing: { before: 0, after: 10 }, children: [
                        new TextRun({ text: company, font: FONT, size: SZ_BODY, italics: true, color: CB }),
                        new TextRun({ text: location ? `  |  ${location}` : "", font: FONT, size: SZ_SMALL, color: CL }),
                    ]})] }),
                new TableCell({ borders: NBS, width: { size: cR, type: WidthType.DXA },
                    children: [new Paragraph({ spacing: { before: 0, after: 0 }, children: [] })] }),
            ]}),
        ]
    });
}
```

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
            eh("Job Title", "Company Name", "City, ST", "2023 \u2013 Present"),
            b("Impact-first bullet point here..."),
            b("Another bullet point..."),

            // EDUCATION
            sh("Education"),
            // Use tables for degree | school | year alignment
            // (see full pattern below)

            // CORE COMPETENCIES
            sh("Core Competencies"),
            // Category: Skill | Skill | Skill format
            // Use SZ_SMALL for competency text to save vertical space
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

## Education Rows

Each education entry is a single-row, two-column table:

```javascript
["MBA, Finance|University of Texas|2024",
 "B.S., Computer Science|Stanford University|2018"].map(row => {
    const [deg, sch, yr] = row.split("|");
    const cL = Math.round(CW * 0.72), cR = CW - cL;
    return new Table({
        width: { size: CW, type: WidthType.DXA },
        columnWidths: [cL, cR],
        rows: [new TableRow({ children: [
            new TableCell({ borders: NBS, width: { size: cL, type: WidthType.DXA },
                children: [new Paragraph({ spacing: { before: 5, after: 5 }, children: [
                    new TextRun({ text: deg, font: FONT, size: SZ_BODY, bold: true, color: C1 }),
                    new TextRun({ text: "  \u2013  ", font: FONT, size: SZ_BODY, color: CB }),
                    new TextRun({ text: sch, font: FONT, size: SZ_BODY, italics: true, color: CB }),
                ]})] }),
            new TableCell({ borders: NBS, width: { size: cR, type: WidthType.DXA },
                children: [new Paragraph({ alignment: AlignmentType.RIGHT, spacing: { before: 5, after: 5 }, children: [
                    new TextRun({ text: yr, font: FONT, size: SZ_SMALL, color: CL }),
                ]})] }),
        ]})]
    });
})
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
