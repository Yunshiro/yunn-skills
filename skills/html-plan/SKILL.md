---
name: html-plan
description: Use this skill when the user asks to create, edit, review, or standardize a planning document that should be delivered as a standalone HTML file. This includes project plans, implementation plans, migration plans, release plans, research plans, and technical execution plans where readable structure, scannable sections, and portable HTML output matter.
---

# HTML Plan Skill

Create clear, standalone HTML planning documents for technical or project work.

## When to Use

Use this skill when the user wants a plan document in HTML format, especially for:

- Implementation plans
- Project or roadmap plans
- Migration or rollout plans
- Release plans
- Research or investigation plans
- Technical decision or execution plans

Do not use this skill for interactive web apps, marketing pages, dashboards, or rich UI prototypes unless the user specifically asks for a plan document.

## Output Requirements

Produce a single self-contained `.html` file unless the user requests otherwise.

The document should:

- Use semantic HTML5: `header`, `main`, `section`, `article`, `nav`, `footer` where appropriate.
- Include a concise document title in both `<title>` and the visible `<h1>`.
- Use a readable structure with stable heading levels: one `h1`, then `h2` for major sections, `h3` for subsections.
- Include a small table of contents when the document has more than four major sections.
- Prefer tables for timelines, milestones, owners, risks, dependencies, and status matrices.
- Keep styling embedded in a single `<style>` block.
- Avoid external scripts, remote fonts, tracking pixels, or network-dependent assets.
- Work when opened directly from the filesystem.

## Recommended Document Structure

Use this structure as a default and adapt it to the user's context:

1. Overview
2. Goals and Non-Goals
3. Scope
4. Assumptions
5. Workstreams or Phases
6. Timeline or Milestones
7. Dependencies
8. Risks and Mitigations
9. Open Questions
10. Next Actions

For technical plans, also consider:

- Architecture Summary
- Implementation Steps
- Testing and Validation
- Rollout and Rollback
- Observability
- Security and Privacy Considerations

## HTML Style Guidelines

Use the fixed **Standard Plan Theme** below for every generated plan document. Do not adapt the plan document's visual style to the surrounding project, product, brand, app UI, selected design option, or existing site palette. The plan document should look like a neutral execution document, not an extension of the product being planned.

Fixed style rules:

- Palette: page `#f6f8fb`, paper `#ffffff`, text `#111827`, muted text `#4b5563`, border `#d1d5db`, accent `#2563eb`, accent-soft `#eff6ff`, code background `#111827`.
- Layout: centered content, max width `1040px`, page padding `32px 16px 48px`.
- Shape: sections, cards, tables, callouts, and code blocks use `8px` radius.
- Typography: system sans-serif only; body `16px`; line height around `1.65`; one `h1`; `h2` for major sections; `h3` for subsections.
- Header: plain document header only. No hero layout, no preview-as-hero, no gradients, no decorative imagery, no product-style composition.
- Navigation: if used, render as a simple table-of-contents card with text links.
- Sections: white cards with subtle border; use light blue callouts for recommendations or notes.
- Tables: full-width, clear headers, light blue header background, visible row borders.
- Code: dark code blocks with light text; inline code uses light blue background.
- Print: include print-friendly CSS that removes background effects and avoids breaking sections.

Use this CSS as the starting point unless the user explicitly provides another document style:

```css
:root {
  color-scheme: light;
  --page: #f6f8fb;
  --paper: #ffffff;
  --ink: #111827;
  --muted: #4b5563;
  --line: #d1d5db;
  --accent: #2563eb;
  --accent-soft: #eff6ff;
  --code-bg: #111827;
  --code-ink: #f9fafb;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: var(--page);
  color: var(--ink);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  font-size: 16px;
  line-height: 1.65;
}

.page {
  width: min(1040px, calc(100vw - 32px));
  margin: 0 auto;
  padding: 32px 0 48px;
}

header,
nav,
section,
footer {
  border: 1px solid var(--line);
  border-radius: 8px;
  background: var(--paper);
  padding: 24px;
}

main {
  display: grid;
  gap: 16px;
}

h1,
h2,
h3,
p {
  margin: 0;
}

h1 {
  font-size: 36px;
  line-height: 1.15;
}

h2 {
  margin-bottom: 12px;
  font-size: 24px;
}

h3 {
  margin: 18px 0 8px;
  font-size: 18px;
}

p,
li,
td {
  color: var(--muted);
}

a {
  color: var(--accent);
}

table {
  width: 100%;
  border-collapse: collapse;
  border: 1px solid var(--line);
  border-radius: 8px;
  overflow: hidden;
}

th,
td {
  border-bottom: 1px solid var(--line);
  padding: 10px 12px;
  text-align: left;
  vertical-align: top;
}

th {
  background: var(--accent-soft);
  color: var(--ink);
}

code {
  border-radius: 4px;
  background: var(--accent-soft);
  color: var(--ink);
  padding: 2px 5px;
}

pre {
  overflow: auto;
  border-radius: 8px;
  background: var(--code-bg);
  color: var(--code-ink);
  padding: 16px;
}

pre code {
  background: transparent;
  color: inherit;
  padding: 0;
}

.callout {
  border-left: 4px solid var(--accent);
  border-radius: 8px;
  background: var(--accent-soft);
  padding: 14px 16px;
}

@media print {
  body {
    background: #fff;
  }

  section {
    break-inside: avoid;
  }
}
```

## Content Guidance

Plans should be specific enough to execute:

- Use concrete action items with owners or owner placeholders.
- Make dependencies and sequencing explicit.
- Mark unknowns as open questions instead of hiding them.
- Separate facts, assumptions, and recommendations.
- Prefer concise bullets over long prose unless the plan requires rationale.

## Quality Checklist

Before finalizing, verify that:

- The HTML file opens without build tooling.
- Heading levels are logical and not skipped.
- Tables have clear headers.
- Dates, owners, and statuses are consistently formatted.
- The plan includes risks, dependencies, and next actions.
- The document title and filename match the plan topic.
