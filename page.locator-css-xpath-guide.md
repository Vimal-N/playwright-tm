# Playwright page.locator() Complete Reference Guide

## When Semantic Locators Don't Work - Your CSS & XPath Fallback

---

## Table of Contents

1. [When to Use page.locator()](#when-to-use-pagelocator)
2. [Basic CSS Selectors](#basic-css-selectors)
3. [Attribute Selectors](#attribute-selectors)
4. [Combining Selectors](#combining-selectors)
5. [Pseudo-Classes](#pseudo-classes)
6. [Filtering Locators](#filtering-locators)
7. [Chaining Locators](#chaining-locators)
8. [XPath Selectors](#xpath-selectors)
9. [Text-Based Locators](#text-based-locators)
10. [Advanced Combinations](#advanced-combinations)
11. [Real-World Scenarios](#real-world-scenarios)
12. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## When to Use page.locator()

### The Locator Priority Pyramid

```
                    ▲
                   /█\
                  / █ \         ← MOST PREFERRED
                 /  █  \           getByRole()
                /───█───\          getByLabel()
               /    █    \         getByText()
              /     █     \
             /──────█──────\    ← WHEN NEEDED
            /       █       \      getByTestId()
           /        █        \
          /─────────█─────────\
         /          █          \ ← LAST RESORT
        /           █           \   page.locator()
       /────────────█────────────\  (CSS/XPath)
      ▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
```

### Use page.locator() When:

| Scenario | Example |
|----------|---------|
| Element has no accessible name | `<div class="price">$99</div>` |
| Need to select by data attributes | `<button data-action="delete">` |
| Complex structural relationships | Parent > child > grandchild |
| No semantic HTML available | Legacy applications |
| Need nth-child or positional selection | Third item in a list |
| Custom components without ARIA | `<app-button>` |

### ⚠️ Remember

```
page.locator() is powerful but fragile!

CSS selectors tied to:
  ❌ Class names → Change with styling updates
  ❌ IDs → May be auto-generated
  ❌ DOM structure → Changes with refactoring

Always try semantic locators first!
```

---

## Basic CSS Selectors

### By ID

```html
<input id="email" type="email" />
<button id="submit-btn">Submit</button>
<div id="error-message">Error occurred</div>
```

```typescript
// Select by ID using # prefix
await page.locator('#email').fill('user@test.com');
await page.locator('#submit-btn').click();
await expect(page.locator('#error-message')).toBeVisible();

// Multiple IDs (any of them)
await page.locator('#login-btn, #signin-btn').click();
```

---

### By Class Name

```html
<button class="btn btn-primary">Submit</button>
<div class="alert alert-error">Error message</div>
<span class="price currency">$99.99</span>
```

```typescript
// Single class using . prefix
await page.locator('.btn-primary').click();

// Multiple classes (element must have ALL classes)
await page.locator('.alert.alert-error').click();
await page.locator('.price.currency').textContent();

// Class contains word (matches class="btn-primary" or class="primary-btn")
// Use attribute selector for partial match
await page.locator('[class*="primary"]').click();
```

---

### By HTML Element (Tag Name)

```html
<button>Click Me</button>
<input type="text" />
<textarea></textarea>
<a href="/home">Home</a>
<h1>Page Title</h1>
```

```typescript
// Select by tag name
await page.locator('button').click();
await page.locator('input').fill('text');
await page.locator('textarea').fill('long text');
await page.locator('a').click();
await page.locator('h1').textContent();

// Specific heading levels
await page.locator('h1').textContent();  // Only h1
await page.locator('h2').textContent();  // Only h2

// All headings
await page.locator('h1, h2, h3, h4, h5, h6').count();
```

---

### Combined: Tag + Class

```html
<button class="primary">Submit</button>
<a class="primary">Link</a>
<div class="primary">Box</div>
```

```typescript
// Tag + Class (more specific)
await page.locator('button.primary').click();  // Only buttons with .primary
await page.locator('a.primary').click();       // Only links with .primary
await page.locator('div.primary').click();     // Only divs with .primary
```

---

### Combined: Tag + ID

```html
<input id="username" type="text" />
<div id="username">Display name</div>
```

```typescript
// Tag + ID (when same ID used on different elements - bad HTML but happens)
await page.locator('input#username').fill('john');
await page.locator('div#username').textContent();
```

---

## Attribute Selectors

### Exact Attribute Match

```html
<input type="email" name="user-email" />
<input type="password" name="user-password" />
<button type="submit">Submit</button>
<a href="/about">About</a>
<img src="logo.png" alt="Company Logo" />
```

```typescript
// [attribute="value"] - Exact match
await page.locator('[type="email"]').fill('user@test.com');
await page.locator('[type="password"]').fill('secret');
await page.locator('[type="submit"]').click();
await page.locator('[href="/about"]').click();
await page.locator('[alt="Company Logo"]').click();

// Name attribute
await page.locator('[name="user-email"]').fill('test@test.com');
await page.locator('[name="user-password"]').fill('password');
```

---

### Partial Attribute Match

```html
<input data-testid="login-email-input" />
<input data-testid="login-password-input" />
<button data-testid="login-submit-button" />
<a href="https://example.com/products/laptop-pro-15">Laptop</a>
<div class="card-product-featured">Featured Product</div>
```

```typescript
// [attribute^="value"] - Starts with
await page.locator('[data-testid^="login"]').count();  // All login-related elements
await page.locator('[href^="https://"]').count();      // All external links
await page.locator('[class^="card-"]').count();        // Classes starting with "card-"

// [attribute$="value"] - Ends with
await page.locator('[data-testid$="-button"]').click();  // Elements ending with "-button"
await page.locator('[href$=".pdf"]').click();            // PDF links
await page.locator('[src$=".png"]').count();             // PNG images

// [attribute*="value"] - Contains
await page.locator('[data-testid*="email"]').fill('test@test.com');  // Contains "email"
await page.locator('[href*="product"]').click();                      // URLs containing "product"
await page.locator('[class*="featured"]').click();                    // Classes containing "featured"
```

---

### Attribute Existence

```html
<input type="text" required />
<input type="text" />
<button disabled>Can't Click</button>
<button>Can Click</button>
<input type="checkbox" checked />
```

```typescript
// [attribute] - Has attribute (any value)
await page.locator('[required]').count();     // All required fields
await page.locator('[disabled]').count();     // All disabled elements
await page.locator('[checked]').count();      // All checked inputs
await page.locator('[readonly]').count();     // All readonly fields

// Combine with :not() for opposite
await page.locator('button:not([disabled])').click();  // Enabled buttons only
await page.locator('input:not([readonly])').fill('text');  // Editable inputs
```

---

### Data Attributes (Custom)

```html
<button data-action="save" data-id="123">Save</button>
<button data-action="delete" data-id="123">Delete</button>
<div data-status="active" data-user-id="456">User</div>
<tr data-row-id="789" data-category="electronics">Product Row</tr>
```

```typescript
// Custom data-* attributes
await page.locator('[data-action="save"]').click();
await page.locator('[data-action="delete"]').click();
await page.locator('[data-status="active"]').count();
await page.locator('[data-user-id="456"]').click();
await page.locator('[data-row-id="789"]').click();
await page.locator('[data-category="electronics"]').count();

// Combine multiple data attributes
await page.locator('[data-action="delete"][data-id="123"]').click();
```

---

## Combining Selectors

### Multiple Conditions (AND)

```html
<input type="email" class="form-input" name="email" required />
```

```typescript
// All conditions must match (no space between selectors)
await page.locator('input[type="email"]').fill('test@test.com');
await page.locator('input.form-input').fill('text');
await page.locator('input[type="email"].form-input').fill('test@test.com');
await page.locator('input[type="email"][name="email"]').fill('test@test.com');
await page.locator('input[type="email"][required]').fill('test@test.com');

// Tag + Class + Attribute
await page.locator('input.form-input[type="email"]').fill('test@test.com');
```

---

### Multiple Options (OR)

```html
<button class="submit">Submit</button>
<button class="save">Save</button>
<input type="submit" value="Send" />
```

```typescript
// Any of these selectors (comma-separated)
await page.locator('.submit, .save, [type="submit"]').click();
await page.locator('button.submit, button.save').click();
await page.locator('#login-btn, #signin-btn, .auth-button').click();
```

---

### Descendant Selectors (Nested Elements)

```html
<div class="card">
  <h2 class="title">Product Name</h2>
  <p class="description">Product description here</p>
  <div class="footer">
    <span class="price">$99.99</span>
    <button class="add-to-cart">Add to Cart</button>
  </div>
</div>
```

```typescript
// Descendant (space) - Any level deep
await page.locator('.card .title').textContent();           // h2 inside .card
await page.locator('.card .price').textContent();           // span inside .card (any depth)
await page.locator('.card button').click();                 // Any button inside .card
await page.locator('.card .footer .price').textContent();   // More specific path

// Direct child (>) - Only immediate children
await page.locator('.card > .title').textContent();         // Direct child only
await page.locator('.footer > .price').textContent();       // Direct child of footer
await page.locator('.card > .footer > button').click();     // Specific path
```

---

### Sibling Selectors

```html
<label for="email">Email</label>
<input id="email" type="email" />
<span class="error">Invalid email</span>

<h2>Section Title</h2>
<p>First paragraph</p>
<p>Second paragraph</p>
```

```typescript
// Adjacent sibling (+) - Immediately following
await page.locator('label + input').fill('text');     // Input right after label
await page.locator('h2 + p').textContent();           // First p after h2

// General sibling (~) - Any following sibling
await page.locator('h2 ~ p').count();                 // All p siblings after h2
await page.locator('input ~ .error').textContent();   // Error span after input
```

---

## Pseudo-Classes

### Position-Based

```html
<ul class="menu">
  <li>First Item</li>
  <li>Second Item</li>
  <li>Third Item</li>
  <li>Fourth Item</li>
  <li>Fifth Item</li>
</ul>
```

```typescript
// :first-child - First element
await page.locator('.menu li:first-child').click();

// :last-child - Last element
await page.locator('.menu li:last-child').click();

// :nth-child(n) - Specific position (1-indexed)
await page.locator('.menu li:nth-child(1)').click();  // First
await page.locator('.menu li:nth-child(2)').click();  // Second
await page.locator('.menu li:nth-child(3)').click();  // Third

// :nth-child(odd) / :nth-child(even)
await page.locator('.menu li:nth-child(odd)').count();   // 1st, 3rd, 5th
await page.locator('.menu li:nth-child(even)').count();  // 2nd, 4th

// :nth-child(formula) - Pattern matching
await page.locator('.menu li:nth-child(2n)').count();    // Every 2nd (2, 4, 6...)
await page.locator('.menu li:nth-child(3n)').count();    // Every 3rd (3, 6, 9...)
await page.locator('.menu li:nth-child(2n+1)').count();  // Odd (1, 3, 5...)

// :nth-last-child(n) - From the end
await page.locator('.menu li:nth-last-child(1)').click();  // Last item
await page.locator('.menu li:nth-last-child(2)').click();  // Second to last
```

---

### Type-Based

```html
<div class="content">
  <h1>Title</h1>
  <p>First paragraph</p>
  <span>Some text</span>
  <p>Second paragraph</p>
  <p>Third paragraph</p>
</div>
```

```typescript
// :first-of-type - First of its tag type
await page.locator('.content p:first-of-type').textContent();  // "First paragraph"

// :last-of-type - Last of its tag type
await page.locator('.content p:last-of-type').textContent();   // "Third paragraph"

// :nth-of-type(n) - Nth of its type
await page.locator('.content p:nth-of-type(2)').textContent(); // "Second paragraph"

// :only-child - Only child of parent
await page.locator('div:only-child').count();

// :only-of-type - Only one of its type in parent
await page.locator('.content h1:only-of-type').textContent();
```

---

### State-Based

```html
<input type="text" disabled />
<input type="text" />
<input type="checkbox" checked />
<input type="checkbox" />
<input type="text" readonly value="Can't edit" />
<a href="#">Link</a>
<a>No href</a>
```

```typescript
// :disabled / :enabled
await page.locator('input:disabled').count();
await page.locator('input:enabled').fill('text');

// :checked
await page.locator('input:checked').count();

// :not() - Negation
await page.locator('input:not([disabled])').fill('text');
await page.locator('input:not(:checked)').check();
await page.locator('button:not(.secondary)').click();

// :empty - Elements with no children
await page.locator('div:empty').count();

// :has() - Contains specific child (Playwright extension)
await page.locator('div:has(button)').click();
await page.locator('tr:has(td:text("Active"))').click();
```

---

### :has() Selector (Powerful!)

```html
<div class="card">
  <h3>Product A</h3>
  <span class="badge sale">SALE</span>
  <button>Buy</button>
</div>
<div class="card">
  <h3>Product B</h3>
  <button>Buy</button>
</div>
<div class="card">
  <h3>Product C</h3>
  <span class="badge new">NEW</span>
  <button>Buy</button>
</div>
```

```typescript
// Find parent that contains specific child
await page.locator('.card:has(.badge.sale)').click();     // Card with SALE badge
await page.locator('.card:has(.badge.new)').click();      // Card with NEW badge
await page.locator('.card:has(h3:text("Product A"))').click();  // Card with specific title

// Combine with other selectors
await page.locator('.card:has(.badge)').count();          // All cards with any badge
await page.locator('.card:not(:has(.badge))').count();    // Cards without badge
```

---

## Filtering Locators

### Filter by Text Content

```html
<ul class="products">
  <li>
    <span class="name">Laptop Pro</span>
    <span class="price">$999</span>
    <button>Add to Cart</button>
  </li>
  <li>
    <span class="name">Wireless Mouse</span>
    <span class="price">$29</span>
    <button>Add to Cart</button>
  </li>
  <li>
    <span class="name">USB Cable</span>
    <span class="price">$9</span>
    <button>Add to Cart</button>
  </li>
</ul>
```

```typescript
// filter({ hasText: }) - Contains text anywhere inside
await page.locator('li').filter({ hasText: 'Laptop Pro' }).locator('button').click();
await page.locator('li').filter({ hasText: '$999' }).locator('button').click();

// Regex for flexible matching
await page.locator('li').filter({ hasText: /laptop/i }).locator('button').click();
await page.locator('li').filter({ hasText: /\$\d+/ }).count();  // Has any price

// filter({ hasNotText: }) - Does NOT contain text
await page.locator('li').filter({ hasNotText: 'Laptop' }).count();  // 2 items
```

---

### Filter by Child Element

```html
<div class="card">
  <h3>Product A</h3>
  <span class="stock in-stock">In Stock</span>
  <button>Buy Now</button>
</div>
<div class="card">
  <h3>Product B</h3>
  <span class="stock out-of-stock">Out of Stock</span>
  <button disabled>Buy Now</button>
</div>
<div class="card">
  <h3>Product C</h3>
  <span class="stock in-stock">In Stock</span>
  <button>Buy Now</button>
</div>
```

```typescript
// filter({ has: }) - Contains specific locator
await page.locator('.card')
  .filter({ has: page.locator('.in-stock') })
  .count();  // 2 cards

await page.locator('.card')
  .filter({ has: page.locator('.out-of-stock') })
  .locator('h3')
  .textContent();  // "Product B"

// Combine has with text
await page.locator('.card')
  .filter({ has: page.locator('.in-stock') })
  .filter({ hasText: 'Product A' })
  .locator('button')
  .click();

// filter({ hasNot: }) - Does NOT contain locator
await page.locator('.card')
  .filter({ hasNot: page.locator('.out-of-stock') })
  .count();  // 2 cards

await page.locator('.card')
  .filter({ hasNot: page.locator('button[disabled]') })
  .count();  // Cards with enabled buttons
```

---

### Filter by Visibility

```html
<button class="action" style="display: none;">Hidden Button</button>
<button class="action">Visible Button</button>
<button class="action" style="visibility: hidden;">Invisible Button</button>
```

```typescript
// Filter to only visible elements
await page.locator('.action').filter({ visible: true }).click();

// This is equivalent to:
await page.locator('.action').locator('visible=true').click();

// Count visible vs all
const allButtons = await page.locator('.action').count();      // 3
const visibleButtons = await page.locator('.action').filter({ visible: true }).count();  // 1
```

---

### Chaining Multiple Filters

```html
<table>
  <tr data-status="active">
    <td>John</td>
    <td>Admin</td>
    <td><button>Edit</button><button>Delete</button></td>
  </tr>
  <tr data-status="inactive">
    <td>Jane</td>
    <td>User</td>
    <td><button>Edit</button><button>Delete</button></td>
  </tr>
  <tr data-status="active">
    <td>Bob</td>
    <td>User</td>
    <td><button>Edit</button><button>Delete</button></td>
  </tr>
</table>
```

```typescript
// Chain multiple filters for precision
await page.locator('tr')
  .filter({ hasText: 'Admin' })
  .filter({ hasText: 'John' })
  .locator('button', { hasText: 'Edit' })
  .click();

// Complex filter: Active users who are not Admins
await page.locator('tr[data-status="active"]')
  .filter({ hasNotText: 'Admin' })
  .count();  // 1 (Bob)
```

---

## Chaining Locators

### Narrowing Down with .locator()

```html
<section class="products">
  <div class="product">
    <h3>Laptop</h3>
    <p class="price">$999</p>
    <button class="buy">Buy Now</button>
  </div>
  <div class="product">
    <h3>Phone</h3>
    <p class="price">$699</p>
    <button class="buy">Buy Now</button>
  </div>
</section>
```

```typescript
// Chain locators to narrow down
const productsSection = page.locator('.products');
const laptopCard = productsSection.locator('.product').filter({ hasText: 'Laptop' });
const buyButton = laptopCard.locator('.buy');

await buyButton.click();

// Or in one chain
await page
  .locator('.products')
  .locator('.product')
  .filter({ hasText: 'Laptop' })
  .locator('.buy')
  .click();
```

---

### Using .first(), .last(), .nth()

```html
<ul class="items">
  <li>First</li>
  <li>Second</li>
  <li>Third</li>
  <li>Fourth</li>
  <li>Fifth</li>
</ul>
```

```typescript
// Position-based selection
await page.locator('.items li').first().click();      // First item
await page.locator('.items li').last().click();       // Last item
await page.locator('.items li').nth(0).click();       // First (0-indexed)
await page.locator('.items li').nth(2).click();       // Third item
await page.locator('.items li').nth(-1).click();      // Last (negative index)
await page.locator('.items li').nth(-2).click();      // Second to last

// ⚠️ Warning: Position-based selectors are fragile!
// Prefer filtering by content when possible
await page.locator('.items li').filter({ hasText: 'Third' }).click();  // Better
```

---

### Using .and() for Multiple Conditions

```html
<button class="btn primary large">Large Primary</button>
<button class="btn primary small">Small Primary</button>
<button class="btn secondary large">Large Secondary</button>
```

```typescript
// .and() - Element must match BOTH locators
await page
  .locator('.btn.primary')
  .and(page.locator('.large'))
  .click();  // Large Primary button

await page
  .locator('button')
  .and(page.locator('[class*="primary"]'))
  .and(page.locator('[class*="small"]'))
  .click();  // Small Primary button
```

---

### Using .or() for Alternatives

```html
<button id="submit">Submit</button>
<!-- OR on some pages: -->
<button id="send">Send</button>
```

```typescript
// .or() - Match either locator
const submitButton = page.locator('#submit').or(page.locator('#send'));
await submitButton.click();

// Useful for handling different page states
const loginOrDashboard = page.locator('.login-form').or(page.locator('.dashboard'));
await expect(loginOrDashboard).toBeVisible();
```

---

## XPath Selectors

### When to Use XPath

```
Use XPath when:
✓ Need to traverse UP the DOM (parent, ancestor)
✓ Need complex text matching
✓ Need to select by partial text content
✓ Working with XML-like structures

Avoid XPath when:
✗ CSS can do the job (CSS is faster)
✗ Simple element selection
✗ Playwright's built-in locators work
```

### Basic XPath Syntax

```html
<div class="container">
  <ul class="list">
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
  </ul>
</div>
```

```typescript
// XPath with // (anywhere in document)
await page.locator('//button').click();
await page.locator('//div[@class="container"]').click();
await page.locator('//li').first().click();

// XPath with explicit prefix
await page.locator('xpath=//button').click();

// Absolute path (not recommended - fragile)
await page.locator('/html/body/div/ul/li').first().click();
```

---

### XPath Attribute Selectors

```html
<input type="email" name="email" placeholder="Enter email" />
<button data-testid="submit" class="btn primary">Submit</button>
```

```typescript
// Attribute exact match
await page.locator('//input[@type="email"]').fill('test@test.com');
await page.locator('//input[@name="email"]').fill('test@test.com');
await page.locator('//button[@data-testid="submit"]').click();

// Multiple attributes
await page.locator('//input[@type="email"][@name="email"]').fill('test@test.com');

// Attribute contains
await page.locator('//button[contains(@class, "primary")]').click();
await page.locator('//input[contains(@placeholder, "email")]').fill('test@test.com');

// Attribute starts with
await page.locator('//button[starts-with(@data-testid, "submit")]').click();
```

---

### XPath Text Selectors

```html
<button>Submit Form</button>
<button>Cancel</button>
<span>Total: $99.99</span>
<p>Welcome to our website. Please login to continue.</p>
```

```typescript
// Exact text match
await page.locator('//button[text()="Submit Form"]').click();
await page.locator('//button[text()="Cancel"]').click();

// Contains text (partial match)
await page.locator('//button[contains(text(), "Submit")]').click();
await page.locator('//span[contains(text(), "Total")]').textContent();
await page.locator('//p[contains(text(), "Welcome")]').textContent();

// Normalize space (handles extra whitespace)
await page.locator('//button[normalize-space()="Submit Form"]').click();

// Case-insensitive (using translate)
await page.locator(
  '//button[contains(translate(text(), "ABCDEFGHIJKLMNOPQRSTUVWXYZ", "abcdefghijklmnopqrstuvwxyz"), "submit")]'
).click();
```

---

### XPath Axes (Navigating the DOM)

```html
<div class="form-group">
  <label>Email</label>
  <input type="email" />
  <span class="error">Invalid email</span>
</div>
```

```typescript
// parent:: - Go to parent
await page.locator('//input[@type="email"]/parent::div').click();

// ancestor:: - Any ancestor
await page.locator('//input[@type="email"]/ancestor::form').count();

// following-sibling:: - Next siblings
await page.locator('//label[text()="Email"]/following-sibling::input').fill('test@test.com');
await page.locator('//input[@type="email"]/following-sibling::span').textContent();

// preceding-sibling:: - Previous siblings
await page.locator('//input[@type="email"]/preceding-sibling::label').textContent();

// child:: - Direct children
await page.locator('//div[@class="form-group"]/child::input').fill('text');

// descendant:: - All descendants
await page.locator('//form/descendant::button').click();
```

---

### XPath Position

```html
<ul>
  <li>First</li>
  <li>Second</li>
  <li>Third</li>
</ul>
```

```typescript
// Position (1-indexed in XPath!)
await page.locator('//ul/li[1]').click();      // First
await page.locator('//ul/li[2]').click();      // Second
await page.locator('//ul/li[last()]').click(); // Last
await page.locator('//ul/li[last()-1]').click(); // Second to last

// Position with conditions
await page.locator('(//button[@type="submit"])[1]').click();  // First submit button
await page.locator('(//button[@type="submit"])[2]').click();  // Second submit button
```

---

## Text-Based Locators

### Playwright's Text Selector

```html
<button>Submit</button>
<button>Submit Form</button>
<a>Click here to continue</a>
<span>Welcome, John!</span>
```

```typescript
// text= selector (substring match by default)
await page.locator('text=Submit').click();           // Matches both buttons!
await page.locator('text=Submit Form').click();      // More specific
await page.locator('text=Click here').click();       // Partial match

// Exact match with quotes
await page.locator('text="Submit"').click();         // Only exact "Submit"
await page.locator("text='Submit Form'").click();    // Exact match

// Case-insensitive (default)
await page.locator('text=submit').click();           // Matches "Submit"
await page.locator('text=SUBMIT').click();           // Matches "Submit"

// Case-sensitive with /s flag
await page.locator('text="Submit"/s').click();       // Case-sensitive

// Regex
await page.locator('text=/submit/i').click();        // Regex, case-insensitive
await page.locator('text=/Welcome, \\w+!/').textContent();  // Pattern match
```

---

### Combining Text with Other Selectors

```html
<div class="card">
  <button>Add to Cart</button>
</div>
<div class="modal">
  <button>Add to Cart</button>
</div>
```

```typescript
// Scope text selector
await page.locator('.card >> text=Add to Cart').click();
await page.locator('.modal >> text=Add to Cart').click();

// Or use filter
await page.locator('.card').locator('text=Add to Cart').click();

// CSS + has-text
await page.locator('button:has-text("Add to Cart")').click();
await page.locator('.card button:has-text("Add")').click();

// :text() pseudo-class (exact match)
await page.locator(':text("Submit")').click();

// :text-is() (exact match, case-sensitive)
await page.locator(':text-is("Submit")').click();

// :text-matches() (regex)
await page.locator(':text-matches("\\\\d+ items")').click();
```

---

## Advanced Combinations

### Complex Real-World Selectors

#### Scenario 1: Table Row by Multiple Criteria

```html
<table>
  <tr data-id="1" class="user-row">
    <td class="name">John Smith</td>
    <td class="email">john@test.com</td>
    <td class="role">Admin</td>
    <td class="status"><span class="badge active">Active</span></td>
    <td class="actions">
      <button class="edit">Edit</button>
      <button class="delete">Delete</button>
    </td>
  </tr>
  <tr data-id="2" class="user-row">
    <td class="name">Jane Doe</td>
    <td class="email">jane@test.com</td>
    <td class="role">User</td>
    <td class="status"><span class="badge inactive">Inactive</span></td>
    <td class="actions">
      <button class="edit">Edit</button>
      <button class="delete">Delete</button>
    </td>
  </tr>
</table>
```

```typescript
// Method 1: Filter by text content
await page.locator('tr.user-row')
  .filter({ hasText: 'John Smith' })
  .locator('button.edit')
  .click();

// Method 2: Filter by child element
await page.locator('tr.user-row')
  .filter({ has: page.locator('.badge.active') })
  .filter({ hasText: 'Admin' })
  .locator('button.delete')
  .click();

// Method 3: Using data attribute + descendant
await page.locator('tr[data-id="1"] button.edit').click();

// Method 4: XPath for complex traversal
await page.locator('//td[text()="john@test.com"]/ancestor::tr//button[@class="edit"]').click();

// Method 5: CSS :has() selector
await page.locator('tr:has(td.email:text("john@test.com")) button.edit').click();
```

---

#### Scenario 2: Dropdown/Select Alternatives

```html
<!-- Custom dropdown (not native select) -->
<div class="dropdown">
  <button class="dropdown-toggle">Select Option</button>
  <ul class="dropdown-menu" style="display: none;">
    <li data-value="opt1">Option 1</li>
    <li data-value="opt2">Option 2</li>
    <li data-value="opt3">Option 3</li>
  </ul>
</div>
```

```typescript
// Open dropdown and select option
await page.locator('.dropdown .dropdown-toggle').click();
await page.locator('.dropdown-menu li[data-value="opt2"]').click();

// Or by text
await page.locator('.dropdown .dropdown-toggle').click();
await page.locator('.dropdown-menu li').filter({ hasText: 'Option 2' }).click();

// With waiting for menu to be visible
await page.locator('.dropdown-toggle').click();
await page.locator('.dropdown-menu').waitFor({ state: 'visible' });
await page.locator('.dropdown-menu li:has-text("Option 2")').click();
```

---

#### Scenario 3: Modal Dialog Content

```html
<div class="modal-overlay">
  <div class="modal" role="dialog">
    <div class="modal-header">
      <h2>Confirm Delete</h2>
      <button class="close">&times;</button>
    </div>
    <div class="modal-body">
      <p>Are you sure you want to delete "Document.pdf"?</p>
    </div>
    <div class="modal-footer">
      <button class="btn-cancel">Cancel</button>
      <button class="btn-confirm btn-danger">Delete</button>
    </div>
  </div>
</div>
```

```typescript
// Scope all actions to the modal
const modal = page.locator('.modal[role="dialog"]');

// Get modal title
await expect(modal.locator('.modal-header h2')).toHaveText('Confirm Delete');

// Click confirm button
await modal.locator('.modal-footer .btn-confirm').click();

// Or using filter to find specific modal
await page.locator('.modal')
  .filter({ hasText: 'Confirm Delete' })
  .locator('.btn-confirm')
  .click();
```

---

#### Scenario 4: Nested Components

```html
<div class="dashboard">
  <aside class="sidebar">
    <nav class="nav-menu">
      <a href="/home" class="nav-item">Home</a>
      <a href="/products" class="nav-item active">Products</a>
      <a href="/orders" class="nav-item">Orders</a>
    </nav>
  </aside>
  <main class="content">
    <nav class="breadcrumb">
      <a href="/home">Home</a>
      <a href="/products">Products</a>
    </nav>
    <div class="product-grid">
      <!-- Products here -->
    </div>
  </main>
</div>
```

```typescript
// Be specific about which nav you mean
await page.locator('.sidebar .nav-menu .nav-item').filter({ hasText: 'Orders' }).click();
await page.locator('.breadcrumb a').filter({ hasText: 'Home' }).click();

// Or use more specific selectors
await page.locator('aside.sidebar a[href="/orders"]').click();
await page.locator('nav.breadcrumb a[href="/home"]').click();

// Find active nav item
await page.locator('.nav-menu .nav-item.active').textContent();
```

---

#### Scenario 5: Form with Dynamic IDs

```html
<!-- IDs are auto-generated and change each load -->
<form>
  <div class="form-field">
    <label for="input_a1b2c3">Email</label>
    <input id="input_a1b2c3" type="email" />
  </div>
  <div class="form-field">
    <label for="input_d4e5f6">Password</label>
    <input id="input_d4e5f6" type="password" />
  </div>
  <button type="submit">Login</button>
</form>
```

```typescript
// ✅ GOOD: Don't rely on dynamic IDs

// Method 1: Use label relationship (best)
await page.getByLabel('Email').fill('test@test.com');
await page.getByLabel('Password').fill('password');

// Method 2: Use structural relationship
await page.locator('.form-field:has(label:text("Email")) input').fill('test@test.com');

// Method 3: Use type attribute
await page.locator('input[type="email"]').fill('test@test.com');
await page.locator('input[type="password"]').fill('password');

// Method 4: XPath with label text
await page.locator('//label[text()="Email"]/following-sibling::input').fill('test@test.com');
```

---

## Real-World Scenarios

### Scenario A: E-commerce Product Card

```html
<div class="product-card" data-product-id="12345">
  <img src="laptop.jpg" alt="Laptop Pro 15" />
  <div class="product-info">
    <h3 class="product-name">Laptop Pro 15</h3>
    <div class="product-rating">
      <span class="stars">★★★★☆</span>
      <span class="count">(128 reviews)</span>
    </div>
    <p class="product-price">
      <span class="original">$1,299.00</span>
      <span class="sale">$999.00</span>
    </p>
    <div class="product-actions">
      <button class="btn-wishlist" aria-label="Add to wishlist">♡</button>
      <button class="btn-cart">Add to Cart</button>
    </div>
  </div>
</div>
```

```typescript
// Create a helper function for product cards
function getProductCard(page: Page, productName: string) {
  return page.locator('.product-card').filter({ hasText: productName });
}

// Usage examples
const laptop = getProductCard(page, 'Laptop Pro 15');

// Get sale price
const salePrice = await laptop.locator('.product-price .sale').textContent();

// Add to cart
await laptop.locator('.btn-cart').click();

// Add to wishlist
await laptop.locator('.btn-wishlist').click();

// Get review count
const reviewCount = await laptop.locator('.product-rating .count').textContent();

// Check if on sale (has .sale element)
const isOnSale = await laptop.locator('.product-price .sale').isVisible();

// Get by data attribute
await page.locator('.product-card[data-product-id="12345"] .btn-cart').click();
```

---

### Scenario B: Data Table with Sorting and Filtering

```html
<div class="data-table-container">
  <div class="table-controls">
    <input type="search" placeholder="Search..." class="search-input" />
    <select class="status-filter">
      <option value="">All Status</option>
      <option value="active">Active</option>
      <option value="inactive">Inactive</option>
    </select>
  </div>
  <table class="data-table">
    <thead>
      <tr>
        <th data-sort="name" class="sortable">Name ▼</th>
        <th data-sort="email" class="sortable">Email</th>
        <th data-sort="status" class="sortable">Status</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr data-row-id="1">
        <td class="cell-name">Alice Johnson</td>
        <td class="cell-email">alice@example.com</td>
        <td class="cell-status"><span class="status active">Active</span></td>
        <td class="cell-actions">
          <button class="btn-view" title="View">👁</button>
          <button class="btn-edit" title="Edit">✏</button>
          <button class="btn-delete" title="Delete">🗑</button>
        </td>
      </tr>
      <!-- More rows -->
    </tbody>
  </table>
  <div class="pagination">
    <button class="prev" disabled>Previous</button>
    <span class="page-info">Page 1 of 10</span>
    <button class="next">Next</button>
  </div>
</div>
```

```typescript
// Table helper class using locators
class DataTableHelper {
  constructor(private page: Page) {}

  get table() {
    return this.page.locator('.data-table');
  }

  get searchInput() {
    return this.page.locator('.search-input');
  }

  get statusFilter() {
    return this.page.locator('.status-filter');
  }

  // Get row by any cell content
  getRowByText(text: string) {
    return this.table.locator('tbody tr').filter({ hasText: text });
  }

  // Get row by specific column
  getRowByEmail(email: string) {
    return this.table.locator(`tbody tr:has(.cell-email:text("${email}"))`);
  }

  // Sort by column
  async sortBy(columnName: string) {
    await this.table.locator(`th[data-sort="${columnName}"]`).click();
  }

  // Filter by status
  async filterByStatus(status: string) {
    await this.statusFilter.selectOption(status);
  }

  // Search
  async search(query: string) {
    await this.searchInput.fill(query);
    await this.searchInput.press('Enter');
  }

  // Actions on specific row
  async viewRow(identifier: string) {
    await this.getRowByText(identifier).locator('.btn-view').click();
  }

  async editRow(identifier: string) {
    await this.getRowByText(identifier).locator('.btn-edit').click();
  }

  async deleteRow(identifier: string) {
    await this.getRowByText(identifier).locator('.btn-delete').click();
  }

  // Pagination
  async nextPage() {
    await this.page.locator('.pagination .next').click();
  }

  async prevPage() {
    await this.page.locator('.pagination .prev').click();
  }

  // Get current page info
  async getPageInfo() {
    return await this.page.locator('.pagination .page-info').textContent();
  }

  // Count visible rows
  async getVisibleRowCount() {
    return await this.table.locator('tbody tr').count();
  }
}

// Usage
const dataTable = new DataTableHelper(page);

await dataTable.search('alice');
await dataTable.filterByStatus('active');
await dataTable.sortBy('name');
await dataTable.editRow('alice@example.com');
```

---

### Scenario C: Multi-Step Form Wizard

```html
<div class="form-wizard">
  <div class="wizard-steps">
    <div class="step completed" data-step="1">Personal Info</div>
    <div class="step active" data-step="2">Address</div>
    <div class="step" data-step="3">Payment</div>
    <div class="step" data-step="4">Review</div>
  </div>
  
  <div class="wizard-content">
    <!-- Step 2: Address (currently visible) -->
    <div class="step-content" data-step="2">
      <div class="form-row">
        <label for="street">Street Address</label>
        <input type="text" id="street" name="street" />
      </div>
      <div class="form-row">
        <label for="city">City</label>
        <input type="text" id="city" name="city" />
      </div>
      <div class="form-row">
        <label for="country">Country</label>
        <select id="country" name="country">
          <option value="">Select...</option>
          <option value="US">United States</option>
          <option value="UK">United Kingdom</option>
        </select>
      </div>
    </div>
  </div>
  
  <div class="wizard-actions">
    <button class="btn-back">Back</button>
    <button class="btn-next">Next</button>
  </div>
</div>
```

```typescript
// Wizard helper class
class FormWizardHelper {
  constructor(private page: Page) {}

  get wizard() {
    return this.page.locator('.form-wizard');
  }

  // Get current step number
  async getCurrentStep(): Promise<number> {
    const activeStep = this.wizard.locator('.wizard-steps .step.active');
    const stepNum = await activeStep.getAttribute('data-step');
    return parseInt(stepNum || '1', 10);
  }

  // Get current step content
  getCurrentStepContent() {
    return this.wizard.locator('.step-content:visible');
  }

  // Navigate
  async clickNext() {
    await this.wizard.locator('.btn-next').click();
  }

  async clickBack() {
    await this.wizard.locator('.btn-back').click();
  }

  // Check if step is completed
  async isStepCompleted(stepNumber: number): Promise<boolean> {
    const step = this.wizard.locator(`.wizard-steps .step[data-step="${stepNumber}"]`);
    return await step.locator('.completed').count() > 0;
  }

  // Fill current step fields
  async fillField(label: string, value: string) {
    const stepContent = this.getCurrentStepContent();
    await stepContent.getByLabel(label).fill(value);
  }

  async selectField(label: string, value: string) {
    const stepContent = this.getCurrentStepContent();
    await stepContent.getByLabel(label).selectOption(value);
  }
}

// Usage
const wizard = new FormWizardHelper(page);

// Fill address step
await wizard.fillField('Street Address', '123 Main St');
await wizard.fillField('City', 'New York');
await wizard.selectField('Country', 'US');
await wizard.clickNext();

// Verify we moved to next step
expect(await wizard.getCurrentStep()).toBe(3);
```

---

## Quick Reference Cheat Sheet

### CSS Selector Syntax

```
┌─────────────────────────────────────────────────────────────────┐
│                    CSS SELECTOR CHEAT SHEET                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BASIC SELECTORS                                                │
│  ───────────────                                                │
│  #id                    Element with id="id"                    │
│  .class                 Element with class="class"              │
│  element                Element by tag name                     │
│  element.class          Tag with class                          │
│  element#id             Tag with id                             │
│                                                                 │
│  ATTRIBUTE SELECTORS                                            │
│  ──────────────────                                             │
│  [attr]                 Has attribute                           │
│  [attr="value"]         Exact match                             │
│  [attr^="value"]        Starts with                             │
│  [attr$="value"]        Ends with                               │
│  [attr*="value"]        Contains                                │
│                                                                 │
│  COMBINATORS                                                    │
│  ───────────                                                    │
│  A B                    B descendant of A (any level)           │
│  A > B                  B direct child of A                     │
│  A + B                  B immediately after A                   │
│  A ~ B                  B sibling after A                       │
│                                                                 │
│  PSEUDO-CLASSES                                                 │
│  ──────────────                                                 │
│  :first-child           First child of parent                   │
│  :last-child            Last child of parent                    │
│  :nth-child(n)          Nth child (1-indexed)                   │
│  :nth-child(odd/even)   Odd or even children                    │
│  :first-of-type         First of its type                       │
│  :last-of-type          Last of its type                        │
│  :not(selector)         Negation                                │
│  :has(selector)         Contains matching element               │
│  :disabled              Disabled elements                       │
│  :enabled               Enabled elements                        │
│  :checked               Checked inputs                          │
│                                                                 │
│  PLAYWRIGHT SPECIFIC                                            │
│  ──────────────────                                             │
│  :has-text("text")      Contains text                           │
│  :text("text")          Has exact text                          │
│  :text-is("text")       Exact text, case-sensitive              │
│  :visible               Visible elements only                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Locator Methods Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                 LOCATOR METHODS CHEAT SHEET                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FILTERING                                                      │
│  ─────────                                                      │
│  .filter({ hasText: 'text' })      Contains text                │
│  .filter({ hasText: /regex/ })     Matches regex                │
│  .filter({ hasNotText: 'text' })   Does not contain text        │
│  .filter({ has: locator })         Contains element             │
│  .filter({ hasNot: locator })      Does not contain element     │
│  .filter({ visible: true })        Only visible elements        │
│                                                                 │
│  POSITION                                                       │
│  ────────                                                       │
│  .first()                          First matching element       │
│  .last()                           Last matching element        │
│  .nth(index)                       Element at index (0-based)   │
│  .nth(-1)                          Last (negative index)        │
│                                                                 │
│  COMBINING                                                      │
│  ─────────                                                      │
│  .and(locator)                     Must match both              │
│  .or(locator)                      Match either                 │
│  .locator(selector)                Narrow down further          │
│                                                                 │
│  CHAINING                                                       │
│  ────────                                                       │
│  page.locator('A').locator('B')    B inside A                   │
│  page.locator('A >> B')            B inside A (shorthand)       │
│  page.locator('A B')               B descendant of A            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Common Patterns Quick Reference

```typescript
// ===== BY ID =====
page.locator('#element-id')

// ===== BY CLASS =====
page.locator('.class-name')
page.locator('.class1.class2')           // Multiple classes

// ===== BY TAG =====
page.locator('button')
page.locator('input')

// ===== BY ATTRIBUTE =====
page.locator('[name="email"]')
page.locator('[type="submit"]')
page.locator('[data-testid="submit"]')
page.locator('[href^="https://"]')       // Starts with
page.locator('[class*="primary"]')       // Contains

// ===== COMBINATIONS =====
page.locator('button.primary')           // Tag + class
page.locator('input#email')              // Tag + id
page.locator('button[type="submit"]')    // Tag + attribute
page.locator('.card .title')             // Descendant
page.locator('.card > .title')           // Direct child

// ===== FILTERING =====
page.locator('.item').filter({ hasText: 'Product' })
page.locator('.card').filter({ has: page.locator('.badge') })
page.locator('button').filter({ hasNotText: 'Cancel' })

// ===== POSITION =====
page.locator('.item').first()
page.locator('.item').last()
page.locator('.item').nth(2)

// ===== PSEUDO-CLASSES =====
page.locator('li:first-child')
page.locator('li:last-child')
page.locator('li:nth-child(3)')
page.locator('button:not([disabled])')
page.locator('.card:has(.sale-badge)')

// ===== TEXT-BASED =====
page.locator('text=Submit')
page.locator('button:has-text("Submit")')
page.locator(':text("Exact Text")')

// ===== XPATH (when needed) =====
page.locator('//button[text()="Submit"]')
page.locator('//input[@type="email"]/ancestor::form')
page.locator('//td[text()="John"]/following-sibling::td')
```

---

## Decision Flowchart: Which Selector to Use?

```
START: Need to locate an element
            │
            ▼
    ┌───────────────────┐
    │ Can you use       │
    │ getByRole()?      │──── YES ──→ Use getByRole() ✅
    └───────────────────┘
            │ NO
            ▼
    ┌───────────────────┐
    │ Is it a form      │
    │ field with label? │──── YES ──→ Use getByLabel() ✅
    └───────────────────┘
            │ NO
            ▼
    ┌───────────────────┐
    │ Does it have      │
    │ placeholder text? │──── YES ──→ Use getByPlaceholder() ✅
    └───────────────────┘
            │ NO
            ▼
    ┌───────────────────┐
    │ Can you identify  │
    │ by visible text?  │──── YES ──→ Use getByText() ✅
    └───────────────────┘
            │ NO
            ▼
    ┌───────────────────┐
    │ Does it have      │
    │ data-testid?      │──── YES ──→ Use getByTestId() ✅
    └───────────────────┘
            │ NO
            ▼
    ┌───────────────────┐
    │ Does it have a    │
    │ stable ID?        │──── YES ──→ page.locator('#id') ⚠️
    └───────────────────┘
            │ NO
            ▼
    ┌───────────────────┐
    │ Does it have a    │
    │ stable class?     │──── YES ──→ page.locator('.class') ⚠️
    └───────────────────┘
            │ NO
            ▼
    ┌───────────────────┐
    │ Can you use       │
    │ data-* attribute? │──── YES ──→ page.locator('[data-*]') ⚠️
    └───────────────────┘
            │ NO
            ▼
    ┌───────────────────┐
    │ Need to traverse  │
    │ UP the DOM?       │──── YES ──→ Use XPath ⚠️⚠️
    └───────────────────┘
            │ NO
            ▼
    Use CSS combination with
    filter() and chaining ⚠️

Legend:
✅ = Recommended
⚠️ = Use with caution
⚠️⚠️ = Last resort
```

---

*This reference guide covers the full spectrum of `page.locator()` capabilities. Remember: always try semantic locators first (getByRole, getByLabel, etc.) before falling back to CSS/XPath selectors!*
