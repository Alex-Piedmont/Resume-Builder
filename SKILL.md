---
name: resume
description: >
  Generate impact-driven, executive-level resumes from a structured markdown source file.
  Use this skill whenever the user wants to: create a tailored resume for a specific job posting,
  build a reusable resume source file from prior experience, generate a .docx resume with
  professional formatting, or rewrite resume bullets to lead with business outcomes.
  Trigger on mentions of "resume", "CV", "job application", "tailor my experience",
  or any request to match experience against a job description.
argument-hint: [job posting URL or description]
allowed-tools: Read, Grep, Glob, Write, Edit, Bash, WebFetch, WebSearch, AskUserQuestion, Task
---

# Impact Resume

Generate polished, single-page executive resumes from a structured markdown source file. The source file captures a person's complete career history, and this skill transforms the relevant pieces into an impact-first narrative tailored to a specific job posting.

User's request: $ARGUMENTS

## Philosophy

Most people describe their work in terms of activities: "Led a team," "Managed a project," "Worked with stakeholders." Hiring managers care about outcomes: what changed because you were there? This skill treats the user's source material as *ingredients*, not text to polish. Every bullet should answer "so what?" before describing "what I did."

**Activity-first (avoid):** Led enterprise BI transformation, migrating from legacy platforms to Snowflake/Sigma
**Impact-first (goal):** Shaped how 3,500+ sellers made pipeline decisions by designing the enterprise BI strategy, owning the migration to Snowflake/Sigma and delivering 9 new dashboards within months

The difference: the impact-first version opens with the business outcome (3,500 people making better decisions), then uses the activity as evidence.

## Workflow

### Step 1: Gather Inputs

You need two things:

1. **A resume source file** — a structured markdown file containing the user's complete career history (see `assets/resume-source-template.md` for the format). If the user doesn't have one yet, help them build it first using the template.

2. **A job posting** — a URL or pasted description of the target role. Extract from it:
   - The exact job title and company
   - Key responsibilities and requirements
   - Preferred qualifications and skills
   - Industry context and team structure

If you can't access the job URL directly, search the web for the posting details using the job title, company, and any job ID provided.

### Step 2: Analyze Fit

Before writing anything, map the user's experience to the job requirements:

- Which roles from the source file are most relevant?
- Which "Key Wins" align with the posting's priorities?
- What skills from the posting does the user demonstrably have?
- Where are the gaps, and can adjacent experience fill them credibly?

Share this analysis with the user briefly. It helps calibrate expectations and surfaces any title or framing decisions.

### Step 3: Pre-Draft Questions

Use `AskUserQuestion` to ask the user targeted questions based on the gap analysis from Step 2. Focus on primary considerations only:

- **Experience gaps** the JD requires that the CV doesn't clearly demonstrate (e.g., "Do you have variable compensation experience?")
- **Title/framing decisions** where multiple options exist (e.g., "Should we use 'Client Director' or 'Practice Group Leader' for Evalueserve?")
- **Overlapping dates or ambiguities** that need clarification (e.g., "Was Piedmont part-time alongside Workday?")
- **Critical context** that would change bullet strategy

No fixed cap on questions — ask as many as needed but only for primary gaps, not secondary concerns. Use multiple `AskUserQuestion` calls if the questions span different topics.

### Step 4: Select, Structure, and Write

Incorporate the user's answers from Step 3 into all decisions below.

**Title selection:** The source file may list multiple relevant titles per role. Pick the one that best mirrors the target posting's language.

**Bullet count per role:** Follow these guidelines based on seniority and recency:
- Most recent / most relevant role: 3-4 bullets
- Second tier roles: 2-3 bullets
- Older or less relevant roles: 1-2 bullets
- Very old roles (10+ years): consolidate into a single entry or omit

**Role consolidation:** Multiple roles at the same company can be:
- Listed separately (when distinct titles strengthen the narrative)
- Consolidated into one entry (when they tell a better combined story)
- The user's source file comments (lines starting with `#`) often contain guidance on this.

**What to include:**
- Professional Summary (3-4 sentences)
- Professional Experience (reverse chronological)
- Education
- Core Competencies (grouped by category)

**Writing impact-first bullets:** For each bullet, follow this pattern:

> [Business outcome / "so what"] + [by/through] + [what you actually did] + [with what scale/evidence]

Concrete techniques:
- **Quantify everything possible:** revenue, team size, time saved, percentage improvement, budget, number of stakeholders affected
- **Name the decision-maker influenced:** "persuaded BU vice president," "informed C-Suite planning," "board-level decision-making"
- **Use active, specific verbs:** "Reshaped," "Grew," "Drove," "Identified," "Built" — not "Responsible for," "Helped with," "Involved in"
- **Show scope:** "$250M business unit," "1,200-family patent portfolio," "9 R&D groups," "3,500+ sellers"
- **Connect to the posting:** if the role asks for "due diligence experience," make sure relevant bullets surface that phrase naturally

The user's source file provides raw material — wins, metrics, context, client names. Your job is to synthesize these into concise, outcome-led statements. Don't just rearrange their words; write original sentences that tell a coherent career story.

**Writing the Professional Summary:** The summary should be 3-4 sentences that:
1. Open with a positioning statement: "[Level] [function] who [core value proposition]"
2. Provide 1-2 proof points with specific numbers from the career
3. Close with credentials and breadth of capability

Example structure:
> Enterprise strategy leader who translates market intelligence into growth. Built and led teams that reshaped how organizations make strategic decisions — from redefining a $250M business unit's addressable market to architecting analytics platforms for 3,500+ stakeholders. Georgia Tech Executive MBA with a decade of competitive analysis, financial modeling, and cross-functional execution delivering measurable P&L impact.

**Generating the .docx:** Read `references/docx-generation.md` for the technical approach to generating the resume as a Word document using the `docx` npm library. The reference includes the complete code pattern with professional formatting (navy/blue color scheme, Calibri font, proper spacing).

Key formatting rules:
- **Target: 1 page** for senior manager level; up to 1.5 pages for VP+ with extensive experience
- **Margins:** 0.5" left/right, ~0.3" top/bottom (tight but professional)
- **Font:** Calibri, body text 9.5pt, name 13pt, section headers 10pt
- **Color scheme:** Navy (#1F3864) for headers, medium blue (#2E75B6) for accents, dark gray (#333333) for body
- **Section order:** Name/Contact → Professional Summary → Professional Experience → Education → Core Competencies
- **ATS-friendly:** no tables, no columns, no text boxes, no images. Use Word heading styles (Heading 2 for sections, Heading 3 for job titles) so ATS parsers can identify document structure. Use tab stops for title/date alignment instead of tables.

Write a Node.js script, run it with `node`, and save the .docx output to the current working directory.

After generating, attempt to convert to PDF (via LibreOffice CLI if available) to verify page count.

### Step 5: Review Agent

Launch a `Task` agent with `subagent_type: general-purpose` to review the draft resume against the full job description. Pass the agent the complete resume content and the full JD text. The agent should evaluate:

1. **Keyword alignment** — missing terms or phrases from the JD that should appear in the resume
2. **Narrative coherence** — does the career story work for this role? Are transitions logical?
3. **Company culture fit** — does the resume signal the right values? (The agent should web-search for well-known features of the target company — leadership principles, culture, interview style — to inform this assessment.)
4. **Specific bullet improvements** — concrete rewrites with rationale
5. **Gaps and risks** — what a hiring manager would question or flag
6. **Overall grade** (A/B/C) with justification

### Step 6: Present Analysis & Post-Review Questions

Present the reviewer's key findings to the user in a clear summary. Then use `AskUserQuestion` to ask about any gaps the user might be able to bridge with additional context or experience not in the source file. **Always do this round**, even if the review is positive — frame it as "anything else you'd like to add or adjust?"

### Step 7: Finalize

Incorporate user feedback from Step 6. Make targeted edits to the resume content, then regenerate the .docx (following the same generation process from Step 4). Present the final version to the user.

## Building a Source File from Scratch

If the user doesn't have a source file yet, help them build one using `assets/resume-source-template.md`. The process:

1. Ask them to share any existing resumes (Word docs, PDFs, LinkedIn export)
2. Extract all unique content, removing duplicates
3. Organize into the template structure with one section per employer
4. Help them fill in the metadata fields (Who is, Timeframe, Key Wins, etc.)
5. Add inline `# comments` for instructions to future Claude sessions

The source file is designed to be reusable across many job applications. It's worth investing time to make it comprehensive.

## Resources

### references/
- `docx-generation.md` — Technical reference for generating .docx resumes with Node.js, including the complete code pattern and formatting constants

### assets/
- `resume-source-template.md` — Sample markdown template showing the expected structure, with placeholder content and inline instructions explaining each field
