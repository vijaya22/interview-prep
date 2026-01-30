# HTML Interview Questions & Answers

---

## 1. What is HTML?

HTML (HyperText Markup Language) is the standard markup language for creating web pages. It defines the structure and content of a web page using elements represented by tags.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page Title</title>
</head>
<body>
  <h1>Hello, World!</h1>
</body>
</html>
```

---

## 2. What is the `<!DOCTYPE html>` declaration?

`<!DOCTYPE html>` tells the browser to render the page in standards mode (HTML5). Without it, the browser may use quirks mode, which emulates older browser behavior and can cause rendering inconsistencies.

- It must be the very first thing in the document (before `<html>`)
- It is not an HTML tag — it is an instruction to the browser
- It is case-insensitive (`<!doctype html>` is also valid)

---

## 3. What are semantic HTML elements?

Semantic elements clearly describe their meaning to both the browser and the developer.

**Semantic elements:**

```html
<header>    <!-- Page or section header -->
<nav>       <!-- Navigation links -->
<main>      <!-- Main content (one per page) -->
<article>   <!-- Self-contained content (blog post, news article) -->
<section>   <!-- Thematic grouping of content -->
<aside>     <!-- Sidebar, tangential content -->
<footer>    <!-- Page or section footer -->
<figure>    <!-- Self-contained content with caption -->
<figcaption><!-- Caption for <figure> -->
<time>      <!-- Date/time -->
<mark>      <!-- Highlighted/relevant text -->
<details>   <!-- Collapsible content -->
<summary>   <!-- Heading for <details> -->
<dialog>    <!-- Modal or dialog box -->
<address>   <!-- Contact information -->
```

**Non-semantic elements:** `<div>`, `<span>` — no inherent meaning.

**Why semantic HTML matters:**
- **Accessibility** — screen readers understand page structure
- **SEO** — search engines can better index content
- **Maintainability** — easier to read and understand code
- **Default behavior** — some elements have built-in functionality (e.g., `<details>`)

---

## 4. What is the difference between block-level and inline elements?

| Feature | Block-level | Inline |
|---|---|---|
| Width | Full width of parent | Only as wide as content |
| New line | Starts on a new line | Flows within text |
| Width/height | Respects `width`/`height` | Ignores `width`/`height` |
| Margin/padding | All directions work | Top/bottom margin ignored |
| Can contain | Block and inline elements | Only inline elements and text |

**Block-level elements:** `<div>`, `<p>`, `<h1>`-`<h6>`, `<ul>`, `<ol>`, `<li>`, `<section>`, `<article>`, `<header>`, `<footer>`, `<form>`, `<table>`, `<blockquote>`, `<pre>`

**Inline elements:** `<span>`, `<a>`, `<strong>`, `<em>`, `<img>`, `<input>`, `<button>`, `<label>`, `<code>`, `<br>`, `<abbr>`, `<cite>`

**Inline-block:** `<img>`, `<input>`, `<button>` — inline flow but respect width/height (replaced elements).

---

## 5. What are `<div>` and `<span>` used for?

- **`<div>`** — generic block-level container for grouping content and applying layout/styling
- **`<span>`** — generic inline container for wrapping text or inline elements for styling

```html
<!-- div for layout -->
<div class="card">
  <div class="card-body">
    <p>Some text with <span class="highlight">highlighted</span> word.</p>
  </div>
</div>
```

Use semantic elements (`<section>`, `<article>`, `<nav>`, etc.) whenever possible. Use `<div>` and `<span>` only when no semantic element fits.

---

## 6. What is the difference between `<head>` and `<body>`?

**`<head>`** — contains metadata about the document (not visible on the page):

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Page description for SEO">
  <meta name="robots" content="index, follow">
  <title>Page Title</title>
  <link rel="stylesheet" href="styles.css">
  <link rel="icon" href="favicon.ico">
  <link rel="canonical" href="https://example.com/page">
  <script src="app.js" defer></script>
  <style>/* inline styles */</style>
</head>
```

**`<body>`** — contains all visible content (text, images, links, forms, etc.).

---

## 7. What are HTML attributes?

Attributes provide additional information about elements. They are written in the opening tag as name-value pairs.

```html
<!-- Global attributes (available on all elements) -->
<div
  id="unique-id"
  class="card primary"
  style="color: red;"
  title="Tooltip text"
  data-user-id="123"
  hidden
  tabindex="0"
  role="button"
  aria-label="Close dialog"
  lang="en"
  dir="ltr"
  contenteditable="true"
  draggable="true"
  spellcheck="true"
>
</div>

<!-- Boolean attributes (presence = true, absence = false) -->
<input disabled>              <!-- disabled is true -->
<input required>              <!-- required is true -->
<input readonly>
<details open>                <!-- open by default -->
<script defer>                <!-- deferred loading -->
<video autoplay muted loop>
```

---

## 8. What are `data-*` attributes?

`data-*` attributes store custom data on HTML elements, accessible via JavaScript.

```html
<button data-action="delete" data-item-id="42" data-confirm="true">
  Delete
</button>
```

```js
const button = document.querySelector('button');

// Access via dataset (camelCase conversion)
button.dataset.action;    // 'delete'
button.dataset.itemId;    // '42' (data-item-id → itemId)
button.dataset.confirm;   // 'true' (always a string)

// Access via getAttribute
button.getAttribute('data-action'); // 'delete'

// Set values
button.dataset.loading = 'true';

// CSS can target data attributes
// [data-action="delete"] { color: red; }
```

Use `data-*` for storing UI state, passing data to JavaScript, or CSS hooks. Avoid storing sensitive data (visible in source).

---

## 9. What are void (self-closing) elements?

Void elements cannot have children or closing tags. They are self-contained.

```html
<br>         <!-- Line break -->
<hr>         <!-- Horizontal rule -->
<img>        <!-- Image -->
<input>      <!-- Form input -->
<meta>       <!-- Metadata -->
<link>       <!-- External resource link -->
<area>       <!-- Image map area -->
<base>       <!-- Base URL -->
<col>        <!-- Table column properties -->
<embed>      <!-- External content (plugin) -->
<source>     <!-- Media source -->
<track>      <!-- Text track for media -->
<wbr>        <!-- Word break opportunity -->
```

The trailing slash (`<br />`) is optional in HTML5 but required in XHTML.

---

## 10. What is the difference between `<script>`, `<script defer>`, and `<script async>`?

| Attribute | HTML Parsing | Script Download | Script Execution | Order Guaranteed |
|---|---|---|---|---|
| `<script>` | Paused | During parse | Immediately after download | Yes |
| `<script defer>` | Not paused | During parse | After HTML is fully parsed | Yes |
| `<script async>` | Not paused | During parse | Immediately after download | No |

```html
<!-- Blocks parsing (avoid placing in <head> without defer/async) -->
<script src="app.js"></script>

<!-- Deferred — runs after DOM is ready, in order -->
<script src="vendor.js" defer></script>
<script src="app.js" defer></script>

<!-- Async — runs as soon as downloaded, order not guaranteed -->
<script src="analytics.js" async></script>
<script src="ads.js" async></script>

<!-- Module scripts are deferred by default -->
<script type="module" src="app.js"></script>
```

**Best practice:** Use `defer` for scripts that need the DOM. Use `async` for independent scripts (analytics, ads). Place scripts at the end of `<body>` if you cannot use `defer`/`async`.

---

## 11. What are HTML forms and form elements?

```html
<form action="/api/submit" method="POST" enctype="multipart/form-data" novalidate>

  <!-- Text inputs -->
  <input type="text" name="name" placeholder="Name" required>
  <input type="email" name="email" required>
  <input type="password" name="password" minlength="8">
  <input type="tel" name="phone" pattern="[0-9]{10}">
  <input type="url" name="website">
  <input type="number" name="age" min="0" max="120" step="1">
  <input type="search" name="query">

  <!-- Date/time inputs -->
  <input type="date" name="birthday">
  <input type="time" name="appointment">
  <input type="datetime-local" name="meeting">
  <input type="month" name="month">
  <input type="week" name="week">

  <!-- Other inputs -->
  <input type="file" name="avatar" accept="image/*">
  <input type="color" name="theme-color">
  <input type="range" name="volume" min="0" max="100">
  <input type="hidden" name="csrf-token" value="abc123">

  <!-- Checkbox and radio -->
  <input type="checkbox" name="agree" id="agree">
  <label for="agree">I agree</label>

  <input type="radio" name="plan" value="free" id="free">
  <label for="free">Free</label>
  <input type="radio" name="plan" value="pro" id="pro">
  <label for="pro">Pro</label>

  <!-- Textarea -->
  <textarea name="message" rows="4" cols="50"></textarea>

  <!-- Select -->
  <select name="country">
    <option value="">Select country</option>
    <optgroup label="North America">
      <option value="us">United States</option>
      <option value="ca">Canada</option>
    </optgroup>
  </select>

  <!-- Datalist (autocomplete) -->
  <input list="browsers" name="browser">
  <datalist id="browsers">
    <option value="Chrome">
    <option value="Firefox">
    <option value="Safari">
  </datalist>

  <!-- Output -->
  <output name="result">0</output>

  <!-- Buttons -->
  <button type="submit">Submit</button>
  <button type="reset">Reset</button>
  <button type="button">Custom Action</button>

</form>
```

---

## 12. What is form validation in HTML?

HTML5 provides built-in form validation without JavaScript.

```html
<!-- Required field -->
<input type="text" required>

<!-- Min/max length -->
<input type="text" minlength="3" maxlength="50">

<!-- Pattern (regex) -->
<input type="text" pattern="[A-Za-z]{3,}" title="At least 3 letters">

<!-- Number constraints -->
<input type="number" min="1" max="100" step="5">

<!-- Email validation (built-in) -->
<input type="email" required>

<!-- URL validation (built-in) -->
<input type="url">

<!-- Custom validity message -->
<input type="text" id="username" required>
<script>
  const input = document.getElementById('username');
  input.addEventListener('input', () => {
    if (input.value.includes(' ')) {
      input.setCustomValidity('No spaces allowed');
    } else {
      input.setCustomValidity('');
    }
  });
</script>
```

**CSS pseudo-classes for validation:**

```css
input:valid   { border-color: green; }
input:invalid { border-color: red; }
input:required { border-left: 3px solid blue; }
input:optional { border-left: 3px solid gray; }
input:in-range { background: #e8f5e9; }
input:out-of-range { background: #ffebee; }
input:placeholder-shown { /* placeholder is visible */ }
```

Use `novalidate` on the `<form>` to disable built-in validation when using custom JavaScript validation.

---

## 13. What is the `<label>` element and why is it important?

`<label>` associates a text label with a form control, improving usability and accessibility.

```html
<!-- Explicit association (using for/id) -->
<label for="email">Email:</label>
<input type="email" id="email" name="email">

<!-- Implicit association (wrapping) -->
<label>
  Email:
  <input type="email" name="email">
</label>
```

**Why labels are important:**
- Clicking the label focuses/activates the associated input
- Screen readers announce the label when the input is focused
- Increases the clickable area (especially important for checkboxes/radio buttons)
- Required for accessibility compliance (WCAG)

---

## 14. What is the difference between `<a>`, `<button>`, and `<input type="submit">`?

| Element | Purpose | Navigation | Form submission | Keyboard |
|---|---|---|---|---|
| `<a>` | Navigate to URL | Yes | No | Enter |
| `<button>` | Trigger action | No (unless JS) | Yes (if in form) | Enter, Space |
| `<input type="submit">` | Submit form | No | Yes | Enter, Space |

```html
<!-- Link — navigates to a URL -->
<a href="/about">About Us</a>
<a href="#section">Jump to section</a>
<a href="mailto:info@example.com">Email us</a>
<a href="tel:+1234567890">Call us</a>
<a href="/file.pdf" download>Download PDF</a>

<!-- Button — triggers an action -->
<button type="button" onclick="handleClick()">Click Me</button>
<button type="submit">Submit Form</button>
<button type="reset">Reset Form</button>

<!-- Input submit — only for form submission, cannot contain HTML -->
<input type="submit" value="Submit">
```

**Rule of thumb:** Use `<a>` for navigation, `<button>` for actions.

---

## 15. What are HTML tables?

Tables should be used for tabular data only, not for layout.

```html
<table>
  <caption>Monthly Sales Report</caption>

  <colgroup>
    <col style="width: 40%">
    <col style="width: 30%">
    <col style="width: 30%">
  </colgroup>

  <thead>
    <tr>
      <th scope="col">Product</th>
      <th scope="col">Sales</th>
      <th scope="col">Revenue</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>Widget A</td>
      <td>150</td>
      <td>$7,500</td>
    </tr>
    <tr>
      <td>Widget B</td>
      <td>230</td>
      <td>$11,500</td>
    </tr>
  </tbody>

  <tfoot>
    <tr>
      <td>Total</td>
      <td>380</td>
      <td>$19,000</td>
    </tr>
  </tfoot>
</table>
```

Key attributes:
- `<caption>` — table title (important for accessibility)
- `<thead>`, `<tbody>`, `<tfoot>` — semantic table sections
- `scope="col"` or `scope="row"` on `<th>` — associates header with column/row
- `colspan` / `rowspan` — merge cells

---

## 16. What are `<picture>`, `<source>`, and responsive images?

```html
<!-- Basic responsive image -->
<img
  src="photo.jpg"
  alt="Descriptive alt text"
  width="800"
  height="600"
  loading="lazy"
  decoding="async"
>

<!-- srcset — resolution switching -->
<img
  src="photo-400.jpg"
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 1000px) 800px, 1200px"
  alt="Photo"
>

<!-- picture — art direction (different crops for different viewports) -->
<picture>
  <source media="(min-width: 1024px)" srcset="hero-wide.jpg">
  <source media="(min-width: 768px)" srcset="hero-medium.jpg">
  <img src="hero-mobile.jpg" alt="Hero image">
</picture>

<!-- picture — format selection -->
<picture>
  <source type="image/avif" srcset="photo.avif">
  <source type="image/webp" srcset="photo.webp">
  <img src="photo.jpg" alt="Photo">
</picture>
```

- `srcset` with `w` descriptors lets the browser choose the best resolution
- `sizes` tells the browser how wide the image will be at each breakpoint
- `<picture>` gives you explicit control over which image to show
- `loading="lazy"` defers offscreen images until they are near the viewport

---

## 17. What are `<video>` and `<audio>` elements?

```html
<!-- Video -->
<video
  src="video.mp4"
  width="640"
  height="360"
  controls                    <!-- show playback controls -->
  autoplay                    <!-- start automatically -->
  muted                       <!-- start muted (required for autoplay in most browsers) -->
  loop                        <!-- loop playback -->
  playsinline                 <!-- play inline on iOS (not fullscreen) -->
  poster="thumbnail.jpg"      <!-- preview image -->
  preload="metadata"          <!-- none | metadata | auto -->
>
  <!-- Multiple sources for format fallback -->
  <source src="video.webm" type="video/webm">
  <source src="video.mp4" type="video/mp4">

  <!-- Subtitles/captions -->
  <track src="subs-en.vtt" kind="subtitles" srclang="en" label="English" default>
  <track src="subs-es.vtt" kind="subtitles" srclang="es" label="Spanish">

  <!-- Fallback for unsupported browsers -->
  <p>Your browser does not support video.</p>
</video>

<!-- Audio -->
<audio controls preload="metadata">
  <source src="song.ogg" type="audio/ogg">
  <source src="song.mp3" type="audio/mpeg">
  <p>Your browser does not support audio.</p>
</audio>
```

---

## 18. What is the `<iframe>` element?

`<iframe>` embeds another HTML page within the current page.

```html
<iframe
  src="https://example.com"
  width="600"
  height="400"
  title="Embedded page"
  loading="lazy"
  sandbox="allow-scripts allow-same-origin"
  allow="camera; microphone; fullscreen"
  referrerpolicy="no-referrer"
></iframe>
```

**`sandbox` attribute values:**
- `allow-scripts` — allow JavaScript
- `allow-same-origin` — treat as same origin
- `allow-forms` — allow form submission
- `allow-popups` — allow popups
- `allow-modals` — allow modal dialogs
- (empty `sandbox`) — maximum restrictions

**Security considerations:**
- Use `sandbox` to restrict iframe capabilities
- Use `Content-Security-Policy` headers to control what can be embedded
- `X-Frame-Options` header prevents your page from being embedded by others

---

## 19. What is accessibility (a11y) in HTML?

Accessibility ensures web content is usable by people with disabilities.

**Key practices:**

```html
<!-- 1. Alt text for images -->
<img src="chart.png" alt="Sales increased 25% in Q4 2025">
<img src="decorative-border.png" alt="">  <!-- empty alt for decorative images -->

<!-- 2. Proper heading hierarchy -->
<h1>Page Title</h1>
  <h2>Section</h2>
    <h3>Subsection</h3>
  <h2>Another Section</h2>
<!-- Don't skip levels (h1 → h3) -->

<!-- 3. Form labels -->
<label for="name">Name:</label>
<input type="text" id="name">

<!-- 4. ARIA attributes -->
<button aria-label="Close dialog" aria-pressed="false">×</button>
<div role="alert">Form submitted successfully</div>
<nav aria-label="Main navigation">...</nav>

<!-- 5. Keyboard navigation -->
<div tabindex="0" role="button" onkeydown="handleKey(event)">
  Custom interactive element
</div>

<!-- 6. Language -->
<html lang="en">
<p>The French word <span lang="fr">bonjour</span> means hello.</p>

<!-- 7. Skip navigation link -->
<a href="#main-content" class="sr-only focus:visible">Skip to main content</a>

<!-- 8. Landmarks -->
<header role="banner">...</header>
<nav role="navigation">...</nav>
<main role="main" id="main-content">...</main>
<footer role="contentinfo">...</footer>
```

---

## 20. What are ARIA attributes?

ARIA (Accessible Rich Internet Applications) attributes enhance accessibility for dynamic content and custom widgets.

```html
<!-- Roles — define what an element is -->
<div role="button">Click me</div>
<div role="alert">Error message</div>
<div role="dialog" aria-modal="true">Modal content</div>
<div role="tablist">
  <button role="tab" aria-selected="true">Tab 1</button>
  <button role="tab" aria-selected="false">Tab 2</button>
</div>
<div role="tabpanel">Tab content</div>

<!-- States — describe current condition -->
<button aria-expanded="false">Menu</button>
<button aria-pressed="true">Bold</button>
<input aria-invalid="true" aria-describedby="error-msg">
<span id="error-msg">Email is required</span>

<!-- Properties — describe relationships -->
<input aria-label="Search">                          <!-- accessible name -->
<input aria-labelledby="label-id">                   <!-- references label element -->
<input aria-describedby="help-text">                 <!-- additional description -->
<div aria-live="polite">Dynamic content updates</div> <!-- live region -->
<button aria-controls="dropdown-menu">Options</button>
<div aria-hidden="true">Hidden from screen readers</div>

<!-- Live regions — announce dynamic changes -->
<div aria-live="polite">  <!-- polite: announced after current speech -->
  3 items in cart
</div>
<div aria-live="assertive"> <!-- assertive: announced immediately -->
  Error: Connection lost
</div>
```

**First rule of ARIA:** Don't use ARIA if a native HTML element provides the behavior. `<button>` is better than `<div role="button">`.

---

## 21. What is the difference between `<strong>`/`<em>` and `<b>`/`<i>`?

| Element | Meaning | Default Style |
|---|---|---|
| `<strong>` | Strong importance (semantic) | Bold |
| `<b>` | Stylistically bold (no extra importance) | Bold |
| `<em>` | Emphasis (semantic, changes meaning) | Italic |
| `<i>` | Alternative voice/mood (technical terms, thoughts) | Italic |

```html
<!-- strong — this text is important -->
<p><strong>Warning:</strong> Do not delete this file.</p>

<!-- b — stylistically bold, no semantic importance -->
<p>The product <b>SuperWidget</b> is now available.</p>

<!-- em — changes the meaning with emphasis -->
<p>I <em>never</em> said that.</p>  <!-- emphasis on "never" -->

<!-- i — alternative voice, technical term, foreign word -->
<p>The <i>Mona Lisa</i> is in the Louvre.</p>
<p>The term <i lang="la">carpe diem</i> means "seize the day."</p>
```

---

## 22. What are `<meta>` tags?

Meta tags provide metadata about the HTML document.

```html
<!-- Character encoding -->
<meta charset="UTF-8">

<!-- Viewport (responsive design) -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- SEO -->
<meta name="description" content="A brief description of the page (150-160 chars)">
<meta name="keywords" content="html, css, javascript">
<meta name="author" content="John Doe">
<meta name="robots" content="index, follow">
<meta name="robots" content="noindex, nofollow">  <!-- prevent indexing -->

<!-- Open Graph (social media sharing) -->
<meta property="og:title" content="Page Title">
<meta property="og:description" content="Page description">
<meta property="og:image" content="https://example.com/image.jpg">
<meta property="og:url" content="https://example.com/page">
<meta property="og:type" content="website">

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Page Title">
<meta name="twitter:description" content="Page description">
<meta name="twitter:image" content="https://example.com/image.jpg">

<!-- Security -->
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">
<meta http-equiv="X-Content-Type-Options" content="nosniff">

<!-- Theme color (browser UI) -->
<meta name="theme-color" content="#3b82f6">

<!-- Refresh/redirect -->
<meta http-equiv="refresh" content="5;url=https://example.com">

<!-- Prevent phone number detection (iOS) -->
<meta name="format-detection" content="telephone=no">
```

---

## 23. What is the difference between `id` and `class`?

| Feature | `id` | `class` |
|---|---|---|
| Uniqueness | Must be unique per page | Can be reused on multiple elements |
| Count per element | One `id` | Multiple classes |
| CSS specificity | Higher (0, 1, 0, 0) | Lower (0, 0, 1, 0) |
| JavaScript | `getElementById` (fast) | `getElementsByClassName`, `querySelectorAll` |
| URL fragment | `#id` links to element | Not linkable |

```html
<!-- id — unique identifier -->
<section id="about">
  <h2>About Us</h2>
</section>
<a href="#about">Jump to About</a>

<!-- class — reusable styling -->
<p class="text-large text-bold text-primary">Styled paragraph</p>
<p class="text-large">Another paragraph</p>
```

Best practice: use `class` for styling, reserve `id` for JavaScript references, anchor links, and form label associations.

---

## 24. What is the `<template>` element?

`<template>` holds HTML that is not rendered on page load but can be instantiated via JavaScript.

```html
<template id="card-template">
  <div class="card">
    <h3 class="card-title"></h3>
    <p class="card-body"></p>
  </div>
</template>

<div id="container"></div>

<script>
  const template = document.getElementById('card-template');
  const container = document.getElementById('container');

  function addCard(title, body) {
    const clone = template.content.cloneNode(true);
    clone.querySelector('.card-title').textContent = title;
    clone.querySelector('.card-body').textContent = body;
    container.appendChild(clone);
  }

  addCard('Hello', 'This is a dynamically created card.');
</script>
```

Content inside `<template>` is:
- Not rendered
- Not part of the DOM
- Images don't load, scripts don't run
- Available for cloning via JavaScript

---

## 25. What is the `<dialog>` element?

`<dialog>` is a native HTML element for creating modal and non-modal dialogs.

```html
<dialog id="modal">
  <h2>Confirm Action</h2>
  <p>Are you sure you want to delete this item?</p>
  <form method="dialog">
    <button value="cancel">Cancel</button>
    <button value="confirm">Confirm</button>
  </form>
</dialog>

<button onclick="document.getElementById('modal').showModal()">
  Open Modal
</button>

<script>
  const dialog = document.getElementById('modal');

  // Show as modal (with backdrop, traps focus)
  dialog.showModal();

  // Show as non-modal
  dialog.show();

  // Close
  dialog.close();

  // Get return value
  dialog.addEventListener('close', () => {
    console.log(dialog.returnValue); // 'cancel' or 'confirm'
  });

  // Close on backdrop click
  dialog.addEventListener('click', (e) => {
    if (e.target === dialog) dialog.close();
  });
</script>

<style>
  /* Style the backdrop */
  dialog::backdrop {
    background: rgba(0, 0, 0, 0.5);
    backdrop-filter: blur(4px);
  }
</style>
```

Benefits over custom modals:
- Built-in focus trapping
- Closes with Escape key
- `::backdrop` pseudo-element
- `method="dialog"` for form handling
- Accessible by default

---

## 26. What is the `<details>` and `<summary>` element?

`<details>` creates a collapsible disclosure widget without JavaScript.

```html
<details>
  <summary>Click to expand</summary>
  <p>This content is hidden by default and revealed on click.</p>
</details>

<!-- Open by default -->
<details open>
  <summary>FAQ: How do I reset my password?</summary>
  <p>Go to Settings → Security → Reset Password.</p>
</details>

<!-- Accordion pattern (with JS to close others) -->
<details name="faq">
  <summary>Question 1</summary>
  <p>Answer 1</p>
</details>
<details name="faq">
  <summary>Question 2</summary>
  <p>Answer 2</p>
</details>

<style>
  details {
    border: 1px solid #e5e7eb;
    border-radius: 4px;
    padding: 8px 16px;
    margin-bottom: 8px;
  }

  summary {
    cursor: pointer;
    font-weight: bold;
  }

  /* Custom marker */
  summary::marker {
    content: '▶ ';
  }

  details[open] summary::marker {
    content: '▼ ';
  }
</style>
```

The `name` attribute (newer browsers) creates exclusive accordions — only one `<details>` with the same `name` can be open at a time.

---

## 27. What is the `<canvas>` element?

`<canvas>` provides a drawing surface for graphics via JavaScript (2D or WebGL 3D).

```html
<canvas id="myCanvas" width="400" height="300"></canvas>

<script>
  const canvas = document.getElementById('myCanvas');
  const ctx = canvas.getContext('2d');

  // Rectangle
  ctx.fillStyle = '#3b82f6';
  ctx.fillRect(10, 10, 150, 100);

  // Line
  ctx.beginPath();
  ctx.moveTo(200, 10);
  ctx.lineTo(350, 100);
  ctx.strokeStyle = '#ef4444';
  ctx.lineWidth = 3;
  ctx.stroke();

  // Circle
  ctx.beginPath();
  ctx.arc(100, 200, 50, 0, Math.PI * 2);
  ctx.fillStyle = '#22c55e';
  ctx.fill();

  // Text
  ctx.font = '20px Arial';
  ctx.fillStyle = '#000';
  ctx.fillText('Hello Canvas', 200, 200);
</script>
```

Use cases: games, data visualization, image manipulation, animations. For simple graphics, SVG is often a better choice since it is scalable and accessible.

---

## 28. What is SVG and how is it used in HTML?

SVG (Scalable Vector Graphics) is an XML-based format for vector graphics that scales without losing quality.

```html
<!-- Inline SVG -->
<svg width="100" height="100" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="40" fill="#3b82f6" />
  <rect x="10" y="10" width="30" height="30" fill="#ef4444" />
  <line x1="0" y1="0" x2="100" y2="100" stroke="#000" stroke-width="2" />
  <text x="50" y="55" text-anchor="middle" fill="white">Hello</text>
</svg>

<!-- External SVG as image -->
<img src="icon.svg" alt="Icon" width="24" height="24">

<!-- SVG as background -->
<!-- background-image: url('pattern.svg'); -->

<!-- SVG as object (allows scripting) -->
<object type="image/svg+xml" data="graphic.svg"></object>
```

**SVG vs Canvas:**

| Feature | SVG | Canvas |
|---|---|---|
| Type | Vector (DOM-based) | Raster (pixel-based) |
| Scalability | Scales perfectly | Gets pixelated |
| Interactivity | Each element is in the DOM (clickable) | Must track coordinates manually |
| Performance | Slower with many elements | Better for many objects |
| Accessibility | Accessible (text, ARIA) | Not inherently accessible |
| Use case | Icons, logos, charts, illustrations | Games, image editing, complex animations |

---

## 29. What is the `loading` attribute?

The `loading` attribute controls when a resource is loaded.

```html
<!-- Lazy loading — defers until near viewport -->
<img src="photo.jpg" alt="Photo" loading="lazy" width="800" height="600">
<iframe src="video.html" loading="lazy"></iframe>

<!-- Eager loading — loads immediately (default) -->
<img src="hero.jpg" alt="Hero" loading="eager">
```

Best practices:
- Use `loading="lazy"` for offscreen images (below the fold)
- Do NOT lazy-load above-the-fold images (LCP images)
- Always specify `width` and `height` to prevent layout shift
- Use `fetchpriority="high"` for critical images

```html
<!-- High priority for LCP image -->
<img src="hero.jpg" alt="Hero" fetchpriority="high" width="1200" height="600">

<!-- Low priority for non-critical images -->
<img src="decoration.jpg" alt="" fetchpriority="low" loading="lazy">
```

---

## 30. What is the `<link>` element and its uses?

```html
<!-- Stylesheet -->
<link rel="stylesheet" href="styles.css">

<!-- Preload critical resources -->
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="hero.jpg" as="image">
<link rel="preload" href="critical.css" as="style">

<!-- Prefetch (likely needed for future navigation) -->
<link rel="prefetch" href="/next-page.html">
<link rel="prefetch" href="data.json" as="fetch">

<!-- DNS prefetch -->
<link rel="dns-prefetch" href="https://api.example.com">

<!-- Preconnect (DNS + TCP + TLS) -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- Favicon -->
<link rel="icon" href="/favicon.ico" sizes="32x32">
<link rel="icon" href="/icon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">

<!-- Canonical URL (prevent duplicate content for SEO) -->
<link rel="canonical" href="https://example.com/page">

<!-- Alternate versions -->
<link rel="alternate" hreflang="es" href="https://example.com/es/page">
<link rel="alternate" type="application/rss+xml" href="/feed.xml">

<!-- Manifest (PWA) -->
<link rel="manifest" href="/manifest.json">
```

---

## 31. What is the difference between `<ol>`, `<ul>`, and `<dl>`?

```html
<!-- Unordered list (bullet points) -->
<ul>
  <li>Item A</li>
  <li>Item B</li>
  <li>Item C</li>
</ul>

<!-- Ordered list (numbered) -->
<ol start="5" reversed type="A">
  <li>Fifth item</li>
  <li>Sixth item</li>
</ol>
<!-- type: 1 (default), A, a, I, i -->

<!-- Description list (key-value pairs) -->
<dl>
  <dt>HTML</dt>
  <dd>HyperText Markup Language</dd>

  <dt>CSS</dt>
  <dd>Cascading Style Sheets</dd>

  <dt>JS</dt>
  <dd>JavaScript</dd>
  <dd>A programming language for the web</dd>  <!-- multiple descriptions allowed -->
</dl>

<!-- Nested lists -->
<ul>
  <li>
    Frontend
    <ul>
      <li>HTML</li>
      <li>CSS</li>
      <li>JavaScript</li>
    </ul>
  </li>
  <li>Backend</li>
</ul>
```

---

## 32. What is `contenteditable`?

`contenteditable` makes an element editable by the user directly in the browser.

```html
<div contenteditable="true">
  Click here and start typing to edit this content.
</div>

<!-- Specific values -->
<p contenteditable="true">Editable</p>
<p contenteditable="false">Not editable</p>
<p contenteditable="plaintext-only">Only plain text (no formatting)</p>

<script>
  const editable = document.querySelector('[contenteditable]');

  editable.addEventListener('input', (e) => {
    console.log('Content changed:', e.target.innerHTML);
  });

  // Get content
  const html = editable.innerHTML;
  const text = editable.textContent;
</script>
```

Use cases: rich text editors, inline editing. For most form data, prefer `<input>` or `<textarea>`.

---

## 33. What is the `<output>` element?

`<output>` represents the result of a calculation or user action.

```html
<form oninput="result.value = parseInt(a.value) + parseInt(b.value)">
  <input type="range" id="a" value="50" min="0" max="100"> +
  <input type="number" id="b" value="25">
  = <output name="result" for="a b">75</output>
</form>
```

The `for` attribute lists the IDs of elements that contributed to the output. Screen readers announce it as a live result.

---

## 34. What is the `draggable` attribute and Drag and Drop API?

```html
<!-- Make element draggable -->
<div draggable="true" id="item">Drag me</div>

<!-- Drop target -->
<div id="drop-zone">Drop here</div>

<script>
  const item = document.getElementById('item');
  const dropZone = document.getElementById('drop-zone');

  // Drag events on the dragged element
  item.addEventListener('dragstart', (e) => {
    e.dataTransfer.setData('text/plain', e.target.id);
    e.dataTransfer.effectAllowed = 'move';
  });

  item.addEventListener('dragend', (e) => {
    console.log('Drag ended');
  });

  // Drop events on the target
  dropZone.addEventListener('dragover', (e) => {
    e.preventDefault(); // required to allow drop
    e.dataTransfer.dropEffect = 'move';
  });

  dropZone.addEventListener('dragenter', (e) => {
    dropZone.classList.add('drag-over');
  });

  dropZone.addEventListener('dragleave', (e) => {
    dropZone.classList.remove('drag-over');
  });

  dropZone.addEventListener('drop', (e) => {
    e.preventDefault();
    const id = e.dataTransfer.getData('text/plain');
    dropZone.appendChild(document.getElementById(id));
  });
</script>
```

---

## 35. What are Web Components?

Web Components are a set of browser APIs for creating reusable, encapsulated custom elements.

```html
<!-- Using a custom element -->
<user-card name="Alice" role="Admin"></user-card>

<script>
  class UserCard extends HTMLElement {
    constructor() {
      super();

      // Attach Shadow DOM
      const shadow = this.attachShadow({ mode: 'open' });

      // Create template
      shadow.innerHTML = `
        <style>
          .card {
            border: 1px solid #e5e7eb;
            border-radius: 8px;
            padding: 16px;
            font-family: system-ui;
          }
          .name { font-weight: bold; font-size: 1.2em; }
          .role { color: #6b7280; }
        </style>
        <div class="card">
          <div class="name">${this.getAttribute('name')}</div>
          <div class="role">${this.getAttribute('role')}</div>
        </div>
      `;
    }

    // Observed attributes
    static get observedAttributes() {
      return ['name', 'role'];
    }

    // Lifecycle callbacks
    connectedCallback() { /* element added to DOM */ }
    disconnectedCallback() { /* element removed from DOM */ }
    attributeChangedCallback(name, oldVal, newVal) {
      /* attribute changed */
    }
  }

  customElements.define('user-card', UserCard);
</script>
```

**Three main technologies:**
1. **Custom Elements** — define new HTML elements
2. **Shadow DOM** — encapsulated DOM and styles
3. **HTML Templates** — `<template>` and `<slot>` for content projection

---

## 36. What is Shadow DOM?

Shadow DOM provides encapsulation — styles and DOM inside a shadow tree don't leak out, and external styles don't leak in.

```js
// Create shadow root
const shadow = element.attachShadow({ mode: 'open' });
// mode: 'open' — accessible via element.shadowRoot
// mode: 'closed' — not accessible from outside

shadow.innerHTML = `
  <style>
    /* These styles ONLY apply inside the shadow DOM */
    p { color: red; }
    :host { display: block; border: 1px solid #ccc; }
    :host(.dark) { background: #333; color: white; }
    ::slotted(span) { font-weight: bold; }
  </style>
  <p>Internal paragraph (red)</p>
  <slot></slot>  <!-- projects children from light DOM -->
`;
```

```html
<my-element class="dark">
  <span>This content is projected via slot</span>
</my-element>
```

---

## 37. What is the `<slot>` element?

`<slot>` is used in Shadow DOM to define placeholders where light DOM content is projected.

```html
<!-- Component definition -->
<template id="alert-template">
  <style>
    .alert { padding: 16px; border-radius: 4px; }
  </style>
  <div class="alert">
    <strong><slot name="title">Default Title</slot></strong>
    <p><slot>Default content</slot></p>
  </div>
</template>

<!-- Usage -->
<my-alert>
  <span slot="title">Warning</span>
  This is the alert body content.
</my-alert>
```

- Default `<slot>` catches all unslotted children
- Named slots (`<slot name="x">`) match elements with `slot="x"` attribute
- Slot content is fallback text shown when nothing is projected

---

## 38. What is the `popover` attribute?

`popover` (HTML spec, supported in modern browsers) creates native popover functionality.

```html
<!-- Basic popover -->
<button popovertarget="my-popover">Toggle Popover</button>

<div id="my-popover" popover>
  <p>This is popover content!</p>
</div>

<!-- Popover types -->
<div popover="auto">Auto-dismisses when clicking outside</div>
<div popover="manual">Must be explicitly closed</div>

<!-- Control actions -->
<button popovertarget="info" popovertargetaction="show">Show</button>
<button popovertarget="info" popovertargetaction="hide">Hide</button>
<button popovertarget="info" popovertargetaction="toggle">Toggle</button>

<script>
  const popover = document.getElementById('my-popover');
  popover.showPopover();
  popover.hidePopover();
  popover.togglePopover();
</script>

<style>
  [popover] {
    padding: 16px;
    border: 1px solid #e5e7eb;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  /* Style the backdrop */
  [popover]::backdrop {
    background: rgba(0, 0, 0, 0.2);
  }
</style>
```

Benefits over custom implementations: top-layer rendering, keyboard dismiss (Escape), light-dismiss (click outside), focus management.

---

## 39. What is the difference between localStorage, sessionStorage, and cookies?

| Feature | localStorage | sessionStorage | Cookies |
|---|---|---|---|
| Capacity | ~5-10 MB | ~5-10 MB | ~4 KB |
| Expiry | Never (persists) | Tab/window close | Set by expiry date |
| Scope | Same origin | Same origin + tab | Same origin (+ path) |
| Sent to server | No | No | Yes (with every request) |
| Access | JavaScript only | JavaScript only | JavaScript + HTTP headers |
| Storage type | Key-value (strings) | Key-value (strings) | Key-value (strings) |

```js
// localStorage
localStorage.setItem('theme', 'dark');
localStorage.getItem('theme');    // 'dark'
localStorage.removeItem('theme');
localStorage.clear();

// sessionStorage (same API)
sessionStorage.setItem('tab-id', '123');

// Cookies
document.cookie = 'name=Alice; max-age=86400; path=/; Secure; SameSite=Strict';
```

---

## 40. What are the Geolocation, Notifications, and other browser APIs?

```js
// Geolocation
navigator.geolocation.getCurrentPosition(
  (pos) => console.log(pos.coords.latitude, pos.coords.longitude),
  (err) => console.error(err),
  { enableHighAccuracy: true, timeout: 5000 }
);

// Notifications
const permission = await Notification.requestPermission();
if (permission === 'granted') {
  new Notification('Hello!', { body: 'You have a new message', icon: 'icon.png' });
}

// Clipboard
await navigator.clipboard.writeText('Copied text');
const text = await navigator.clipboard.readText();

// Fullscreen
element.requestFullscreen();
document.exitFullscreen();

// Page Visibility
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    // Tab is hidden — pause video, reduce updates
  }
});

// Intersection Observer (lazy loading, infinite scroll)
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      loadContent(entry.target);
    }
  });
}, { threshold: 0.1 });

observer.observe(element);

// Resize Observer
const resizeObserver = new ResizeObserver((entries) => {
  for (const entry of entries) {
    console.log('New size:', entry.contentRect.width);
  }
});
resizeObserver.observe(element);

// Share API (mobile)
navigator.share({ title: 'Page', text: 'Check this out', url: location.href });
```

---

## 41. What are HTML entities?

HTML entities are special characters represented by codes to avoid conflicts with HTML syntax.

```html
<!-- Named entities -->
&lt;       <!-- < (less than) -->
&gt;       <!-- > (greater than) -->
&amp;      <!-- & (ampersand) -->
&quot;     <!-- " (double quote) -->
&apos;     <!-- ' (single quote) -->
&nbsp;     <!-- non-breaking space -->
&copy;     <!-- © (copyright) -->
&reg;      <!-- ® (registered) -->
&trade;    <!-- ™ (trademark) -->
&mdash;    <!-- — (em dash) -->
&ndash;    <!-- – (en dash) -->
&hellip;   <!-- … (ellipsis) -->
&laquo;    <!-- « (left guillemet) -->
&raquo;    <!-- » (right guillemet) -->
&times;    <!-- × (multiplication) -->
&divide;   <!-- ÷ (division) -->

<!-- Numeric entities -->
&#169;     <!-- © -->
&#x00A9;   <!-- © (hex) -->

<!-- Usage example -->
<p>Price: 5 &lt; 10 &amp; 10 &gt; 5</p>
<!-- Renders: Price: 5 < 10 & 10 > 5 -->
```

---

## 42. What is the `<base>` element?

`<base>` sets the base URL for all relative URLs in the document.

```html
<head>
  <base href="https://example.com/blog/" target="_blank">
</head>

<body>
  <!-- Resolves to https://example.com/blog/post-1 -->
  <a href="post-1">Post 1</a>

  <!-- Resolves to https://example.com/blog/images/photo.jpg -->
  <img src="images/photo.jpg" alt="Photo">

  <!-- Absolute URLs are not affected -->
  <a href="https://other.com">Other Site</a>
</body>
```

Only one `<base>` element is allowed per document. Use with caution — it affects all relative URLs including anchors, forms, and scripts.

---

## 43. What are `<fieldset>` and `<legend>`?

`<fieldset>` groups related form controls, and `<legend>` provides a caption for the group.

```html
<form>
  <fieldset>
    <legend>Personal Information</legend>
    <label for="fname">First Name:</label>
    <input type="text" id="fname" name="fname">

    <label for="lname">Last Name:</label>
    <input type="text" id="lname" name="lname">
  </fieldset>

  <fieldset>
    <legend>Shipping Method</legend>
    <label>
      <input type="radio" name="shipping" value="standard"> Standard
    </label>
    <label>
      <input type="radio" name="shipping" value="express"> Express
    </label>
  </fieldset>

  <fieldset disabled>
    <legend>Premium Options (Upgrade Required)</legend>
    <label>
      <input type="checkbox" name="gift-wrap"> Gift Wrap
    </label>
  </fieldset>
</form>
```

The `disabled` attribute on `<fieldset>` disables all form controls within it.

---

## 44. What is the `<progress>` and `<meter>` element?

```html
<!-- Progress — task completion -->
<label for="file-upload">Upload Progress:</label>
<progress id="file-upload" value="65" max="100">65%</progress>

<!-- Indeterminate progress (no value) -->
<progress>Loading...</progress>

<!-- Meter — measurement within a known range -->
<label for="disk">Disk Usage:</label>
<meter
  id="disk"
  value="0.7"
  min="0"
  max="1"
  low="0.3"
  high="0.7"
  optimum="0.5"
>70%</meter>
```

**Difference:**
- `<progress>` — for task completion (loading, upload progress)
- `<meter>` — for scalar measurements (disk usage, battery level, scores)

---

## 45. What is the `<time>` element?

`<time>` represents a specific date, time, or duration in a machine-readable format.

```html
<!-- Date -->
<time datetime="2025-12-25">December 25, 2025</time>

<!-- Date and time -->
<time datetime="2025-12-25T14:30:00">Dec 25 at 2:30 PM</time>

<!-- With timezone -->
<time datetime="2025-12-25T14:30:00-05:00">2:30 PM EST</time>

<!-- Duration -->
<time datetime="PT2H30M">2 hours 30 minutes</time>

<!-- Relative display with machine-readable value -->
<p>Posted <time datetime="2025-01-15">3 days ago</time></p>

<!-- Year-month -->
<time datetime="2025-06">June 2025</time>

<!-- Week -->
<time datetime="2025-W05">Fifth week of 2025</time>
```

The `datetime` attribute provides the machine-readable format. The text content between tags is the human-readable display.

---

## 46. What is the `inputmode` attribute?

`inputmode` hints to the browser which virtual keyboard to show on mobile devices.

```html
<!-- Numeric keyboard (no decimals) -->
<input type="text" inputmode="numeric" pattern="[0-9]*">

<!-- Telephone keyboard -->
<input type="text" inputmode="tel">

<!-- Email keyboard (@ and . easily accessible) -->
<input type="text" inputmode="email">

<!-- URL keyboard (/ and . easily accessible) -->
<input type="text" inputmode="url">

<!-- Search keyboard (may show search/go button) -->
<input type="text" inputmode="search">

<!-- Decimal keyboard (numbers with decimal point) -->
<input type="text" inputmode="decimal">

<!-- No keyboard -->
<input type="text" inputmode="none">
```

`inputmode` is different from `type` — it only affects the keyboard shown, not validation. Use `type="email"` for email validation and `inputmode="email"` for keyboard hints.

---

## 47. What is the `autocomplete` attribute?

`autocomplete` helps browsers autofill form fields.

```html
<form autocomplete="on">
  <!-- Name -->
  <input type="text" autocomplete="name">
  <input type="text" autocomplete="given-name">
  <input type="text" autocomplete="family-name">

  <!-- Contact -->
  <input type="email" autocomplete="email">
  <input type="tel" autocomplete="tel">

  <!-- Address -->
  <input type="text" autocomplete="street-address">
  <input type="text" autocomplete="address-level2"> <!-- city -->
  <input type="text" autocomplete="address-level1"> <!-- state -->
  <input type="text" autocomplete="postal-code">
  <input type="text" autocomplete="country">

  <!-- Payment -->
  <input type="text" autocomplete="cc-name">
  <input type="text" autocomplete="cc-number">
  <input type="text" autocomplete="cc-exp">
  <input type="text" autocomplete="cc-csc">

  <!-- Login -->
  <input type="text" autocomplete="username">
  <input type="password" autocomplete="current-password">
  <input type="password" autocomplete="new-password">

  <!-- One-time code (SMS verification) -->
  <input type="text" autocomplete="one-time-code">

  <!-- Disable autocomplete -->
  <input type="text" autocomplete="off">
</form>
```

Proper `autocomplete` values improve user experience and accessibility.

---

## 48. What are common HTML performance optimizations?

1. **Minimize DOM size** — fewer elements = faster rendering and memory usage

2. **Resource hints:**
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="dns-prefetch" href="https://api.example.com">
<link rel="preload" href="critical.css" as="style">
<link rel="prefetch" href="next-page.html">
```

3. **Lazy loading:**
```html
<img loading="lazy" src="photo.jpg" width="400" height="300" alt="Photo">
<iframe loading="lazy" src="widget.html"></iframe>
```

4. **Image optimization:**
```html
<picture>
  <source type="image/avif" srcset="photo.avif">
  <source type="image/webp" srcset="photo.webp">
  <img src="photo.jpg" alt="Photo" width="800" height="600">
</picture>
```

5. **Script loading:**
```html
<script src="app.js" defer></script>
<script src="analytics.js" async></script>
```

6. **Prevent layout shift:**
```html
<!-- Always specify dimensions -->
<img src="photo.jpg" width="800" height="600" alt="Photo">
<video width="640" height="360"></video>
```

7. **Reduce render-blocking resources:**
```html
<!-- Critical CSS inline -->
<style>/* above-the-fold styles */</style>
<!-- Non-critical CSS loaded asynchronously -->
<link rel="preload" href="styles.css" as="style" onload="this.rel='stylesheet'">
```

---

## 49. What are common HTML SEO best practices?

1. **One `<h1>` per page** — clear page title
2. **Proper heading hierarchy** — `h1 > h2 > h3` (no skipping)
3. **Descriptive `<title>`** — unique, under 60 characters
4. **Meta description** — unique, 150-160 characters
5. **Alt text on images** — descriptive for content images, empty for decorative
6. **Semantic elements** — `<article>`, `<nav>`, `<main>`, etc.
7. **Canonical URLs** — prevent duplicate content
8. **Open Graph tags** — better social media sharing
9. **Structured data (JSON-LD):**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Article Title",
  "author": {
    "@type": "Person",
    "name": "John Doe"
  },
  "datePublished": "2025-01-15"
}
</script>
```

10. **Mobile-friendly viewport:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

---

## 50. What are common HTML anti-patterns?

1. **Using `<div>` for everything** — use semantic elements instead
2. **Missing `alt` attributes** — every `<img>` needs an `alt` (empty string for decorative)
3. **Using `<br>` for spacing** — use CSS margin/padding instead
4. **Tables for layout** — use CSS Grid or Flexbox
5. **Missing form labels** — every input needs an associated `<label>`
6. **Inline styles** — use classes and external stylesheets
7. **Skipping heading levels** — `h1` → `h3` (never skip `h2`)
8. **Not setting `lang` attribute** — `<html lang="en">` is important for screen readers
9. **Missing `viewport` meta tag** — breaks responsive design on mobile
10. **Using deprecated elements** — `<center>`, `<font>`, `<marquee>`, `<blink>`, `<frameset>`
