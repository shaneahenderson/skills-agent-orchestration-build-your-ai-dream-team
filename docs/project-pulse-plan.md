# Project Pulse Dashboard Implementation Plan

## Summary

Build Mona's Project Pulse dashboard as a small static app that shows contributors:

- which projects are active
- who owns each project
- each project's status
- recent activity
- priority or risk level
- a short summary

The app has three files in `app/`: `index.html`, `styles.css`, and `project-data.json`. It also needs `.vscode/launch.json`, which provides a **Run Project Pulse Dashboard** launch configuration. That configuration serves `app/` over HTTP and opens `http://localhost:%s/index.html`, so learners see the dashboard and not a directory listing.

Designer owns `app/styles.css`. Coder owns `app/index.html`, `app/project-data.json`, and `.vscode/launch.json`. No file has more than one owner.

## Repository facts

- `app/` exists but is empty. None of the four deliverables exist yet, so all four are new files.
- `.vscode/` contains only `tasks.json`, which opens the Copilot CLI terminal on folder open. Do not modify it. `.vscode/launch.json` is a separate new file.
- `.github/project-pulse-brief.md` defines the requirements. It requires:
  - a top-level `projects` array in `app/project-data.json`
  - per-project fields `name`, `owner`, `status`, `recentActivity`, and `priority`
  - a summary and a polished card layout with status badges
  - a launch configuration that serves from `app/` and opens `index.html`
- `.github/agents/designer.agent.md` requires:
  - a polished dashboard, not a bare page
  - visible project cards, status badges, clear priority treatment, readable spacing, and a responsive layout
  - the CSS hooks `.dashboard` and `.project-card`
  - rounded corners, shadows, contrast, and clear typography
  - a first view that clearly reads as a Project Pulse dashboard
  - no Git operations
- `.github/agents/coder.agent.md` requires:
  - `.vscode/launch.json` as strict JSON with no comments
  - `cwd` set to `${workspaceFolder}/app`
  - `index.html` opened, not a directory listing
  - deterministic names, commands, ports, and URLs
  - validation before the Coder reports completion
  - no Git operations
- The Designer's tools are `read`, `edit`, `search`, `web`, `memory`, and `todo`. It has no `execute`, so it cannot run or preview anything. The Coder has `execute` and does all runtime validation.
- The Orchestrator may run tasks in parallel only when file scopes do not overlap and there are no data dependencies.
- The exercise's Step 3 workflow checks for these exact strings:
  - `.dashboard`, `.project-card`, `border-radius`, and `box-shadow` in `app/styles.css`
  - `project-card`, `Project Pulse`, `styles.css`, `project-data.json`, `name`, `recentActivity`, and `priority` in the app files
  - `Run Project Pulse Dashboard` and `index.html` in `.vscode/launch.json`, which must also pass `python3 -m json.tool`
- Step 3's prompt fixes these values:
  - launch command `python3 -m http.server 5500`
  - `serverReadyAction` opening `http://localhost:%s/index.html`
  - the exact title "Project Pulse"
  - the class name `project-card`
  - a `.dashboard` selector
- Git staging, commits, and pushes are left to the learner through Copilot CLI. No agent runs them.

## Shared contract

This contract lets Designer and Coder work in parallel without sharing files.

### Data schema: `app/project-data.json`

- Top-level object with a `projects` array.
- Each project has these string fields:
  - `name`
  - `owner`
  - `status`, one of `Active`, `At Risk`, `On Hold`, or `Completed`
  - `recentActivity`
  - `priority`, one of `High`, `Medium`, or `Low`
  - `summary`, a short contributor-friendly sentence
- Include five to six projects covering every status and priority value.
- Use no real personal data. Use fictional or Mona-themed team handles.

### Markup and CSS hooks

| Hook | Used for |
| --- | --- |
| `.dashboard` | Root `<main>` wrapper |
| `.dashboard-header` | Title and subtitle |
| `.project-grid` | Card container, with `role="list"` |
| `.project-card` | Each card, an `<article>` |
| `.project-card__name` | Project name heading |
| `.project-card__owner` | Owner line |
| `.project-card__summary` | Summary text |
| `.project-card__activity` | Recent activity block |
| `.badge` with `.badge--status-<slug>` | Status badge |
| `.badge` with `.badge--priority-<slug>` | Priority badge |
| `.dashboard-state` with `.dashboard-state--error` | Loading and error message region |

- `<slug>` is the lowercase, hyphenated value, for example `at-risk`, `on-hold`, `active`, `completed`, `high`, `medium`, and `low`.
- Each badge shows its text label as well as its color.

## Ordered implementation steps

### Step 0: Lock the contract

The Orchestrator copies the shared contract into both specialist prompts. No files change.

### Step 1a: Designer builds `app/styles.css`

**Responsibilities:**

- Define the visual system:
  - CSS custom properties for color, spacing, radius, and shadow
  - a system font stack
  - a type scale
- Style `.dashboard`, `.dashboard-header`, and `.project-grid`. The grid uses `repeat(auto-fill, minmax(~280px, 1fr))` and collapses to one column on narrow screens.
- Style `.project-card`:
  - `border-radius`, `box-shadow`, and padding
  - a hover or focus lift that is disabled under `prefers-reduced-motion`
  - a visible `:focus-visible` outline
- Style badges for each status and priority slug:
  - at least 4.5:1 text contrast
  - a text label alongside the color
  - `.badge--priority-high` is the most visually prominent, for example with a bold weight or a left-border accent on the card
- Style `.dashboard-state` and `.dashboard-state--error`.
- Add a `@media (max-width: ...)` rule for small screens and optionally `prefers-color-scheme`. Use `.dashboard` and `.project-card` as literal selectors.
- Report the design rationale and accessibility notes to the Orchestrator.

### Step 1b: Coder builds `app/project-data.json` and `.vscode/launch.json`

**Responsibilities:**

- `app/project-data.json`: create it per the schema above, as valid strict JSON.
- `.vscode/launch.json`: create it as strict JSON with no comments and no trailing commas.
  - Use `"version": "0.2.0"` and one configuration named exactly `Run Project Pulse Dashboard`.
  - Use `"type": "node-terminal"` with `"request": "launch"`, `"command": "python3 -m http.server 5500"`, and `"cwd": "${workspaceFolder}/app"`.
  - Add `serverReadyAction` with `"action": "openExternally"`, `"uriFormat": "http://localhost:%s/index.html"`, and a custom `pattern`.
  - The pattern must be `Serving HTTP on .* port ([0-9]+)`. VS Code's default pattern looks for "listening on", which Python's `http.server` never prints, so the browser would not open. If `node-terminal` is unavailable in this Codespace, fall back to a `preLaunchTask` plus a `chrome` or `msedge` launch with `url` set to `http://localhost:5500/index.html`. In that case the Coder must report the change.
- Do not modify `.vscode/tasks.json`.

### Step 2: Coder builds `app/index.html`

This starts after Steps 1a and 1b finish.

**Responsibilities:**

- Write semantic HTML:
  - `<!doctype html>` and `<html lang="en">`
  - the viewport meta tag
  - `<title>Project Pulse</title>`
  - `<link rel="stylesheet" href="styles.css">`
  - a `<main class="dashboard">` containing a header with an `<h1>Project Pulse</h1>`, a one-line subtitle, and the project grid
- Add a small inline `<script>` that:
  - runs `fetch('project-data.json')`
  - checks `response.ok` and that `projects` is an array
  - renders one `<article class="project-card">` per project using the shared hooks
  - builds nodes with `textContent`, not `innerHTML`
  - shows `status`, `recentActivity`, and `priority` as badges, plus `name`, `owner`, and `summary`
- Handle states:
  - a loading message
  - an explicit error message that includes the HTTP status or parse error and a hint to run via the launch configuration
  - an empty-state message if `projects` is empty
- Use relative paths only. Do not add a build step, external dependency, CDN, or font.

### Step 3: Validate and integrate

The Coder validates the work, then the Orchestrator integrates the reports. Any defects return to the file's owner.

## File assignments

| File | Owner | Action | Notes |
| --- | --- | --- | --- |
| `app/styles.css` | Designer | Create | Only Designer edits it. |
| `app/index.html` | Coder | Create | Uses the shared hooks; no inline `<style>` blocks. |
| `app/project-data.json` | Coder | Create | Follows the schema. |
| `.vscode/launch.json` | Coder | Create | Strict JSON. |
| `docs/project-pulse-plan.md` | Planner output, saved by the Orchestrator | Create | This document. |
| `.vscode/tasks.json`, `.github/**`, `scripts/**`, `.devcontainer/**` | Nobody | Read-only | Out of scope. |

## Dependencies

- Step 0 comes before everything because the contract is the shared interface.
- Step 1a depends only on Step 0. It needs the class hooks and slugs, not the HTML or data.
- Step 1b depends only on Step 0. It needs the schema, not the CSS or HTML.
- Step 2 depends on Steps 1a, 1b, and 0:
  - It reads the JSON field names, so it needs Step 1b.
  - The Coder confirms every emitted hook has a rule in `styles.css`, so it needs Step 1a.
- Step 3 depends on all preceding steps.
- The launch configuration depends on `index.html` existing at runtime, but not at authoring time.

## Parallel and sequential work

### Parallel: Steps 1a and 1b

The Designer's `app/styles.css` work and the Coder's `app/project-data.json` and `.vscode/launch.json` work can run in parallel. They have distinct file ownership, neither reads the other's output, and both rely only on the shared contract.

### Sequential: Step 2 after Steps 1a and 1b

`app/index.html` is the integration point. It consumes the JSON field names and CSS hook names. Building it earlier would risk markup drifting from the CSS and data. One owner prevents edit conflicts.

### Sequential: Step 3 last

Validation needs every file present and verifies the integrated result.

## Edge cases and risks

- **Opening the file directly:** `file://` blocks local JSON `fetch` in most browsers. The error state must explain that the page needs the launch configuration.
- **Python output pattern:** `http.server` prints `Serving HTTP on 0.0.0.0 port 5500 ...`. A wrong `serverReadyAction` pattern prevents the browser from opening.
- **Port 5500 already in use:** `http.server` exits with an address-in-use error. Stop the existing preview first.
- **Directory listing:** The URI must end in `/index.html`, and `cwd` must be `${workspaceFolder}/app`.
- **`%s` in `uriFormat`:** Keep VS Code's port substitution literal; do not hard-code the port.
- **JSON strictness:** A comment or trailing comma in `launch.json` or the data file fails `python3 -m json.tool`.
- **Data variety:** Unknown status or priority values should receive a neutral badge style; missing fields should show `Not provided`, not `undefined`; long values must wrap without breaking the card layout.
- **Injection:** Render data with `textContent`, so JSON values cannot execute as markup.
- **Accessibility:** Use one `<h1>` and `<h2>` card headings; color is never the only signal; preserve visible focus and 4.5:1 contrast; announce loading and error messages with `role="status"` or `role="alert"`.
- **Responsive layout:** Check widths from 320px to 1440px for horizontal scrolling and card overflow.
- **Cross-file drift:** If class slugs differ, badges render unstyled. The shared contract and hook checks catch this.
- **Scope creep:** Do not add frameworks, build tooling, filters, or editing features.
- **Designer cannot run the app:** Visual and runtime verification belongs to the Coder and learner.

## Validation expectations

The Coder runs these checks and reports the results:

1. **Files and JSON**
   - All four deliverables exist.
   - `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` both succeed.
2. **Data shape**
   - A top-level `projects` array has at least five entries.
   - Every entry has non-empty `name`, `owner`, `status`, `recentActivity`, `priority`, and `summary`.
   - Statuses and priorities cover the full vocabulary.
3. **Required strings**
   - `app/index.html` contains `Project Pulse`, `styles.css`, `project-data.json`, and `project-card`.
   - `app/index.html` renders `status`, `recentActivity`, and `priority`.
   - `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, and `box-shadow`.
   - `.vscode/launch.json` contains `Run Project Pulse Dashboard`, `index.html`, `${workspaceFolder}/app`, `python3 -m http.server 5500`, and `http://localhost:%s/index.html`.
4. **Hook coverage**
   - Every class the script emits has a matching rule in `styles.css`, including each status and priority slug present in the data.
5. **Runtime**
   - From `app/`, run `python3 -m http.server 5500` in the background.
   - `curl -s http://localhost:5500/index.html` returns the HTML with the Project Pulse title, not a directory listing.
   - `curl -s http://localhost:5500/project-data.json` returns the JSON, and `styles.css` returns 200.
   - Stop the server afterward.
6. **Manual check by the learner**
   - Run and Debug → **Run Project Pulse Dashboard** opens the browser at `/index.html`.
   - The page shows the title, one card per project, status and priority badges, owner, and recent activity.
   - Shrinking the window to about 360px produces one column with no horizontal scroll.
   - Tab navigation shows visible focus.
7. **Error state**
   - Temporarily renaming `project-data.json` shows the explicit error message. Restore the file afterward.
8. **Scope**
   - Only the four assigned files were created or changed, and nothing was staged, committed, or pushed.

The Orchestrator confirms that the Designer's and Coder's reports agree on the shared contract and reports any remaining risk. The learner then stages, commits, and pushes through Copilot CLI.
