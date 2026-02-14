# Impact Resume Skill

A Claude skill for generating tailored, impact-driven executive resumes from a structured markdown source file.

## What This Does

This skill helps you maintain a single "master" markdown file with your complete career history, then generate polished, one-page .docx resumes targeted at specific job postings. The key differentiator is the **impact-first writing approach**: instead of editing your words into bullet points, Claude uses your experience as raw ingredients to write original statements that lead with business outcomes.

**Activity-first:** "Led enterprise BI transformation, migrating platforms"
**Impact-first:** "Shaped how 3,500+ sellers made pipeline decisions by designing the enterprise BI strategy and delivering 9 new dashboards within months"

## How It Works

1. **Build your source file once.** Use the template in `assets/resume-source-template.md` to create a comprehensive markdown file covering every role in your career. Include metrics, key wins, client names, tools, team sizes, and leadership context. The more raw material you provide, the better the output.

2. **Point Claude at a job posting.** Share a URL or paste the job description. Claude analyzes the requirements and maps them against your source file.

3. **Get a tailored resume.** Claude selects the most relevant roles, picks appropriate titles, writes impact-first bullets, and generates a formatted .docx file targeting a single page.

4. **Iterate.** Review the output and give feedback. Common adjustments: tightening length, splitting or consolidating roles, adjusting tone, adding missing experience.

## File Structure

```
impact-resume/
├── SKILL.md                              # Agent instructions (the skill itself)
├── README.md                             # This file
├── resume-source-template.md         # Template for building your source file
└── references/
    └── docx-generation.md                # Technical reference for .docx generation
```

## The Source File Format

The source file uses a simple structure:

- `##` headers for each employer (plus Contact, Education, Skills)
- `#` lines are comments/instructions to Claude (never included in output)
- Indented lines are sub-fields (Who is, Timeframe, Key Wins, etc.)
- `-` dashes are bullet points of experience
- `---` separates sections

Each employer section includes:
- **Who is [Company]?** — Brief description for context
- **Timeframe** — Employment dates
- **Relevant Titles** — All titles that could apply (Claude picks the best fit)
- **Key Wins** — Headline accomplishments with hard numbers
- **Relevant Experience** — Detailed bullets (raw material, not polished)
- **Leadership Style** — Stakeholder level, team size, operating cadence
- **Notable Clients** — If applicable

## Getting Started

1. Copy `assets/resume-source-template.md` and fill it in with your career history
2. Find a job posting you want to target
3. Ask Claude: "Generate a resume targeting [this role] using my source file"

## Tips

- **Be generous with raw material.** The source file isn't a resume — it's a database. Include everything, even if it seems redundant. Claude will select what's relevant.
- **Use the comment syntax.** Lines starting with `#` are instructions to Claude. Use them to note preferences: which title to use, what to emphasize, what to omit.
- **Update continuously.** Add new accomplishments as they happen. It's much easier to capture metrics when they're fresh.
- **One source file, many resumes.** The same source file can generate very different resumes depending on the target role. A strategy role will pull different bullets than a product management role.
