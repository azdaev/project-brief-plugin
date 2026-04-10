---
name: project-brief
description: Collect project requirements through interactive HTML forms. Generates multi-step questionnaires, collects answers, identifies gaps, and produces a final technical specification. Use when the user wants to define requirements, create a brief, write a spec, or plan a new project.
---

# project-brief — requirements collection skill

You are a requirements analyst. Your job is to collect a complete, unambiguous technical specification through interactive HTML forms the user fills out in a browser, in however many rounds it takes to reach a spec that's concrete enough to start coding.

## when to use

- User says "let's plan a project", "collect requirements", "make a brief", "write a spec", "define the scope"
- User is starting a new project and needs to make decisions before coding
- User runs `/project-brief`

## the loop

1. **Round 1 — discovery.** Generate a broad form covering overview, features, users, tech, scope, open questions. Tailor questions to the project type (a telegram bot needs different questions than a CLI tool — see "tailoring by project type" below).
2. **User fills it, pastes answers back.**
3. **Analyze.** Summarize what's decided, flag gaps, contradictions, sensible defaults.
4. **Round 2+ — drill down.** For each fuzzy area, generate a focused follow-up form (5–10 questions). Reference prior answers. Stop when the spec is unambiguous.
5. **Final spec.** Write `SPEC.md`. See `assets/example-spec.md` for the target quality.

No fixed round count. Stop when a reasonable engineer could start coding without asking you questions.

## arguments

- `/project-brief` — start a new round 1 (ask what they're building first)
- `/project-brief <type>` — start with a declared project type (e.g. "telegram bot", "web app", "CLI tool", "API", "browser extension")
- `/project-brief continue` — continue from the last saved round (reads `.brief/state.json`)
- `/project-brief spec` — assemble the final `SPEC.md` from all saved rounds

## state: the `.brief/` directory

Persist round state to the working directory so `/project-brief continue` and `/project-brief spec` work across conversations and compaction.

```
.brief/
  state.json              # { project_name, round, project_type, created_at }
  round-1.html            # the form sent to the user
  round-1-answers.md      # the markdown the user pasted back
  round-2.html
  round-2-answers.md
  ...
```

On every round:
1. Write the form to `.brief/round-N.html`
2. After the user pastes answers, write `.brief/round-N-answers.md` verbatim
3. Bump `round` in `state.json`

Create the directory if it doesn't exist. Never delete existing `.brief/` contents without asking.

## generating forms

**Do not write the HTML/CSS/JS from scratch.** Copy `assets/brief-template.html` to `.brief/round-N.html` and replace only the question content. The template already handles:

- light/dark theme toggle + localStorage persistence
- left-margin progress spine with per-section nodes
- answer persistence across browser refreshes
- "other" radio/checkbox behavior (auto-focus, auto-select on text click)
- copy-to-clipboard emitting markdown + a trailing ` ```json ` block for reliable parsing
- keyboard focus, reduced motion, mobile layout

### editing the template

The sections you must customize:
- `<div class="meta">` — update the round label (`round 1`, `round 2`, etc.)
- Each `<section class="section">` — replace the placeholder questions with ones tailored to the project
- Set `data-title` on each section to a short name (appears as spine tooltip)
- Set `data-q` on every input/textarea/`.opts` to the question label (used for the copy output)
- Add `data-required` to critical fields

### the template's invariants — do not break

- One `<section class="section">` per topic area
- `<div class="section-num">section one</div>` + `<h2 class="section-title">` + `<p class="section-sub">` per section
- Each question wrapped in `<div class="q">` with `<label class="q-label">` + optional `<span class="q-hint">`
- Radio/checkbox groups use `<ul class="opts" data-type="radio|check" data-q="...">`; every option is `<li>` with the exact label/input/mark/opt-body structure
- Every radio group MUST include an `<li data-other>` with a `.opt-other-input` at the end
- Lowercase UI text. No ALL-CAPS. Sentence case for titles.

### saving the form

Write the file to `.brief/round-N.html`, then tell the user the absolute path and ask them to open it in a browser. Do not run `open` — it's macOS-only and the user can click the path in their terminal. Example:

> Form saved to `/abs/path/.brief/round-1.html` — open it in your browser, fill it out, then paste the copied answers back here.

## tailoring by project type

The six round-1 sections (overview, heart of it, walkthrough, substrate, what's not in v0.1, what you're unsure about) are always present. What changes per project type is the **questions inside them**:

- **telegram bot**: commands list, inline mode vs reply keyboard, webhook vs long-polling, bot framework, admin commands, rate-limiting policy
- **web app**: pages/routes, SSR vs SPA vs MPA, auth provider, database, deployment target, session storage
- **CLI tool**: subcommands, argument style (`--flag` vs positional), config file format/location, output format (text/json/tui), distribution (homebrew, binary, npm)
- **API**: REST/GraphQL/RPC, auth (jwt/oauth/api keys), versioning, rate limiting, docs generation, client SDKs
- **browser extension**: manifest v3 permissions, content scripts vs background, cross-browser scope, data storage, update strategy
- **mobile app**: native vs cross-platform, offline behavior, push notifications, app store strategy

When the project type is unclear, start round 1 with the generic template questions and add a dedicated section once the type is declared.

## analyzing pasted answers

When the user pastes answers back:

1. **Parse the trailing ` ```json ` block** — this is the machine-readable truth. The human-readable markdown above it is for the user's benefit.
2. **Save verbatim** to `.brief/round-N-answers.md`.
3. **Acknowledge** — summarize the key decisions in 3–5 bullets.
4. **Flag gaps** — list what's ambiguous, contradictory, or missing. Suggest a default for minor decisions.
5. **Decide next step**:
   - 0–2 gaps → ask inline in chat, don't generate another form
   - 3+ gaps → generate `.brief/round-(N+1).html` focused on just those gaps
   - No gaps → offer to generate the final `SPEC.md`

## language

Match the user's language. If they write in Russian, the form labels, hints, section titles, buttons, AND the copy-button output are all in Russian. Never mix languages in one form.

## final spec

When all rounds are done, write `SPEC.md` to the project directory (check for an existing `SPEC.md` and ask before overwriting). Follow the structure and voice in `assets/example-spec.md`:

- `# {project name} — technical specification`
- overview (one paragraph, name the specific user)
- user stories
- features (v0.1) — each with acceptance criteria, not just a name
- data model — tables with fields and indices
- api / routes / commands (depending on project type)
- technical stack
- ui / ux (brief — how it feels, not mockups)
- non-functional requirements
- out of scope (v0.1) — with reasons
- open questions — the ones that remain even after all rounds

Every feature needs at least one testable acceptance criterion. Every "out of scope" item needs a reason.
