# Plan: Design System Prior to Remaining Artifacts

## Why a design system first

All remaining artifacts — `confirm.html`, `sample-snapshots.json` viewer, `scenarios/*.md` print stylesheet — share the same visual vocabulary already defined in `explorer/index.html`. That file carries its design as 120+ lines of embedded `<style>`. If we build `confirm.html` next without extracting those styles first, we either copy-paste and drift, or rebuild from scratch.

Establishing a thin, file-based design system now means every artifact that follows opens with two `<link>` tags and inherits a consistent foundation rather than defining its own.

---

## What "design system" means here

No build tools. No framework. No npm. This repo is static files — the design system is **two CSS files** that live in `design-system/` and are linked by each HTML artifact.

```
design-system/
  tokens.css       ← design decisions expressed as CSS custom properties
  components.css   ← reusable CSS classes consumed by all HTML artifacts
```

---

## Layer 1 — `design-system/tokens.css`

All decision-bearing values in one place. Every artifact `var()`-references these; nothing hard-codes a hex colour or font size outside this file.

### Colour tokens

**Surface / structure**
```css
--color-bg          /* page background          #f7f8fa */
--color-surface     /* panel / card background  #ffffff */
--color-border      /* dividers, card borders   #e2e6ec */
--color-surface-sub /* subtle inset fill        #eef1f5 */
```

**Text**
```css
--color-ink         /* primary text    #1c2430 */
--color-muted       /* secondary text  #5c6776 */
--color-ink-inverse /* text on dark bg #ffffff */
--color-ink-dim     /* header subtitle #b7c0cd */
```

**Accent (interactive)**
```css
--color-accent      /* links, focus, active tab indicator  #2f6fed */
```

**Semantic — lifecycle classification (the core domain colours)**
```css
--color-prog        /* progression text   #1f9d57 */
--color-prog-bg     /* progression fill   #e6f6ec */
--color-regr        /* regression text    #c77700 */
--color-regr-bg     /* regression fill    #fcf1de */
--color-lat         /* lateral text       #2f6fed */
--color-lat-bg      /* lateral fill       #e8f0fe */
--color-danger      /* not-in-spec text   #d23b3b */
--color-danger-bg   /* not-in-spec fill   #fdeaea */
--color-term        /* terminal text      #6b4fbb */
--color-term-bg     /* terminal fill      #efeafb */
```

### Typography tokens
```css
--font-sans         /* system stack: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, … */
--font-size-xs      /* 11px */
--font-size-sm      /* 12px */
--font-size-base    /* 13px */
--font-size-md      /* 14px */
--font-size-lg      /* 15px */
--font-size-xl      /* 20px */
--font-weight-normal /* 400 */
--font-weight-semi  /* 550 */
--font-weight-bold  /* 650 */
--font-weight-heavy /* 700 */
--line-height-base  /* 1.5 */
```

### Spacing tokens (4-point scale)
```css
--space-1  /* 4px  */
--space-2  /* 8px  */
--space-3  /* 12px */
--space-4  /* 16px */
--space-5  /* 20px */
--space-6  /* 24px */
--space-8  /* 32px */
--space-10 /* 40px */
```

### Shape / elevation tokens
```css
--radius-sm   /* 6px  */
--radius-md   /* 8px  */
--radius-lg   /* 10px */
--radius-pill /* 20px */
--shadow-card /* 0 2px 10px rgba(47,111,237,.08) */
```

### Base reset (also in tokens.css)
```css
*, *::before, *::after { box-sizing: border-box; }
body { margin: 0; font-family: var(--font-sans); font-size: var(--font-size-base);
       color: var(--color-ink); background: var(--color-bg); line-height: var(--line-height-base); }
```

---

## Layer 2 — `design-system/components.css`

Reusable CSS classes covering every pattern already used in `explorer/index.html` and needed by `confirm.html` and the scenario pages. Class names stay exactly as they are — no rename, so `index.html` needs only a `<link>` swap, not a class rename sweep.

### Layout
| Class | Purpose |
|---|---|
| `.wrap` | Max-width 1180px centred content area, horizontal padding |
| `.cols` | Two-column responsive grid (collapses at 820px) |
| `.row-inline` | Flex row with wrapping, used for form controls |

### Navigation
| Class | Purpose |
|---|---|
| `.tabs` | Tab bar container |
| `.tab` | Individual tab; `.tab.active` gets accent underline |

### Panels & cards
| Class | Purpose |
|---|---|
| `.panel` | White card with border and padding — the main content container |
| `.entity-meta` | Borderless panel with 0 8px 8px 8px radius (connects flush below tabs) |
| `.grid` | Auto-fill card grid, 180px minimum column |
| `.card` | Clickable status card; `.card.selected` shows accent ring |
| `.req-item` | Requirement/flag item — heading + note + remove button |

### Typography
| Class | Purpose |
|---|---|
| `h2.section` | Section divider heading — uppercase, muted, spaced |
| `.pill` | Inline label chip — for canonical sequence and branch labels |
| `.arrow` | Muted chevron separator used in canonical sequences |
| `.empty-hint` | Italic muted placeholder text |

### Classification & status
| Class | Purpose |
|---|---|
| `.badge` | Base badge style — min-width, uppercase, small, rounded |
| `.badge.progression` | Green fill |
| `.badge.regression` | Amber fill |
| `.badge.lateral` | Blue fill |
| `.badge.none` | Red fill — undocumented transition |
| `.mini-badge` | Smaller inline badge variant for table cells |
| `.term-flag` | Purple "terminal" or "end" pill used on terminal-state cards and warnings |

### Page header
| Class | Purpose |
|---|---|
| `header.top` | Full-width dark header bar |

### Forms & controls
| Class | Purpose |
|---|---|
| `select`, `input[type=text]` | Base form control styling |
| `button` | Primary button (accent fill) |
| `button.ghost` | Secondary button (white, bordered) |
| `button.small` | Compact button modifier |
| `.result-box` | Classifier result container — hidden until `.result-box.show` |
| `label.fld` | Uppercase field label above a control |

### Data
| Class | Purpose |
|---|---|
| `table.journey` | Step-log table — full width, collapsed borders |
| `.trans-row` | Flex row: badge + body (cause, handling note) |
| `.trans-body` | Right side of a trans-row |
| `.handling` | Amber inset box for regression handling notes |

### Feedback
| Class | Purpose |
|---|---|
| `.toast` | Fixed bottom toast notification — shown via `.toast.show` |

---

## Refactoring `explorer/index.html`

The `<style>` block (lines 8–130 approximately) is **deleted** and replaced with:

```html
<link rel="stylesheet" href="../design-system/tokens.css">
<link rel="stylesheet" href="../design-system/components.css">
```

No class names change. No JS changes. The visual output is identical — only the location of the CSS moves.

This is the only existing file that needs touching.

---

## How future artifacts use it

Every subsequent HTML file opens with the same two `<link>` tags and gets the full token set and component library for free. File-specific overrides (if needed) go in a small `<style>` block after the links — not in the shared files.

```html
<!-- confirm.html -->
<link rel="stylesheet" href="../design-system/tokens.css">
<link rel="stylesheet" href="../design-system/components.css">
<!-- any confirm-specific overrides: -->
<style>
  .check-card { … }
</style>
```

---

## Scenario pages (`scenarios/*.md`)

These are Markdown, not HTML. They inherit design via a separate **print/web stylesheet**:
`design-system/prose.css` — a lightweight reading stylesheet for rendered Markdown (margins, max-width, heading scale, table styles). This is added in the plan but not listed as a blocker for the HTML artifacts.

---

## Build order

1. `design-system/tokens.css` — write token file
2. `design-system/components.css` — extract all classes from `explorer/index.html`
3. Refactor `explorer/index.html` — swap `<style>` block for two `<link>` tags, verify visually unchanged
4. Then build `explorer/confirm.html` — starts from the design system, not from scratch
5. Then `scenarios/*.md` + optional `design-system/prose.css`
