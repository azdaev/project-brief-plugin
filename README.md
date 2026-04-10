# Project Brief Plugin

A Claude Code plugin that collects project requirements through interactive HTML forms and generates technical specifications.

## Install

In Claude Code, run:

```
/plugin marketplace add azdaev/project-brief-plugin
/plugin install project-brief@amady-plugins
```

## Usage

- `/project-brief` — Start collecting requirements for a new project
- `/project-brief <type>` — Start with a specific project type (e.g., "web app", "CLI tool", "telegram bot")
- `/project-brief continue` — Continue from the last saved round
- `/project-brief spec` — Generate the final technical specification

## How it works

1. You run `/project-brief` and describe what you're building
2. The plugin generates an interactive HTML form with questions tailored to the project type
3. You fill it out in the browser and paste the copied answers back into chat
4. It analyzes your answers, identifies gaps, and generates follow-up forms if needed
5. Once everything is clear, it produces a full `SPEC.md` technical specification

Rounds and answers are saved to `.brief/` in your working directory so you can pick up where you left off across sessions.

### The form

A single, calm reading surface — notebook feel, not dashboard. Features:

- Light and dark modes (click the half-filled disc in the top-right)
- Left-margin progress spine: one dot per section, fills as you complete them
- Underline-only inputs, no boxed cards, one reading column
- Radio groups always include an "other" option with a free-text input
- Copy button emits clean markdown plus a machine-readable JSON block for reliable re-parsing
- Answers auto-save to localStorage so a refresh doesn't lose work
