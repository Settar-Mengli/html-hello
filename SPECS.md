# AgentHub Admin Panel — Specification

> Version 1.0 · Single source of truth for the prototype build.
> An AI coding agent or human developer must be able to read this document and build the prototype without asking questions. If something is ambiguous in this spec, the spec is wrong.

---

## 1. Product Description

**AgentHub** is a SaaS platform where companies rent AI agents — pre-configured intelligent assistants equipped with **skills** (capabilities such as web browsing, document reading, calendar management, and email drafting) and deployed for specific business tasks.

This document specifies the **internal admin panel** used by the AgentHub operations team. The admin user is a non-engineer staff member responsible for monitoring revenue, managing user accounts, overseeing active agents, curating the skill catalog, reviewing rental contracts, and triaging runtime errors.

The deliverable is a **fully designed HTML prototype** with hardcoded data — no backend, no API calls. Its purpose is to serve as a visual contract that the founding team can review and validate before backend integration begins.

---

## 2. Tech Stack & Constraints

### Allowed
- **HTML5** with semantic tags (`<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<table>`).
- **Tailwind CSS via CDN** loaded from `https://cdn.tailwindcss.com` — single `<script>` tag in `<head>`.
- **Vanilla JavaScript** in a single `<script>` block at the end of `<body>`.
- **Google Fonts** loaded via `<link>` tags.

### Forbidden
- No external CSS files (no `styles.css`, no separate stylesheets).
- No inline `style` attributes anywhere in the markup.
- No frameworks (React, Vue, Svelte, Alpine, etc.).
- No jQuery or other JavaScript libraries.
- No build tools (no npm, no bundlers, no preprocessors).
- No backend, no `fetch` calls, no API integration.
- No `localStorage`, `sessionStorage`, or any browser persistence — state lives in JavaScript variables only.

### File Structure
- A single `index.html` at the root of the repository.
- All six panel sections live in this one file. Section switching is JS-controlled — sections are `<section>` elements whose visibility is toggled via the `hidden` class. No real navigation, no separate HTML files.
- Rationale: this keeps dark-mode state and hardcoded data trivially consistent across "navigation" without persistence layers.

### Tailwind Dark Mode Configuration
The brief requires a manual toggle (not OS-preference-based). Tailwind must be configured for class-based dark mode. Place this **before** the CDN script tag:

```html
<script>
  tailwind.config = {
    darkMode: 'class',
    theme: {
      extend: {
        fontFamily: {
          sans: ['"IBM Plex Sans"', 'sans-serif'],
          display: ['"Hanken Grotesk"', 'sans-serif'],
          mono: ['"IBM Plex Mono"', 'monospace']
        }
      }
    }
  };
</script>
<script src="https://cdn.tailwindcss.com"></script>
```

The toggle works by adding/removing the `dark` class on the `<html>` element.

---

## 3. Visual Design System

### 3.1 Typography

Two display/body Google Fonts plus a monospaced font, loaded together via a single `<link>`:

- **Headings, section titles, metric values:** `Hanken Grotesk` — weights 600 and 700.
- **Body, table cells, labels, descriptions:** `IBM Plex Sans` — weights 400 and 500.
- **Monospaced (system prompts, error traces, timestamps):** `IBM Plex Mono` — weight 400.

Apply `font-sans` (mapped to IBM Plex Sans) globally on `<body>`. Use `font-display` on `h1`, `h2`, `h3`, and on metric card values. Use `font-mono` on system-prompt textareas, error trace blocks, and timestamps in the Error Log.

**Forbidden fonts:** `Inter`, `Roboto`, `Arial`, generic system stack. The chosen pair is intentional — do not substitute.

### 3.2 Color Tokens

All colors come from Tailwind's default palette. Use these exact tokens:

| Token | Light mode | Dark mode |
|---|---|---|
| Page background | `bg-slate-50` | `dark:bg-slate-950` |
| Surface (cards, sidebar) | `bg-white` | `dark:bg-slate-900` |
| Surface elevated (modal panel) | `bg-white` | `dark:bg-slate-900` |
| Border | `border-slate-200` | `dark:border-slate-800` |
| Heading text | `text-slate-900` | `dark:text-slate-50` |
| Body text | `text-slate-700` | `dark:text-slate-300` |
| Muted text | `text-slate-500` | `dark:text-slate-400` |
| Accent (primary) | `bg-indigo-600 text-white` | `dark:bg-indigo-500` |
| Accent hover | `hover:bg-indigo-700` | `dark:hover:bg-indigo-400` |
| Accent text | `text-indigo-600` | `dark:text-indigo-400` |

**Status / semantic palette** (used in badges, error types, and metric icons):

| Status | Background tint | Text |
|---|---|---|
| Active / Success | `bg-emerald-100 dark:bg-emerald-900/40` | `text-emerald-700 dark:text-emerald-300` |
| Inactive / Neutral | `bg-slate-100 dark:bg-slate-800` | `text-slate-600 dark:text-slate-300` |
| Warning | `bg-amber-100 dark:bg-amber-900/40` | `text-amber-700 dark:text-amber-300` |
| Failing / Critical | `bg-rose-100 dark:bg-rose-900/40` | `text-rose-700 dark:text-rose-300` |
| Info | `bg-sky-100 dark:bg-sky-900/40` | `text-sky-700 dark:text-sky-300` |

### 3.3 Spacing & Radius

- Tailwind's default 4 px scale.
- Cards and the modal panel use `rounded-xl` (12 px).
- Buttons, inputs, and rectangular badges use `rounded-md` (6 px).
- Status pills use `rounded-full`.
- Section content padding: `px-8 py-6` on desktop, `px-4 py-4` on tablet.

### 3.4 Elevation & Borders

- Light mode: cards use `shadow-sm` plus `border border-slate-200`.
- Dark mode: cards drop the shadow and rely on `dark:border-slate-800` only.
- The modal uses `shadow-2xl` in light mode and `dark:border-slate-700` in dark mode.

### 3.5 Iconography

Use **inline SVG icons only** — no icon font, no external icon library.

- Default size inside table rows and dropdown menus: 16×16 (`w-4 h-4`).
- Default size inside metric cards and sidebar links: 20×20 (`w-5 h-5`).
- Stroke: `currentColor`, stroke-width `1.75`, stroke-linecap `round`, stroke-linejoin `round`.
- The `⋮` action-dropdown trigger is rendered as inline SVG (three vertically stacked dots) for crisp rendering at any zoom level.

Suggested icons per metric card:
- Total revenue → dollar-sign
- Discount losses → trending-down arrow
- Active agents → bot / cpu-chip
- Failing agents → alert-triangle

---

## 4. Layout Structure

The page renders three top-level regions:

1. **Sidebar** — fixed-width persistent navigation, left-aligned, full viewport height.
2. **Top bar** — sticky header inside the main column, contains the section title and the dark-mode toggle.
3. **Section content** — the active section; all others have the `hidden` class.

### 4.1 Sidebar

- Width: `w-64` on desktop (≥ 768 px), `w-20` icon-only on tablet (< 768 px).
- Header: AgentHub wordmark — `<h1 class="font-display font-bold text-xl">AgentHub</h1>` — plus a small "Admin" tag below it in muted text.
- Navigation: a `<nav>` containing six `<a>` links (one per section). Each link has an inline SVG icon and a text label. The `href` is `#section-{name}`; the click handler calls `showSection(name)` and prevents default.
- Active link styling: `bg-indigo-50 text-indigo-600 dark:bg-indigo-500/10 dark:text-indigo-400` plus a 2 px left accent bar (`border-l-2 border-indigo-600`).
- Footer: a small block with a circular avatar placeholder, the admin name "Settar Mengli", and the email "admin@agenthub.io" in muted text.

### 4.2 Top Bar

- Sticky (`sticky top-0 z-10`), full width of the main column, height `h-16`.
- Left: section title rendered dynamically — same text as the active sidebar link, in `font-display text-lg font-semibold`.
- Right: dark-mode toggle button. The button shows a **moon** icon when in light mode (suggesting "click to go dark") and a **sun** icon when in dark mode.

### 4.3 Main Content Area

- Each section is a `<section id="section-{name}">` with consistent inner padding (`px-8 py-6`).
- Only one section visible at a time; others have the `hidden` class.
- Section opening element: `<h2 class="font-display text-2xl font-semibold mb-1">{Section title}</h2>` plus a one-line description in `text-sm text-slate-500 mb-6`.

---

## 5. Section Specifications

Each section below lists at least three concrete, verifiable specs covering layout, components, and behavior.

### 5.1 Dashboard

**5.1.1** Four metric cards in a responsive grid: `grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-4 gap-4`. Each card is `rounded-xl border bg-white p-5 dark:bg-slate-900 dark:border-slate-800 shadow-sm dark:shadow-none` and stacks vertically: a 36×36 rounded icon container (`w-9 h-9 rounded-lg flex items-center justify-center`) tinted with the metric's accent background, a label in muted text (`text-sm text-slate-500`), and the hardcoded value in `font-display text-3xl font-bold text-slate-900 dark:text-slate-50`.

**5.1.2** Each card carries a distinct accent color applied to the icon container only — Total revenue uses emerald, Discount losses uses rose, Active agents uses indigo, Failing agents uses amber. The card surface itself stays neutral. A small bottom delta indicator (e.g., `+12.4% vs last month`) renders in the matching semantic color.

**5.1.3** Below the cards, a full-width placeholder block represents the weekly activity chart: `border-2 border-dashed border-slate-300 dark:border-slate-700 rounded-xl h-72 flex flex-col items-center justify-center`, with a centered label "Weekly activity chart" and a smaller subtitle "Wired to backend in Phase 2" in muted text.

**5.1.4** Hardcoded values: Total revenue — `$48,250` (delta `+12.4%`), Discount losses — `$3,180` (delta `+8.1%`), Active agents — `27` (delta `+3`), Failing agents — `4` (delta `+1`).

### 5.2 User Management

**5.2.1** A `<table>` wrapped in `overflow-x-auto` for tablet support. Columns: Name, Email, Plan, Status, Actions. The table header uses `text-xs uppercase tracking-wider text-slate-500` with a bottom border. Each row has `hover:bg-slate-50 dark:hover:bg-slate-800/50`. The Plan column shows a neutral pill (`bg-slate-100 dark:bg-slate-800 text-xs px-2 py-0.5 rounded-md`). The Status column shows a semantic pill from §3.2 (Active = emerald, Inactive = slate).

**5.2.2** Each row ends with a `⋮` action-dropdown trigger — a `<button>` with `aria-haspopup="menu"`. Clicking opens a small menu, absolutely positioned below the trigger, with two items: "View detail" (with an eye icon) and "Delete" (with a trash icon, `text-rose-600`). At most one dropdown is open at any time across the entire panel.

**5.2.3** "View detail" opens the **user-record modal**. The modal title is the user's name in `font-display text-xl`. The body is a `<dl>` listing all fields: Name, Email, Plan, Status, Joined, Last active. The modal closes via three pathways: (a) the `×` close button in its top-right corner, (b) clicking the backdrop, (c) pressing the Escape key.

**5.2.4** The table renders all 5 hardcoded users from §7.1 in the order listed there.

### 5.3 Agent Management

**5.3.1** A vertical stack of agent rows (not a table — each row is a card-like `<article>` with `rounded-xl border bg-white p-4 dark:bg-slate-900 dark:border-slate-800`). The row's top line contains, left-to-right: the agent name (`font-display text-lg font-semibold`), the owner name (muted, `text-sm`), the status pill (semantic palette — Active emerald, Inactive slate, Failing rose), an "expand skills" chevron-toggle button on the right, and the `⋮` action-dropdown trigger.

**5.3.2** The skill list is **collapsed by default** — implemented as a child `<div>` with `max-h-0 overflow-hidden` transitioning to `max-h-40` when expanded, using `transition-all duration-300 ease-in-out`. The chevron icon rotates 180° on expand via `transition-transform duration-300`. When expanded, the skill list shows the agent's skills as small neutral pills (`bg-slate-100 dark:bg-slate-800 text-xs px-2 py-0.5 rounded-md`) inside a horizontally wrapped flex container.

**5.3.3** Each row's action dropdown contains "Configure" and "Delete". "Configure" opens an **agent-system-prompt modal** whose title is the agent name plus the subtitle "System prompt", and whose body is an editable `<textarea>` with 10 rows, `font-mono text-sm`, `bg-slate-50 dark:bg-slate-900`, pre-filled with the agent's hardcoded system prompt from §7.2. The modal includes a "Save" button that simply closes the modal — there is no submit logic.

**5.3.4** The list renders all 4 hardcoded agents from §7.2 in the order: Atlas, Beacon, Ledger, Scribe.

### 5.4 Skills

**5.4.1** A short `<p>` block at the top of the section explains what a "skill" is in the AgentHub context, in muted text inside `max-w-prose`. **Exact copy:** *"In AgentHub, a skill is a reusable capability — a pluggable behavior such as web browsing, document parsing, or calendar access — that can be attached to an agent at rental time. Skills are versioned independently and priced per use."*

**5.4.2** Below the explanation, a `grid grid-cols-1 md:grid-cols-2 gap-4` of skill cards. Each card has three vertically stacked regions: a header row with the skill name (`font-display text-lg`) on the left and the `⋮` action dropdown on the right; a body paragraph with the description (`text-sm text-slate-600 dark:text-slate-400`); and a footer line `Enabled on N agents` in muted text, prefixed with a small indigo dot (`w-1.5 h-1.5 rounded-full bg-indigo-500`).

**5.4.3** Each skill card's action dropdown contains "View detail" and "Delete". "View detail" opens a **skill-detail modal** whose body shows the skill name, full description, version, price, and a bulleted list of agents that currently have it enabled. All values come from §7.3.

**5.4.4** The catalog renders all 4 hardcoded skills from §7.3.

### 5.5 Agent Contracts

**5.5.1** A `<table>` (wrapped in `overflow-x-auto`) with columns: Client, Rented Agent, Skills, Start, End, Amount, Actions. The Skills column shows the contracted skills as small inline neutral pills. Amount is right-aligned (`text-right tabular-nums`). The Start and End columns use `text-sm` and the Amount column uses `font-medium`.

**5.5.2** Each row's action dropdown contains "View detail" and "Delete". "View detail" opens a **contract-detail modal** whose body shows: client name, agent name, contract dates (Start → End), an itemized inner table listing each contracted skill with its individual price, and a final summary row showing the total amount paid in bold.

**5.5.3** Past contracts (end date earlier than today, **2026-04-28**) display their dates in muted text (`text-slate-400`). Active contracts use normal body text. There is no separate Status column — the visual state of the dates implies the status.

**5.5.4** The table renders all 4 hardcoded contracts from §7.4.

### 5.6 Error Log

**5.6.1** A `<table>` with columns: Timestamp, Agent, Type, Description, Actions. Timestamps render in `font-mono text-xs text-slate-500`. The Type column is a color-coded badge using the semantic palette: Critical = rose, Warning = amber, Info = sky.

**5.6.2** Each row's action dropdown contains "View detail" and "Mark as resolved". "View detail" opens an **error-detail modal** whose body shows the timestamp, agent, type, description, and a full error trace inside a `<pre>` block (`font-mono text-xs bg-slate-100 dark:bg-slate-800 rounded-md p-4 overflow-x-auto whitespace-pre`). The trace text is hardcoded plausible content (5–8 lines per entry — see §7.5 note).

**5.6.3** "Mark as resolved" applies in-place styling to the row: the description cell gets `line-through text-slate-400`, the entire row gets `opacity-60`, and the row's action-dropdown trigger gets `disabled` plus `cursor-not-allowed`. The row is **not removed**.

**5.6.4** The table renders all 6 hardcoded entries from §7.5 in reverse-chronological order (newest first).

---

## 6. Component Inventory

These reusable components appear across multiple sections. Each is implemented once (one DOM template, one set of JS handlers) and varies only by data.

| # | Component | Used in | Notes |
|---|---|---|---|
| 6.1 | Sidebar | Global (always visible) | Persistent left nav with 6 links + active indicator. |
| 6.2 | Top Bar | Global (always visible) | Sticky header with section title + dark-mode toggle. |
| 6.3 | Metric Card | Dashboard | Icon container, label, value, delta. Configurable accent color. |
| 6.4 | Status Badge (pill) | User Mgmt, Agent Mgmt, Error Log | Semantic palette. Rounded-full. |
| 6.5 | Action Dropdown (`⋮`) | User Mgmt, Agent Mgmt, Skills, Contracts, Error Log | Trigger + absolutely positioned menu. Closes on outside click and Escape. |
| 6.6 | Modal | User Mgmt, Agent Mgmt, Skills, Contracts, Error Log | Backdrop + centered panel. Closes via × button, backdrop click, Escape. |
| 6.7 | Collapsible Skill List | Agent Mgmt | Smooth max-height transition + rotating chevron. |
| 6.8 | Color-Coded Error Type Badge | Error Log | Variant of status badge with rose / amber / sky semantics. |
| 6.9 | Dark Mode Toggle | Top Bar | Sun/moon icon swap, toggles `dark` class on `<html>`. |

**Implementation rule:** where components share behavior (action dropdown, modal), implement a single JS function for each behavior parameterized by element ID — not duplicate copies per section. There must be exactly one `openModal(id)` / `closeModal(id)` pair, exactly one `toggleDropdown(id)` function, and exactly one global outside-click handler that closes any open dropdown.

---

## 7. Hardcoded Data

Canonical values used throughout the panel. **Names must match exactly across sections.** Agent Management ↔ Agent Contracts ↔ Error Log share agent names. User Management ↔ Agent owners (§7.2) share user names.

### 7.1 Users (5)

| Name | Email | Plan | Status | Joined | Last active |
|---|---|---|---|---|---|
| Maya Chen | maya.chen@northwind.com | Pro | Active | 2025-08-12 | 2026-04-28 |
| Daniel Ruiz | d.ruiz@helios-labs.io | Enterprise | Active | 2025-06-04 | 2026-04-27 |
| Priya Patel | priya@stellargrid.co | Free | Inactive | 2025-12-19 | 2026-02-11 |
| Tomás Oliveira | tomas.o@bluepeak.dev | Pro | Active | 2026-01-08 | 2026-04-28 |
| Sarah Kim | s.kim@meridianfin.com | Enterprise | Active | 2025-04-30 | 2026-04-26 |

### 7.2 Agents (4)

| Name | Owner (from §7.1) | Status | Skills | System prompt (excerpt — extend in build) |
|---|---|---|---|---|
| Atlas | Daniel Ruiz | Active | Web Search, Document Reader | "You are Atlas, a research analyst. Investigate topics by combining web search with document parsing. Cite every claim with a source URL or document name. Prefer primary sources over aggregators." |
| Beacon | Maya Chen | Active | Calendar Manager, Email Composer | "You are Beacon, a scheduling assistant. Coordinate meetings across the user's calendar and draft polite, concise emails to confirm and reschedule. Always offer two alternative time slots." |
| Ledger | Sarah Kim | Failing | Document Reader, Web Search | "You are Ledger, a financial document analyst. Extract figures from earnings reports and PDFs, and cross-check them with public web sources. Flag discrepancies of more than 1%." |
| Scribe | Tomás Oliveira | Inactive | Web Search, Email Composer | "You are Scribe, a documentation writer. Draft technical write-ups from raw notes and refine them for an external audience. Prefer plain language over jargon." |

In the modal, the system prompt is shown in full — the build phase should extend each excerpt to a 6–10 line realistic prompt with constraints, output format guidance, and a refusal clause.

### 7.3 Skills (4)

| Name | Description | Version | Price | Enabled on (count) | Enabled on (agents) |
|---|---|---|---|---|---|
| Web Search | Browse the web and retrieve up-to-date information from public sources. | v1.4.2 | $0.80 / 1,000 invocations | 3 | Atlas, Ledger, Scribe |
| Document Reader | Parse and summarize PDFs, DOCX, and spreadsheets up to 200 pages. | v2.0.1 | $1.20 / 1,000 invocations | 2 | Atlas, Ledger |
| Calendar Manager | Read and create events on connected Google and Outlook calendars. | v1.1.0 | $0.50 / 1,000 invocations | 1 | Beacon |
| Email Composer | Draft and review emails based on conversational context and tone hints. | v1.3.5 | $0.95 / 1,000 invocations | 2 | Beacon, Scribe |

Cross-check: skill counts here must match the skills listed under each agent in §7.2.

### 7.4 Agent Contracts (4)

| Client | Agent | Skills | Start | End | Amount | Itemized prices |
|---|---|---|---|---|---|---|
| Helios Labs | Atlas | Web Search, Document Reader | 2025-11-01 | 2026-05-01 | $3,200 | Web Search $1,800 · Document Reader $1,400 |
| Northwind Logistics | Beacon | Calendar Manager, Email Composer | 2026-01-15 | 2026-07-15 | $2,400 | Calendar Manager $1,500 · Email Composer $900 |
| Meridian Financial | Ledger | Document Reader, Web Search | 2025-09-10 | 2026-03-10 | $4,100 | Document Reader $2,500 · Web Search $1,600 |
| BluePeak Studios | Scribe | Web Search, Email Composer | 2026-02-20 | 2026-08-20 | $1,950 | Web Search $1,200 · Email Composer $750 |

**Today's date for active/past determination: 2026-04-28.** Active contracts: Helios Labs / Atlas, Northwind / Beacon, BluePeak / Scribe. Past contracts: Meridian Financial / Ledger (ended 2026-03-10).

Each client name in this table corresponds to the email domain of one user in §7.1 (e.g., Maya Chen at northwind.com → Northwind Logistics).

### 7.5 Error Log Entries (6)

| Timestamp | Agent | Type | Description |
|---|---|---|---|
| 2026-04-28 09:14 | Ledger | Critical | Skill 'Document Reader' timed out after 30s |
| 2026-04-27 18:42 | Ledger | Critical | Failed to authenticate to data source after 3 retries |
| 2026-04-27 11:03 | Atlas | Warning | Rate limit reached on Web Search; throttled for 60s |
| 2026-04-26 22:17 | Scribe | Warning | Output truncated: response exceeded max_tokens |
| 2026-04-26 08:55 | Beacon | Info | Calendar permissions refreshed successfully |
| 2026-04-25 14:20 | Ledger | Critical | Unhandled exception in pipeline: NullReferenceError |

`Ledger` appears 3× in the log — this explains its `Failing` status in §7.2.

Each entry has a hardcoded full trace string for the View-detail modal. Build phase: produce realistic 5–8 line traces (function names, file paths, line numbers) matching each description.

---

## 8. Global Interactive Behaviors

**8.1 Section switching.** Sidebar links call `showSection(name)`. The function: (a) adds `hidden` to all `<section>` elements with an `id` starting with `section-`, (b) removes `hidden` from `#section-{name}`, (c) updates the active link styling on the sidebar, (d) updates the section title in the top bar. The browser URL is **not** updated; `history.pushState` is not used.

**8.2 Action dropdowns.** All `⋮` triggers share one `toggleDropdown(id)` handler. Opening a dropdown closes any other currently open dropdown. A single `document` click listener closes the open dropdown when the click target is outside its menu (use `event.target.closest('.dropdown-menu')` to detect inside-clicks). Pressing Escape closes the open dropdown. There is at most one open dropdown at any time across the entire panel.

**8.3 Modals.** All modals share one `openModal(id)` / `closeModal(id)` pair. Opening a modal makes its backdrop visible (full-viewport, `bg-black/50 backdrop-blur-sm fixed inset-0`) and centers the panel inside it. Closing happens via three pathways: (a) the `×` close button inside the panel, (b) a click on the backdrop (use `event.stopPropagation` on the panel so clicks inside the panel do not close the modal), (c) the Escape key. When a modal is open, the page background must not scroll (`document.body.classList.add('overflow-hidden')`).

**8.4 Collapsible skill lists (Agent Management).** Each row's expand control toggles a child element's max-height between `0` and a value tall enough to reveal all skills, animated via Tailwind's `transition-all duration-300 ease-in-out`. The chevron rotates 180° via `transition-transform duration-300`. State is held per row (e.g., a `data-expanded` attribute on the row).

**8.5 Dark mode toggle.** The button in the top bar toggles the `dark` class on `<html>`. The icon swaps from moon (light mode) to sun (dark mode). State lives in a single JS variable plus the class on `<html>`. Because the entire app is one HTML page, the chosen mode is automatically preserved across section switches.

**8.6 Mark-as-resolved (Error Log).** Clicking applies `line-through text-slate-400` to the description cell, `opacity-60` to the row, and `disabled` + `cursor-not-allowed` to the row's `⋮` trigger. The row remains visible.

**8.7 Responsive behavior.** Sidebar collapses to icon-only at the `md` breakpoint downward (`< 768 px`). Tables become horizontally scrollable inside an `overflow-x-auto` wrapper at the same breakpoint. The metric grid drops from 4 → 2 → 1 columns at the `xl`, `sm`, and base breakpoints.

---

## 9. Acceptance Criteria

The prototype is **complete** when **all** of the following are verifiable:

1. `SPECS.md` was committed in a separate Git commit **before** the first commit that contains `index.html`. (Verifiable via `git log --oneline -- SPECS.md index.html`.)
2. The repository contains exactly one HTML file (`index.html`) at the root, and no external CSS files.
3. Tailwind CSS is loaded via the official CDN script tag and configured with `darkMode: 'class'`.
4. The page renders six distinct sections — Dashboard, User Management, Agent Management, Skills, Agent Contracts, Error Log — accessible from a persistent sidebar.
5. The sidebar shows an active-state indicator on the link of the currently visible section, and only one section is visible at a time.
6. The Dashboard renders four metric cards in a responsive grid plus a dashed-border placeholder for the weekly chart below them.
7. User Management renders a `<table>` with at least 5 hardcoded user rows (names from §7.1).
8. Agent Management renders at least 4 agent rows (names from §7.2). Each agent's skill list is collapsed by default, expands with a visible CSS transition on click, and collapses on a second click.
9. Skills renders at least 4 skill cards (names from §7.3) plus the in-panel explanation paragraph from §5.4.1.
10. Agent Contracts renders a `<table>` with at least 4 contract rows (clients from §7.4).
11. Error Log renders at least 6 entries (timestamps from §7.5), each with a color-coded type badge using the rose / amber / sky palette.
12. Every list row across User Management, Agent Management, Skills, Agent Contracts, and Error Log has a working `⋮` action dropdown that opens on click, closes on a second click of the same trigger, and closes when the user clicks outside the menu area.
13. "View detail" opens a modal in **at least four** sections (User Management, Skills, Agent Contracts, Error Log).
14. Every modal closes via (a) its close button, (b) a click on the backdrop, **and** (c) pressing Escape.
15. The Agent Management "Configure" action opens a modal containing the agent's system prompt inside an editable `<textarea>`.
16. The dark/light mode toggle in the top bar adds/removes the `dark` class on `<html>`, switches both background and text across all sections in one click, and the chosen mode is preserved when switching between sections.
17. Hardcoded data is consistent across sections: every agent name in §7.2 appears in at least one Error Log entry **and** at least one Agent Contract; every contract client (§7.4) maps to a user in §7.1 via email domain.
18. Markup uses semantic tags — at minimum one each of `<header>`, `<nav>`, `<main>`, `<section>`, `<table>`. No inline `style` attributes anywhere in the file.
19. The layout is usable at desktop (≥ 1280 px), small desktop (1024 px), and tablet (768 px) viewports without horizontal page scroll. Tables may scroll horizontally inside their container at narrow widths.
20. Typography uses `Hanken Grotesk` (display) and `IBM Plex Sans` (body) loaded from Google Fonts. `Inter`, `Roboto`, and `Arial` do not appear anywhere in the markup or config.

---

*End of specification.*