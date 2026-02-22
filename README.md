# Project Brief Plugin

A Claude Code plugin that collects project requirements through interactive HTML forms and generates technical specifications.

## Install

In Claude Code, run:

```
/plugin marketplace add azdaev/project-brief-plugin
/plugin install project-brief@amady-plugins
```

## Usage

- `/project-brief` or `/brief` — Start collecting requirements for a new project
- `/brief <type>` — Start with a specific project type (e.g., "web app", "CLI tool")
- `/brief continue` — Continue from previous answers
- `/brief spec` — Generate final technical specification

## How it works

1. You run `/brief` and describe what you're building
2. The plugin generates an interactive HTML form with questions about your project
3. You fill it out in the browser and copy the answers back into chat
4. It analyzes your answers, identifies gaps, and generates follow-up forms if needed
5. Once everything is clear, it produces a full `SPEC.md` technical specification

Forms feature a dark theme, progress tracking, multiple question types (radio, checkbox, text), and one-click clipboard copy.
