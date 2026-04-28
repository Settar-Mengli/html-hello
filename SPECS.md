# AgentHub — Admin Panel Specification

---

## 1. Product Description

**AgentHub** is a web-based administration panel for an AI Agent Rental Platform. The platform enables businesses and individuals to rent specialized AI agents (e.g., data analysts, writers, customer support bots, code reviewers) on a contract basis. Administrators use AgentHub to oversee every aspect of the platform: who the users are, which agents are listed, what skills those agents offer, which rental contracts are active, and what errors the system has logged.

**Target users:** Platform administrators only. There is no public-facing user portal in this panel.

**Core responsibilities of the admin:**
- Monitor platform health via the Dashboard
- Create, edit, suspend, or delete user accounts
- Approve, edit, or deactivate AI agent listings
- Define and manage the skill taxonomy available to agents
- View and terminate rental contracts
- Review, resolve, and export system error logs

---

## 2. Tech Stack & Constraints

| Concern | Decision |
|---|---|
| Markup | HTML5 only |
| Styling | Tailwind CSS v4 via CDN — `<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>` |
| Scripting | Vanilla JavaScript (ES6+) — no jQuery, no frameworks |
| Data | All data hardcoded as JavaScript arrays/objects inside `<script>` tags |
| Backend | None — zero network requests |
| Build tool | None — open `index.html` directly in a browser or via `python3 server.py` |
| File structure | Single `index.html` is acceptable; separate `style.css` and `app.js` are optional |
| Browser target | Modern Chromium-based browsers (Chrome 110+, Edge 110+) |
| Minimum viewport | 1280px wide |

**Prohibited:**
- Any JavaScript framework or library (React, Vue, Alpine, htmx, jQuery, Chart.js, etc.)
- Any CSS framework other than Tailwind via the CDN above
- Any `fetch()`, `XMLHttpRequest`, or WebSocket calls
- Any `localStorage`, `sessionStorage`, or `IndexedDB` usage

---

## 3. Global Layout

The admin panel is a Single-Page Application (SPA). Navigation between sections never reloads the page — JavaScript shows and hides section containers.

### 3.1 Shell Structure

```
┌──────────────────────────────────────────────────────┐
│  SIDEBAR (fixed, 240px wide, full viewport height)   │
│  ┌────────────────────────────────────────────────┐  │
│  │ Logo: "AgentHub" wordmark + robot icon         │  │
│  ├────────────────────────────────────────────────┤  │
│  │ Nav Links (vertical list):                     │  │
│  │   📊 Dashboard                                 │  │
│  │   👥 User Management                           │  │
│  │   🤖 Agent Management                          │  │
│  │   🧠 Skills                                    │  │
│  │   📄 Agent Contracts                           │  │
│  │   🔴 Error Log                                 │  │
│  └────────────────────────────────────────────────┘  │
├──────────────────────────────────────────────────────┤
│  TOP BAR (fixed, full width minus sidebar)           │
│  Left: Page title (updates on nav)                   │
│  Right: Admin avatar circle + "Admin" label          │
├──────────────────────────────────────────────────────┤
│  MAIN CONTENT AREA (scrollable, padding 24px)        │
│  [Active section renders here]                       │
└──────────────────────────────────────────────────────┘
```

### 3.2 Sidebar Behavior
- Width: 240px, fixed position, full viewport height, dark background (`bg-gray-900`), white text.
- Logo area at top: icon + "AgentHub" text, ~64px tall.
- Nav items: full-width clickable areas, `py-3 px-4`, icon + label.
- **Active state:** Background `bg-indigo-600`, white text, left border `border-l-4 border-white`.
- **Inactive hover state:** Background `bg-gray-700`.
- On click: JS removes active class from all nav items, adds it to clicked item, hides all sections, shows the target section.

### 3.3 Top Bar
- Height: 64px, fixed, `bg-white`, bottom border `border-b border-gray-200`, shadow.
- Left: `<h1>` element with the current section name (updated by JS on each nav click).
- Right: Circular avatar placeholder (`bg-indigo-500`, white initials "SA") + text "Super Admin".

### 3.4 Content Area
- Positioned to the right of the sidebar, below the top bar.
- Padding: `p-6` all sides.
- Background: `bg-gray-50`.
- Each of the 6 sections is a `<div>` with `id="section-{name}"`. All are `display: none` by default; the active one is `display: block`.
- Dashboard is shown by default on page load.

---

## 4. Component Inventory

These components recur across sections. Each implementation must be consistent in appearance.

### 4.1 KPI Card
**Used in:** Dashboard

**Structure:**
```
┌─────────────────────────────┐
│  [Icon]        [Trend Badge]│
│  Label                      │
│  Value (large, bold)        │
└─────────────────────────────┘
```
- Background: `bg-white`, rounded, shadow, padding `p-5`.
- Icon: 40×40px colored circle with an SVG icon inside (e.g., user icon, robot icon, file icon, dollar icon).
- Trend Badge: small pill, `bg-green-100 text-green-700` for positive (↑), `bg-red-100 text-red-700` for negative (↓). Displays percentage change, e.g. "+8%" or "-3%".
- Label: small gray text (`text-sm text-gray-500`).
- Value: large bold number (`text-3xl font-bold text-gray-800`).

### 4.2 Data Table
**Used in:** User Management, Agent Management, Agent Contracts, Error Log

**Structure:**
- Wrapper: `overflow-x-auto`, `bg-white`, rounded, shadow.
- `<table class="w-full text-sm">` with `border-collapse`.
- `<thead>`: `bg-gray-50`, `text-xs uppercase text-gray-500`, `font-semibold`, bottom border.
- `<tbody>`: alternating row shading — odd rows `bg-white`, even rows `bg-gray-50`.
- Hover: `hover:bg-indigo-50` on each `<tr>`.
- Each `<td>`: `px-4 py-3`, `text-gray-700`.
- Column headers `<th>`: `px-4 py-3`, `text-left`.

### 4.3 Search Input
**Used in:** All tabular sections and Skills

**Structure:**
- Wrapper div: `relative`.
- Magnifier SVG icon: positioned absolutely, left side, vertically centered, `text-gray-400`.
- `<input type="text">`: `pl-10 pr-4 py-2`, `border border-gray-300`, `rounded-lg`, `text-sm`, `w-64`, `focus:outline-none focus:ring-2 focus:ring-indigo-500`.
- Placeholder text: "Search…" (contextual label optional).
- `oninput` handler: calls the section's filter function on every keystroke.

### 4.4 Dropdown Filter
**Used in:** All tabular sections

**Structure:**
- `<select>` element: `border border-gray-300`, `rounded-lg`, `px-3 py-2`, `text-sm`, `bg-white`, `focus:outline-none focus:ring-2 focus:ring-indigo-500`.
- First `<option>`: "All [FieldName]" with value `""`.
- Remaining options: one per distinct filter value.
- `onchange` handler: calls the section's filter function.

### 4.5 Pagination Bar
**Used in:** User Management, Agent Management, Agent Contracts, Error Log

**Structure:**
```
[Previous]  [1]  [2]  [3]  [Next]       Showing 1–10 of 20
```
- Wrapper: `flex items-center justify-between mt-4`.
- Prev/Next: `<button>` styled as outlined pill, disabled when at first/last page (`opacity-50 cursor-not-allowed`).
- Page number pills: current page has `bg-indigo-600 text-white`, others `bg-white text-gray-600 border`.
- "Showing X–Y of Z": right-aligned gray text.
- Behavior: clicking a page number or Next/Prev re-renders the visible table rows from the in-memory filtered data array.

### 4.6 Modal
**Used in:** All sections (view, edit, confirm, add)

**Structure:**
```
[Backdrop overlay — semi-transparent, full viewport]
  ┌──────────────────────────────────────────┐
  │ Title                            [X]     │
  ├──────────────────────────────────────────┤
  │ Body content (form fields or read-only   │
  │ details)                                 │
  ├──────────────────────────────────────────┤
  │                      [Cancel]  [Action]  │
  └──────────────────────────────────────────┘
```
- Backdrop: `fixed inset-0 bg-black bg-opacity-50 z-40`, clicking it closes the modal.
- Modal box: `fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2`, `bg-white`, `rounded-xl`, `shadow-2xl`, `z-50`, `w-full max-w-lg`, `max-h-[90vh] overflow-y-auto`.
- Header: `px-6 py-4`, `border-b`, title as `font-semibold text-lg text-gray-800`, X button top-right.
- Body: `px-6 py-4`.
- Footer: `px-6 py-4`, `border-t`, `flex justify-end gap-3`.
- Cancel button: `bg-white border border-gray-300 text-gray-700 px-4 py-2 rounded-lg`.
- Action button: `bg-indigo-600 text-white px-4 py-2 rounded-lg` (or `bg-red-600` for destructive actions).
- Close triggers: X button, Cancel button, backdrop click. All call a `closeModal(id)` function.

### 4.7 Status Badge
**Used in:** User Management, Agent Management, Agent Contracts

| Status | Classes |
|---|---|
| Active | `bg-green-100 text-green-700` |
| Inactive | `bg-gray-100 text-gray-600` |
| Suspended | `bg-red-100 text-red-700` |
| Pending | `bg-yellow-100 text-yellow-700` |
| Under Review | `bg-blue-100 text-blue-700` |
| Completed | `bg-indigo-100 text-indigo-700` |
| Cancelled | `bg-gray-100 text-gray-500` |

Common classes: `inline-block px-2.5 py-0.5 rounded-full text-xs font-medium`.

### 4.8 Action Button Group
**Used in:** All tabular sections, Skill cards

A horizontal group of small icon buttons at the end of each row or card.

| Button | Color | Icon | Action |
|---|---|---|---|
| View | `bg-blue-50 text-blue-600 hover:bg-blue-100` | Eye icon | Open read-only modal |
| Edit | `bg-indigo-50 text-indigo-600 hover:bg-indigo-100` | Pencil icon | Open edit modal |
| Suspend/Activate | `bg-yellow-50 text-yellow-600 hover:bg-yellow-100` | Lock/Unlock icon | Toggle status, show toast |
| Deactivate/Activate | `bg-orange-50 text-orange-600 hover:bg-orange-100` | Power icon | Toggle status, show toast |
| Terminate | `bg-red-50 text-red-600 hover:bg-red-100` | X-circle icon | Confirmation modal → cancel contract |
| Delete | `bg-red-50 text-red-600 hover:bg-red-100` | Trash icon | Confirmation modal → remove row |
| Mark Resolved | `bg-green-50 text-green-600 hover:bg-green-100` | Check icon | Toggle error status |

Each button: `p-1.5 rounded-md`, tooltip via `title` attribute.

### 4.9 Toast Notification
**Used in:** All sections (triggered by state-changing actions)

**Structure:**
- Fixed position: `fixed bottom-6 right-6 z-50`.
- `bg-gray-800 text-white px-5 py-3 rounded-lg shadow-lg text-sm`.
- Optional left-side colored dot: green for success, red for error/destructive.
- Appears with a slide-up CSS transition (`translate-y-full → translate-y-0`).
- Auto-dismisses after 2000ms; uses `setTimeout` to remove from DOM.
- Only one toast is shown at a time — previous toast is removed before showing a new one.

### 4.10 Severity Badge
**Used in:** Error Log

| Severity | Classes |
|---|---|
| Critical | `bg-red-100 text-red-700 border border-red-200` |
| Warning | `bg-orange-100 text-orange-700 border border-orange-200` |
| Info | `bg-blue-100 text-blue-700 border border-blue-200` |

Common classes: `inline-block px-2.5 py-0.5 rounded-full text-xs font-medium`.

### 4.11 Section Header Bar
**Used in:** Every section (top of content area)

**Structure:**
```
[Section Title (h2)]                [Primary Action Button (optional)]
[Subtitle / breadcrumb text]
```
- Title: `text-2xl font-bold text-gray-800`.
- Subtitle: `text-sm text-gray-500 mt-1`.
- Primary action button (when present): `bg-indigo-600 text-white px-4 py-2 rounded-lg text-sm font-medium hover:bg-indigo-700`, positioned right-aligned via `flex justify-between items-start`.

---

## 5. Section Specifications

### 5.1 Dashboard

**Purpose:** At-a-glance platform health snapshot for the administrator.

**Layout (top to bottom):**
1. Section Header Bar (title: "Dashboard", subtitle: "Welcome back, Super Admin")
2. KPI Cards Row (4 cards, equal width columns)
3. Two-column row: Bar Chart (left, 60%) + Recent Activity Feed (right, 40%)
4. Full-width Top Agents Table

---

#### 5.1.1 KPI Cards

Four cards in a `grid grid-cols-4 gap-4` container.

| Card | Icon | Value | Trend |
|---|---|---|---|
| Total Users | Users icon (indigo) | 1,284 | +8% |
| Active Agents | Robot/CPU icon (emerald) | 347 | +12% |
| Active Contracts | Document icon (blue) | 93 | -3% |
| Monthly Revenue | Dollar icon (amber) | $48,200 | +21% |

No click interaction on KPI cards. They are display-only.

---

#### 5.1.2 Bar Chart — "Agent Rentals – Last 7 Days"

**Implementation:** Pure CSS flexbox bars. No `<canvas>`, no SVG library, no JavaScript charting.

Container: `bg-white rounded-xl shadow p-5`.

Chart structure:
```
[Chart Title]
┌─────────────────────────────────────────────┐
│   █                                         │
│   █          █                              │
│   █    █     █    █    █         █          │
│   █    █     █    █    █    █    █          │
│  Mon  Tue   Wed  Thu  Fri  Sat  Sun         │
└─────────────────────────────────────────────┘
```

- Outer wrapper: `flex items-end gap-2 h-40` (the chart area).
- Each bar column: `flex flex-col items-center gap-1`.
- Bar element: `bg-indigo-500 rounded-t w-10`, height set via inline `style="height: Xpx"`.
- Value label above bar: `text-xs text-gray-600 font-medium`.
- Day label below: `text-xs text-gray-500`.

Hardcoded data (day → rentals → bar height in px):

| Day | Rentals | Bar height |
|---|---|---|
| Mon | 42 | 84px |
| Tue | 31 | 62px |
| Wed | 58 | 116px |
| Thu | 47 | 94px |
| Fri | 63 | 126px |
| Sat | 29 | 58px |
| Sun | 51 | 102px |

---

#### 5.1.3 Recent Activity Feed

Container: `bg-white rounded-xl shadow p-5`, title "Recent Activity".

A vertical list of 8 hardcoded events. Each event:
- Left: colored icon circle (12px, varies by event type).
- Center: description text (`text-sm text-gray-700`) + relative timestamp below (`text-xs text-gray-400`).
- No click interaction.

Hardcoded events (newest first):

| # | Description | Timestamp | Icon color |
|---|---|---|---|
| 1 | New contract signed by Acme Corp | 2 minutes ago | green |
| 2 | Agent "DataBot v2" approved | 14 minutes ago | indigo |
| 3 | User maria@example.com suspended | 32 minutes ago | red |
| 4 | Skill "RAG Pipeline" added | 1 hour ago | blue |
| 5 | Contract #CT-0091 terminated | 2 hours ago | orange |
| 6 | Agent "WriterPro" deactivated | 3 hours ago | gray |
| 7 | New user registered: j.smith@corp.io | 5 hours ago | green |
| 8 | Critical error resolved: NullRef in Agent #A-019 | 8 hours ago | purple |

---

#### 5.1.4 Top Agents Table

Container: `bg-white rounded-xl shadow p-5`, title "Top Agents This Month".

Simple `<table>` — no search, no filter, no pagination.

Columns: Rank | Agent Name | Category | Rentals (30d) | Avg Rating | Revenue (30d)

| Rank | Agent Name | Category | Rentals | Rating | Revenue |
|---|---|---|---|---|---|
| 1 | DataBot v2 | Data Analysis | 87 | ⭐ 4.9 | $12,180 |
| 2 | WriterPro Elite | Writing | 74 | ⭐ 4.8 | $8,880 |
| 3 | SupportAce | Customer Support | 68 | ⭐ 4.7 | $6,120 |
| 4 | CodeSense | Code Generation | 61 | ⭐ 4.8 | $9,760 |
| 5 | MarketMind | Marketing | 55 | ⭐ 4.6 | $7,150 |

Rank column: bold number with gold medal emoji for rank 1 (🥇), silver for 2 (🥈), bronze for 3 (🥉), plain number for 4–5.

---

### 5.2 User Management

**Purpose:** View, create, edit, suspend, and delete platform user accounts.

**Layout (top to bottom):**
1. Section Header Bar (title: "User Management", subtitle: "Manage platform users and their roles")
2. Filter Bar (search + 2 dropdowns)
3. Data Table
4. Pagination Bar

---

#### 5.2.1 Filter Bar

`flex items-center gap-3 mb-4`

- Search Input: placeholder "Search by name or email…", width `w-72`.
- Dropdown 1 — Role: options `All Roles | Admin | Renter | Agent Owner`.
- Dropdown 2 — Status: options `All Statuses | Active | Suspended | Pending`.

Filter logic (applied simultaneously via AND):
- Search matches against `name` OR `email` fields (case-insensitive substring).
- Role dropdown matches exact `role` field (skipped if "All Roles").
- Status dropdown matches exact `status` field (skipped if "All Statuses").

Whenever any filter changes, reset to page 1 and re-render the table.

---

#### 5.2.2 User Table

Columns: `#` | Name | Email | Role | Status | Joined | Actions

**Hardcoded Users (20 total):**

| ID | Name | Email | Role | Status | Joined |
|---|---|---|---|---|---|
| U-001 | Alice Johnson | alice@example.com | Admin | Active | 2024-01-15 |
| U-002 | Bob Martinez | bob@corp.io | Renter | Active | 2024-02-03 |
| U-003 | Carol Singh | carol@agency.net | Agent Owner | Active | 2024-02-18 |
| U-004 | David Lee | david@startup.co | Renter | Suspended | 2024-03-05 |
| U-005 | Emma Wilson | emma@freelance.dev | Agent Owner | Active | 2024-03-22 |
| U-006 | Frank Chen | frank@enterprise.com | Renter | Active | 2024-04-01 |
| U-007 | Grace Patel | grace@techco.io | Admin | Active | 2024-04-14 |
| U-008 | Henry O'Brien | henry@media.org | Renter | Pending | 2024-04-29 |
| U-009 | Iris Nakamura | iris@design.studio | Agent Owner | Active | 2024-05-10 |
| U-010 | James Kim | james@logistics.biz | Renter | Active | 2024-05-23 |
| U-011 | Karen Reyes | karen@healthtech.co | Renter | Active | 2024-06-02 |
| U-012 | Liam Foster | liam@eduplatform.org | Agent Owner | Suspended | 2024-06-18 |
| U-013 | Mia Thompson | mia@retail.shop | Renter | Active | 2024-07-01 |
| U-014 | Noah Davis | noah@fintech.io | Renter | Active | 2024-07-14 |
| U-015 | Olivia Brown | olivia@legal.firm | Agent Owner | Pending | 2024-07-28 |
| U-016 | Paul Garcia | paul@realestate.biz | Renter | Active | 2024-08-05 |
| U-017 | Quinn Adams | quinn@pharma.co | Renter | Active | 2024-08-19 |
| U-018 | Rachel Moore | rachel@ngo.org | Agent Owner | Active | 2024-09-01 |
| U-019 | Sam Taylor | sam@gaming.studio | Renter | Suspended | 2024-09-15 |
| U-020 | Tina White | tina@fashion.brand | Agent Owner | Active | 2024-09-30 |

---

#### 5.2.3 User Row Actions

**Edit button:**
- Opens the Edit User Modal pre-populated with that user's current data.
- Modal title: "Edit User — [User Name]".
- Form fields:
  - Name: `<input type="text">`, pre-filled.
  - Email: `<input type="email">`, pre-filled.
  - Role: `<select>` with options `Admin | Renter | Agent Owner`, pre-selected.
  - Status: `<select>` with options `Active | Suspended | Pending`, pre-selected.
- Save button: updates the in-memory user object, re-renders that table row, closes modal, shows toast "User updated successfully".

**Suspend / Activate button:**
- If user status is `Active`: button label/icon = "Suspend". Clicking sets status to `Suspended`, updates badge in row, shows toast "User suspended".
- If user status is `Suspended`: button label/icon = "Activate". Clicking sets status to `Active`, updates badge, shows toast "User activated".
- If user status is `Pending`: button is disabled (grayed out, not clickable).

**Delete button:**
- Opens a Confirmation Modal.
- Modal title: "Delete User".
- Modal body: "Are you sure you want to delete [User Name]? This action cannot be undone."
- Cancel button: closes modal, no change.
- Delete button (`bg-red-600`): removes the user from the in-memory array, re-renders the table, closes modal, shows toast "User deleted".

---

#### 5.2.4 Pagination

- 10 rows per page.
- With 20 users: 2 pages.
- After filtering, page count recalculates based on filtered result count.
- "Showing 1–10 of 20 users" label updates dynamically.

---

### 5.3 Agent Management

**Purpose:** Browse, register, edit, and deactivate AI agents listed on the platform.

**Layout (top to bottom):**
1. Section Header Bar with "Add New Agent" button (right-aligned)
2. Filter Bar (search + 2 dropdowns)
3. Data Table
4. Pagination Bar

---

#### 5.3.1 Filter Bar

- Search Input: placeholder "Search by agent name…", `w-64`.
- Dropdown 1 — Category: `All Categories | Data Analysis | Writing | Customer Support | Code Generation | Marketing`.
- Dropdown 2 — Status: `All Statuses | Active | Inactive | Under Review`.

Filter logic: same AND pattern as User Management (search on `name`, dropdowns exact match).

---

#### 5.3.2 Agent Table

Columns: `#` | Agent Name | Category | Skills | Owner | Hourly Rate | Status | Actions

**Hardcoded Agents (18 total):**

| ID | Name | Category | Skills | Owner | Rate | Status |
|---|---|---|---|---|---|---|
| A-001 | DataBot v2 | Data Analysis | NLP, Data Viz, SQL | Carol Singh | $45/hr | Active |
| A-002 | WriterPro Elite | Writing | Copywriting, SEO, Summarization | Emma Wilson | $35/hr | Active |
| A-003 | SupportAce | Customer Support | Sentiment Analysis, FAQ Handling, Escalation | Iris Nakamura | $30/hr | Active |
| A-004 | CodeSense | Code Generation | Python, JavaScript, Code Review | Emma Wilson | $55/hr | Active |
| A-005 | MarketMind | Marketing | SEO, Ad Copy, Campaign Planning | Rachel Moore | $40/hr | Active |
| A-006 | InsightPro | Data Analysis | Data Viz, Forecasting, Reporting | Carol Singh | $50/hr | Active |
| A-007 | NarratorAI | Writing | Storytelling, Summarization, Translation | Tina White | $28/hr | Inactive |
| A-008 | HelpBot Ultra | Customer Support | FAQ Handling, Live Chat, CSAT | Iris Nakamura | $25/hr | Active |
| A-009 | SQLWizard | Data Analysis | SQL, ETL, Data Cleaning | Carol Singh | $48/hr | Under Review |
| A-010 | ContentFlow | Writing | Blog Writing, SEO, Social Media | Tina White | $32/hr | Active |
| A-011 | DevAssist | Code Generation | TypeScript, Code Review, Debugging | Emma Wilson | $60/hr | Active |
| A-012 | BrandVoice | Marketing | Brand Strategy, Ad Copy, Email Marketing | Rachel Moore | $42/hr | Active |
| A-013 | LogicBot | Code Generation | Python, Data Pipelines, API Integration | Emma Wilson | $52/hr | Inactive |
| A-014 | TrendSpotter | Marketing | Social Listening, Trend Analysis, Reporting | Rachel Moore | $38/hr | Under Review |
| A-015 | LexiBot | Writing | Legal Writing, Document Review, Summarization | Olivia Brown | $45/hr | Active |
| A-016 | QueryMaster | Data Analysis | SQL, BI Dashboards, KPI Tracking | Carol Singh | $47/hr | Active |
| A-017 | ReplyBot | Customer Support | Email Handling, Sentiment Analysis, Routing | Iris Nakamura | $27/hr | Active |
| A-018 | CampaignAI | Marketing | Paid Ads, A/B Testing, ROI Analysis | Rachel Moore | $44/hr | Inactive |

---

#### 5.3.3 Agent Row Actions

**View button:**
- Opens a read-only Agent Detail Modal.
- Modal title: "Agent Details — [Agent Name]".
- Modal body sections:
  - **Info:** Name, Category, Owner, Hourly Rate, Status (badge), Created Date (hardcode to a plausible date per agent).
  - **Description:** 2–3 sentence paragraph describing what the agent does (hardcoded per agent).
  - **Skills:** displayed as a row of gray pill badges.
  - **Performance Stats:** three stat boxes in a row:
    - Uptime: e.g. "99.2%"
    - Avg Rating: e.g. "4.9 ⭐"
    - Total Rentals: e.g. "187"
  (All values hardcoded per agent.)
- Footer: only a "Close" button.

Hardcoded agent detail data:

| Agent | Description | Uptime | Rating | Rentals |
|---|---|---|---|---|
| DataBot v2 | Specializes in turning raw datasets into actionable insights using advanced SQL queries and automated visualizations. Ideal for business analytics teams. | 99.2% | 4.9 | 187 |
| WriterPro Elite | A versatile writing agent capable of producing SEO-optimized blog posts, ad copy, and executive summaries. Adapts tone to brand voice. | 98.7% | 4.8 | 152 |
| SupportAce | Handles frontline customer inquiries with empathy-aware sentiment detection. Routes complex issues to human agents automatically. | 99.8% | 4.7 | 214 |
| CodeSense | Generates production-ready Python and JavaScript code from natural language specifications. Includes inline documentation and unit tests. | 98.1% | 4.8 | 139 |
| MarketMind | Builds end-to-end marketing campaigns including keyword research, ad copy, and performance tracking. Integrates with major ad platforms via reports. | 97.5% | 4.6 | 118 |
| InsightPro | Delivers scheduled BI reports with trend forecasting powered by time-series models. Used by finance and operations teams. | 99.0% | 4.7 | 103 |
| NarratorAI | Transforms structured data and briefs into compelling narratives for internal and external communications. | 96.4% | 4.5 | 67 |
| HelpBot Ultra | High-volume customer support agent designed for e-commerce. Manages returns, FAQs, and CSAT surveys autonomously. | 99.9% | 4.7 | 198 |
| SQLWizard | Converts plain-English business questions into optimized SQL queries across multiple database dialects. | 97.8% | 4.6 | 88 |
| ContentFlow | Produces a steady stream of blog content, social posts, and newsletter issues on a scheduled basis. | 98.3% | 4.5 | 95 |
| DevAssist | Pairs with engineering teams to review pull requests, catch bugs, and suggest TypeScript improvements. | 98.9% | 4.8 | 112 |
| BrandVoice | Maintains consistent brand messaging across all channels; generates email campaigns and brand guidelines. | 97.1% | 4.6 | 79 |
| LogicBot | Builds robust Python data pipelines and REST API integrations with built-in error handling. | 96.0% | 4.4 | 54 |
| TrendSpotter | Monitors social media and search trends in real time, generating daily insight reports. | 95.5% | 4.3 | 41 |
| LexiBot | Drafts legal documents, summarizes contracts, and flags compliance risks for legal teams. | 99.1% | 4.9 | 76 |
| QueryMaster | Powers executive dashboards with real-time KPI tracking and anomaly detection. | 98.6% | 4.7 | 109 |
| ReplyBot | Handles high-volume email queues with intelligent routing and auto-reply generation. | 99.4% | 4.6 | 163 |
| CampaignAI | Plans and optimizes paid ad campaigns across Google Ads and Meta with A/B testing reports. | 96.8% | 4.5 | 62 |

**Edit button:**
- Opens the Add/Edit Agent Modal pre-populated.
- Same form fields as the Add New Agent modal (see 5.3.4).
- Save updates the in-memory agent object and re-renders that row. Toast: "Agent updated".

**Deactivate / Activate button:**
- If `Active`: sets status to `Inactive`. Toast: "Agent deactivated".
- If `Inactive`: sets status to `Active`. Toast: "Agent activated".
- If `Under Review`: button is disabled (grayed out, not clickable).

---

#### 5.3.4 Add New Agent Modal

Triggered by the "Add New Agent" button in the section header.

Modal title: "Add New Agent"

Form fields (all required):
- Agent Name: `<input type="text">` placeholder "e.g. DataBot Pro"
- Category: `<select>` — `Data Analysis | Writing | Customer Support | Code Generation | Marketing`
- Description: `<textarea rows="3">` placeholder "Describe what this agent does…"
- Skills: `<input type="text">` placeholder "e.g. NLP, SQL, Data Viz (comma-separated)"
- Owner: `<input type="text">` placeholder "Owner's full name"
- Hourly Rate ($): `<input type="number" min="1">` placeholder "45"
- Status: `<select>` — `Active | Inactive | Under Review`

On submit:
- Validates all fields are non-empty (shows inline red error text under each empty field if not).
- Generates new ID as `A-0{n+1}` (where n = current array length).
- Prepends the new agent to the hardcoded array.
- Re-renders the agent table (new agent appears on page 1, row 1).
- Closes modal.
- Shows toast: "Agent added successfully".

---

#### 5.3.5 Pagination

- 10 rows per page.
- With 18 agents: 2 pages (page 2 has 8 rows).
- Recalculates after filtering.

---

### 5.4 Skills

**Purpose:** Define and manage the skill taxonomy that agents can be tagged with.

**Layout (top to bottom):**
1. Section Header Bar (title: "Skills", subtitle: "Manage the skill library for AI agents") with "Add Skill" button right-aligned
2. Search Input (left-aligned, below header)
3. 3-column card grid (`grid grid-cols-3 gap-4`)

---

#### 5.4.1 Search Input

- Placeholder: "Search skills…", `w-72`.
- Filters cards in real-time by skill name OR category (case-insensitive substring).
- Cards not matching are hidden (`display: none`).
- No pagination for Skills — all 15 cards are in the DOM; filtering is purely show/hide.

---

#### 5.4.2 Skill Card Structure

```
┌───────────────────────────────────────┐
│  [Emoji Icon]   [Category Badge]      │
│  Skill Name (bold, lg)                │
│  Description (2 lines, gray, sm)      │
│                                       │
│  👤 X Agents                  [✏️][🗑️] │
└───────────────────────────────────────┘
```

- Background: `bg-white`, `rounded-xl`, `shadow`, `p-5`.
- Emoji icon: 32px, displayed in a colored circle background.
- Category badge: small colored pill (same palette as Status Badge but category-colored).
- Skill Name: `text-lg font-semibold text-gray-800`.
- Description: `text-sm text-gray-500`, clamped to 2 lines (`line-clamp-2`).
- Agent count: `text-xs text-gray-500`, with a small user icon.
- Edit (pencil) and Delete (trash) icon buttons at bottom-right of card.

**Hardcoded Skills (15 total):**

| # | Name | Category | Icon | Agents | Description |
|---|---|---|---|---|---|
| 1 | Natural Language Processing | AI/ML | 🧠 | 8 | Understands and generates human language with contextual awareness. |
| 2 | Data Visualization | Analytics | 📊 | 6 | Transforms raw data into clear, interactive charts and dashboards. |
| 3 | Sentiment Analysis | AI/ML | 💬 | 5 | Detects emotional tone and intent in text for customer feedback analysis. |
| 4 | Code Review | Development | 🔍 | 4 | Analyzes code for bugs, style violations, and performance issues. |
| 5 | SEO Optimization | Marketing | 🔎 | 5 | Improves content discoverability through keyword and structure analysis. |
| 6 | SQL | Analytics | 🗄️ | 7 | Writes and optimizes database queries across relational databases. |
| 7 | Copywriting | Content | ✍️ | 6 | Crafts persuasive marketing copy tailored to target audiences. |
| 8 | Forecasting | Analytics | 📈 | 3 | Predicts future trends using statistical and machine learning models. |
| 9 | Python | Development | 🐍 | 5 | Develops scripts, pipelines, and automation using Python. |
| 10 | FAQ Handling | Support | ❓ | 4 | Resolves common customer questions from a structured knowledge base. |
| 11 | Summarization | Content | 📝 | 7 | Condenses long-form documents into concise, readable summaries. |
| 12 | Translation | Content | 🌐 | 3 | Translates content across 50+ languages with cultural nuance. |
| 13 | A/B Testing | Marketing | 🧪 | 2 | Designs and analyzes split tests to optimize conversion rates. |
| 14 | ETL Pipelines | Analytics | 🔄 | 3 | Extracts, transforms, and loads data between systems reliably. |
| 15 | Brand Strategy | Marketing | 🎯 | 2 | Develops cohesive brand positioning, voice, and messaging frameworks. |

---

#### 5.4.3 Add / Edit Skill Modal

**Add Skill modal** (triggered by header button): empty form.
**Edit modal** (triggered by pencil icon on card): pre-populated form.

Modal title: "Add Skill" or "Edit Skill — [Skill Name]"

Form fields:
- Skill Name: `<input type="text">`, required.
- Category: `<select>` — `AI/ML | Analytics | Development | Content | Marketing | Support`.
- Icon (Emoji): `<input type="text" maxlength="2">`, placeholder "e.g. 🧠", required.
- Description: `<textarea rows="2">`, required.

On Add submit: generates new card, appends to grid, closes modal, toast "Skill added".
On Edit submit: updates that card's content in place, closes modal, toast "Skill updated".

---

#### 5.4.4 Delete Skill

Confirmation Modal:
- Title: "Delete Skill"
- Body: "Are you sure you want to delete '[Skill Name]'? Agents currently using this skill will retain the tag."
- Cancel: close, no change.
- Delete (`bg-red-600`): removes card from DOM, updates in-memory array, toast "Skill deleted".

---

### 5.5 Agent Contracts

**Purpose:** View all rental contracts, inspect details, and terminate active or pending contracts.

**Layout (top to bottom):**
1. Section Header Bar (title: "Agent Contracts", subtitle: "View and manage rental agreements") — no primary action button
2. Filter Bar
3. Data Table
4. Pagination Bar

---

#### 5.5.1 Filter Bar

- Search Input: placeholder "Search by contract ID or renter…", `w-72`.
- Dropdown — Status: `All Statuses | Active | Pending | Completed | Cancelled`.
- Date input — Start From: `<input type="date">`, label "From:".
- Date input — Start To: `<input type="date">`, label "To:".

Filter logic:
- Search matches `contractId` OR `renter` (case-insensitive substring).
- Status dropdown: exact match (skip if "All Statuses").
- Date range: show rows where `startDate >= From AND startDate <= To` (each date filter applied only if the input has a value).
- All filters AND'd simultaneously.

---

#### 5.5.2 Contract Table

Columns: Contract ID | Renter | Agent | Start Date | End Date | Duration | Amount | Status | Actions

**Hardcoded Contracts (22 total):**

| Contract ID | Renter | Agent | Start | End | Duration | Amount | Status |
|---|---|---|---|---|---|---|---|
| CT-0001 | Acme Corp | DataBot v2 | 2024-10-01 | 2024-10-31 | 30 days | $1,350 | Completed |
| CT-0002 | Bright Media | WriterPro Elite | 2024-10-05 | 2024-11-04 | 30 days | $1,050 | Completed |
| CT-0003 | CloudStack Inc | SupportAce | 2024-10-10 | 2025-01-10 | 90 days | $2,700 | Active |
| CT-0004 | Delta Retail | CodeSense | 2024-10-15 | 2024-12-15 | 60 days | $3,300 | Active |
| CT-0005 | Echo Analytics | InsightPro | 2024-10-20 | 2024-11-20 | 30 days | $1,500 | Completed |
| CT-0006 | FutureLabs | MarketMind | 2024-11-01 | 2024-12-01 | 30 days | $1,200 | Completed |
| CT-0007 | GreenPath | HelpBot Ultra | 2024-11-05 | 2025-02-05 | 90 days | $2,250 | Active |
| CT-0008 | Harbor Freight | DataBot v2 | 2024-11-10 | 2024-12-10 | 30 days | $1,350 | Completed |
| CT-0009 | Ignite Ventures | CodeSense | 2024-11-15 | 2025-01-15 | 60 days | $3,300 | Active |
| CT-0010 | Jupiter Corp | LexiBot | 2024-11-20 | 2025-02-20 | 90 days | $4,050 | Active |
| CT-0011 | Kinetic Labs | WriterPro Elite | 2024-12-01 | 2024-12-31 | 30 days | $1,050 | Cancelled |
| CT-0012 | LuminaTech | TrendSpotter | 2024-12-05 | 2025-01-05 | 30 days | $1,140 | Completed |
| CT-0013 | Meridian Health | SupportAce | 2024-12-10 | 2025-03-10 | 90 days | $2,700 | Active |
| CT-0014 | Nova Systems | DevAssist | 2024-12-15 | 2025-03-15 | 90 days | $5,400 | Active |
| CT-0015 | Orbit Dynamics | MarketMind | 2025-01-01 | 2025-02-01 | 30 days | $1,200 | Active |
| CT-0016 | Pinnacle Group | QueryMaster | 2025-01-05 | 2025-04-05 | 90 days | $4,230 | Active |
| CT-0017 | Quantum Bridge | ReplyBot | 2025-01-10 | 2025-02-10 | 30 days | $810 | Active |
| CT-0018 | Riviera Brand | BrandVoice | 2025-01-15 | 2025-02-15 | 30 days | $1,260 | Active |
| CT-0019 | Summit Capital | DataBot v2 | 2025-01-20 | — | — | $1,350 | Pending |
| CT-0020 | Terrapin AI | CampaignAI | 2025-01-22 | — | — | $1,320 | Pending |
| CT-0021 | Unity Works | ContentFlow | 2025-01-25 | 2025-04-25 | 90 days | $2,880 | Active |
| CT-0022 | Vertex Labs | SQLWizard | 2025-01-28 | 2025-02-28 | 30 days | $1,440 | Cancelled |

For Pending contracts, End Date and Duration display as "—" (TBD).

---

#### 5.5.3 Contract Row Actions

**View button:**
- Opens a read-only Contract Detail Modal.
- Modal title: "Contract Details — [Contract ID]"
- Modal body sections:
  - **Parties:** Renter name, Agent name (with category).
  - **Dates:** Start Date, End Date (or "TBD"), Duration.
  - **Financials:** Hourly Rate, Total Hours (Duration × 8hrs/day), Amount.
  - **Payment Breakdown:** table with rows:

    | Item | Amount |
    |---|---|
    | Base Fee | [stated Amount] |
    | Platform Fee (10%) | [Amount × 0.10] |
    | Tax (7%) | [Amount × 0.07] |
    | **Total** | **[Base + Platform Fee + Tax]** |

  - **Status:** Status badge.
  - **Terms (hardcoded):** "The renter agrees to use the agent solely for lawful business purposes. The platform reserves the right to terminate service if misuse is detected. All outputs generated by the agent remain the intellectual property of the renter."
- Footer: "Close" button only.

**Terminate button:**
- Visible and enabled ONLY for contracts with status `Active` or `Pending`.
- For `Completed` and `Cancelled` contracts: the Terminate button is not rendered.
- Clicking opens a Confirmation Modal:
  - Title: "Terminate Contract"
  - Body: "Are you sure you want to terminate contract [CT-XXXX]? This action cannot be undone."
  - Cancel: close, no change.
  - Terminate (`bg-red-600`): sets contract status to `Cancelled`, updates badge in row, closes modal, toast "Contract terminated".

---

#### 5.5.4 Pagination

- 10 rows per page.
- With 22 contracts: 3 pages (page 3 has 2 rows).
- Recalculates based on filtered results.

---

### 5.6 Error Log

**Purpose:** Monitor, investigate, and resolve system errors generated by AI agents.

**Layout (top to bottom):**
1. Section Header Bar (title: "Error Log", subtitle: "Monitor and resolve system errors") with "Export" button right-aligned
2. Filter Bar
3. Data Table
4. Pagination Bar

---

#### 5.6.1 Filter Bar

`flex items-center gap-3 flex-wrap mb-4`

- Search Input: placeholder "Search by message or agent…", `w-72`.
- Dropdown — Severity: `All Severities | Critical | Warning | Info`.
- Dropdown — Status: `All Statuses | Resolved | Unresolved`.
- Date input — From: `<input type="date">`, label "From:", shows rows where timestamp date >= selected date.

All four filters AND'd simultaneously. Reset to page 1 on any change.

**Export button** (in section header, right side):
- Styled: `bg-white border border-gray-300 text-gray-700 px-4 py-2 rounded-lg text-sm`.
- On click: shows toast "Export initiated — CSV will be ready shortly". No file is produced.

---

#### 5.6.2 Error Log Table

Columns: Error ID | Timestamp | Agent | User | Type | Message (truncated) | Severity | Status | Actions

Message column: display first 40 characters + "…" if the full message is longer. Full message shown in View modal.

**Hardcoded Errors (25 total):**

| Error ID | Timestamp | Agent | User | Type | Full Message | Severity | Status |
|---|---|---|---|---|---|---|---|
| E-001 | 2025-01-10 08:14:22 | DataBot v2 | Acme Corp | NullReferenceError | Null pointer exception in data pipeline at step 3 | Critical | Resolved |
| E-002 | 2025-01-10 09:31:05 | WriterPro Elite | Bright Media | TimeoutError | Request timed out after 30s — LLM response delayed | Warning | Resolved |
| E-003 | 2025-01-11 11:02:44 | SupportAce | CloudStack Inc | AuthError | Invalid API token provided during session initialization | Critical | Unresolved |
| E-004 | 2025-01-11 13:45:19 | CodeSense | Delta Retail | SyntaxError | Generated code contains unmatched brackets in output block | Warning | Resolved |
| E-005 | 2025-01-12 07:28:01 | MarketMind | FutureLabs | RateLimitError | Upstream rate limit exceeded — retrying in 60 seconds | Info | Resolved |
| E-006 | 2025-01-12 10:55:33 | InsightPro | Echo Analytics | DataFormatError | Unexpected null in column "revenue_Q4" at row 847 | Warning | Unresolved |
| E-007 | 2025-01-13 08:00:15 | HelpBot Ultra | GreenPath | MemoryError | Agent heap exceeded 512MB during bulk ticket processing | Critical | Resolved |
| E-008 | 2025-01-13 14:22:47 | DataBot v2 | Harbor Freight | QueryError | SQL JOIN produced Cartesian product — query aborted | Warning | Resolved |
| E-009 | 2025-01-14 09:10:08 | CodeSense | Ignite Ventures | RuntimeError | Unhandled exception in sandbox execution environment | Critical | Unresolved |
| E-010 | 2025-01-14 16:05:52 | LexiBot | Jupiter Corp | ParseError | Unable to parse PDF attachment — file may be corrupted | Warning | Resolved |
| E-011 | 2025-01-15 10:30:00 | TrendSpotter | LuminaTech | APIError | Third-party trends API returned HTTP 503 | Info | Resolved |
| E-012 | 2025-01-15 11:48:36 | SupportAce | Meridian Health | SessionError | User session expired mid-conversation — context lost | Info | Unresolved |
| E-013 | 2025-01-16 08:05:14 | DevAssist | Nova Systems | TypeMismatch | Expected string, received integer at parameter "userId" | Warning | Resolved |
| E-014 | 2025-01-16 12:33:29 | MarketMind | Orbit Dynamics | CacheError | Cache invalidation failed — serving stale campaign data | Info | Resolved |
| E-015 | 2025-01-17 09:21:55 | QueryMaster | Pinnacle Group | PermissionError | Insufficient database permissions for schema "prod_data" | Critical | Unresolved |
| E-016 | 2025-01-17 14:00:03 | ReplyBot | Quantum Bridge | EncodingError | UTF-8 decode error in customer email attachment | Warning | Resolved |
| E-017 | 2025-01-18 08:45:17 | BrandVoice | Riviera Brand | ModelError | Output token limit exceeded — response truncated at 4096 | Info | Resolved |
| E-018 | 2025-01-18 11:12:44 | DataBot v2 | Summit Capital | NetworkError | Connection to data warehouse timed out after 10 retries | Critical | Unresolved |
| E-019 | 2025-01-19 09:00:00 | CampaignAI | Terrapin AI | ConfigError | Missing required environment variable "META_ACCESS_TOKEN" | Critical | Resolved |
| E-020 | 2025-01-19 13:55:08 | ContentFlow | Unity Works | ValidationError | Output failed content policy check — flagged for review | Warning | Unresolved |
| E-021 | 2025-01-20 08:30:22 | SQLWizard | Vertex Labs | QueryError | Ambiguous column reference in auto-generated JOIN clause | Warning | Resolved |
| E-022 | 2025-01-20 10:44:10 | HelpBot Ultra | GreenPath | OverloadError | Concurrent session limit reached — 50/50 slots occupied | Info | Unresolved |
| E-023 | 2025-01-21 08:15:00 | InsightPro | Echo Analytics | DataFormatError | ISO 8601 date parse failure in column "last_updated" | Info | Resolved |
| E-024 | 2025-01-21 13:28:55 | DevAssist | Nova Systems | SecurityError | Detected potentially unsafe shell command in generated script | Critical | Resolved |
| E-025 | 2025-01-22 09:05:40 | LexiBot | Jupiter Corp | NullReferenceError | Document metadata returned null — unable to extract clauses | Warning | Unresolved |

---

#### 5.6.3 Error Log Row Actions

**View button:**
- Opens an Error Detail Modal.
- Modal title: "Error Details — [Error ID]"
- Modal body sections:
  - **Overview:** Error ID, Timestamp, Severity badge, Status badge.
  - **Context:** Agent name, User (renter), Error Type.
  - **Full Message:** full (untruncated) message text, styled `bg-gray-50 rounded p-3 text-sm font-mono text-gray-800`.
  - **Stack Trace (placeholder):** same code block styling, containing a fictional but realistic trace:
    ```
    at AgentRuntime.execute (runtime.js:142)
    at Pipeline.run (pipeline.js:87)
    at TaskQueue.process (queue.js:33)
    ```
  - **Resolution Status:** current status badge. If Resolved, show a "Resolved at" timestamp hardcoded ~2 hours after the error timestamp.
- Footer: Close button + Mark Resolved / Mark Unresolved button (same behavior as row action).

**Mark Resolved / Mark Unresolved toggle:**
- Appears both as a row action and in the View modal footer.
- Unresolved → "Mark Resolved" (`bg-green-600`): sets status to `Resolved`, updates table row badge, shows toast "Error marked as resolved".
- Resolved → "Mark Unresolved" (`bg-orange-500`): sets status to `Unresolved`, updates table row badge, shows toast "Error marked as unresolved".
- If modal is open when toggled via row action, the modal badge also updates.

---

#### 5.6.4 Pagination

- 10 rows per page.
- With 25 errors: 3 pages (page 3 has 5 rows).
- Recalculates based on filtered results.

---

## 6. Navigation & SPA Behavior

### 6.1 Initial State (page load)
- Dashboard section is visible.
- Dashboard nav item has active class.
- All other 5 sections have `display: none`.
- Page title in top bar reads "Dashboard".

### 6.2 On Nav Click (pseudocode)

```javascript
function navigateTo(sectionId, label) {
  document.querySelectorAll('.section').forEach(s => s.style.display = 'none');
  document.getElementById(sectionId).style.display = 'block';
  document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
  document.getElementById('nav-' + sectionId).classList.add('active');
  document.getElementById('page-title').textContent = label;
}
```

---

## 7. Data Architecture

All data lives in JavaScript variables declared in a `<script>` block (or `app.js`). No data is fetched or persisted.

```javascript
const users     = [ { id, name, email, role, status, joined }, ... ];       // 20 entries
const agents    = [ { id, name, category, skills, owner, rate, status }, ... ]; // 18 entries
const skills    = [ { id, name, category, icon, agentCount, description }, ... ]; // 15 entries
const contracts = [ { id, renter, agent, startDate, endDate, duration, amount, status }, ... ]; // 22 entries
const errors    = [ { id, timestamp, agent, user, type, message, severity, status }, ... ]; // 25 entries
```

State mutations (edits, deletes, status toggles, additions) modify the in-memory arrays directly and re-render only the affected section. A full page refresh resets all data to hardcoded initial values.

---

## 8. Acceptance Criteria

All 72 behaviors below must work correctly for the implementation to be considered complete.

**Navigation**
1. Clicking a sidebar nav item shows only that section's container; all other 5 are hidden.
2. The active sidebar nav item displays an indigo background and left border; all others do not.
3. The top bar page title updates to the clicked section's name on every nav click.

**Dashboard**
4. All 4 KPI cards display the correct hardcoded values and trend percentages.
5. Trend badges display green (↑) for positive and red (↓) for negative trends.
6. The bar chart renders exactly 7 labeled bars using only CSS height — no canvas, no SVG charting library, no JS charting library.
7. Bar heights are proportional to hardcoded rental values (tallest bar = Fri at 63 rentals, 126px).
8. All 8 activity feed items render with correct descriptions, timestamps, and colored dot icons.
9. The Top Agents table renders all 5 rows with correct data and medal emojis for ranks 1–3.

**User Management — Filtering**
10. Typing in the User Management search box immediately hides rows whose name and email do not match the substring.
11. Selecting a role from the Role dropdown hides rows with a different role.
12. Selecting a status from the Status dropdown hides rows with a different status.
13. All three User Management filters apply simultaneously with AND logic.
14. Clearing a filter restores previously hidden rows.

**User Management — Actions**
15. Clicking Edit on any user row opens a modal pre-filled with that user's current data.
16. Saving the Edit User modal updates the Name, Email, Role, and Status in that table row without a page reload.
17. Clicking Suspend on an Active user changes the Status badge to "Suspended" and shows a toast.
18. Clicking Activate on a Suspended user changes the Status badge to "Active" and shows a toast.
19. The Suspend/Activate button is disabled for Pending users.
20. Clicking Delete opens a confirmation modal; Cancel closes it with no change.
21. Confirming Delete removes the row and shows a toast.

**User Management — Pagination**
22. Only 10 user rows are visible per page.
23. Next advances to the next page; Prev returns to the previous page.
24. The "Showing X–Y of Z" label reflects the current page and filtered total.
25. Any filter change resets the view to page 1.

**Agent Management**
26. Clicking "Add New Agent" opens a modal with all form fields empty.
27. Submitting the Add Agent form with all fields filled adds a new row at the top of the agent table.
28. Submitting with any empty field shows inline validation errors and does not close the modal.
29. Clicking View opens a read-only modal with description, skills pills, and all three performance stats.
30. Clicking Edit opens a modal pre-filled with that agent's current data.
31. Saving the Edit Agent modal updates that row in the table.
32. Clicking Deactivate on an Active agent sets its badge to "Inactive" and shows a toast.
33. Clicking Activate on an Inactive agent sets its badge to "Active" and shows a toast.
34. The Deactivate/Activate button is disabled for agents with status "Under Review".
35. Agent Management category and status dropdowns filter the table (AND'd with search).

**Skills**
36. All 15 skill cards render in a 3-column grid with correct name, category, icon, description, and agent count.
37. Typing in the Skills search box immediately hides cards whose name and category do not match.
38. Clicking "Add Skill" opens an empty modal form.
39. Submitting the Add Skill form appends a new card to the grid and shows a toast.
40. Clicking Edit on a skill card opens a pre-populated modal.
41. Saving the Edit Skill modal updates the card's displayed content in place.
42. Clicking Delete on a skill card opens a confirmation modal.
43. Confirming Delete removes the card from the grid and shows a toast.

**Agent Contracts**
44. The Contracts search input filters by contract ID and renter name simultaneously.
45. The Status dropdown filters to show only contracts matching the selected status.
46. The Start From and Start To date inputs filter rows to only those whose start date falls within the range.
47. All four Contract filters apply simultaneously with AND logic.
48. Clicking View opens a read-only modal with all fields, payment breakdown (base fee, platform fee 10%, tax 7%, total), and the terms text.
49. The Terminate button is not rendered for contracts with status "Completed" or "Cancelled".
50. Clicking Terminate for an Active or Pending contract opens a confirmation modal.
51. Confirming termination sets the contract's status badge to "Cancelled" and shows a toast.

**Error Log**
52. The Message column is truncated to 40 characters with "…" appended for longer messages.
53. The search input filters rows by both message text and agent name.
54. The Severity dropdown filters rows to only the selected severity.
55. The Status dropdown filters rows to only Resolved or Unresolved.
56. The From date input filters rows to only those with a timestamp on or after the selected date.
57. All four Error Log filters apply simultaneously with AND logic.
58. Clicking View opens a modal with the full untruncated message, stack trace, and all metadata.
59. Clicking "Mark Resolved" on an Unresolved error sets its badge to "Resolved" and shows a toast.
60. Clicking "Mark Unresolved" on a Resolved error sets its badge to "Unresolved" and shows a toast.
61. The toggle button in the error row and the same button in the View modal both update the same record.
62. Clicking the Export button shows a toast "Export initiated — CSV will be ready shortly" (no file download).

**Global Behaviors**
63. All modals close when clicking the X button in the modal header.
64. All modals close when clicking the Cancel button in the modal footer.
65. All modals close when clicking the semi-transparent backdrop behind the modal.
66. Toast notifications appear in the bottom-right corner of the viewport.
67. Toast notifications automatically disappear after 2 seconds.
68. Only one toast is visible at a time; a new toast replaces any currently visible toast.
69. Zero network requests are made at any point — no `fetch`, `XMLHttpRequest`, or external image/font URLs other than the Tailwind CDN script.
70. The only external resource loaded is the Tailwind CSS v4 CDN script tag.
71. The layout does not break or overflow horizontally at a 1280px viewport width.
72. All sidebar navigation, filters, modals, and actions function correctly without any page reload.

## 9. UI State Rules
- Only one section is visible at a time.
- Only one modal can be open at a time.
- Only one toast can exist at a time.
- All filters reset pagination to page 1.
- All actions immediately update the UI without a page reload.

## 10. Empty States
When a table or grid has no results after filtering:
- Show a centered message: "No results found"
- Show subtext: "Try adjusting your filters or search query"
- Hide table body rows or cards that do not match the filters.
- Keep table headers visible for table-based sections.

## 11. UI Error Handling
- Form validation errors show inline below inputs in red text.
- Invalid submissions do not close modals.
- Disabled buttons use `opacity-50 cursor-not-allowed`.
- All required fields must be validated before submission.

## 12. Performance Considerations
- Tables re-render only visible rows.
- Filtering operates on in-memory arrays.
- Avoid unnecessary full-page re-renders.

## 13. Suggested File Structure

* index.html
* app.js
* styles.css (optional)

## 14. Naming Conventions

* Section IDs: section-{name}
* Nav IDs: nav-{section}
* Data arrays: camelCase (users, agents, skills, contracts, errors)
* Functions: verb-based (renderUsers, filterAgents, openModal, closeModal)

## 15. UI State Variables

* currentSection (string)
* currentPage (object per table)
* activeFilters (object per section)
* activeModalId (string or null)
* toastVisible (boolean)

## 16. Rendering Strategy

* Each section must have a dedicated render function.
* Tables re-render when:

  * filters change
  * pagination changes
  * data is updated (add/edit/delete)
* DOM updates should target only affected sections, not the entire page.
