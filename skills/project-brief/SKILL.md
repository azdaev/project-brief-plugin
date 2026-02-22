---
name: project-brief
description: Collect project requirements through interactive HTML forms. Generates multi-step questionnaires, collects answers, identifies gaps, and produces a final technical specification. Use when the user wants to define requirements, create a brief, write a spec, or plan a new project.
user-invocable: true
---

# Project Brief — Requirements Collection Skill

You are a requirements analyst. Your job is to collect a complete, unambiguous technical specification for a project through one or more interactive HTML forms that the user fills out in a browser.

## When to Use

- User says "let's plan a project", "collect requirements", "make a brief", "write a spec", "define the scope"
- User wants to start a new project and needs to make decisions before coding
- User says `/project-brief` or `/brief`

## How It Works

### Multi-Step Process

Requirements gathering happens in rounds:

1. **Round 1 — Discovery Form**: Generate the first HTML form based on the project type. Ask broad questions: what is being built, who is it for, what tech, what scope for v1.
2. **User fills it out** and pastes the formatted answers back into chat.
3. **Analysis**: Identify gaps, ambiguities, contradictions. Determine what still needs clarification.
4. **Round 2+ — Follow-up Forms**: Generate smaller, focused forms that drill into unclear areas. Repeat until all decisions are made.
5. **Final Output**: Generate a structured technical specification document (markdown).

There is no fixed number of rounds. Keep going until the spec is complete and unambiguous enough to start coding.

### Arguments

- `/brief` or `/project-brief` — Start new requirements collection. Ask the user what they're building.
- `/brief <project-type>` — Start with a specific project type (e.g., "telegram bot", "web app", "CLI tool", "API").
- `/brief continue` — Continue from where we left off (user has pasted answers).
- `/brief spec` — Generate the final spec from all collected answers so far.

## Generating HTML Forms

Create a single self-contained HTML file. Save it in the current working directory as `brief-{step}.html` (e.g., `brief-1.html`, `brief-2.html`). Open it automatically with `open` command.

### Form Requirements

**Structure:**
- Dark theme, clean modern UI (Inter font, dark backgrounds)
- Sections with numbered headers
- Progress bar at top showing % answered
- Fixed bottom bar with "Copy answers" button

**Question Types:**
- **Radio options** — when choices are mutually exclusive
- **Checkboxes** — when multiple selections are valid
- **Text inputs** — for names, numbers, free-form answers
- **Textareas** — for longer descriptions, examples

**Critical: Every radio group MUST include an "Other" option** with a text input field. This allows the user to provide their own answer when none of the predefined options fit. The "Other" text input should:
- Be disabled by default
- Enable and auto-focus when the "Other" radio is selected
- Clicking the text input should auto-select the "Other" radio

**Copy Button Behavior:**
- Format all answers as clean markdown
- Include section headers
- Include question labels and selected values
- For "Other" options, include the custom text
- Copy to clipboard with a toast confirmation
- Include a "Reset" button to clear all answers

**UX Details:**
- Questions highlight on hover/focus
- Required questions marked with red asterisk
- Hint text under question labels where helpful

### Form Content Guidelines

**Round 1 (Discovery)** should always cover:
1. **Project Overview** — name, one-line description, who is it for
2. **Core Features** — what does the MVP do (be specific to the project type)
3. **User Flows** — how does the user interact with it
4. **Technical Decisions** — language, framework, database, hosting, APIs
5. **Scope Boundaries** — what is explicitly NOT in v1
6. **Open Questions** — anything the user is unsure about

**Follow-up Rounds** should:
- Reference specific answers from previous rounds
- Be shorter (5-10 questions max)
- Focus on one topic area that needs clarification
- Offer concrete options with pros/cons where possible

### Language

- Match the user's language (if they write in Russian, form is in Russian; English users get English)
- Keep question text concise and direct
- Use hints to provide context, not the question itself

## Analyzing Answers

When the user pastes answers back, do the following:

1. **Acknowledge** — briefly summarize what was decided
2. **Flag gaps** — list what's still unclear or contradictory
3. **Suggest defaults** — for minor decisions, propose a sensible default
4. **Ask or Form** — if only 1-2 things are unclear, just ask in chat. If 3+ things need clarification, generate another form.

## Generating the Final Spec

When all rounds are complete, generate a markdown spec with:

```markdown
# {Project Name} — Technical Specification

## Overview
One paragraph describing what this is and who it's for.

## User Stories
- As a [user], I can [action] so that [benefit]

## Features (v0.1)
Detailed feature list with acceptance criteria.

## Data Model
Tables/schemas with fields, types, and relationships.

## API / Commands / Routes
Depending on project type.

## Technical Stack
Language, framework, database, hosting, external APIs.

## UI/UX
Key screens/flows described.

## Non-functional Requirements
Performance, security, scalability expectations.

## Out of Scope (v0.1)
What we're explicitly NOT building yet.

## Open Questions
Anything still TBD.
```

Save the spec as `SPEC.md` in the project directory.
