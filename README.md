# AgentHub — Admin Panel Prototype

> Internal admin panel for AgentHub, a SaaS platform where companies rent AI agents equipped with skills and deploy them for business tasks.

A fully designed HTML prototype of the operations team's admin panel. All data is hardcoded — this is a frontend deliverable intended for review and validation before backend integration.

**Status:** Specification complete · Prototype build in progress

---

## Sections

The panel covers six operational views, accessible from a persistent sidebar:

| # | Section | Purpose |
|---|---|---|
| 1 | Dashboard | Revenue, losses, active and failing agents at a glance |
| 2 | User Management | Registered users with plan and status |
| 3 | Agent Management | Agents with collapsible skill lists and editable system prompts |
| 4 | Skills | Catalog of capabilities that can be attached to agents |
| 5 | Agent Contracts | Active and past rental contracts with itemized pricing |
| 6 | Error Log | Runtime errors triaged by severity |

## Interactive features

- **Light / dark mode toggle** — switches the entire panel via Tailwind `dark:` utilities; the chosen mode is preserved across sections.
- **Action dropdowns** (`⋮`) — open one at a time, close on a second click, on outside click, and on Escape.
- **Modals** — backdrop overlay, close via button, backdrop click, or Escape; page scroll is locked while a modal is open.
- **Collapsible skill lists** — agents' skills hidden by default, smooth max-height transition on expand.
- **Mark-as-resolved** — Error Log entries can be visually struck through in place without removing the row.

## Tech stack

- **HTML5** with semantic markup (`<header>`, `<nav>`, `<main>`, `<section>`, `<table>`).
- **Tailwind CSS via CDN** — no build step, no custom CSS files, no inline styles.
- **Vanilla JavaScript** — no frameworks, no jQuery, no libraries, no bundlers.
- **Google Fonts** — Hanken Grotesk (display), IBM Plex Sans (body), IBM Plex Mono (code).

## Run locally

The prototype is a single static HTML file. Two options:

**Option A — VS Code Live Server** *(recommended for development)*

1. Open the project folder in VS Code.
2. Install the *Live Server* extension if you don't already have it.
3. Right-click `index.html` → *Open with Live Server*.

**Option B — any browser, no server**

Double-click `index.html`. Tailwind loads from its CDN; nothing else is required.

## Project structure

```
html-hello/
├── index.html       # The full admin panel prototype
├── SPECS.md         # The specification — single source of truth
├── README.md        # This file
└── learn.json       # 4Geeks Academy platform metadata
```

## Specification

The build follows [`SPECS.md`](./SPECS.md) — a complete specification document defining the design system, layout, six section specs, component inventory, hardcoded data, interactive behaviors, and 20 numbered acceptance criteria. The spec was committed **before** any HTML was written, following a *vision-to-spec* approach.

## About

Built as a frontend project for the **4Geeks Academy** web development bootcamp, in the *Frontend development with Coding Agents* module. The project demonstrates a spec-first development workflow with AI coding agents.

Author — **Settar Mengli**