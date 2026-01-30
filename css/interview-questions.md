# CSS Interview Questions & Answers

---

## 1. What is the CSS Box Model?

Every HTML element is treated as a rectangular box. The box model consists of four layers (inside out):

1. **Content** — the actual content (text, images)
2. **Padding** — space between content and border
3. **Border** — surrounds the padding
4. **Margin** — space outside the border (between elements)

```css
.box {
  width: 200px;
  padding: 20px;
  border: 5px solid #000;
  margin: 10px;
}

/* Total width = 200 + 20*2 + 5*2 + 10*2 = 270px (content-box) */
```

**`box-sizing` property:**

```css
/* content-box (default) — width/height applies to content only */
.box { box-sizing: content-box; width: 200px; padding: 20px; }
/* Total rendered width = 200 + 20 + 20 = 240px */

/* border-box — width/height includes padding and border */
.box { box-sizing: border-box; width: 200px; padding: 20px; }
/* Total rendered width = 200px (content shrinks to 160px) */
```

Best practice: apply `border-box` globally:

```css
*, *::before, *::after {
  box-sizing: border-box;
}
```

---

## 2. What is the difference between `margin` and `padding`?

| Feature | Margin | Padding |
|---|---|---|
| Location | Outside the border | Inside the border |
| Background | Transparent (shows parent background) | Takes element's background |
| Collapsing | Vertical margins collapse | Never collapses |
| Negative values | Allowed | Not allowed |
| Click area | Not part of element | Part of element (clickable) |

**Margin collapsing:** When two vertical margins meet, they collapse into the larger one.

```css
.box-a { margin-bottom: 30px; }
.box-b { margin-top: 20px; }
/* Actual gap = 30px (not 50px) — the larger margin wins */
```

Margin collapsing does NOT happen with:
- Floated elements
- Flex/grid items
- Absolutely positioned elements
- Elements with `overflow` other than `visible`

---

## 3. What is CSS specificity?

Specificity determines which CSS rule applies when multiple rules target the same element. It is calculated as a four-part value: `(inline, ID, class, element)`.

| Selector | Specificity | Score |
|---|---|---|
| `*` | (0, 0, 0, 0) | 0 |
| `div` | (0, 0, 0, 1) | 1 |
| `.class` | (0, 0, 1, 0) | 10 |
| `#id` | (0, 1, 0, 0) | 100 |
| `style=""` (inline) | (1, 0, 0, 0) | 1000 |
| `!important` | Overrides everything | — |

```css
/* Specificity: (0, 0, 0, 1) */
p { color: blue; }

/* Specificity: (0, 0, 1, 1) — wins */
p.intro { color: red; }

/* Specificity: (0, 1, 0, 1) — wins over both */
#main p { color: green; }
```

**Rules:**
1. Higher specificity wins
2. Equal specificity — last rule wins (source order)
3. `!important` overrides all specificity (avoid using it)
4. Inline styles override stylesheets (unless `!important` is used)

---

## 4. What is the CSS cascade?

The cascade is the algorithm that determines which CSS declarations apply to an element. It resolves conflicts using this priority order:

1. **Origin and importance:**
   - User agent `!important`
   - User `!important`
   - Author `!important`
   - CSS animations (`@keyframes`)
   - Author styles
   - User styles
   - User agent (browser defaults)

2. **Specificity** — higher specificity wins

3. **Source order** — later declarations win

4. **Layers** (`@layer`) — later layers have higher priority

```css
@layer base, components, utilities;

@layer base {
  p { color: blue; }
}

@layer utilities {
  .text-red { color: red; } /* wins over base layer */
}
```

---

## 5. What is Flexbox?

Flexbox is a one-dimensional layout model for arranging items in rows or columns.

```css
.container {
  display: flex;

  /* Direction */
  flex-direction: row;          /* row | row-reverse | column | column-reverse */

  /* Wrapping */
  flex-wrap: wrap;              /* nowrap | wrap | wrap-reverse */

  /* Main axis alignment */
  justify-content: center;     /* flex-start | flex-end | center | space-between | space-around | space-evenly */

  /* Cross axis alignment */
  align-items: center;         /* flex-start | flex-end | center | stretch | baseline */

  /* Multi-line cross axis */
  align-content: space-between; /* same values as justify-content + stretch */

  /* Gap between items */
  gap: 16px;
}

.item {
  /* Growth factor */
  flex-grow: 1;     /* how much to grow relative to siblings */

  /* Shrink factor */
  flex-shrink: 0;   /* how much to shrink (0 = don't shrink) */

  /* Base size */
  flex-basis: 200px; /* initial size before growing/shrinking */

  /* Shorthand */
  flex: 1 0 200px;  /* grow shrink basis */
  flex: 1;          /* same as 1 1 0 */

  /* Individual cross-axis alignment */
  align-self: flex-end;

  /* Order */
  order: 2;
}
```

**Common patterns:**

```css
/* Center anything */
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* Sticky footer */
body {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}
main { flex: 1; }

/* Holy grail layout */
.layout {
  display: flex;
}
.sidebar { flex: 0 0 250px; }
.content { flex: 1; }
```

---

## 6. What is CSS Grid?

CSS Grid is a two-dimensional layout system for creating complex layouts with rows and columns.

```css
.container {
  display: grid;

  /* Define columns */
  grid-template-columns: 200px 1fr 200px;     /* fixed + flexible */
  grid-template-columns: repeat(3, 1fr);       /* three equal columns */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); /* responsive */

  /* Define rows */
  grid-template-rows: auto 1fr auto;

  /* Gap */
  gap: 16px;                   /* row and column gap */
  row-gap: 16px;
  column-gap: 24px;

  /* Named areas */
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";

  /* Alignment */
  justify-items: center;       /* horizontal alignment of items within cells */
  align-items: center;         /* vertical alignment of items within cells */
  justify-content: center;     /* horizontal alignment of the grid itself */
  align-content: center;       /* vertical alignment of the grid itself */
}

.item {
  /* Place by line numbers */
  grid-column: 1 / 3;         /* span from line 1 to 3 */
  grid-row: 2 / 4;

  /* Span shorthand */
  grid-column: span 2;        /* span 2 columns */

  /* Place by area name */
  grid-area: header;

  /* Self alignment */
  justify-self: end;
  align-self: center;
}
```

**Common patterns:**

```css
/* Responsive card grid */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 24px;
}

/* Full page layout */
.page {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}
.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }
```

---

## 7. When should you use Flexbox vs Grid?

| Feature | Flexbox | Grid |
|---|---|---|
| Dimensions | One-dimensional (row OR column) | Two-dimensional (rows AND columns) |
| Content vs layout | Content-driven (items define size) | Layout-driven (grid defines size) |
| Alignment | Excellent on one axis | Excellent on both axes |
| Use case | Navigation, toolbars, single-row/column | Page layouts, card grids, dashboards |
| Item placement | Sequential | Precise (row + column) |
| Overlap | No | Yes (`grid-area` overlap) |

**Use Flexbox for:**
- Navigation bars
- Centering content
- Equal-height columns
- Distributing space among items

**Use Grid for:**
- Page layouts
- Card grids
- Complex multi-area layouts
- Overlapping elements
- When you need control over both axes

They work well together — Grid for the macro layout, Flexbox for component-level alignment.

---

## 8. What is the `position` property?

| Value | Description |
|---|---|
| `static` | Default. Follows normal document flow. `top/right/bottom/left` have no effect. |
| `relative` | Positioned relative to its normal position. Still takes up space in flow. |
| `absolute` | Removed from flow. Positioned relative to nearest positioned ancestor. |
| `fixed` | Removed from flow. Positioned relative to the viewport. Stays on scroll. |
| `sticky` | Acts like `relative` until a scroll threshold, then acts like `fixed`. |

```css
/* Relative — offset from normal position */
.relative {
  position: relative;
  top: 10px;    /* moved 10px down from normal position */
  left: 20px;   /* moved 20px right from normal position */
}

/* Absolute — positioned within nearest positioned ancestor */
.parent { position: relative; }
.child {
  position: absolute;
  top: 0;
  right: 0;     /* top-right corner of parent */
}

/* Fixed — stays in viewport */
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 100;
}

/* Sticky — sticks when scrolled past threshold */
.section-header {
  position: sticky;
  top: 0;       /* sticks to top when scrolled past */
}
```

---

## 9. What is the `display` property?

```css
/* Block — takes full width, starts on new line */
display: block;       /* div, p, h1-h6, section */

/* Inline — flows with text, width/height ignored */
display: inline;      /* span, a, strong, em */

/* Inline-block — flows inline but respects width/height */
display: inline-block;

/* None — removed from layout and accessibility tree */
display: none;

/* Flex — flexbox container */
display: flex;
display: inline-flex;

/* Grid — grid container */
display: grid;
display: inline-grid;

/* Contents — element disappears, children move up */
display: contents;

/* Table layouts */
display: table;
display: table-row;
display: table-cell;
```

**`display: none` vs `visibility: hidden` vs `opacity: 0`:**

| Property | Space | Clickable | Screen readers | Transitions |
|---|---|---|---|---|
| `display: none` | No | No | Hidden | No |
| `visibility: hidden` | Yes | No | Hidden | Yes |
| `opacity: 0` | Yes | Yes | Visible | Yes |

---

## 10. What are CSS selectors?

```css
/* Basic selectors */
*               { }  /* Universal */
div             { }  /* Type/element */
.class          { }  /* Class */
#id             { }  /* ID */
[attr]          { }  /* Has attribute */
[attr="val"]    { }  /* Attribute equals */
[attr^="val"]   { }  /* Starts with */
[attr$="val"]   { }  /* Ends with */
[attr*="val"]   { }  /* Contains */

/* Combinators */
div p           { }  /* Descendant (any depth) */
div > p         { }  /* Direct child */
div + p         { }  /* Adjacent sibling (next) */
div ~ p         { }  /* General sibling (all after) */

/* Pseudo-classes */
:hover          { }  /* Mouse over */
:focus          { }  /* Has focus */
:focus-visible  { }  /* Keyboard focus (not mouse) */
:active         { }  /* Being clicked */
:visited        { }  /* Visited link */

:first-child    { }  /* First child of parent */
:last-child     { }  /* Last child */
:nth-child(2)   { }  /* 2nd child */
:nth-child(odd) { }  /* Odd children */
:nth-child(3n)  { }  /* Every 3rd */
:nth-of-type(2) { }  /* 2nd of its type */
:only-child     { }  /* Only child of parent */

:not(.class)    { }  /* Negation */
:is(h1, h2, h3) { } /* Matches any (forgiving) */
:where(h1, h2)  { } /* Like :is but 0 specificity */
:has(.child)     { } /* Parent selector (contains .child) */

:empty          { }  /* No children or text */
:disabled       { }  /* Disabled form element */
:checked        { }  /* Checked checkbox/radio */
:required       { }  /* Required form field */
:valid          { }  /* Valid form field */
:invalid        { }  /* Invalid form field */
:placeholder-shown { } /* Placeholder visible */

/* Pseudo-elements */
::before        { content: ''; }  /* Insert before content */
::after         { content: ''; }  /* Insert after content */
::first-line    { }                /* First line of text */
::first-letter  { }                /* First letter */
::selection     { }                /* Selected text */
::placeholder   { }                /* Input placeholder */
```

---

## 11. What is the `:has()` selector?

`:has()` is a relational pseudo-class that selects an element based on its descendants or siblings. It effectively acts as a "parent selector."

```css
/* Select a card that contains an image */
.card:has(img) {
  grid-column: span 2;
}

/* Style a form group when its input is invalid */
.form-group:has(:invalid) {
  border-color: red;
}

/* Style label when its sibling input is focused */
label:has(+ input:focus) {
  color: blue;
}

/* Page has a modal open */
body:has(.modal.open) {
  overflow: hidden;
}

/* Select figure only if it has a figcaption */
figure:has(figcaption) {
  border: 1px solid #ccc;
}

/* Quantity selector — different items in grid */
.grid:has(> :nth-child(4)) {
  grid-template-columns: repeat(2, 1fr); /* switch to 2 columns when 4+ items */
}
```

---

## 12. What are CSS Custom Properties (Variables)?

Custom properties are reusable values defined with `--` prefix and accessed with `var()`.

```css
:root {
  --color-primary: #3b82f6;
  --color-secondary: #64748b;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 32px;
  --border-radius: 8px;
  --font-sans: 'Inter', system-ui, sans-serif;
}

.button {
  background: var(--color-primary);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--border-radius);
  font-family: var(--font-sans);
}

/* Fallback value */
.box {
  color: var(--text-color, #333);
}

/* Scoped variables */
.card {
  --card-padding: 24px;
  padding: var(--card-padding);
}

/* Dynamic with JavaScript */
document.documentElement.style.setProperty('--color-primary', '#ef4444');

/* Dark mode with variables */
:root {
  --bg: #ffffff;
  --text: #1a1a1a;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #1a1a1a;
    --text: #ffffff;
  }
}

/* Computation with calc() */
.container {
  padding: calc(var(--spacing-md) * 2);
  width: calc(100% - var(--sidebar-width, 250px));
}
```

---

## 13. What are CSS transitions?

Transitions animate property changes smoothly over a duration.

```css
.button {
  background: #3b82f6;
  color: white;
  transform: scale(1);

  /* Individual properties */
  transition-property: background, transform;
  transition-duration: 0.3s;
  transition-timing-function: ease-in-out;
  transition-delay: 0s;

  /* Shorthand */
  transition: background 0.3s ease-in-out, transform 0.2s ease;

  /* All properties */
  transition: all 0.3s ease;
}

.button:hover {
  background: #2563eb;
  transform: scale(1.05);
}
```

**Timing functions:**

```css
transition-timing-function: ease;         /* default — slow start and end */
transition-timing-function: linear;       /* constant speed */
transition-timing-function: ease-in;      /* slow start */
transition-timing-function: ease-out;     /* slow end */
transition-timing-function: ease-in-out;  /* slow start and end */
transition-timing-function: cubic-bezier(0.68, -0.55, 0.27, 1.55); /* custom */
transition-timing-function: steps(4);     /* stepped animation */
```

**Properties that can be transitioned:** Any property with interpolatable values (colors, lengths, opacity, transforms). Properties like `display` and `visibility` cannot be smoothly transitioned (though `display` transitions are coming via `@starting-style`).

---

## 14. What are CSS animations?

Animations provide more control than transitions with keyframes and multiple states.

```css
/* Define keyframes */
@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-100px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes pulse {
  0%   { transform: scale(1); }
  50%  { transform: scale(1.1); }
  100% { transform: scale(1); }
}

/* Apply animation */
.element {
  animation-name: slideIn;
  animation-duration: 0.5s;
  animation-timing-function: ease-out;
  animation-delay: 0.2s;
  animation-iteration-count: 1;          /* number or infinite */
  animation-direction: normal;           /* normal | reverse | alternate */
  animation-fill-mode: forwards;         /* none | forwards | backwards | both */
  animation-play-state: running;         /* running | paused */

  /* Shorthand */
  animation: slideIn 0.5s ease-out 0.2s 1 normal forwards;
}

/* Multiple animations */
.element {
  animation:
    fadeIn 0.3s ease-out,
    slideUp 0.5s ease-out 0.2s;
}

/* Pause on hover */
.animated:hover {
  animation-play-state: paused;
}
```

**`animation-fill-mode`:**
- `none` — no styles before/after
- `forwards` — keeps final keyframe state after animation ends
- `backwards` — applies first keyframe during delay
- `both` — combines forwards and backwards

---

## 15. What are CSS transforms?

Transforms modify the visual rendering of an element without affecting layout.

```css
/* 2D transforms */
.element {
  transform: translate(50px, 100px);   /* move X, Y */
  transform: translateX(50px);
  transform: translateY(100px);

  transform: scale(1.5);               /* uniform scale */
  transform: scale(2, 0.5);            /* scaleX, scaleY */

  transform: rotate(45deg);            /* clockwise rotation */
  transform: rotate(-90deg);

  transform: skew(10deg, 20deg);       /* skew X, Y */

  /* Multiple transforms (order matters) */
  transform: translate(50px, 0) rotate(45deg) scale(1.2);

  /* Transform origin (default: center) */
  transform-origin: top left;
  transform-origin: 50% 50%;
}

/* 3D transforms */
.card {
  transform: perspective(800px) rotateY(30deg);
  transform: rotateX(45deg);
  transform: translateZ(100px);
  transform-style: preserve-3d;    /* enable 3D for children */
  backface-visibility: hidden;     /* hide back face */
}

/* Centering with transform */
.centered {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

---

## 16. What are media queries?

Media queries apply styles based on device characteristics.

```css
/* Viewport width */
@media (min-width: 768px) { /* tablet and up */ }
@media (max-width: 767px) { /* mobile only */ }

/* Range syntax (modern) */
@media (width >= 768px) { }
@media (768px <= width <= 1024px) { }

/* Common breakpoints (mobile-first) */
/* Base styles = mobile */
@media (min-width: 640px)  { /* sm */ }
@media (min-width: 768px)  { /* md */ }
@media (min-width: 1024px) { /* lg */ }
@media (min-width: 1280px) { /* xl */ }

/* Orientation */
@media (orientation: landscape) { }
@media (orientation: portrait) { }

/* Color scheme preference */
@media (prefers-color-scheme: dark) {
  :root { --bg: #1a1a1a; --text: #fff; }
}

/* Reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  * {
    animation: none !important;
    transition-duration: 0.01ms !important;
  }
}

/* Hover capability */
@media (hover: hover) {
  .button:hover { background: #ddd; } /* only apply hover on devices that support it */
}

/* Print */
@media print {
  .no-print { display: none; }
  body { font-size: 12pt; }
}

/* High DPI / Retina */
@media (min-resolution: 2dppx) { }
@media (-webkit-min-device-pixel-ratio: 2) { }

/* Combining */
@media (min-width: 768px) and (orientation: landscape) { }
@media (min-width: 768px), (orientation: landscape) { /* OR */ }
```

---

## 17. What are container queries?

Container queries style elements based on the size of a parent container, not the viewport.

```css
/* Define a containment context */
.card-container {
  container-type: inline-size;  /* size | inline-size | normal */
  container-name: card;         /* optional name */
}

/* Query the container */
@container card (min-width: 400px) {
  .card {
    display: flex;
    flex-direction: row;
  }
}

@container card (max-width: 399px) {
  .card {
    display: flex;
    flex-direction: column;
  }
}

/* Shorthand */
.card-container {
  container: card / inline-size;
}

/* Without name (queries nearest ancestor container) */
@container (min-width: 600px) {
  .content { font-size: 1.2rem; }
}

/* Container query units */
.element {
  font-size: 5cqi;   /* 5% of container inline size */
  padding: 2cqb;     /* 2% of container block size */
  width: 50cqw;      /* 50% of container width */
  height: 10cqh;     /* 10% of container height */
}
```

Container queries solve the problem of reusable components that need to adapt to their available space rather than the viewport.

---

## 18. What is responsive design?

Responsive design makes websites adapt to different screen sizes and devices.

**Key techniques:**

```css
/* 1. Fluid widths */
.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 16px;
}

/* 2. Flexible images */
img {
  max-width: 100%;
  height: auto;
}

/* 3. Responsive typography */
html {
  font-size: clamp(14px, 1vw + 10px, 18px);
}

h1 {
  font-size: clamp(1.5rem, 3vw + 1rem, 3rem);
}

/* 4. Mobile-first media queries */
.grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
}

@media (min-width: 768px) {
  .grid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 1024px) {
  .grid { grid-template-columns: repeat(3, 1fr); }
}

/* 5. Viewport meta tag (HTML) */
/* <meta name="viewport" content="width=device-width, initial-scale=1"> */

/* 6. Responsive spacing */
.section {
  padding: clamp(24px, 5vw, 80px);
}
```

---

## 19. What are the `clamp()`, `min()`, and `max()` functions?

```css
/* clamp(minimum, preferred, maximum) */
.text {
  font-size: clamp(1rem, 2.5vw, 2rem);
  /* Never smaller than 1rem, grows with viewport, never larger than 2rem */
}

.container {
  width: clamp(320px, 90%, 1200px);
}

/* min() — returns the smallest value */
.sidebar {
  width: min(300px, 100%);
  /* Whichever is smaller: 300px or 100% of parent */
}

/* max() — returns the largest value */
.element {
  width: max(50%, 300px);
  /* Whichever is larger: 50% or 300px */
}

/* Combining */
.responsive {
  padding: max(16px, 3vw);
  margin: min(5vw, 48px);
  font-size: clamp(0.875rem, 1vw + 0.5rem, 1.125rem);
}

/* Responsive spacing without media queries */
.gap {
  gap: clamp(8px, 2vw, 32px);
}
```

---

## 20. What is `z-index` and stacking context?

`z-index` controls the stacking order of positioned elements on the z-axis. It only works on positioned elements (`position` other than `static`) and flex/grid items.

```css
.behind  { z-index: 1; position: relative; }
.infront { z-index: 2; position: relative; }
```

**Stacking context** is created by:
- Root element (`<html>`)
- `position` + `z-index` (not `auto`)
- `position: fixed` or `position: sticky`
- `opacity` less than 1
- `transform`, `filter`, `perspective`, `clip-path` (any non-`none` value)
- `isolation: isolate`
- Flex/grid child with `z-index` (not `auto`)
- `will-change` with a value that creates a stacking context
- `contain: paint` or `contain: layout`

```css
/* Create isolated stacking context */
.modal-overlay {
  isolation: isolate; /* prevents z-index leaking */
  z-index: 1000;
  position: fixed;
}
```

Key concept: `z-index` values are scoped to their stacking context. A child with `z-index: 9999` cannot appear above another stacking context with a lower `z-index` on the parent.

---

## 21. What are pseudo-elements?

Pseudo-elements create virtual elements that don't exist in the DOM.

```css
/* ::before and ::after */
.quote::before {
  content: '\201C'; /* opening quote character */
  font-size: 2em;
  color: #ccc;
}

.required-field::after {
  content: ' *';
  color: red;
}

/* Decorative elements */
.badge::after {
  content: '';
  position: absolute;
  top: -5px;
  right: -5px;
  width: 10px;
  height: 10px;
  background: red;
  border-radius: 50%;
}

/* Counter */
.list { counter-reset: item; }
.list li::before {
  counter-increment: item;
  content: counter(item) '. ';
  font-weight: bold;
}

/* Attribute value as content */
a[href^="http"]::after {
  content: ' (' attr(href) ')';
}

/* ::first-line and ::first-letter */
p::first-letter {
  font-size: 3em;
  float: left;
  line-height: 0.8;
}

p::first-line {
  font-weight: bold;
}

/* ::selection */
::selection {
  background: #3b82f6;
  color: white;
}

/* ::placeholder */
input::placeholder {
  color: #94a3b8;
  font-style: italic;
}
```

---

## 22. What is the `overflow` property?

```css
/* Values */
overflow: visible;   /* default — content overflows the box */
overflow: hidden;    /* clips overflowing content */
overflow: scroll;    /* always shows scrollbars */
overflow: auto;      /* shows scrollbars only when needed */
overflow: clip;      /* like hidden but no programmatic scrolling */

/* Separate axes */
overflow-x: auto;    /* horizontal */
overflow-y: hidden;  /* vertical */

/* Text overflow (for single-line truncation) */
.truncate {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Multi-line truncation */
.line-clamp {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* Scroll behavior */
html {
  scroll-behavior: smooth;
}

/* Scrollbar styling */
.scrollable::-webkit-scrollbar {
  width: 8px;
}
.scrollable::-webkit-scrollbar-thumb {
  background: #ccc;
  border-radius: 4px;
}

/* Modern scrollbar styling */
.scrollable {
  scrollbar-width: thin;          /* auto | thin | none */
  scrollbar-color: #888 #f1f1f1; /* thumb track */
}
```

---

## 23. What is BEM naming convention?

BEM (Block, Element, Modifier) is a CSS naming methodology for creating reusable and maintainable class names.

```css
/* Block — standalone component */
.card { }

/* Element — part of a block (double underscore) */
.card__title { }
.card__image { }
.card__body { }
.card__footer { }

/* Modifier — variation of block or element (double hyphen) */
.card--featured { }
.card--dark { }
.card__title--large { }
.card__button--disabled { }
```

```html
<div class="card card--featured">
  <img class="card__image" src="photo.jpg" alt="Photo">
  <div class="card__body">
    <h2 class="card__title card__title--large">Title</h2>
    <p class="card__text">Description</p>
  </div>
  <div class="card__footer">
    <button class="card__button card__button--disabled">Read More</button>
  </div>
</div>
```

Benefits:
- Clear relationship between HTML and CSS
- Avoids deep nesting and specificity issues
- Self-documenting class names
- Easy to identify component boundaries

---

## 24. What are CSS preprocessors?

CSS preprocessors extend CSS with programming features. They compile down to standard CSS.

**SCSS (Sass):**

```scss
// Variables
$primary: #3b82f6;
$spacing: 16px;

// Nesting
.card {
  padding: $spacing;

  &__title {
    font-size: 1.5rem;
    color: $primary;
  }

  &--featured {
    border: 2px solid $primary;
  }

  &:hover {
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }
}

// Mixins
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.hero { @include flex-center; }

// Mixin with parameters
@mixin breakpoint($size) {
  @if $size == md { @media (min-width: 768px) { @content; } }
  @if $size == lg { @media (min-width: 1024px) { @content; } }
}

.grid {
  grid-template-columns: 1fr;
  @include breakpoint(md) {
    grid-template-columns: repeat(2, 1fr);
  }
}

// Functions
@function rem($px) {
  @return calc($px / 16) * 1rem;
}

.text { font-size: rem(18); }

// Extend
%visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  clip: rect(0, 0, 0, 0);
  overflow: hidden;
}

.sr-only { @extend %visually-hidden; }

// Partials and imports
@use 'variables';
@use 'mixins';
```

With CSS custom properties and modern CSS features, preprocessors are less essential but still widely used.

---

## 25. What is the difference between `em`, `rem`, `%`, `vw`, `vh`, and `px`?

| Unit | Relative to | Use case |
|---|---|---|
| `px` | Absolute (1/96 of an inch) | Borders, shadows, precise values |
| `em` | Parent element's font size | Component-scoped spacing |
| `rem` | Root element's (`<html>`) font size | Typography, global spacing |
| `%` | Parent element's dimension | Fluid widths |
| `vw` | 1% of viewport width | Full-width sections, fluid typography |
| `vh` | 1% of viewport height | Full-height sections |
| `dvh` | 1% of dynamic viewport height | Mobile-safe full height |
| `svh` | 1% of small viewport height | Mobile (with address bar) |
| `lvh` | 1% of large viewport height | Mobile (without address bar) |
| `ch` | Width of the "0" character | Max-width for text readability |
| `lh` | Element's line-height | Vertical spacing relative to text |

```css
html { font-size: 16px; } /* 1rem = 16px */

.parent { font-size: 20px; }
.child {
  font-size: 1.5em;  /* 30px (1.5 * 20px parent) */
  padding: 1rem;     /* 16px (1 * 16px root) */
  width: 50%;        /* 50% of parent width */
  max-width: 65ch;   /* ~65 characters wide */
}

/* Mobile-safe full height */
.hero {
  min-height: 100dvh; /* accounts for mobile browser chrome */
}
```

---

## 26. What is the `float` property?

`float` removes an element from normal flow and shifts it to the left or right, allowing inline content to wrap around it.

```css
.image-left {
  float: left;
  margin-right: 16px;
  margin-bottom: 8px;
}

.image-right {
  float: right;
  margin-left: 16px;
}
```

**Clearing floats:**

```css
/* Clear on sibling */
.clear { clear: both; } /* left | right | both */

/* Clearfix (on parent) */
.clearfix::after {
  content: '';
  display: table;
  clear: both;
}

/* Modern alternative — use overflow */
.parent { overflow: auto; }

/* Best alternative — use display: flow-root */
.parent { display: flow-root; }
```

Floats are largely replaced by Flexbox and Grid for layout purposes. They are still useful for wrapping text around images.

---

## 27. What are CSS logical properties?

Logical properties adapt to writing direction (LTR/RTL) and writing mode (horizontal/vertical), making internationalization easier.

```css
/* Physical properties → Logical equivalents */
margin-top     → margin-block-start
margin-bottom  → margin-block-end
margin-left    → margin-inline-start
margin-right   → margin-inline-end

padding-top    → padding-block-start
padding-left   → padding-inline-start

width          → inline-size
height         → block-size
min-width      → min-inline-size
max-height     → max-block-size

top            → inset-block-start
right          → inset-inline-end
bottom         → inset-block-end
left           → inset-inline-start

border-top     → border-block-start
text-align: left → text-align: start
text-align: right → text-align: end

/* Shorthand */
.element {
  margin-block: 16px;         /* top + bottom */
  margin-inline: auto;        /* left + right */
  padding-block: 8px 16px;    /* block-start block-end */
  padding-inline: 24px;
  inset: 0;                   /* all four sides */
  border-inline: 1px solid;
}
```

---

## 28. What is the `calc()` function?

`calc()` performs mathematical calculations for CSS property values, allowing mixing of units.

```css
.sidebar {
  width: calc(100% - 250px);
}

.element {
  /* Mix units */
  padding: calc(1rem + 5px);
  margin: calc(100vh - 80px);
  font-size: calc(14px + 0.5vw);

  /* Arithmetic */
  width: calc(100% / 3);
  height: calc(50vh - 2 * 32px);

  /* With CSS variables */
  --header-height: 60px;
  min-height: calc(100vh - var(--header-height));

  /* Nested calc */
  width: calc(calc(100% - 40px) / 3);
  /* Same as: */
  width: calc((100% - 40px) / 3);
}

/* Grid with calc */
.grid-item {
  width: calc((100% - 2 * 16px) / 3); /* 3 columns with 16px gaps */
}
```

---

## 29. What are CSS filters?

Filters apply graphical effects like blur, brightness, and contrast to elements.

```css
.element {
  filter: blur(5px);                 /* gaussian blur */
  filter: brightness(1.5);           /* 0 = black, 1 = normal, 2 = bright */
  filter: contrast(1.5);             /* 0 = gray, 1 = normal */
  filter: grayscale(100%);           /* 0% = color, 100% = grayscale */
  filter: sepia(100%);               /* sepia tone */
  filter: saturate(2);               /* 0 = desaturated, 1 = normal */
  filter: hue-rotate(90deg);         /* rotate hue */
  filter: invert(100%);              /* invert colors */
  filter: opacity(50%);              /* same as opacity property */
  filter: drop-shadow(4px 4px 8px rgba(0,0,0,0.3)); /* follows shape, not box */

  /* Multiple filters */
  filter: brightness(1.2) contrast(1.1) saturate(1.3);
}

/* Hover effect */
.image:hover {
  filter: brightness(1.1) saturate(1.2);
  transition: filter 0.3s ease;
}

/* Backdrop filter (blurs content behind element) */
.glass {
  backdrop-filter: blur(10px) saturate(180%);
  background: rgba(255, 255, 255, 0.2);
}
```

`drop-shadow` vs `box-shadow`: `drop-shadow` follows the shape of the element (respects transparency), while `box-shadow` is always rectangular.

---

## 30. What is `clip-path`?

`clip-path` clips an element to a specific shape.

```css
/* Basic shapes */
.circle {
  clip-path: circle(50%);
}

.ellipse {
  clip-path: ellipse(50% 30% at 50% 50%);
}

.triangle {
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
}

.diamond {
  clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%);
}

.hexagon {
  clip-path: polygon(25% 0%, 75% 0%, 100% 50%, 75% 100%, 25% 100%, 0% 50%);
}

/* Inset (rounded rectangle) */
.rounded {
  clip-path: inset(10px 20px 30px 40px round 15px);
}

/* Animate clip-path */
.reveal {
  clip-path: circle(0% at 50% 50%);
  transition: clip-path 0.5s ease;
}
.reveal:hover {
  clip-path: circle(100% at 50% 50%);
}
```

---

## 31. What is the `aspect-ratio` property?

`aspect-ratio` sets a preferred aspect ratio for an element.

```css
/* Fixed aspect ratios */
.video {
  aspect-ratio: 16 / 9;
  width: 100%;
}

.square {
  aspect-ratio: 1;   /* same as 1 / 1 */
  width: 200px;
}

.portrait {
  aspect-ratio: 3 / 4;
}

/* Responsive images */
img {
  aspect-ratio: 4 / 3;
  width: 100%;
  object-fit: cover;       /* cover | contain | fill | none */
  object-position: center; /* position within the element */
}

/* Before aspect-ratio existed (padding trick) */
.old-ratio {
  position: relative;
  padding-top: 56.25%; /* 9/16 = 56.25% for 16:9 */
}
.old-ratio > * {
  position: absolute;
  inset: 0;
}
```

---

## 32. What is the `object-fit` property?

`object-fit` defines how replaced elements (images, videos) fit within their container.

```css
img {
  width: 300px;
  height: 200px;
}

/* Values */
object-fit: fill;      /* default — stretches to fill (may distort) */
object-fit: contain;   /* scales to fit, maintains aspect ratio (may letterbox) */
object-fit: cover;     /* scales to cover, maintains aspect ratio (may crop) */
object-fit: none;      /* no resizing (uses original size) */
object-fit: scale-down; /* like contain or none, whichever is smaller */

/* Position the object within its box */
object-position: top;
object-position: center;
object-position: 20% 80%;
```

---

## 33. What is the difference between `visibility: hidden` and `display: none`?

| Feature | `display: none` | `visibility: hidden` |
|---|---|---|
| Space in layout | Removed | Still occupies space |
| Accessible | Not to screen readers | Not to screen readers |
| Transitions | Cannot animate | Can animate |
| Child override | Children are all hidden | Children can set `visibility: visible` |
| Events | No events fired | No events fired |

```css
/* Accessible hiding (visually hidden but readable by screen readers) */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

---

## 34. What are CSS counters?

CSS counters are variables controlled by CSS rules that track how many times they are used.

```css
/* Automatic numbering */
.toc {
  counter-reset: section;
}

.toc h2::before {
  counter-increment: section;
  content: counter(section) '. ';
}

/* Nested counters */
.outline {
  counter-reset: chapter;
}

.outline h2 {
  counter-increment: chapter;
  counter-reset: section;
}

.outline h2::before {
  content: 'Chapter ' counter(chapter) ': ';
}

.outline h3 {
  counter-increment: section;
}

.outline h3::before {
  content: counter(chapter) '.' counter(section) ' ';
}

/* Custom list styles */
ol {
  counter-reset: list;
  list-style: none;
}

ol li {
  counter-increment: list;
}

ol li::before {
  content: counter(list, upper-roman) ') ';
  font-weight: bold;
}
```

---

## 35. What are CSS Grid subgrid?

`subgrid` allows a grid item's children to participate in the parent grid's track sizing.

```css
.parent-grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  gap: 16px;
}

.child {
  grid-column: 1 / -1;        /* spans all columns */
  display: grid;
  grid-template-columns: subgrid; /* inherits parent column tracks */
}

/* Practical use: consistent card layouts */
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

.card {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 3;           /* card spans 3 row tracks */
}
/* All card titles, content, and footers align across cards */
```

Without subgrid, nested grid children cannot align to the parent's grid tracks.

---

## 36. What is the `will-change` property?

`will-change` hints to the browser about which properties will change, allowing it to set up optimizations ahead of time.

```css
/* Hint that transform and opacity will change */
.animated {
  will-change: transform, opacity;
}

/* Apply on hover intent, not permanently */
.card:hover {
  will-change: transform;
}

.card:active {
  transform: scale(0.95);
}
```

**Rules:**
- Do NOT apply `will-change` to too many elements (wastes memory)
- Do NOT apply it permanently — add before animation, remove after
- It creates a new stacking context and containing block
- Use sparingly for elements that actually have performance issues

---

## 37. What are CSS blend modes?

Blend modes determine how an element's colors blend with the content behind it.

```css
/* mix-blend-mode — element blends with what's behind it */
.overlay-text {
  mix-blend-mode: multiply;    /* darkens */
  mix-blend-mode: screen;      /* lightens */
  mix-blend-mode: overlay;     /* contrast */
  mix-blend-mode: difference;  /* invert colors */
  mix-blend-mode: color-dodge; /* bright effect */
}

/* background-blend-mode — blend multiple backgrounds */
.hero {
  background-image: url('photo.jpg'), linear-gradient(#3b82f6, #8b5cf6);
  background-blend-mode: overlay;
}

/* Common blend modes */
multiply     /* dark: keeps dark areas, removes white */
screen       /* light: keeps light areas, removes black */
overlay      /* contrast: multiply dark, screen light */
darken       /* keeps the darker pixel */
lighten      /* keeps the lighter pixel */
color-dodge  /* bright highlights */
color-burn   /* deep shadows */
difference   /* inverts based on brightness */
exclusion    /* softer difference */
hue          /* applies hue from top layer */
saturation   /* applies saturation from top layer */
color        /* applies hue + saturation */
luminosity   /* applies luminosity */
```

---

## 38. What are CSS scroll-snap properties?

Scroll snap forces scrollable containers to snap to specific positions.

```css
/* Container */
.carousel {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;     /* x | y | both, mandatory | proximity */
  scroll-behavior: smooth;
  -webkit-overflow-scrolling: touch;
}

/* Items */
.carousel-item {
  flex: 0 0 100%;
  scroll-snap-align: start;          /* start | center | end */
  scroll-snap-stop: always;          /* normal | always (prevents skipping) */
}

/* Vertical scroll snap */
.sections {
  height: 100vh;
  overflow-y: auto;
  scroll-snap-type: y mandatory;
}

.section {
  height: 100vh;
  scroll-snap-align: start;
}

/* Scroll padding (offset snap position) */
.container {
  scroll-snap-type: y mandatory;
  scroll-padding-top: 80px;          /* accounts for fixed header */
}

/* Horizontal gallery with proximity snap */
.gallery {
  scroll-snap-type: x proximity;     /* snaps when close enough */
}
```

---

## 39. What is the `@supports` rule?

`@supports` (feature query) applies styles only if the browser supports a given CSS feature.

```css
/* Check support */
@supports (display: grid) {
  .layout { display: grid; }
}

/* Negation */
@supports not (display: grid) {
  .layout { display: flex; }
}

/* Multiple conditions */
@supports (display: grid) and (gap: 16px) {
  .grid { display: grid; gap: 16px; }
}

@supports (display: grid) or (display: flex) {
  .layout { /* ... */ }
}

/* Check selector support */
@supports selector(:has(*)) {
  .parent:has(.child) { color: red; }
}

/* Progressive enhancement pattern */
.card-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

@supports (display: grid) {
  .card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  }
}
```

---

## 40. What are the new CSS color functions?

```css
/* Modern color formats */

/* oklch — perceptually uniform, great for design systems */
color: oklch(70% 0.15 240);          /* lightness chroma hue */
color: oklch(70% 0.15 240 / 50%);    /* with alpha */

/* oklab — perceptually uniform lab color */
color: oklab(70% -0.05 -0.1);

/* hsl — hue, saturation, lightness */
color: hsl(220 90% 50%);             /* modern syntax (no commas) */
color: hsl(220 90% 50% / 80%);       /* with alpha */

/* hwb — hue, whiteness, blackness */
color: hwb(220 10% 20%);

/* color-mix — mix two colors */
color: color-mix(in oklch, #3b82f6, white 30%);
background: color-mix(in srgb, var(--primary), transparent 80%);

/* Relative color syntax — modify existing colors */
--primary: #3b82f6;
--primary-light: oklch(from var(--primary) calc(l + 0.2) c h);
--primary-dark: oklch(from var(--primary) calc(l - 0.2) c h);
--primary-50: oklch(from var(--primary) l c h / 50%);

/* light-dark() — automatic light/dark values */
color-scheme: light dark;
background: light-dark(#ffffff, #1a1a1a);
color: light-dark(#1a1a1a, #ffffff);
```

---

## 41. What is the `@layer` rule?

`@layer` (cascade layers) gives control over the cascade order, regardless of specificity or source order.

```css
/* Define layer order (first = lowest priority) */
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; box-sizing: border-box; }
}

@layer base {
  body { font-family: system-ui; line-height: 1.5; }
  a { color: blue; }
}

@layer components {
  .button { padding: 8px 16px; border-radius: 4px; }
  .card { padding: 24px; border: 1px solid #e5e7eb; }
}

@layer utilities {
  .text-center { text-align: center; }
  .mt-4 { margin-top: 16px; }
}

/* Unlayered styles have highest priority */
.override { color: red; } /* wins over any layer */

/* Import with layer */
@import url('reset.css') layer(reset);
@import url('library.css') layer(lib);

/* Nested layers */
@layer components {
  @layer buttons {
    .btn { /* ... */ }
  }
  @layer cards {
    .card { /* ... */ }
  }
}
```

Layers solve specificity battles in large codebases and CSS frameworks.

---

## 42. What is `accent-color`?

`accent-color` controls the color of form elements like checkboxes, radio buttons, range sliders, and progress bars.

```css
/* Global accent color */
:root {
  accent-color: #3b82f6;
}

/* Per-element */
input[type="checkbox"] { accent-color: green; }
input[type="radio"] { accent-color: purple; }
input[type="range"] { accent-color: orange; }
progress { accent-color: blue; }
```

Before `accent-color`, styling these elements required custom implementations or hiding the native element.

---

## 43. What is the `content-visibility` property?

`content-visibility` controls whether an element's content is rendered, enabling rendering performance optimization.

```css
/* auto — skip rendering off-screen content */
.card {
  content-visibility: auto;
  contain-intrinsic-size: auto 200px; /* estimated height to prevent layout shifts */
}

/* hidden — like display:none but preserves state */
.tab-panel {
  content-visibility: hidden;
}
.tab-panel.active {
  content-visibility: visible;
}
```

`content-visibility: auto` can significantly improve initial rendering performance for long pages with many off-screen elements.

---

## 44. What is the `color-scheme` property?

`color-scheme` tells the browser which color schemes the element supports, affecting form controls, scrollbars, and system colors.

```css
/* Support both light and dark */
:root {
  color-scheme: light dark;
}

/* Force dark mode for specific element */
.dark-section {
  color-scheme: dark;
}

/* Meta tag equivalent */
/* <meta name="color-scheme" content="light dark"> */
```

When `color-scheme: dark` is set, the browser automatically adjusts:
- Form controls (inputs, selects, buttons)
- Scrollbars
- System colors (`Canvas`, `CanvasText`, etc.)
- Default background and text colors

---

## 45. What are CSS nesting?

Native CSS nesting (supported in modern browsers) allows nesting selectors similar to Sass.

```css
/* Native CSS nesting */
.card {
  padding: 24px;
  border: 1px solid #e5e7eb;

  & .title {
    font-size: 1.5rem;
    font-weight: bold;
  }

  & .body {
    margin-top: 12px;
  }

  /* & is optional for pseudo-classes and pseudo-elements */
  &:hover {
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  &::after {
    content: '';
    display: block;
  }

  /* Media queries nest too */
  @media (min-width: 768px) {
    padding: 32px;
  }

  /* Compound selectors */
  &.featured {
    border-color: gold;
  }

  /* Nesting combinators */
  & > p {
    margin: 0;
  }

  & + & {
    margin-top: 16px;
  }
}
```

---

## 46. What are common CSS layout patterns?

**Centering:**

```css
/* Flexbox center */
.center-flex {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* Grid center */
.center-grid {
  display: grid;
  place-items: center;
}

/* Margin auto (block element with known width) */
.center-margin {
  width: 600px;
  margin-inline: auto;
}
```

**Holy grail layout:**

```css
.layout {
  display: grid;
  grid-template: auto 1fr auto / 200px 1fr 200px;
  min-height: 100vh;
}
header { grid-column: 1 / -1; }
footer { grid-column: 1 / -1; }
```

**Sidebar layout:**

```css
.with-sidebar {
  display: grid;
  grid-template-columns: minmax(200px, 25%) 1fr;
  gap: 24px;
}
```

**Pancake stack:**

```css
.stack {
  display: grid;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}
```

**Masonry-like (columns):**

```css
.masonry {
  columns: 3;
  column-gap: 16px;
}
.masonry > * {
  break-inside: avoid;
  margin-bottom: 16px;
}
```

---

## 47. What is the `gap` property?

`gap` sets spacing between items in flex, grid, and multi-column layouts.

```css
/* Grid gap */
.grid {
  display: grid;
  gap: 16px;            /* row and column */
  row-gap: 16px;
  column-gap: 24px;
  gap: 16px 24px;       /* row column */
}

/* Flex gap */
.flex {
  display: flex;
  gap: 12px;
}

/* Multi-column gap */
.columns {
  columns: 3;
  column-gap: 24px;
}
```

`gap` is preferred over margin-based spacing because:
- No double margins between items
- No negative margins needed
- No "last-child" margin removal
- Works consistently in both flex and grid

---

## 48. What is the `isolation` property?

`isolation` creates a new stacking context without needing `z-index` or other side effects.

```css
.component {
  isolation: isolate;
}
```

Use case: preventing `z-index` from leaking out of a component and interfering with other parts of the page.

```css
/* Without isolation, z-index can bleed across components */
.modal {
  isolation: isolate; /* contains all z-index values within */
}

.modal-backdrop { z-index: 1; }
.modal-content  { z-index: 2; }
/* These z-indexes only compete within .modal's stacking context */
```

---

## 49. What are common CSS performance tips?

1. **Avoid expensive selectors** — reduce complex selectors like `div > ul > li > a.active`
2. **Use `will-change` sparingly** — only for known animated properties
3. **Prefer `transform` and `opacity` for animations** — they run on the GPU compositor
4. **Avoid layout thrashing** — animating `width`, `height`, `top`, `left` triggers layout recalculation
5. **Use `content-visibility: auto`** — skip rendering off-screen elements
6. **Minimize reflows** — batch DOM changes, avoid reading layout properties mid-update
7. **Use `contain` property** — isolate elements from the rest of the page

```css
/* Contain — limits browser recalculation scope */
.widget {
  contain: layout style paint; /* or contain: strict; contain: content; */
}

/* Prefer compositable properties */
/* Bad — triggers layout */
.animate { left: 0; transition: left 0.3s; }
.animate:hover { left: 100px; }

/* Good — runs on compositor */
.animate { transform: translateX(0); transition: transform 0.3s; }
.animate:hover { transform: translateX(100px); }
```

8. **Reduce CSS file size** — remove unused CSS, use minification
9. **Critical CSS** — inline above-the-fold CSS for faster first paint
10. **Use `font-display: swap`** — prevent invisible text while fonts load

```css
@font-face {
  font-family: 'CustomFont';
  src: url('font.woff2') format('woff2');
  font-display: swap;
}
```

---

## 50. What are common CSS anti-patterns?

1. **Using `!important`** — breaks the cascade, makes overriding difficult
2. **Over-nesting selectors** — `.page .content .sidebar .nav .list .item a` has unnecessary specificity
3. **Using IDs for styling** — high specificity, not reusable
4. **Magic numbers** — hardcoded values like `margin-top: 37px` without explanation
5. **Not using a reset/normalize** — inconsistent cross-browser defaults
6. **Using `px` for font sizes** — prevents user font-size preferences
7. **Animating layout properties** — `width`, `height`, `top`, `left` cause layout recalculation
8. **Not using CSS custom properties** — repeated values instead of variables
9. **Desktop-first approach** — harder to maintain than mobile-first
10. **Not considering accessibility** — missing focus styles, insufficient contrast, disabling zoom
