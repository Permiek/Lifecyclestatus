# Plan: Business-User Artifacts for Lifecycle Status Scenarios

## Context

This repository documents how four entities (Order, User Account, Support Ticket, Document) move through lifecycle statuses in a **snapshot-ingestion system** that observes — but does not enforce — transitions. Business users need to:

1. **Understand** what statuses exist and what each transition means
2. **Confirm** that documented progressions, regressions, and lateral moves match their actual processes
3. **Explore** "what happens if…" scenarios interactively before implementation teams build against the spec

---

## Artifacts to Build

### 1. `data/transitions.json` — Machine-Readable Transition Registry

A structured JSON file encoding every known transition across all four entities. This is the **data foundation** that powers the interactive HTML tools.

Each record contains:
- `entity`, `from`, `to`
- `classification`: `"progression"` | `"regression"` | `"lateral"`
- `ordinal_from`, `ordinal_to` (canonical positions; `null` for branch statuses)
- `typical_cause` — the one-line business reason
- `handling_notes` — what the downstream system should do (particularly for regressions)

Business value: implementation teams can use this directly as test fixtures; it also becomes a single authoritative reference that avoids spec drift between the HTML tools and the markdown docs.

---

### 2. `data/sample-snapshots.json` — Realistic Snapshot Sequences

Concrete example data showing exactly what the ingestion system would **receive and classify**. Structured as named scenarios per entity, e.g.:

```json
{
  "entity": "order",
  "scenario": "carrier-returns-package",
  "snapshots": [
    { "seq": 1, "entity_id": "ORD-001", "timestamp": "2024-01-10T09:00:00Z", "status": "Processing" },
    { "seq": 2, "entity_id": "ORD-001", "timestamp": "2024-01-10T14:30:00Z", "status": "Shipped" },
    { "seq": 3, "entity_id": "ORD-001", "timestamp": "2024-01-12T08:00:00Z", "status": "Processing",
      "inferred_transition": { "from": "Shipped", "to": "Processing", "classification": "regression" } }
  ]
}
```

Scenarios covered per entity:
- Happy path (full canonical journey)
- A regression (with business cause annotated)
- A skipped-state gap (snapshots jump multiple positions)
- A terminal-state anomaly (transition out of Completed / Deactivated / etc.)

Business value: business users can read real-looking data rather than abstract rules; developers get ready-made test fixtures.

---

### 3. `explorer/index.html` — Interactive State Machine Explorer

A **self-contained, single HTML file** (no build tools, no external dependencies — opens directly in any browser). This is the primary interactive artifact.

#### Features

**Entity tabs** — Order | User Account | Support Ticket | Document

**Status card grid** — each status rendered as a clickable card showing:
- Status name and description
- Canonical position badge (or "branch" label)
- Click to select as current state

**Transition panel (context-sensitive)** — once a state is selected, shows:
- All valid outbound transitions, grouped by classification
- Color-coded badges: `PROGRESSION` (green) | `REGRESSION` (amber) | `LATERAL` (blue)
- Typical cause and handling note for each

**Journey Tracer** — lets a business user walk through a sequence of transitions step by step:
- Click a starting state → click the next state → repeat
- Each step is logged in a running table: From | To | Classification | Cause
- Invalid transitions (not in spec) shown in red with an alert
- "Reset" button to start a new journey
- "Export" button to copy the journey log as plain text

**Instant Transition Classifier** — two dropdowns (From / To) + a "Classify" button:
- Returns the classification, typical cause, and handling note
- Returns "Not a documented transition — verify with upstream" for unknown pairs

---

### 4. `explorer/confirm.html` — Business Confirmation Checklist

A structured HTML form business users fill out to formally sign off on the documented scenarios before implementation begins.

#### Structure

Each entity has a **section** with pre-populated scenario cards. Example cards:

> **Scenario: Order regression — carrier returns package**
> The system receives a snapshot showing `Shipped → Processing`. This is classified as a **regression**. The upstream system indicates the carrier returned the package before delivery. The ingestion system should flag the regression, suppress any delivery-confirmation downstream events, and alert the fulfillment team.
>
> Does this match your business process?
> ○ Yes, as documented  ○ Partially — see notes  ○ No — needs revision  ○ Not applicable to us
> Notes: _______________

Scenarios pre-populated:
- 1 happy-path scenario per entity
- All documented regression scenarios (highest business-confirmation value)
- 1 terminal-state anomaly scenario per entity

**Completion tracker** — shows `X of Y scenarios confirmed` with a colour indicator (red → amber → green).

**Export** — "Copy responses as JSON" button for attaching confirmation to a ticket or Confluence page. Also a clean **print stylesheet** for physical sign-off if required.

---

### 5. `scenarios/` — Narrative Scenario Walkthroughs (4 Markdown files)

One file per entity, written as **business stories** rather than spec tables. These give reviewers who prefer prose over diagrams a way to read the lifecycle as a narrative.

Structure per file:

1. **Happy path** — full story from creation to terminal state, annotated at each status with what the snapshot system sees and classifies
2. **Regression in context** — the most operationally significant regression for that entity, explained with a realistic business cause and the expected system response
3. **Snapshot gap / skipped states** — a scenario where an intermediate status was never observed; explains what the system records and why that is not an error
4. **Terminal-state anomaly** — a snapshot that arrives after the entity reached a terminal state; explains the alert behavior

Files: `scenarios/order-scenarios.md`, `scenarios/user-account-scenarios.md`, `scenarios/support-ticket-scenarios.md`, `scenarios/document-scenarios.md`

---

## File Layout

```
Lifecyclestatus/
├── README.md                          (existing)
├── order.md                           (existing)
├── user-account.md                    (existing)
├── support-ticket.md                  (existing)
├── document.md                        (existing)
├── data/
│   ├── transitions.json               NEW — full transition registry
│   └── sample-snapshots.json          NEW — example snapshot sequences
├── explorer/
│   ├── index.html                     NEW — interactive state machine explorer
│   └── confirm.html                   NEW — business confirmation checklist
└── scenarios/
    ├── order-scenarios.md             NEW — order narrative walkthrough
    ├── user-account-scenarios.md      NEW — user account narrative walkthrough
    ├── support-ticket-scenarios.md    NEW — support ticket narrative walkthrough
    └── document-scenarios.md          NEW — document narrative walkthrough
```

---

## Build Order

1. `data/transitions.json` — foundation; all other artifacts derive from it
2. `data/sample-snapshots.json` — concrete test data
3. `explorer/index.html` — primary interactive tool (embeds transition data inline so it works without a server)
4. `explorer/confirm.html` — confirmation workflow
5. `scenarios/*.md` — narrative walkthroughs (four files in parallel)
