# Project Brief Plugin

A Claude Code plugin that collects project requirements through interactive HTML forms and generates technical specifications.

## What it does

- `/project-brief` or `/brief` — Start collecting requirements for a new project
- `/brief <type>` — Start with a specific project type (e.g., "web app", "CLI tool")
- `/brief continue` — Continue from previous answers
- `/brief spec` — Generate final technical specification

The skill generates dark-themed HTML forms with progress tracking, multiple question types, and clipboard copy. It works in rounds — each round narrows down requirements until the spec is complete.

## Install

```
/plugin marketplace add <your-github-username>/project-brief-plugin
/plugin install project-brief@amady-plugins
```

## Structure

```
.claude-plugin/
  marketplace.json    # Marketplace catalog
  plugin.json         # Plugin manifest
skills/
  project-brief/
    SKILL.md          # The skill definition
```
