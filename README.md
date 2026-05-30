# Lifecycle Status Explained

This directory contains explanation materials for how logical data entities move through defined lifecycle statuses, as observed by a **snapshot-ingestion system**.

---

## The Snapshot Model

This system does not drive or enforce transitions. It receives periodic **snapshots** of entity data from an upstream source and infers what has changed by comparing the current snapshot to the previously recorded state.

Key consequences:

- **The system observes transitions; it does not cause them.** The upstream source is the authority on what status an entity holds. Guard conditions and business rules live there, not here.
- **Transitions can be forward or backward.** The upstream source may correct errors, reverse operations, or apply compensating actions. A `Processing` order legitimately revert to `Confirmed`. The system must handle this without treating it as corrupt data.
- **Intermediate states may be skipped.** If snapshots are infrequent, an entity might have passed through one or more intermediate statuses between two observed snapshots. The system sees only the before and after.
- **No transition is received as an event.** A transition is inferred by the system when `snapshot.status != last_known_status`.

---

## Progression vs. Regression

Each entity's statuses have a **canonical ordering** — a sequence that represents the normal forward journey. A transition is classified as:

| Classification | Meaning |
|---------------|---------|
| **Progression** | Status moves forward in the canonical order (e.g., `Confirmed → Processing`). |
| **Regression** | Status moves backward in the canonical order (e.g., `Processing → Confirmed`). |
| **Lateral** | Status moves to a branch outside the main sequence (e.g., `Confirmed → Cancelled`). |

The canonical order for each entity is defined in its own file. The system assigns each status an ordinal position; comparing ordinals of before and after determines the classification.

Regressions are **valid and expected** — they are not errors. However, they may warrant:
- Different downstream processing logic
- Alerting or flagging for human review
- Rollback of any actions taken upon the earlier forward transition (e.g., reversing an invoice issued when the order reached `Processing`)

---

## What the System Does With a Transition

When a snapshot reveals a status change, the system should:

1. **Record the observed transition** — previous status, new status, entity ID, snapshot timestamp, classification (progression / regression / lateral).
2. **Classify the direction** — compare ordinal positions.
3. **Route to appropriate handlers** — forward and backward transitions for the same status pair may require different downstream actions.
4. **Handle skipped states** — if the ordinal gap is greater than 1, consider whether intermediate states need synthetic events or compensating logic.

---

## Entities Covered

| File | Entity | Core concern |
|------|--------|-------------|
| [order.md](order.md) | Order | Fulfillment pipeline from cart to delivery |
| [user-account.md](user-account.md) | User Account | Identity and access from registration to deactivation |
| [support-ticket.md](support-ticket.md) | Support Ticket | Issue resolution from open to closed |
| [document.md](document.md) | Document / Content | Editorial workflow from draft to archive |

---

## Reading the Diagrams

Each entity file contains a state diagram in [Mermaid](https://mermaid.js.org/) syntax (rendered by GitHub, GitLab, and most markdown viewers).

- Solid arrows (`-->`) show **forward / lateral** transitions.
- Dashed arrows (`-.->`) show **regression** transitions (backward along the canonical order).
- The canonical order is listed explicitly in each file as a numbered sequence.
- Terminal states have no outbound transitions.
