# HTML Tags Mastery for Test Automation Engineers

A comprehensive guide to understanding HTML structure, tag purposes, and their implications for Playwright test automation and UI development discussions.

---

## 📚 Table of Contents

1. [Document Structure](#1-html-fundamentals--document-structure)
2. [Text & Typography](#2-text--typography-tags)
3. [Semantic Layout (Modern)](#3-semantic-layout-tags-html5-modern)
4. [Legacy Layout Patterns](#4-legacy-layout-patterns)
5. [Form Elements](#5-form-elements--input-types)
6. [Tables & Data](#6-tables--data-display)
7. [Media & Embedded](#7-media--embedded-content)
8. [Interactive Elements](#8-interactive-elements)
9. [Essential Attributes](#9-essential-html-attributes)
10. [Playwright Locators](#10-playwright-locator-strategies)
11. [Influencing Design](#11-influencing-design--development-decisions)

---

## 1. HTML Fundamentals & Document Structure

Every web page is built on HTML. Understanding document structure helps you navigate the DOM efficiently during test automation.

```html
<!-- Document Type Declaration -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Page Title</title>
  </head>
  <body>
    <!-- All visible content -->
  </body>
</html>
```

### `<!DOCTYPE html>`
**Document Type Declaration**

**Purpose:** Tells browser to use HTML5 standards mode. Missing DOCTYPE causes quirks mode with inconsistent rendering.

---

### `<html>`
**Root Element**

**Purpose:** Container for all HTML content. The `lang` attribute specifies document language for screen readers.

**Example:**
```html
<html lang="en">
```

**🎭 Playwright Impact:**
```javascript
// No direct locator, but sets page language context
await page.locator('html').getAttribute('lang'); // "en"
```

---

### `<head>`
**Document Metadata Container**

**Purpose:** Contains metadata, links to CSS, scripts, and page title. Not visible on page but essential for SEO and functionality.

**Common Children:**
- `<meta>` - Metadata (charset, viewport, description)
- `<title>` - Browser tab title
- `<link>` - Stylesheets, favicons
- `<script>` - JavaScript
- `<style>` - Embedded CSS

---

### `<body>`
**Visible Content Container**

**Purpose:** Contains all visible page content. Everything users see and interact with goes here.

**🎭 Playwright Impact:**
```javascript
// Target the entire body if needed
await page.locator('body').screenshot();
```

---

### `<meta>`
**Metadata Tag**

**Purpose:** Provides metadata about the HTML document. Common uses: character encoding, viewport settings, SEO description.

**Examples:**
```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="description" content="Page description for SEO">
```

---

### `<title>`
**Page Title**

**Purpose:** Defines browser tab title and appears in search results. Critical for SEO and user navigation.

**Example:**
```html
<title>My Amazing App - Login</title>
```

**🎭 Playwright Impact:**
```javascript
await expect(page).toHaveTitle('My Amazing App - Login');
await expect(page).toHaveTitle(/Login/);
```

---

### `<link>`
**External Resource Link**

**Purpose:** Links external resources like CSS stylesheets, favicons, fonts.

**Examples:**
```html
<link rel="stylesheet" href="styles.css">
<link rel="icon" href="favicon.ico">
<link rel="preconnect" href="https://fonts.googleapis.com">
```

---

### `<script>`
**JavaScript Container**

**Purpose:** Embeds or links JavaScript code. Can be inline or external.

**Examples:**
```html
<script src="app.js"></script>
<script>
  console.log('Inline script');
</script>
<script src="analytics.js" async></script>
```

**Attributes:**
- `defer` - Execute after HTML parsing
- `async` - Execute asynchronously
- `type="module"` - ES6 module

---

## 2. Text & Typography Tags

### `<h1>` to `<h6>`
**Headings**

**Purpose:** Define document hierarchy and structure. `<h1>` is most important, `<h6>` is least important.

**Example:**
```html
<h1>Main Page Title</h1>
<h2>Section Title</h2>
<h3>Sub-section</h3>
```

**SEO Impact:** Search engines use headings to understand content structure. Use only ONE `<h1>` per page.

**🎭 Playwright Impact:**
```javascript
await page.getByRole('heading', { name: 'Main Page Title' });
await page.getByRole('heading', { level: 1 });
await page.getByRole('heading', { level: 2, name: 'Section Title' });
```

---

### `<p>`
**Paragraph**

**Purpose:** Defines a paragraph of text. Block-level element with default spacing.

**Example:**
```html
<p>This is a paragraph of text.</p>
```

**🎭 Playwright Impact:**
```javascript
await page.getByText('This is a paragraph of text');
await page.locator('p').filter({ hasText: 'paragraph' });
```

---

### `<span>`
**Inline Container**

**Purpose:** Generic inline container for styling or scripting. No semantic meaning.

**Example:**
```html
<p>Price: <span class="highlight">$49.99</span></p>
```

**🎭 Playwright Impact:**
```javascript
await page.locator('span.highlight').textContent(); // "$49.99"
```

---

### `<div>`
**Division/Container**

**Purpose:** Generic block-level container for grouping content. No semantic meaning but widely used for layout.

**Example:**
```html
<div class="card">
  <h3>Card Title</h3>
  <p>Card content</p>
</div>
```

**⚠️ Modern Alternative:** Use semantic elements like `<section>`, `<article>`, `<header>` when appropriate.

---

### `<strong>` vs `<b>`
**Strong Emphasis vs Bold**

**`<strong>`** - Semantic importance
```html
<p><strong>Warning:</strong> Do not proceed</p>
```

**`<b>`** - Visual boldness without semantic meaning
```html
<p><b>Product Name</b> - $49.99</p>
```

**Best Practice:** Use `<strong>` for important content, `<b>` for keywords without importance.

---

### `<em>` vs `<i>`
**Emphasis vs Italic**

**`<em>`** - Semantic emphasis (stress emphasis)
```html
<p>I <em>really</em> mean it!</p>
```

**`<i>`** - Visual italic without semantic meaning (technical terms, foreign words)
```html
<p>The term <i>carpe diem</i> means "seize the day"</p>
```

---

### `<br>`
**Line Break**

**Purpose:** Forces a line break within text. Self-closing tag.

**Example:**
```html
<p>First line<br>Second line</p>
```

**⚠️ Avoid:** Excessive use for spacing. Use CSS `margin` or `padding` instead.

---

### `<hr>`
**Horizontal Rule**

**Purpose:** Thematic break or section divider. Self-closing tag.

**Example:**
```html
<h2>Section 1</h2>
<p>Content here</p>
<hr>
<h2>Section 2</h2>
```

---

### `<blockquote>`
**Block Quotation**

**Purpose:** Represents a section quoted from another source.

**Example:**
```html
<blockquote cite="https://example.com">
  <p>"The only way to do great work is to love what you do."</p>
  <footer>— Steve Jobs</footer>
</blockquote>
```

---

### `<code>`
**Inline Code**

**Purpose:** Displays inline code or programming text with monospace font.

**Example:**
```html
<p>Use the <code>console.log()</code> function for debugging.</p>
```

---

### `<pre>`
**Preformatted Text**

**Purpose:** Preserves whitespace and line breaks. Typically used with `<code>` for code blocks.

**Example:**
```html
<pre><code>
function hello() {
  console.log("Hello, World!");
}
</code></pre>
```

---

## 3. Semantic Layout Tags (HTML5 Modern)

### Why Semantic Tags Matter
- **Accessibility:** Screen readers understand page structure
- **SEO:** Search engines better understand content hierarchy
- **Maintainability:** Code is self-documenting
- **Testing:** Playwright can use role-based locators

---

### `<header>`
**Page/Section Header**

**Purpose:** Container for introductory content or navigation. Can be used multiple times in a page.

**Example:**
```html
<header>
  <h1>Company Logo</h1>
  <nav><!-- navigation --></nav>
</header>
```

**Implicit ARIA Role:** `banner` (when direct child of `<body>`)

**🎭 Playwright Impact:**
```javascript
await page.getByRole('banner');
await page.locator('header').getByRole('navigation');
```

---

### `<nav>`
**Navigation Section**

**Purpose:** Container for navigation links. Major navigation blocks.

**Example:**
```html
<nav>
  <a href="/">Home</a>
  <a href="/about">About</a>
  <a href="/contact">Contact</a>
</nav>
```

**Implicit ARIA Role:** `navigation`

**🎭 Playwright Impact:**
```javascript
await page.getByRole('navigation').getByRole('link', { name: 'Home' });
```

---

### `<main>`
**Main Content**

**Purpose:** Primary content of the document. Should be unique per page (only ONE per page).

**Example:**
```html
<main>
  <h1>Article Title</h1>
  <article><!-- main content --></article>
</main>
```

**Implicit ARIA Role:** `main`

**🎭 Playwright Impact:**
```javascript
await page.getByRole('main');
await page.locator('main').getByRole('heading', { level: 1 });
```

---

### `<section>`
**Thematic Section**

**Purpose:** Groups related content with a heading. Represents a standalone section of a document.

**Example:**
```html
<section>
  <h2>Features</h2>
  <p>Feature description...</p>
</section>
```

**🎭 Playwright Impact:**
```javascript
await page.locator('section').filter({ has: page.getByRole('heading', { name: 'Features' }) });
```

---

### `<article>`
**Self-Contained Content**

**Purpose:** Independent, self-contained content that could be distributed separately (blog post, news article, forum post).

**Example:**
```html
<article>
  <h2>Blog Post Title</h2>
  <p>Post content...</p>
  <footer>Published on <time>2024-01-15</time></footer>
</article>
```

**Implicit ARIA Role:** `article`

---

### `<aside>`
**Sidebar Content**

**Purpose:** Content tangentially related to main content. Often used for sidebars, callouts, related links.

**Example:**
```html
<aside>
  <h3>Related Articles</h3>
  <ul>
    <li><a href="/article1">Article 1</a></li>
  </ul>
</aside>
```

**Implicit ARIA Role:** `complementary`

---

### `<footer>`
**Page/Section Footer**

**Purpose:** Footer for page or section. Contains metadata, copyright, links.

**Example:**
```html
<footer>
  <p>&copy; 2024 Company Name</p>
  <nav><!-- footer navigation --></nav>
</footer>
```

**Implicit ARIA Role:** `contentinfo` (when direct child of `<body>`)

**🎭 Playwright Impact:**
```javascript
await page.getByRole('contentinfo');
```

---

## 4. Legacy Layout Patterns

### The "Div Soup" Era

Before HTML5 semantic elements, developers used `<div>` with classes:

```html
<!-- ❌ Old Way (Still common in legacy apps) -->
<div class="header">
  <div class="logo">Logo</div>
  <div class="navigation">
    <div class="nav-item"><a href="/">Home</a></div>
  </div>
</div>

<!-- ✅ Modern Way -->
<header>
  <div class="logo">Logo</div>
  <nav>
    <a href="/">Home</a>
  </nav>
</header>
```

### Why You'll See Legacy Patterns

1. **Old Applications:** Code written before HTML5 (pre-2014)
2. **Framework Generated:** Some frameworks generate div-heavy markup
3. **Resistance to Change:** "If it works, don't change it" mentality
4. **Learning Curve:** Developers unfamiliar with semantic HTML

### Testing Legacy HTML

**🎭 Playwright Strategies:**

```javascript
// Legacy apps often require class-based selectors
await page.locator('.header .navigation .nav-item a').click();

// Or use test IDs
await page.getByTestId('nav-home-link').click();

// Text content can be reliable
await page.getByText('Home', { exact: true }).click();
```

---

## 5. Form Elements & Input Types

Forms are critical for test automation. Understanding input types and attributes is essential.

### `<form>`
**Form Container**

**Purpose:** Groups form controls and defines how data is submitted.

**Example:**
```html
<form action="/submit" method="POST">
  <!-- form controls -->
</form>
```

**Key Attributes:**
- `action` - URL where form data is sent
- `method` - HTTP method (GET/POST)
- `enctype` - How form data is encoded

**🎭 Playwright Impact:**
```javascript
await page.locator('form').submit();
await page.locator('form[action="/submit"]').locator('button[type="submit"]').click();
```

---

### `<input>` Types

#### `<input type="text">`
**Single-line Text Input**

```html
<label for="username">Username:</label>
<input type="text" id="username" name="username">
```

**🎭 Playwright:**
```javascript
await page.getByLabel('Username').fill('john_doe');
await page.getByRole('textbox', { name: 'Username' }).fill('john_doe');
```

---

#### `<input type="email">`
**Email Input with Validation**

```html
<input type="email" id="email" required>
```

**Browser Behavior:** Validates email format on submit.

**🎭 Playwright:**
```javascript
await page.getByLabel('Email').fill('user@example.com');
```

---

#### `<input type="password">`
**Password Input (Hidden Text)**

```html
<input type="password" id="password" autocomplete="current-password">
```

**🎭 Playwright:**
```javascript
await page.getByLabel('Password').fill('SecurePass123!');
```

---

#### `<input type="checkbox">`
**Checkbox Toggle**

```html
<input type="checkbox" id="agree" name="agree">
<label for="agree">I agree to terms</label>
```

**🎭 Playwright:**
```javascript
await page.getByLabel('I agree to terms').check();
await page.getByLabel('I agree to terms').uncheck();
await page.getByRole('checkbox', { name: 'I agree to terms' }).check();
```

---

#### `<input type="radio">`
**Radio Button (Exclusive Selection)**

```html
<input type="radio" id="male" name="gender" value="male">
<label for="male">Male</label>
<input type="radio" id="female" name="gender" value="female">
<label for="female">Female</label>
```

**🎭 Playwright:**
```javascript
await page.getByLabel('Male').check();
await page.getByRole('radio', { name: 'Female' }).check();
```

---

#### `<input type="file">`
**File Upload**

```html
<input type="file" id="upload" accept=".pdf,.jpg">
```

**🎭 Playwright:**
```javascript
await page.getByLabel('Upload').setInputFiles('path/to/file.pdf');
await page.getByLabel('Upload').setInputFiles(['file1.pdf', 'file2.jpg']);
```

---

#### `<input type="date">`
**Date Picker**

```html
<input type="date" id="birthdate">
```

**🎭 Playwright:**
```javascript
await page.getByLabel('Birth Date').fill('2024-01-15');
```

---

#### `<input type="number">`
**Numeric Input**

```html
<input type="number" id="quantity" min="1" max="100" step="1">
```

**🎭 Playwright:**
```javascript
await page.getByLabel('Quantity').fill('5');
```

---

#### `<input type="range">`
**Slider Control**

```html
<input type="range" id="volume" min="0" max="100" value="50">
```

**🎭 Playwright:**
```javascript
await page.locator('#volume').fill('75');
```

---

### `<textarea>`
**Multi-line Text Input**

**Purpose:** Accepts multi-line text input.

**Example:**
```html
<label for="bio">Biography:</label>
<textarea id="bio" rows="4" cols="50"></textarea>
```

**🎭 Playwright:**
```javascript
await page.getByLabel('Biography').fill('Multi-line\ntext here');
```

---

### `<select>` and `<option>`
**Dropdown Menu**

**Purpose:** Creates a dropdown selection list.

**Example:**
```html
<label for="country">Country:</label>
<select id="country" name="country">
  <option value="">Select...</option>
  <option value="us">United States</option>
  <option value="uk">United Kingdom</option>
  <option value="ca">Canada</option>
</select>
```

**🎭 Playwright:**
```javascript
await page.getByLabel('Country').selectOption('us');
await page.getByLabel('Country').selectOption({ label: 'United States' });
await page.getByRole('combobox', { name: 'Country' }).selectOption('us');
```

---

### `<button>`
**Button Element**

**Purpose:** Clickable button. More semantic than `<div onclick>` or `<input type="button">`.

**Types:**
```html
<button type="submit">Submit Form</button>
<button type="button">Just a Button</button>
<button type="reset">Reset Form</button>
```

**🎭 Playwright:**
```javascript
await page.getByRole('button', { name: 'Submit Form' }).click();
await page.getByText('Submit Form').click();
```

---

### `<label>`
**Form Label**

**Purpose:** Labels form controls. Clicking label focuses associated input.

**Two Ways to Associate:**

**Implicit:**
```html
<label>
  Username:
  <input type="text">
</label>
```

**Explicit (Preferred):**
```html
<label for="username">Username:</label>
<input type="text" id="username">
```

**🎭 Playwright Impact:**
```javascript
// Labels enable getByLabel()
await page.getByLabel('Username').fill('john');
```

---

### `<fieldset>` and `<legend>`
**Group Form Controls**

**Purpose:** Groups related form controls with a caption.

**Example:**
```html
<fieldset>
  <legend>Personal Information</legend>
  <label for="fname">First Name:</label>
  <input type="text" id="fname">
  <label for="lname">Last Name:</label>
  <input type="text" id="lname">
</fieldset>
```

---

## 6. Tables & Data Display

Tables are for tabular data, not layout (that's CSS's job).

### `<table>`
**Table Container**

**Purpose:** Displays data in rows and columns.

**Basic Structure:**
```html
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>John Doe</td>
      <td>john@example.com</td>
    </tr>
  </tbody>
</table>
```

**🎭 Playwright:**
```javascript
await page.getByRole('table');
await page.getByRole('row').filter({ hasText: 'John Doe' });
await page.getByRole('cell', { name: 'john@example.com' });
```

---

### `<thead>`, `<tbody>`, `<tfoot>`
**Table Sections**

**Purpose:**
- `<thead>` - Table header rows
- `<tbody>` - Table body (data rows)
- `<tfoot>` - Table footer (summaries)

**Example:**
```html
<table>
  <thead>
    <tr><th>Product</th><th>Price</th></tr>
  </thead>
  <tbody>
    <tr><td>Widget</td><td>$10</td></tr>
    <tr><td>Gadget</td><td>$20</td></tr>
  </tbody>
  <tfoot>
    <tr><td>Total</td><td>$30</td></tr>
  </tfoot>
</table>
```

---

### `<tr>`, `<th>`, `<td>`
**Table Row and Cells**

- `<tr>` - Table row
- `<th>` - Table header cell (bold, centered by default)
- `<td>` - Table data cell

**Example:**
```html
<tr>
  <th>Header 1</th>
  <th>Header 2</th>
</tr>
<tr>
  <td>Data 1</td>
  <td>Data 2</td>
</tr>
```

**🎭 Playwright:**
```javascript
// Find specific cell
await page.getByRole('cell', { name: 'Data 1' });

// Find row containing text
await page.getByRole('row', { name: /Widget/ });

// Column headers
await page.getByRole('columnheader', { name: 'Price' });
```

---

### Table Attributes

**colspan** - Cell spans multiple columns
```html
<td colspan="2">Spans 2 columns</td>
```

**rowspan** - Cell spans multiple rows
```html
<td rowspan="3">Spans 3 rows</td>
```

---

## 7. Media & Embedded Content

### `<img>`
**Image Element**

**Purpose:** Embeds images in the page.

**Example:**
```html
<img src="logo.png" alt="Company Logo" width="200" height="100">
```

**Critical Attributes:**
- `src` - Image source URL (required)
- `alt` - Alternative text for accessibility (required)
- `width`, `height` - Dimensions (optional but recommended)

**🎭 Playwright:**
```javascript
await page.getByAltText('Company Logo');
await page.getByRole('img', { name: 'Company Logo' });
await expect(page.getByAltText('Logo')).toBeVisible();
```

---

### `<a>` (Anchor/Link)
**Hyperlink**

**Purpose:** Creates clickable links to other pages or sections.

**Examples:**
```html
<!-- External link -->
<a href="https://example.com">Visit Example</a>

<!-- Internal link -->
<a href="/about">About Us</a>

<!-- Section link -->
<a href="#section1">Jump to Section 1</a>

<!-- Email link -->
<a href="mailto:info@example.com">Email Us</a>

<!-- Phone link -->
<a href="tel:+1234567890">Call Us</a>
```

**🎭 Playwright:**
```javascript
await page.getByRole('link', { name: 'Visit Example' }).click();
await page.getByText('About Us').click();
```

---

### `<video>`
**Video Player**

**Purpose:** Embeds video content.

**Example:**
```html
<video src="movie.mp4" controls width="640" height="360">
  <p>Your browser doesn't support video.</p>
</video>
```

**Attributes:**
- `controls` - Show play/pause controls
- `autoplay` - Auto-start video
- `loop` - Loop video
- `muted` - Start muted

**🎭 Playwright:**
```javascript
await page.locator('video').click(); // Play/pause
await expect(page.locator('video')).toHaveJSProperty('paused', false);
```

---

### `<audio>`
**Audio Player**

**Purpose:** Embeds audio content.

**Example:**
```html
<audio src="podcast.mp3" controls>
  <p>Your browser doesn't support audio.</p>
</audio>
```

---

### `<iframe>`
**Inline Frame**

**Purpose:** Embeds another HTML document within the current page.

**Example:**
```html
<iframe src="https://example.com" width="600" height="400" title="Example"></iframe>
```

**🎭 Playwright:**
```javascript
// Switch to iframe context
const frame = page.frameLocator('iframe[title="Example"]');
await frame.getByRole('button', { name: 'Click Me' }).click();
```

---

### `<svg>`
**Scalable Vector Graphics**

**Purpose:** Embeds vector graphics directly in HTML.

**Example:**
```html
<svg width="100" height="100">
  <circle cx="50" cy="50" r="40" fill="blue" />
</svg>
```

**🎭 Playwright:**
```javascript
await page.locator('svg circle').click();
```

---

## 8. Interactive Elements

### `<details>` and `<summary>`
**Disclosure Widget**

**Purpose:** Creates expandable/collapsible content.

**Example:**
```html
<details>
  <summary>Click to expand</summary>
  <p>Hidden content appears here</p>
</details>
```

**🎭 Playwright:**
```javascript
await page.locator('summary').click(); // Toggle open/close
await expect(page.locator('details')).toHaveAttribute('open', '');
```

---

### `<dialog>`
**Dialog Modal**

**Purpose:** Creates modal or non-modal dialogs.

**Example:**
```html
<dialog id="myDialog">
  <h2>Dialog Title</h2>
  <p>Dialog content</p>
  <button onclick="document.getElementById('myDialog').close()">Close</button>
</dialog>
```

**🎭 Playwright:**
```javascript
await page.getByRole('dialog');
await page.locator('dialog').getByRole('button', { name: 'Close' }).click();
```

---

## 9. Essential HTML Attributes

Attributes modify element behavior and provide metadata for browsers and assistive technologies.

### Core Attributes

#### `id`
**Unique Identifier**

**Purpose:** Uniquely identifies an element on the page. Must be unique.

**Example:**
```html
<div id="main-content">Content</div>
```

**Uses:**
- CSS styling: `#main-content { }`
- JavaScript: `document.getElementById('main-content')`
- Label association: `<label for="email">`
- Fragment links: `<a href="#main-content">`

**🎭 Playwright:**
```javascript
await page.locator('#main-content');
await page.locator('div#main-content');
```

---

#### `class`
**CSS Class Names**

**Purpose:** Assigns one or more class names for styling or scripting. Reusable across multiple elements.

**Example:**
```html
<div class="card featured">Content</div>
```

**Multiple Classes:** Space-separated

**🎭 Playwright:**
```javascript
await page.locator('.card.featured');
await page.locator('div.card');
```

---

#### `name`
**Form Control Name**

**Purpose:** Names form controls for form submission. The key in key-value pairs sent to server.

**Example:**
```html
<input type="text" name="username" id="username">
```

**Form Data:** `username=john_doe`

---

#### `value`
**Input Value**

**Purpose:** Specifies the value of form controls.

**Examples:**
```html
<input type="text" value="Default text">
<option value="us">United States</option>
<button value="submit">Submit</button>
```

**🎭 Playwright:**
```javascript
await page.locator('input').inputValue(); // Get value
await expect(page.locator('input')).toHaveValue('Default text');
```

---

### Form Attributes

#### `placeholder`
**Input Placeholder Text**

**Purpose:** Shows hint text inside input when empty.

**Example:**
```html
<input type="email" placeholder="Enter your email">
```

**🎭 Playwright:**
```javascript
await page.getByPlaceholder('Enter your email').fill('user@example.com');
```

---

#### `required`
**Required Field**

**Purpose:** Makes form field mandatory. Browser validates before submit.

**Example:**
```html
<input type="text" required>
```

**🎭 Playwright:**
```javascript
await expect(page.locator('input[required]')).toHaveAttribute('required', '');
```

---

#### `disabled`
**Disabled State**

**Purpose:** Disables form control. Cannot be interacted with.

**Example:**
```html
<button disabled>Submit</button>
```

**🎭 Playwright:**
```javascript
await expect(page.getByRole('button', { name: 'Submit' })).toBeDisabled();
```

---

#### `readonly`
**Read-Only Input**

**Purpose:** Input value cannot be changed by user, but can be submitted.

**Example:**
```html
<input type="text" value="Read-only value" readonly>
```

---

#### `autocomplete`
**Form Autocomplete**

**Purpose:** Controls browser autofill behavior.

**Examples:**
```html
<input type="email" autocomplete="email">
<input type="password" autocomplete="current-password">
<input type="text" autocomplete="off">
```

**Common Values:**
- `email`, `username`, `new-password`, `current-password`
- `tel`, `address-line1`, `postal-code`
- `off` - Disable autocomplete

---

### Data Attributes

#### `data-*`
**Custom Data Attributes**

**Purpose:** Store custom data on elements. Accessible via JavaScript and CSS.

**Example:**
```html
<button data-user-id="12345" data-action="delete">Delete</button>
```

**JavaScript:**
```javascript
element.dataset.userId; // "12345"
element.dataset.action; // "delete"
```

**🎭 Playwright:**
```javascript
await page.locator('[data-user-id="12345"]').click();
await page.locator('button[data-action="delete"]').click();
```

---

#### `data-testid`
**Testing Identifier**

**Purpose:** Dedicated attribute for test automation. Doesn't affect functionality or styling.

**Example:**
```html
<button data-testid="submit-btn">Submit</button>
```

**🎭 Playwright:**
```javascript
await page.getByTestId('submit-btn').click();
```

**Best Practice:** Use when semantic selectors (role, label, text) aren't available or reliable.

---

### Accessibility (ARIA) Attributes

ARIA (Accessible Rich Internet Applications) attributes improve accessibility for screen readers and assistive technologies.

#### Common ARIA Attributes

**`aria-label`** - Accessible name for element
```html
<button aria-label="Close dialog">✕</button>
```

**`aria-labelledby`** - Reference another element for label
```html
<div role="dialog" aria-labelledby="title">
  <h2 id="title">Dialog Title</h2>
</div>
```

**`aria-expanded`** - Expandable element state
```html
<button aria-expanded="false">Menu</button>
```

**`aria-hidden`** - Hide from screen readers
```html
<svg aria-hidden="true">...</svg>
```

**`role`** - Define element's role
```html
<div role="alert">Error!</div>
```

**🎭 Playwright ARIA Usage:**
```javascript
await page.getByLabel('Close').click();
await page.getByRole('dialog', { name: 'Settings' }).locator('button').click();
await expect(page.locator('button')).toHaveAttribute('aria-expanded', 'true');
await page.getByRole('alert').toHaveText('Error!');
```

---

## 10. Playwright Locator Strategies

### Locator Priority (Best to Worst)

| # | Method | Example | Why |
|---|--------|---------|-----|
| 1 | `getByRole()` | `getByRole('button', { name: 'Submit' })` | Semantic, accessible |
| 2 | `getByLabel()` | `getByLabel('Email')` | Works with labels |
| 3 | `getByPlaceholder()` | `getByPlaceholder('Search')` | When no labels |
| 4 | `getByText()` | `getByText('Welcome')` | Visible content |
| 5 | `getByTestId()` | `getByTestId('submit-btn')` | Dedicated attribute |
| 6 | `locator()` | `locator('#id')` | Last resort |

### Role Reference

| Element | Role | Playwright |
|---------|------|------------|
| `<button>` | button | `getByRole('button')` |
| `<a href>` | link | `getByRole('link')` |
| `<input text>` | textbox | `getByRole('textbox')` |
| `<input checkbox>` | checkbox | `getByRole('checkbox')` |
| `<select>` | combobox | `getByRole('combobox')` |
| `<table>` | table | `getByRole('table')` |
| `<header>` | banner | `getByRole('banner')` |
| `<nav>` | navigation | `getByRole('navigation')` |
| `<main>` | main | `getByRole('main')` |
| `<footer>` | contentinfo | `getByRole('contentinfo')` |
| `<h1-h6>` | heading | `getByRole('heading', { level: 1 })` |

---

## 11. Influencing Design & Development Decisions

As a QA engineer, you can advocate for better HTML that improves accessibility AND testability.

### Key Discussion Points

#### 💡 1. Request Semantic Elements

**Instead of:** "This is hard to test"

**Say:** "If we use `<button>` instead of `<div onclick>`, screen readers work AND our tests are more stable."

---

#### 💡 2. Advocate for Proper Labels

**Instead of:** "I can't find this input"

**Say:** "Adding `<label for="email">` lets us use `getByLabel()` AND improves accessibility."

---

#### 💡 3. Suggest Test IDs When Needed

**Instead of:** "No stable selector"

**Say:** "Can we add `data-testid` here? It won't affect styling but gives us a stable hook."

---

#### 💡 4. Frame Accessibility as Testing Issues

**Instead of:** "Playwright can't find this"

**Say:** "This element has no role or accessible name. That's both an accessibility bug AND makes it untestable."

---

### Pre-Development Checklist

#### ✅ HTML Review Checklist

- ☐ Semantic elements used (header, nav, main, section)?
- ☐ Interactive elements have roles (button, link)?
- ☐ Form inputs properly labeled?
- ☐ Images have meaningful alt text?
- ☐ Headings are hierarchical (h1 → h2 → h3)?
- ☐ Modals use dialog semantics?
- ☐ Custom components have ARIA?
- ☐ data-testid planned for dynamic content?

---

### ✅ Benefits of Good HTML

- **Faster test writing:** Semantic selectors are self-documenting
- **Stable tests:** Role-based selectors survive CSS changes
- **Accessibility:** Meets WCAG compliance
- **Easier debugging:** Clear structure = obvious issues
- **Future-proof:** Works with new tools and browsers

---

## Conclusion

Understanding HTML tags, their purposes, and their implications for test automation is essential for modern QA engineers. This knowledge enables you to:

1. Write more reliable and maintainable tests
2. Collaborate effectively with developers
3. Advocate for accessible and testable applications
4. Navigate the DOM efficiently
5. Choose the right locator strategies

**Remember:** Good HTML isn't just about making tests easier—it's about building better, more accessible applications for everyone.

---

**HTML Tags Training Guide for Test Automation Engineers**

*Created for Playwright test automation training*
