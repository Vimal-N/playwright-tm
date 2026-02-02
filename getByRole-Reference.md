# Playwright getByRole() Complete Reference Guide

## Why getByRole() is the Recommended Locator

`getByRole()` is Playwright's **most recommended** locator strategy because it:

1. **Mirrors user perception** - Finds elements the way users and screen readers see them
2. **Ensures accessibility** - If your tests work with roles, your app is more accessible
3. **Survives refactoring** - CSS classes and IDs change; semantic roles don't
4. **Self-documenting** - `getByRole('button', { name: 'Submit' })` is instantly readable

```typescript
// ✅ RECOMMENDED - Semantic, accessible, resilient
await page.getByRole('button', { name: 'Submit' }).click();

// ❌ AVOID - Fragile, tied to implementation
await page.locator('#submit-btn').click();
await page.locator('.btn-primary').click();
```

---

## Quick Syntax Reference

```typescript
// Basic usage
page.getByRole('button')

// With accessible name (recommended)
page.getByRole('button', { name: 'Submit' })

// Case-insensitive with regex
page.getByRole('button', { name: /submit/i })

// Exact match
page.getByRole('button', { name: 'Submit', exact: true })

// Additional options
page.getByRole('heading', { level: 1 })           // Heading level
page.getByRole('checkbox', { checked: true })      // State
page.getByRole('button', { pressed: true })        // Pressed state
page.getByRole('button', { disabled: true })       // Disabled state
page.getByRole('combobox', { expanded: true })     // Expanded state
```

---

## Common Roles (Daily Use)

### button
**HTML Elements:** `<button>`, `<input type="submit">`, `<input type="button">`, `<input type="reset">`

```html
<button>Click Me</button>
<button type="submit">Submit Form</button>
<input type="button" value="Action" />
<div role="button">Custom Button</div>
```

```typescript
// Click a button by its text
await page.getByRole('button', { name: 'Click Me' }).click();

// Submit button
await page.getByRole('button', { name: 'Submit Form' }).click();

// Partial match with regex
await page.getByRole('button', { name: /submit/i }).click();

// Disabled button assertion
await expect(page.getByRole('button', { name: 'Save' })).toBeDisabled();
```

---

### link
**HTML Elements:** `<a href="...">`

```html
<a href="/home">Home</a>
<a href="/products">View Products</a>
<a href="https://external.com" target="_blank">External Link</a>
```

```typescript
// Click a navigation link
await page.getByRole('link', { name: 'Home' }).click();

// Verify link exists
await expect(page.getByRole('link', { name: 'View Products' })).toBeVisible();

// Check link href
await expect(page.getByRole('link', { name: 'Home' })).toHaveAttribute('href', '/home');
```

---

### textbox
**HTML Elements:** `<input type="text">`, `<input type="email">`, `<input type="tel">`, `<input type="url">`, `<input type="password">`, `<textarea>`

```html
<label>
  Email
  <input type="email" />
</label>

<label for="message">Message</label>
<textarea id="message"></textarea>
```

```typescript
// Fill a text input
await page.getByRole('textbox', { name: 'Email' }).fill('user@example.com');

// Fill textarea
await page.getByRole('textbox', { name: 'Message' }).fill('Hello world');

// Clear and type
await page.getByRole('textbox', { name: 'Search' }).clear();
await page.getByRole('textbox', { name: 'Search' }).fill('query');

// Verify value
await expect(page.getByRole('textbox', { name: 'Email' })).toHaveValue('user@example.com');
```

> **Note:** For password fields, use `getByLabel('Password')` instead, as password inputs don't have a textbox role for security reasons.

---

### checkbox
**HTML Elements:** `<input type="checkbox">`

```html
<label>
  <input type="checkbox" /> Remember me
</label>

<input type="checkbox" id="terms" />
<label for="terms">I agree to the terms</label>
```

```typescript
// Check a checkbox
await page.getByRole('checkbox', { name: 'Remember me' }).check();

// Uncheck
await page.getByRole('checkbox', { name: 'Remember me' }).uncheck();

// Toggle
await page.getByRole('checkbox', { name: 'Subscribe' }).setChecked(true);

// Assert checked state
await expect(page.getByRole('checkbox', { name: 'Terms' })).toBeChecked();
await expect(page.getByRole('checkbox', { name: 'Terms' })).not.toBeChecked();

// Find only checked checkboxes
page.getByRole('checkbox', { checked: true })
```

---

### radio
**HTML Elements:** `<input type="radio">`

```html
<fieldset>
  <legend>Payment Method</legend>
  <label><input type="radio" name="payment" /> Credit Card</label>
  <label><input type="radio" name="payment" /> PayPal</label>
  <label><input type="radio" name="payment" /> Bank Transfer</label>
</fieldset>
```

```typescript
// Select a radio option
await page.getByRole('radio', { name: 'Credit Card' }).check();

// Verify selection
await expect(page.getByRole('radio', { name: 'Credit Card' })).toBeChecked();
await expect(page.getByRole('radio', { name: 'PayPal' })).not.toBeChecked();
```

---

### combobox
**HTML Elements:** `<select>`, autocomplete inputs with `role="combobox"`

```html
<label>
  Country
  <select>
    <option value="">Select...</option>
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
    <option value="in">India</option>
  </select>
</label>
```

```typescript
// Select by visible text
await page.getByRole('combobox', { name: 'Country' }).selectOption('India');

// Select by value
await page.getByRole('combobox', { name: 'Country' }).selectOption({ value: 'in' });

// Select by label
await page.getByRole('combobox', { name: 'Country' }).selectOption({ label: 'India' });

// Verify selected option
await expect(page.getByRole('combobox', { name: 'Country' })).toHaveValue('in');
```

---

### heading
**HTML Elements:** `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>`

```html
<h1>Welcome to Our Site</h1>
<h2>Featured Products</h2>
<h3>Electronics</h3>
```

```typescript
// Find any heading
await expect(page.getByRole('heading', { name: 'Welcome to Our Site' })).toBeVisible();

// Find heading by level
await expect(page.getByRole('heading', { level: 1 })).toHaveText('Welcome to Our Site');
await expect(page.getByRole('heading', { level: 2 })).toHaveText('Featured Products');

// Combine name and level
await expect(page.getByRole('heading', { name: 'Electronics', level: 3 })).toBeVisible();

// Count headings
await expect(page.getByRole('heading', { level: 2 })).toHaveCount(3);
```

---

### img
**HTML Elements:** `<img>`

```html
<img src="logo.png" alt="Company Logo" />
<img src="product.jpg" alt="Wireless Headphones" />
```

```typescript
// Find image by alt text
await expect(page.getByRole('img', { name: 'Company Logo' })).toBeVisible();

// Click an image
await page.getByRole('img', { name: 'Wireless Headphones' }).click();

// Verify image loaded (check natural width)
const img = page.getByRole('img', { name: 'Company Logo' });
await expect(img).toHaveJSProperty('naturalWidth', { greaterThan: 0 });
```

---

## Table Roles

### table, row, cell, columnheader, rowheader

```html
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th>Status</th>
      <th>Actions</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>John Doe</td>
      <td>john@example.com</td>
      <td>Active</td>
      <td><button>Edit</button></td>
    </tr>
    <tr>
      <td>Jane Smith</td>
      <td>jane@example.com</td>
      <td>Inactive</td>
      <td><button>Edit</button></td>
    </tr>
  </tbody>
</table>
```

```typescript
// Find the table
const table = page.getByRole('table');

// Find a specific row by content
const johnRow = page.getByRole('row', { name: /John Doe/ });

// Click Edit button in John's row
await page.getByRole('row', { name: /John Doe/ })
  .getByRole('button', { name: 'Edit' })
  .click();

// Verify a cell value
await expect(page.getByRole('row', { name: /Jane Smith/ }))
  .toContainText('Inactive');

// Count rows (excluding header)
await expect(page.getByRole('row')).toHaveCount(3); // header + 2 data rows

// Find column header
await expect(page.getByRole('columnheader', { name: 'Status' })).toBeVisible();

// Get specific cell
const statusCell = page.getByRole('row', { name: /John Doe/ })
  .getByRole('cell', { name: 'Active' });
await expect(statusCell).toBeVisible();
```

---

## List Roles

### list, listitem

```html
<ul>
  <li>Apple</li>
  <li>Banana</li>
  <li>Orange</li>
</ul>

<ol>
  <li>First step</li>
  <li>Second step</li>
  <li>Third step</li>
</ol>
```

```typescript
// Find list
const fruitList = page.getByRole('list');

// Count list items
await expect(page.getByRole('listitem')).toHaveCount(3);

// Find specific item
await page.getByRole('listitem').filter({ hasText: 'Banana' }).click();

// Verify all items text
await expect(page.getByRole('listitem')).toHaveText(['Apple', 'Banana', 'Orange']);

// Get first/last item
await page.getByRole('listitem').first().click();
await page.getByRole('listitem').last().click();

// Get nth item (0-indexed)
await page.getByRole('listitem').nth(1).click(); // Banana
```

---

## Navigation & Structure Roles

### navigation
**HTML Elements:** `<nav>`

```html
<nav aria-label="Main">
  <a href="/">Home</a>
  <a href="/products">Products</a>
  <a href="/about">About</a>
  <a href="/contact">Contact</a>
</nav>

<nav aria-label="Footer">
  <a href="/privacy">Privacy</a>
  <a href="/terms">Terms</a>
</nav>
```

```typescript
// Find main navigation and click link
await page.getByRole('navigation', { name: 'Main' })
  .getByRole('link', { name: 'Products' })
  .click();

// Find footer navigation
await page.getByRole('navigation', { name: 'Footer' })
  .getByRole('link', { name: 'Privacy' })
  .click();

// If only one nav, no name needed
await page.getByRole('navigation').getByRole('link', { name: 'Home' }).click();
```

---

### main
**HTML Elements:** `<main>`

```html
<main>
  <h1>Page Title</h1>
  <p>Main content here...</p>
</main>
```

```typescript
// Scope interactions to main content area
await page.getByRole('main').getByRole('heading', { level: 1 });

// Verify main content loaded
await expect(page.getByRole('main')).toBeVisible();
```

---

### banner
**HTML Elements:** `<header>` (top-level only)

```html
<header>
  <img src="logo.png" alt="Logo" />
  <nav>...</nav>
</header>
```

```typescript
// Find logo in header
await page.getByRole('banner').getByRole('img', { name: 'Logo' }).click();
```

---

### contentinfo
**HTML Elements:** `<footer>` (top-level only)

```html
<footer>
  <p>© 2024 Company Name</p>
  <nav>...</nav>
</footer>
```

```typescript
// Find footer content
await expect(page.getByRole('contentinfo')).toContainText('© 2024');
```

---

### region
**HTML Elements:** `<section>` with aria-label/aria-labelledby

```html
<section aria-label="Sidebar">
  <h2>Related Articles</h2>
  ...
</section>

<section aria-label="Comments">
  <h2>User Comments</h2>
  ...
</section>
```

```typescript
// Find specific region
await page.getByRole('region', { name: 'Sidebar' }).getByRole('link').first().click();

// Verify comments section
await expect(page.getByRole('region', { name: 'Comments' })).toBeVisible();
```

---

### article
**HTML Elements:** `<article>`

```html
<article>
  <h2>Blog Post Title</h2>
  <p>Article content...</p>
</article>
```

```typescript
// Find article
await expect(page.getByRole('article')).toContainText('Blog Post Title');

// Multiple articles
await expect(page.getByRole('article')).toHaveCount(5);
await page.getByRole('article').filter({ hasText: 'Specific Post' }).click();
```

---

## Dialog & Alert Roles

### dialog
**HTML Elements:** `<dialog>`, elements with `role="dialog"`

```html
<dialog open>
  <h2>Confirm Action</h2>
  <p>Are you sure you want to proceed?</p>
  <button>Cancel</button>
  <button>Confirm</button>
</dialog>
```

```typescript
// Wait for dialog to appear
await expect(page.getByRole('dialog')).toBeVisible();

// Interact with dialog content
await page.getByRole('dialog').getByRole('button', { name: 'Confirm' }).click();

// Verify dialog closed
await expect(page.getByRole('dialog')).toBeHidden();

// Named dialog
await page.getByRole('dialog', { name: 'Confirm Action' }).getByRole('button', { name: 'Cancel' }).click();
```

---

### alertdialog
**HTML Elements:** Confirmation dialogs with `role="alertdialog"`

```html
<div role="alertdialog" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Delete Item?</h2>
  <p>This action cannot be undone.</p>
  <button>Cancel</button>
  <button>Delete</button>
</div>
```

```typescript
// Handle alert dialog
await expect(page.getByRole('alertdialog')).toBeVisible();
await page.getByRole('alertdialog').getByRole('button', { name: 'Delete' }).click();
```

---

### alert
**HTML Elements:** Elements with `role="alert"`

```html
<div role="alert">
  Form submitted successfully!
</div>

<div role="alert">
  Error: Invalid email address
</div>
```

```typescript
// Verify success message
await expect(page.getByRole('alert')).toHaveText('Form submitted successfully!');

// Verify error message
await expect(page.getByRole('alert')).toContainText('Error');

// Wait for alert to appear
await expect(page.getByRole('alert')).toBeVisible({ timeout: 5000 });
```

---

## Form Roles

### form
**HTML Elements:** `<form>` with accessible name

```html
<form aria-label="Login">
  <input type="email" />
  <input type="password" />
  <button type="submit">Sign In</button>
</form>

<form aria-label="Search">
  <input type="search" />
  <button type="submit">Search</button>
</form>
```

```typescript
// Interact with specific form
await page.getByRole('form', { name: 'Login' })
  .getByRole('button', { name: 'Sign In' })
  .click();

// Scope to search form
await page.getByRole('form', { name: 'Search' })
  .getByRole('searchbox')
  .fill('query');
```

---

### searchbox
**HTML Elements:** `<input type="search">`

```html
<label>
  Search Products
  <input type="search" placeholder="Enter keywords..." />
</label>
```

```typescript
// Fill search box
await page.getByRole('searchbox', { name: 'Search Products' }).fill('laptop');

// Or without name if only one search box
await page.getByRole('searchbox').fill('laptop');
```

---

### spinbutton
**HTML Elements:** `<input type="number">`

```html
<label>
  Quantity
  <input type="number" min="1" max="99" value="1" />
</label>
```

```typescript
// Set quantity
await page.getByRole('spinbutton', { name: 'Quantity' }).fill('5');

// Verify value
await expect(page.getByRole('spinbutton', { name: 'Quantity' })).toHaveValue('5');

// Clear and set
await page.getByRole('spinbutton', { name: 'Quantity' }).clear();
await page.getByRole('spinbutton', { name: 'Quantity' }).fill('10');
```

---

### slider
**HTML Elements:** `<input type="range">`

```html
<label>
  Volume
  <input type="range" min="0" max="100" value="50" />
</label>
```

```typescript
// Set slider value
await page.getByRole('slider', { name: 'Volume' }).fill('75');

// Verify value
await expect(page.getByRole('slider', { name: 'Volume' })).toHaveValue('75');
```

---

### switch
**HTML Elements:** Toggle switches with `role="switch"`

```html
<button role="switch" aria-checked="false" aria-label="Dark mode">
  <span>Dark Mode</span>
</button>
```

```typescript
// Toggle switch on
await page.getByRole('switch', { name: 'Dark mode' }).click();

// Verify switch state
await expect(page.getByRole('switch', { name: 'Dark mode' })).toBeChecked();
```

---

### option
**HTML Elements:** `<option>` inside select

```html
<select>
  <option value="red">Red</option>
  <option value="green">Green</option>
  <option value="blue">Blue</option>
</select>
```

```typescript
// Usually you interact with combobox, not options directly
// But you can verify options exist:
await expect(page.getByRole('option', { name: 'Red' })).toBeAttached();

// Count options
await expect(page.getByRole('option')).toHaveCount(3);
```

---

## Tab Roles

### tab, tablist, tabpanel

```html
<div role="tablist">
  <button role="tab" aria-selected="true" aria-controls="panel-1">General</button>
  <button role="tab" aria-selected="false" aria-controls="panel-2">Settings</button>
  <button role="tab" aria-selected="false" aria-controls="panel-3">Advanced</button>
</div>

<div role="tabpanel" id="panel-1">General content...</div>
<div role="tabpanel" id="panel-2" hidden>Settings content...</div>
<div role="tabpanel" id="panel-3" hidden>Advanced content...</div>
```

```typescript
// Click a tab
await page.getByRole('tab', { name: 'Settings' }).click();

// Verify tab is selected
await expect(page.getByRole('tab', { name: 'Settings' })).toHaveAttribute('aria-selected', 'true');

// Verify tab panel content
await expect(page.getByRole('tabpanel')).toContainText('Settings content');

// Find selected tab
page.getByRole('tab', { selected: true })
```

---

## Menu Roles

### menu, menuitem, menubar

```html
<button aria-haspopup="menu" aria-expanded="false">Actions</button>
<ul role="menu">
  <li role="menuitem">Edit</li>
  <li role="menuitem">Duplicate</li>
  <li role="menuitem">Delete</li>
</ul>
```

```typescript
// Open menu and click item
await page.getByRole('button', { name: 'Actions' }).click();
await page.getByRole('menuitem', { name: 'Delete' }).click();

// Verify menu is visible
await expect(page.getByRole('menu')).toBeVisible();

// Count menu items
await expect(page.getByRole('menuitem')).toHaveCount(3);
```

---

## Other Useful Roles

### progressbar
**HTML Elements:** `<progress>`

```html
<progress value="70" max="100" aria-label="Upload progress">70%</progress>
```

```typescript
// Verify progress
await expect(page.getByRole('progressbar', { name: 'Upload progress' })).toHaveAttribute('value', '70');
```

---

### status
**HTML Elements:** Elements with `role="status"`

```html
<div role="status">3 items in cart</div>
```

```typescript
await expect(page.getByRole('status')).toHaveText('3 items in cart');
```

---

### tooltip
**HTML Elements:** Elements with `role="tooltip"`

```html
<button aria-describedby="tip">Hover me</button>
<div role="tooltip" id="tip">Helpful information</div>
```

```typescript
await page.getByRole('button', { name: 'Hover me' }).hover();
await expect(page.getByRole('tooltip')).toHaveText('Helpful information');
```

---

### separator
**HTML Elements:** `<hr>`

```html
<hr />
```

```typescript
await expect(page.getByRole('separator')).toBeVisible();
```

---

### group
**HTML Elements:** `<fieldset>`, `<optgroup>`

```html
<fieldset>
  <legend>Shipping Options</legend>
  <label><input type="radio" name="ship" /> Standard</label>
  <label><input type="radio" name="ship" /> Express</label>
</fieldset>
```

```typescript
await page.getByRole('group', { name: 'Shipping Options' })
  .getByRole('radio', { name: 'Express' })
  .check();
```

---

## Role Options Reference

| Option | Type | Description | Example |
|--------|------|-------------|---------|
| `name` | string \| RegExp | Accessible name | `{ name: 'Submit' }` or `{ name: /submit/i }` |
| `exact` | boolean | Exact name match | `{ name: 'Log', exact: true }` |
| `level` | number | Heading level (1-6) | `{ level: 1 }` |
| `checked` | boolean | Checkbox/radio state | `{ checked: true }` |
| `pressed` | boolean | Toggle button state | `{ pressed: true }` |
| `selected` | boolean | Tab/option selection | `{ selected: true }` |
| `expanded` | boolean | Expandable state | `{ expanded: true }` |
| `disabled` | boolean | Disabled state | `{ disabled: true }` |
| `includeHidden` | boolean | Include hidden elements | `{ includeHidden: true }` |

---

## Complete Role List (Alphabetical)

| Role | Common HTML Element(s) |
|------|------------------------|
| `alert` | `role="alert"` |
| `alertdialog` | `role="alertdialog"` |
| `article` | `<article>` |
| `banner` | `<header>` (top-level) |
| `button` | `<button>`, `<input type="button/submit/reset">` |
| `cell` | `<td>` |
| `checkbox` | `<input type="checkbox">` |
| `columnheader` | `<th>` |
| `combobox` | `<select>`, `role="combobox"` |
| `complementary` | `<aside>` |
| `contentinfo` | `<footer>` (top-level) |
| `definition` | `<dd>` |
| `dialog` | `<dialog>`, `role="dialog"` |
| `document` | `role="document"` |
| `figure` | `<figure>` |
| `form` | `<form>` (with name) |
| `grid` | `role="grid"` |
| `gridcell` | `role="gridcell"` |
| `group` | `<fieldset>`, `<optgroup>` |
| `heading` | `<h1>` - `<h6>` |
| `img` | `<img>` |
| `link` | `<a href="...">` |
| `list` | `<ul>`, `<ol>` |
| `listbox` | `role="listbox"` |
| `listitem` | `<li>` |
| `main` | `<main>` |
| `menu` | `role="menu"` |
| `menubar` | `role="menubar"` |
| `menuitem` | `role="menuitem"` |
| `menuitemcheckbox` | `role="menuitemcheckbox"` |
| `menuitemradio` | `role="menuitemradio"` |
| `meter` | `<meter>` |
| `navigation` | `<nav>` |
| `option` | `<option>` |
| `paragraph` | `<p>` |
| `progressbar` | `<progress>` |
| `radio` | `<input type="radio">` |
| `radiogroup` | `role="radiogroup"` |
| `region` | `<section>` (with name) |
| `row` | `<tr>` |
| `rowgroup` | `<tbody>`, `<thead>`, `<tfoot>` |
| `rowheader` | `<th scope="row">` |
| `scrollbar` | `role="scrollbar"` |
| `searchbox` | `<input type="search">` |
| `separator` | `<hr>` |
| `slider` | `<input type="range">` |
| `spinbutton` | `<input type="number">` |
| `status` | `role="status"` |
| `switch` | `role="switch"` |
| `tab` | `role="tab"` |
| `table` | `<table>` |
| `tablist` | `role="tablist"` |
| `tabpanel` | `role="tabpanel"` |
| `term` | `<dt>` |
| `textbox` | `<input type="text/email/tel/url">`, `<textarea>` |
| `toolbar` | `role="toolbar"` |
| `tooltip` | `role="tooltip"` |
| `tree` | `role="tree"` |
| `treeitem` | `role="treeitem"` |

---

## When to Use getByRole vs Other Locators

| Scenario | Recommended Locator |
|----------|---------------------|
| Buttons, links, inputs | `getByRole()` ✅ |
| Form fields with labels | `getByLabel()` ✅ |
| Inputs without labels | `getByPlaceholder()` |
| Static text content | `getByText()` |
| Images | `getByRole('img')` or `getByAltText()` |
| No semantic meaning | `getByTestId()` |
| Complex nested structures | Chained `getByRole()` calls |

---

## Resources

- [WAI-ARIA Roles Specification](https://www.w3.org/TR/wai-aria-1.2/#role_definitions)
- [Playwright Locators Documentation](https://playwright.dev/docs/locators)
- [HTML Element to ARIA Role Mappings](https://www.w3.org/TR/html-aria/#docconformance)
- [Chrome DevTools Accessibility Panel](https://developer.chrome.com/docs/devtools/accessibility/reference/) - Great for discovering roles

---

*This reference is part of the Playwright Training Curriculum. For complete training materials, see the main curriculum document.*
