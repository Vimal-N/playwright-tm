# Playwright UI Automation Training Curriculum

## Complete Framework Building Guide for QA Engineers

**Version:** 1.0  
**Target Audience:** Manual testers transitioning to automation & Automation engineers learning framework development  
**Duration:** 10-15 training sessions (2-3 hours each)

---

## Table of Contents

1. [Introduction & Course Overview](#module-1-introduction--course-overview)
2. [Environment Setup & First Test](#module-2-environment-setup--first-test)
3. [Understanding Locators - The Foundation](#module-3-understanding-locators---the-foundation)
4. [Mastering Selectors - Decision Framework](#module-4-mastering-selectors---decision-framework)
5. [Page Object Model (POM) Architecture](#module-5-page-object-model-pom-architecture)
6. [Fixtures & Test Organization](#module-6-fixtures--test-organization)
7. [Authentication Handling](#module-7-authentication-handling)
8. [Configuration & Multi-Browser Testing](#module-8-configuration--multi-browser-testing)
9. [Advanced Patterns & Best Practices](#module-9-advanced-patterns--best-practices)
10. [Building a Complete Framework](#module-10-building-a-complete-framework)
11. [Exercises & Projects](#exercises--hands-on-projects)
12. [Quick Reference Cards](#quick-reference-cards)

---

## Module 1: Introduction & Course Overview

### Learning Objectives
- Understand what Playwright is and why it's superior to other frameworks
- Understand the architecture and how it differs from Selenium
- Know the key features that make tests reliable

### What is Playwright?

Playwright is an end-to-end testing framework developed by Microsoft that enables reliable cross-browser automation. Unlike Selenium, Playwright:

- **Runs out-of-process**: No in-process limitations, runs outside the browser
- **Auto-waits**: Automatically waits for elements to be actionable
- **Has native support** for modern web features (Shadow DOM, iframes, etc.)
- **Provides isolation**: Each test runs in a fresh browser context

### Key Features

| Feature | Benefit |
|---------|---------|
| Auto-waiting | Eliminates flaky tests from timing issues |
| Web-first assertions | Retries until conditions are met |
| Trace viewer | Debug failures with full execution history |
| Parallel execution | Tests run fast by utilizing CPU cores |
| Multiple browsers | Single API for Chromium, Firefox, WebKit |
| Codegen | Generate tests by recording actions |

### Why Playwright Over Other Tools?

```
┌─────────────────────────────────────────────────────────────────┐
│                    PLAYWRIGHT ADVANTAGES                        │
├─────────────────────────────────────────────────────────────────┤
│ ✓ Auto-waiting built into every action                         │
│ ✓ Browser contexts provide complete test isolation              │
│ ✓ Network interception and mocking out of the box              │
│ ✓ Mobile emulation without additional setup                    │
│ ✓ Powerful debugging tools (Trace Viewer, UI Mode)             │
│ ✓ First-class TypeScript support                               │
│ ✓ Active development by Microsoft                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Module 2: Environment Setup & First Test

### Prerequisites
- Node.js (version 20.x, 22.x, or 24.x)
- VS Code with Playwright extension (recommended)
- Basic JavaScript/TypeScript knowledge

### Installation Steps

```bash
# Step 1: Create a new project directory
mkdir playwright-framework
cd playwright-framework

# Step 2: Initialize Playwright
npm init playwright@latest

# When prompted, choose:
# - TypeScript (recommended for larger projects)
# - tests folder name: tests (or e2e)
# - Add GitHub Actions workflow: Yes (for CI/CD)
# - Install Playwright browsers: Yes
```

### What Gets Created

```
playwright-framework/
├── playwright.config.ts    # Test configuration
├── package.json            # Project dependencies
├── tests/
│   └── example.spec.ts     # Sample test file
├── tests-examples/         # Additional examples (can delete)
└── .github/
    └── workflows/          # CI configuration
```

### Your First Test

Create `tests/first-test.spec.ts`:

```typescript
import { test, expect } from '@playwright/test';

// Test suite description
test.describe('My First Test Suite', () => {
  
  // Individual test case
  test('should load the page and verify title', async ({ page }) => {
    // Navigate to a website
    await page.goto('https://playwright.dev');
    
    // Assert the title contains expected text
    await expect(page).toHaveTitle(/Playwright/);
  });

  test('should click a button and verify navigation', async ({ page }) => {
    await page.goto('https://playwright.dev');
    
    // Click on "Get started" link
    await page.getByRole('link', { name: 'Get started' }).click();
    
    // Verify we navigated to the installation page
    await expect(page.getByRole('heading', { name: 'Installation' })).toBeVisible();
  });
});
```

### Running Tests

```bash
# Run all tests
npx playwright test

# Run in headed mode (see browser)
npx playwright test --headed

# Run specific test file
npx playwright test tests/first-test.spec.ts

# Run with UI Mode (interactive)
npx playwright test --ui

# Run on specific browser
npx playwright test --project=chromium

# Show HTML report
npx playwright show-report
```

### Understanding Test Structure

```typescript
import { test, expect } from '@playwright/test';

test.describe('Suite Name', () => {
  // Runs before ALL tests in this describe block
  test.beforeAll(async () => {
    // One-time setup
  });

  // Runs before EACH test
  test.beforeEach(async ({ page }) => {
    // Setup before every test
    await page.goto('https://example.com');
  });

  test('test name', async ({ page }) => {
    // Test code here
  });

  // Runs after EACH test
  test.afterEach(async ({ page }) => {
    // Cleanup after every test
  });

  // Runs after ALL tests
  test.afterAll(async () => {
    // Final cleanup
  });
});
```

---

## Module 3: Understanding Locators - The Foundation

### What are Locators?

Locators are the way Playwright finds elements on a page. They are:
- **Lazy**: Elements are found when action is performed
- **Strict**: Throw error if multiple elements match
- **Auto-waiting**: Wait for element to be actionable

### The Locator Hierarchy (Priority Order)

```
┌─────────────────────────────────────────────────────────────────┐
│           LOCATOR PRIORITY (MOST TO LEAST PREFERRED)           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. getByRole()        ← MOST RECOMMENDED                      │
│     - Reflects how users/assistive tech see the page            │
│     - Most resilient to UI changes                              │
│                                                                 │
│  2. getByLabel()                                                │
│     - Best for form controls                                    │
│                                                                 │
│  3. getByPlaceholder()                                          │
│     - For inputs without labels                                 │
│                                                                 │
│  4. getByText()                                                 │
│     - For non-interactive elements (div, span, p)               │
│                                                                 │
│  5. getByTestId()                                               │
│     - When other methods don't work                             │
│     - Explicit testing contract                                 │
│                                                                 │
│  6. getByAltText()                                              │
│     - For images                                                │
│                                                                 │
│  7. getByTitle()                                                │
│     - For elements with title attribute                         │
│                                                                 │
│  8. CSS/XPath         ← AVOID WHEN POSSIBLE                    │
│     - Fragile, tied to DOM structure                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Built-in Locators Explained

#### 1. getByRole() - The Champion

```typescript
// Buttons
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByRole('button', { name: /submit/i }).click(); // Case insensitive

// Links
await page.getByRole('link', { name: 'Home' }).click();

// Headings
await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible();
await page.getByRole('heading', { level: 1 }); // <h1>

// Form elements
await page.getByRole('textbox', { name: 'Email' }).fill('test@email.com');
await page.getByRole('checkbox', { name: 'Remember me' }).check();
await page.getByRole('radio', { name: 'Option 1' }).click();

// Navigation and lists
await page.getByRole('navigation').getByRole('link', { name: 'About' }).click();
await page.getByRole('list').getByRole('listitem').first();

// Tables
await page.getByRole('table').getByRole('row').filter({ hasText: 'John' });
```

**Common ARIA Roles:**
| Role | HTML Elements |
|------|---------------|
| button | `<button>`, `<input type="button">` |
| checkbox | `<input type="checkbox">` |
| heading | `<h1>` through `<h6>` |
| link | `<a href="...">` |
| listitem | `<li>` |
| textbox | `<input type="text">`, `<textarea>` |
| combobox | `<select>` |
| radio | `<input type="radio">` |
| navigation | `<nav>` |

#### 2. getByLabel() - For Form Controls

```typescript
// Works with <label> elements
// HTML: <label>Email <input type="email" /></label>
await page.getByLabel('Email').fill('user@example.com');

// Works with for/id association
// HTML: <label for="pwd">Password</label><input id="pwd" />
await page.getByLabel('Password').fill('secret');

// Exact match
await page.getByLabel('Email', { exact: true }).fill('test@test.com');
```

#### 3. getByPlaceholder() - For Inputs Without Labels

```typescript
// HTML: <input placeholder="Enter your email" />
await page.getByPlaceholder('Enter your email').fill('test@test.com');

// Case insensitive with regex
await page.getByPlaceholder(/enter your email/i).fill('test@test.com');
```

#### 4. getByText() - For Static Content

```typescript
// Find element containing text
await expect(page.getByText('Welcome, John!')).toBeVisible();

// Exact match
await page.getByText('Submit', { exact: true }).click();

// Regex for partial/flexible matching
await expect(page.getByText(/order #\d+/i)).toBeVisible();
```

**⚠️ Important:** Use getByText for NON-interactive elements. For buttons/links, use getByRole.

#### 5. getByTestId() - Explicit Testing Contract

```typescript
// HTML: <button data-testid="submit-btn">Submit</button>
await page.getByTestId('submit-btn').click();

// Configure custom test ID attribute in playwright.config.ts
// use: { testIdAttribute: 'data-pw' }
```

#### 6. getByAltText() - For Images

```typescript
// HTML: <img alt="Company Logo" src="logo.png" />
await page.getByAltText('Company Logo').click();
```

#### 7. getByTitle() - For Title Attributes

```typescript
// HTML: <span title="Full details">More...</span>
await page.getByTitle('Full details').click();
```

---

## Module 4: Mastering Selectors - Decision Framework

### The Selector Decision Tree

```
Start Here: What element do you need to interact with?
                │
                ▼
┌─────────────────────────────────────────────────────────────┐
│ Is it a form control (button, input, checkbox, etc.)?      │
└─────────────────────────────────────────────────────────────┘
        │                               │
       YES                              NO
        │                               │
        ▼                               ▼
┌───────────────────┐      ┌─────────────────────────────────┐
│ Does it have a    │      │ Is it displaying text content?  │
│ visible label?    │      └─────────────────────────────────┘
└───────────────────┘               │               │
    │           │                  YES              NO
   YES          NO                  │               │
    │           │                   ▼               ▼
    ▼           ▼          ┌──────────────┐  ┌──────────────┐
┌────────┐  ┌────────────┐ │ getByText()  │  │ getByTestId()│
│getByRole│ │getByRole() │ └──────────────┘  │ or CSS/XPath │
│  with   │ │  or        │                   └──────────────┘
│  name   │ │getByPlace- │
└────────┘  │ holder()   │
            └────────────┘
```

### Real-World Selector Examples

#### Scenario 1: Login Form

```html
<form>
  <label for="email">Email Address</label>
  <input type="email" id="email" placeholder="Enter email" />
  
  <label for="password">Password</label>
  <input type="password" id="password" />
  
  <button type="submit">Sign In</button>
</form>
```

```typescript
// ✅ GOOD - Using semantic locators
await page.getByLabel('Email Address').fill('user@test.com');
await page.getByLabel('Password').fill('secret123');
await page.getByRole('button', { name: 'Sign In' }).click();

// ❌ BAD - Fragile CSS selectors
await page.locator('#email').fill('user@test.com');
await page.locator('form > input:nth-child(2)').fill('secret123');
await page.locator('button[type="submit"]').click();
```

#### Scenario 2: Product List

```html
<ul class="products">
  <li>
    <h3>Product A</h3>
    <span class="price">$10.00</span>
    <button>Add to Cart</button>
  </li>
  <li>
    <h3>Product B</h3>
    <span class="price">$20.00</span>
    <button>Add to Cart</button>
  </li>
</ul>
```

```typescript
// Click "Add to Cart" for Product B
// ✅ GOOD - Filter then find button
await page
  .getByRole('listitem')
  .filter({ hasText: 'Product B' })
  .getByRole('button', { name: 'Add to Cart' })
  .click();

// ❌ BAD - Position-based
await page.locator('.products li:nth-child(2) button').click();
```

#### Scenario 3: Dynamic Table

```html
<table>
  <thead>
    <tr><th>Name</th><th>Status</th><th>Actions</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>John Doe</td>
      <td>Active</td>
      <td><button>Edit</button></td>
    </tr>
    <tr>
      <td>Jane Smith</td>
      <td>Inactive</td>
      <td><button>Edit</button></td>
    </tr>
  </tbody>
</table>
```

```typescript
// Edit John Doe's record
// ✅ GOOD
await page
  .getByRole('row', { name: /John Doe/ })
  .getByRole('button', { name: 'Edit' })
  .click();

// ❌ BAD
await page.locator('table tr:nth-child(1) button').click();
```

### Locator Chaining and Filtering

```typescript
// Chaining - narrow down by nesting
const nav = page.getByRole('navigation');
const homeLink = nav.getByRole('link', { name: 'Home' });

// Filtering - narrow down by conditions
const products = page.getByRole('listitem');

// Filter by text content
const expensiveProduct = products.filter({ hasText: '$99.99' });

// Filter by having a child element
const productsWithDiscount = products.filter({
  has: page.getByText('SALE')
});

// Filter by NOT having something
const regularPriceProducts = products.filter({
  hasNot: page.getByText('SALE')
});

// Combine multiple filters
const specificProduct = products
  .filter({ hasText: 'Laptop' })
  .filter({ has: page.getByText('In Stock') });
```

### When CSS/XPath is Necessary

Sometimes you need CSS or XPath as a last resort:

```typescript
// Complex structural relationships
await page.locator('article:has(h2:text("Featured")) + aside').click();

// Attribute-based when no semantic meaning
await page.locator('[data-status="pending"]').click();

// XPath for text contains (last resort)
await page.locator('xpath=//div[contains(text(), "partial")]').click();
```

**⚠️ Anti-patterns to AVOID:**

```typescript
// ❌ NEVER use long CSS chains
await page.locator('#app > div > main > div.content > ul > li:first-child > button').click();

// ❌ NEVER use positional XPath
await page.locator('//*[@id="form"]/div[2]/div[1]/input').fill('text');

// ❌ NEVER rely on auto-generated classes
await page.locator('.css-1a2b3c4').click();
```

---

## Module 5: Page Object Model (POM) Architecture

### Why Use Page Object Model?

```
┌─────────────────────────────────────────────────────────────────┐
│                      WITHOUT POM                                │
├─────────────────────────────────────────────────────────────────┤
│ Test 1: await page.locator('#email').fill('user@test.com');    │
│ Test 2: await page.locator('#email').fill('admin@test.com');   │
│ Test 3: await page.locator('#email').fill('test@test.com');    │
│                                                                 │
│ ⚠️ If #email changes to .email-input, update ALL tests!        │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       WITH POM                                  │
├─────────────────────────────────────────────────────────────────┤
│ LoginPage: emailInput = page.getByLabel('Email');              │
│                                                                 │
│ Test 1: await loginPage.fillEmail('user@test.com');            │
│ Test 2: await loginPage.fillEmail('admin@test.com');           │
│ Test 3: await loginPage.fillEmail('test@test.com');            │
│                                                                 │
│ ✅ If selector changes, update ONLY LoginPage!                  │
└─────────────────────────────────────────────────────────────────┘
```

### Benefits of POM

1. **Single Responsibility**: Each page has one class
2. **Reusability**: Methods can be used across tests
3. **Maintainability**: Locators in one place
4. **Readability**: Tests describe WHAT, pages describe HOW

### Project Structure

```
playwright-framework/
├── playwright.config.ts
├── package.json
├── tests/
│   ├── login.spec.ts
│   ├── checkout.spec.ts
│   └── profile.spec.ts
├── pages/
│   ├── BasePage.ts
│   ├── LoginPage.ts
│   ├── HomePage.ts
│   ├── ProductPage.ts
│   └── CheckoutPage.ts
├── fixtures/
│   └── test-fixtures.ts
├── utils/
│   ├── test-data.ts
│   └── helpers.ts
└── data/
    └── users.json
```

### Implementing Page Objects

#### Base Page (Optional but Recommended)

```typescript
// pages/BasePage.ts
import { Page, Locator } from '@playwright/test';

export class BasePage {
  readonly page: Page;
  
  constructor(page: Page) {
    this.page = page;
  }

  // Common navigation
  async navigateTo(path: string) {
    await this.page.goto(path);
  }

  // Common wait for page load
  async waitForPageLoad() {
    await this.page.waitForLoadState('networkidle');
  }

  // Common element interactions
  async clickAndWait(locator: Locator) {
    await locator.click();
    await this.waitForPageLoad();
  }
}
```

#### Login Page

```typescript
// pages/LoginPage.ts
import { Page, Locator, expect } from '@playwright/test';
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  // Locators - defined as readonly properties
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;
  readonly rememberMeCheckbox: Locator;

  constructor(page: Page) {
    super(page);
    
    // Initialize locators in constructor
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign In' });
    this.errorMessage = page.getByRole('alert');
    this.rememberMeCheckbox = page.getByRole('checkbox', { name: 'Remember me' });
  }

  // Navigation
  async goto() {
    await this.page.goto('/login');
  }

  // Actions
  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async loginWithRememberMe(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.rememberMeCheckbox.check();
    await this.submitButton.click();
  }

  // Assertions - Keep assertions in tests, not pages
  // But helper methods for common checks are okay
  async getErrorText(): Promise<string> {
    return await this.errorMessage.textContent() || '';
  }

  async isLoginFormVisible(): Promise<boolean> {
    return await this.submitButton.isVisible();
  }
}
```

#### Home Page

```typescript
// pages/HomePage.ts
import { Page, Locator } from '@playwright/test';
import { BasePage } from './BasePage';

export class HomePage extends BasePage {
  readonly welcomeMessage: Locator;
  readonly searchInput: Locator;
  readonly searchButton: Locator;
  readonly navigationMenu: Locator;
  readonly userProfileLink: Locator;
  readonly logoutButton: Locator;

  constructor(page: Page) {
    super(page);
    
    this.welcomeMessage = page.getByRole('heading', { name: /Welcome/i });
    this.searchInput = page.getByPlaceholder('Search products...');
    this.searchButton = page.getByRole('button', { name: 'Search' });
    this.navigationMenu = page.getByRole('navigation');
    this.userProfileLink = page.getByRole('link', { name: 'My Profile' });
    this.logoutButton = page.getByRole('button', { name: 'Logout' });
  }

  async goto() {
    await this.page.goto('/');
  }

  async search(query: string) {
    await this.searchInput.fill(query);
    await this.searchButton.click();
  }

  async navigateToCategory(categoryName: string) {
    await this.navigationMenu
      .getByRole('link', { name: categoryName })
      .click();
  }

  async getUserName(): Promise<string> {
    const text = await this.welcomeMessage.textContent();
    // Extract name from "Welcome, John!"
    const match = text?.match(/Welcome,\s+(.+)!/);
    return match ? match[1] : '';
  }

  async logout() {
    await this.logoutButton.click();
  }
}
```

### Using Page Objects in Tests

```typescript
// tests/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { HomePage } from '../pages/HomePage';

test.describe('Login Functionality', () => {
  let loginPage: LoginPage;
  let homePage: HomePage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    homePage = new HomePage(page);
    await loginPage.goto();
  });

  test('successful login redirects to home page', async ({ page }) => {
    await loginPage.login('valid@user.com', 'validPassword');
    
    // Assert we're on home page
    await expect(homePage.welcomeMessage).toBeVisible();
    await expect(page).toHaveURL('/home');
  });

  test('invalid credentials show error message', async () => {
    await loginPage.login('invalid@user.com', 'wrongPassword');
    
    // Assert error is shown
    await expect(loginPage.errorMessage).toBeVisible();
    await expect(loginPage.errorMessage).toHaveText('Invalid credentials');
  });

  test('remember me keeps user logged in', async ({ page }) => {
    await loginPage.loginWithRememberMe('valid@user.com', 'validPassword');
    
    // Verify login successful
    await expect(homePage.welcomeMessage).toBeVisible();
    
    // Additional verification could involve session checking
  });
});
```

### POM Best Practices

```
┌─────────────────────────────────────────────────────────────────┐
│                    POM BEST PRACTICES                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ ✅ DO:                                                          │
│    • One page class per page/component                          │
│    • Define locators as readonly properties                     │
│    • Keep actions as simple methods                             │
│    • Use descriptive method names (login, addToCart)            │
│    • Initialize all locators in constructor                     │
│    • Return page objects for chaining when navigating           │
│                                                                 │
│ ❌ DON'T:                                                       │
│    • Put assertions in page objects                             │
│    • Store state in page objects                                │
│    • Make page objects too granular                             │
│    • Duplicate locators across pages                            │
│    • Use hard-coded waits in page objects                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Module 6: Fixtures & Test Organization

### What are Fixtures?

Fixtures are a way to establish environment for tests. They provide:
- Setup and teardown in one place
- Reusability across test files
- On-demand execution
- Better organization than beforeEach/afterEach

### Built-in Fixtures

```typescript
test('example', async ({ 
  page,           // New page for each test
  context,        // Browser context (isolated session)
  browser,        // Browser instance
  browserName,    // 'chromium', 'firefox', or 'webkit'
  request,        // API request context
}) => {
  // Your test code
});
```

### Creating Custom Fixtures

```typescript
// fixtures/test-fixtures.ts
import { test as base, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { HomePage } from '../pages/HomePage';
import { ProductPage } from '../pages/ProductPage';

// Define fixture types
type MyFixtures = {
  loginPage: LoginPage;
  homePage: HomePage;
  productPage: ProductPage;
  authenticatedPage: any; // For logged-in tests
};

// Extend base test with custom fixtures
export const test = base.extend<MyFixtures>({
  // Simple page object fixture
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await use(loginPage);
  },

  homePage: async ({ page }, use) => {
    const homePage = new HomePage(page);
    await use(homePage);
  },

  productPage: async ({ page }, use) => {
    const productPage = new ProductPage(page);
    await use(productPage);
  },

  // Fixture with setup and teardown
  authenticatedPage: async ({ page }, use) => {
    // Setup: Login before test
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login('test@user.com', 'password123');
    
    // Wait for login to complete
    await page.waitForURL('/home');
    
    // Provide the page to the test
    await use(page);
    
    // Teardown: Logout after test (optional)
    const homePage = new HomePage(page);
    await homePage.logout();
  },
});

// Re-export expect for convenience
export { expect } from '@playwright/test';
```

### Using Custom Fixtures in Tests

```typescript
// tests/product.spec.ts
import { test, expect } from '../fixtures/test-fixtures';

test.describe('Product Features', () => {
  
  // Uses authenticatedPage fixture - already logged in!
  test('logged in user can add product to cart', async ({ 
    authenticatedPage, 
    productPage 
  }) => {
    await productPage.goto();
    await productPage.addToCart('Laptop');
    await expect(productPage.cartCount).toHaveText('1');
  });

  // Uses loginPage fixture - starts at login
  test('guest user sees login prompt on checkout', async ({ 
    loginPage, 
    productPage 
  }) => {
    await productPage.goto();
    await productPage.clickCheckout();
    await expect(loginPage.submitButton).toBeVisible();
  });
});
```

### Auto-running Fixtures

```typescript
// fixtures/test-fixtures.ts
export const test = base.extend<MyFixtures>({
  // Auto-run fixture - executes for every test automatically
  autoLogger: [async ({}, use, testInfo) => {
    console.log(`Starting test: ${testInfo.title}`);
    const startTime = Date.now();
    
    await use(); // Test runs here
    
    const duration = Date.now() - startTime;
    console.log(`Finished ${testInfo.title} in ${duration}ms`);
  }, { auto: true }], // auto: true makes it run automatically
});
```

### Worker-Scoped Fixtures

```typescript
// For expensive setup that should be shared across tests in a worker
export const test = base.extend<{}, { sharedDatabase: Database }>({
  sharedDatabase: [async ({}, use) => {
    // Setup - runs once per worker
    const db = await setupTestDatabase();
    
    await use(db);
    
    // Teardown - runs when worker shuts down
    await db.cleanup();
  }, { scope: 'worker' }],
});
```

### Fixture Dependencies

```typescript
export const test = base.extend<MyFixtures>({
  // Base fixture
  apiClient: async ({ request }, use) => {
    const client = new ApiClient(request);
    await use(client);
  },

  // Fixture that depends on another fixture
  testUser: async ({ apiClient }, use) => {
    // Create user using API
    const user = await apiClient.createUser({
      email: `test-${Date.now()}@test.com`,
      password: 'Test123!'
    });
    
    await use(user);
    
    // Cleanup: Delete user after test
    await apiClient.deleteUser(user.id);
  },

  // Fixture that depends on testUser
  loggedInPage: async ({ page, testUser }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login(testUser.email, testUser.password);
    await use(page);
  },
});
```

---

## Module 7: Authentication Handling

### The Problem

```
❌ BAD: Login before EVERY test
   Test 1: Login → Test → Logout
   Test 2: Login → Test → Logout
   Test 3: Login → Test → Logout
   
   ⏱️ Wastes time on repeated logins
   🔄 More points of failure
```

```
✅ GOOD: Login ONCE, reuse session
   Setup: Login → Save state
   Test 1: Load state → Test
   Test 2: Load state → Test
   Test 3: Load state → Test
   
   ⏱️ Fast execution
   ✅ Tests are isolated but efficient
```

### Authentication Strategy Overview

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  projects: [
    // Setup project - runs first, performs login
    {
      name: 'setup',
      testMatch: /.*\.setup\.ts/,
    },
    
    // Tests that need authentication
    {
      name: 'chromium',
      use: {
        storageState: 'playwright/.auth/user.json', // Use saved state
      },
      dependencies: ['setup'], // Wait for setup to complete
    },
  ],
});
```

### Step-by-Step Implementation

#### Step 1: Create Auth Directory

```bash
mkdir -p playwright/.auth
echo "playwright/.auth" >> .gitignore
```

#### Step 2: Create Setup File

```typescript
// tests/auth.setup.ts
import { test as setup, expect } from '@playwright/test';

const authFile = 'playwright/.auth/user.json';

setup('authenticate', async ({ page }) => {
  // Navigate to login page
  await page.goto('/login');
  
  // Fill login form
  await page.getByLabel('Email').fill(process.env.TEST_USER_EMAIL!);
  await page.getByLabel('Password').fill(process.env.TEST_USER_PASSWORD!);
  
  // Submit
  await page.getByRole('button', { name: 'Sign In' }).click();
  
  // Wait for successful login
  // Option 1: Wait for URL change
  await page.waitForURL('/dashboard');
  
  // Option 2: Wait for element that only appears when logged in
  await expect(page.getByRole('button', { name: 'Logout' })).toBeVisible();
  
  // Save authentication state
  await page.context().storageState({ path: authFile });
});
```

#### Step 3: Configure Playwright

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  
  projects: [
    // Authentication setup
    {
      name: 'setup',
      testMatch: /.*\.setup\.ts/,
    },
    
    // Desktop browsers with auth
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
    
    // Tests that DON'T need authentication (login page tests)
    {
      name: 'chromium-no-auth',
      use: { ...devices['Desktop Chrome'] },
      testMatch: /.*login\.spec\.ts/,
    },
  ],
});
```

#### Step 4: Write Tests (No Login Needed!)

```typescript
// tests/dashboard.spec.ts
import { test, expect } from '@playwright/test';

// These tests run with pre-authenticated state
test('user can see dashboard', async ({ page }) => {
  await page.goto('/dashboard');
  
  // Already logged in - no login step needed!
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});

test('user can view profile', async ({ page }) => {
  await page.goto('/profile');
  
  await expect(page.getByText('My Profile')).toBeVisible();
});
```

### Multiple User Roles

```typescript
// tests/auth.setup.ts
import { test as setup } from '@playwright/test';

const adminFile = 'playwright/.auth/admin.json';
const userFile = 'playwright/.auth/user.json';

setup('authenticate as admin', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.ADMIN_EMAIL!);
  await page.getByLabel('Password').fill(process.env.ADMIN_PASSWORD!);
  await page.getByRole('button', { name: 'Sign In' }).click();
  await page.waitForURL('/admin/dashboard');
  await page.context().storageState({ path: adminFile });
});

setup('authenticate as user', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.USER_EMAIL!);
  await page.getByLabel('Password').fill(process.env.USER_PASSWORD!);
  await page.getByRole('button', { name: 'Sign In' }).click();
  await page.waitForURL('/dashboard');
  await page.context().storageState({ path: userFile });
});
```

```typescript
// playwright.config.ts
projects: [
  { name: 'setup', testMatch: /.*\.setup\.ts/ },
  
  {
    name: 'admin-tests',
    use: { storageState: 'playwright/.auth/admin.json' },
    dependencies: ['setup'],
    testMatch: /.*admin.*\.spec\.ts/,
  },
  
  {
    name: 'user-tests',
    use: { storageState: 'playwright/.auth/user.json' },
    dependencies: ['setup'],
    testMatch: /.*user.*\.spec\.ts/,
  },
],
```

### API-Based Authentication

```typescript
// tests/auth.setup.ts
import { test as setup } from '@playwright/test';

setup('authenticate via API', async ({ request }) => {
  // Login via API (faster than UI)
  const response = await request.post('/api/login', {
    data: {
      email: process.env.TEST_USER_EMAIL,
      password: process.env.TEST_USER_PASSWORD,
    },
  });
  
  // Save state including cookies from API response
  await request.storageState({ path: 'playwright/.auth/user.json' });
});
```

### Session Storage Handling

If your app uses sessionStorage (not supported by storageState):

```typescript
// tests/auth.setup.ts
import { test as setup } from '@playwright/test';
import fs from 'fs';

setup('authenticate with session storage', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('user@test.com');
  await page.getByLabel('Password').fill('password');
  await page.getByRole('button', { name: 'Sign In' }).click();
  await page.waitForURL('/dashboard');
  
  // Save regular storage state
  await page.context().storageState({ path: 'playwright/.auth/user.json' });
  
  // Save session storage separately
  const sessionStorage = await page.evaluate(() => 
    JSON.stringify(sessionStorage)
  );
  fs.writeFileSync(
    'playwright/.auth/session.json',
    sessionStorage,
    'utf-8'
  );
});
```

```typescript
// fixtures/test-fixtures.ts
export const test = base.extend({
  page: async ({ page }, use) => {
    // Restore session storage before test
    const sessionStorage = JSON.parse(
      fs.readFileSync('playwright/.auth/session.json', 'utf-8')
    );
    
    await page.addInitScript((storage) => {
      for (const [key, value] of Object.entries(storage)) {
        window.sessionStorage.setItem(key, value as string);
      }
    }, JSON.parse(sessionStorage));
    
    await use(page);
  },
});
```

---

## Module 8: Configuration & Multi-Browser Testing

### Complete Configuration File

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  // Test directory
  testDir: './tests',
  
  // Test file pattern
  testMatch: '**/*.spec.ts',
  
  // Ignore patterns
  testIgnore: '**/helpers/**',
  
  // Run tests in parallel
  fullyParallel: true,
  
  // Fail build if test.only is left in code
  forbidOnly: !!process.env.CI,
  
  // Retry failed tests
  retries: process.env.CI ? 2 : 0,
  
  // Limit parallel workers
  workers: process.env.CI ? 1 : undefined,
  
  // Reporter configuration
  reporter: [
    ['html', { outputFolder: 'playwright-report' }],
    ['list'],
    ['junit', { outputFile: 'results.xml' }],
  ],
  
  // Shared settings for all projects
  use: {
    // Base URL for all tests
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    
    // Collect trace on first retry
    trace: 'on-first-retry',
    
    // Screenshot on failure
    screenshot: 'only-on-failure',
    
    // Record video on retry
    video: 'on-first-retry',
    
    // Timeouts
    actionTimeout: 15000,
    navigationTimeout: 30000,
  },
  
  // Global timeout per test
  timeout: 60000,
  
  // Expect timeout
  expect: {
    timeout: 10000,
  },
  
  // Project configurations
  projects: [
    // Setup project
    {
      name: 'setup',
      testMatch: /.*\.setup\.ts/,
    },
    
    // Desktop browsers
    {
      name: 'chromium',
      use: { 
        ...devices['Desktop Chrome'],
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
    
    {
      name: 'firefox',
      use: { 
        ...devices['Desktop Firefox'],
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
    
    {
      name: 'webkit',
      use: { 
        ...devices['Desktop Safari'],
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
    
    // Mobile browsers
    {
      name: 'mobile-chrome',
      use: { 
        ...devices['Pixel 5'],
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
    
    {
      name: 'mobile-safari',
      use: { 
        ...devices['iPhone 12'],
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
    
    // Branded browsers
    {
      name: 'google-chrome',
      use: { 
        ...devices['Desktop Chrome'],
        channel: 'chrome',
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
  ],
  
  // Local dev server
  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
});
```

### Environment-Based Configuration

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

const environment = process.env.TEST_ENV || 'local';

const envConfig = {
  local: {
    baseURL: 'http://localhost:3000',
    retries: 0,
  },
  staging: {
    baseURL: 'https://staging.example.com',
    retries: 2,
  },
  production: {
    baseURL: 'https://www.example.com',
    retries: 3,
  },
};

export default defineConfig({
  use: {
    baseURL: envConfig[environment].baseURL,
  },
  retries: envConfig[environment].retries,
});
```

### Running on Specific Browsers/Projects

```bash
# Run on all projects
npx playwright test

# Run on specific project
npx playwright test --project=chromium
npx playwright test --project=mobile-safari

# Run on multiple projects
npx playwright test --project=chromium --project=firefox

# Run with specific browser headed
npx playwright test --project=chromium --headed

# Skip certain projects
npx playwright test --project=chromium --project=firefox
```

### Timeout Configuration

```typescript
// playwright.config.ts
export default defineConfig({
  // Global test timeout
  timeout: 60000, // 60 seconds
  
  // Expect timeout (for assertions)
  expect: {
    timeout: 10000, // 10 seconds
  },
  
  use: {
    // Action timeout (click, fill, etc.)
    actionTimeout: 15000,
    
    // Navigation timeout
    navigationTimeout: 30000,
  },
});
```

```typescript
// Override in specific tests
test('slow test', async ({ page }) => {
  test.setTimeout(120000); // 2 minutes for this test
  
  await page.goto('/slow-page');
});

// Slow test annotation
test('another slow test', async ({ page }) => {
  test.slow(); // Multiplies timeout by 3
  
  await page.goto('/slow-page');
});
```

---

## Module 9: Advanced Patterns & Best Practices

### Web-First Assertions

```typescript
// ✅ GOOD - Auto-retrying assertions
await expect(page.getByRole('alert')).toBeVisible();
await expect(page.getByRole('heading')).toHaveText('Welcome');
await expect(page.getByRole('list')).toHaveCount(5);
await expect(page).toHaveURL(/dashboard/);
await expect(page).toHaveTitle('Home Page');

// ❌ BAD - Manual waits
await page.waitForTimeout(2000); // NEVER use fixed waits
const isVisible = await page.getByRole('alert').isVisible(); // Single check
```

### Common Assertions Reference

```typescript
// Visibility
await expect(locator).toBeVisible();
await expect(locator).toBeHidden();
await expect(locator).toBeAttached();
await expect(locator).toBeDetached();

// Text content
await expect(locator).toHaveText('exact text');
await expect(locator).toHaveText(/partial/i);
await expect(locator).toContainText('partial');

// Attributes
await expect(locator).toHaveAttribute('href', '/about');
await expect(locator).toHaveClass(/active/);
await expect(locator).toHaveId('main-content');

// State
await expect(locator).toBeEnabled();
await expect(locator).toBeDisabled();
await expect(locator).toBeChecked();
await expect(locator).toBeFocused();
await expect(locator).toBeEditable();

// Input values
await expect(locator).toHaveValue('input value');
await expect(locator).toHaveValues(['option1', 'option2']);

// Count
await expect(locator).toHaveCount(5);

// Page-level
await expect(page).toHaveURL('/dashboard');
await expect(page).toHaveURL(/dashboard/);
await expect(page).toHaveTitle(/Home/);
```

### Network Interception and Mocking

```typescript
test('mock API response', async ({ page }) => {
  // Mock API endpoint
  await page.route('**/api/users', async (route) => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify([
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' },
      ]),
    });
  });
  
  await page.goto('/users');
  await expect(page.getByText('John Doe')).toBeVisible();
});

test('simulate error response', async ({ page }) => {
  await page.route('**/api/products', async (route) => {
    await route.fulfill({
      status: 500,
      body: 'Internal Server Error',
    });
  });
  
  await page.goto('/products');
  await expect(page.getByText('Failed to load products')).toBeVisible();
});

test('modify response', async ({ page }) => {
  await page.route('**/api/user/profile', async (route) => {
    const response = await route.fetch();
    const json = await response.json();
    
    // Modify response
    json.premiumUser = true;
    
    await route.fulfill({
      response,
      json,
    });
  });
  
  await page.goto('/profile');
  await expect(page.getByText('Premium Features')).toBeVisible();
});

test('wait for specific request', async ({ page }) => {
  // Start waiting before navigation
  const responsePromise = page.waitForResponse('**/api/data');
  
  await page.goto('/dashboard');
  
  const response = await responsePromise;
  expect(response.status()).toBe(200);
});
```

### Handling Dialogs

```typescript
test('handle JavaScript alerts', async ({ page }) => {
  // Setup dialog handler BEFORE triggering
  page.on('dialog', async (dialog) => {
    expect(dialog.type()).toBe('confirm');
    expect(dialog.message()).toBe('Are you sure?');
    await dialog.accept(); // or dialog.dismiss()
  });
  
  await page.getByRole('button', { name: 'Delete' }).click();
});

test('handle prompt dialog', async ({ page }) => {
  page.on('dialog', async (dialog) => {
    await dialog.accept('User input value');
  });
  
  await page.getByRole('button', { name: 'Rename' }).click();
});
```

### File Upload and Download

```typescript
test('upload file', async ({ page }) => {
  await page.goto('/upload');
  
  // File input upload
  await page.getByLabel('Upload file').setInputFiles('test-files/document.pdf');
  
  // Multiple files
  await page.getByLabel('Upload files').setInputFiles([
    'test-files/doc1.pdf',
    'test-files/doc2.pdf',
  ]);
  
  // Clear files
  await page.getByLabel('Upload file').setInputFiles([]);
});

test('download file', async ({ page }) => {
  // Wait for download event
  const downloadPromise = page.waitForEvent('download');
  
  await page.getByRole('button', { name: 'Download Report' }).click();
  
  const download = await downloadPromise;
  
  // Save to specific path
  await download.saveAs('downloads/' + download.suggestedFilename());
  
  // Or get the path where it was saved
  const path = await download.path();
});
```

### Debugging Techniques

```typescript
// Pause test execution
test('debug test', async ({ page }) => {
  await page.goto('/login');
  
  await page.pause(); // Opens Playwright Inspector
  
  await page.getByLabel('Email').fill('test@test.com');
});

// Screenshot at any point
test('capture state', async ({ page }) => {
  await page.goto('/dashboard');
  
  await page.screenshot({ path: 'screenshots/dashboard.png', fullPage: true });
});

// Console log in browser context
test('browser console', async ({ page }) => {
  page.on('console', (msg) => {
    console.log(`Browser console: ${msg.text()}`);
  });
  
  await page.goto('/app');
});

// Evaluate JavaScript
test('check JS variable', async ({ page }) => {
  await page.goto('/app');
  
  const appVersion = await page.evaluate(() => {
    return (window as any).APP_VERSION;
  });
  
  console.log(`App version: ${appVersion}`);
});
```

### Test Data Management

```typescript
// data/test-users.ts
export const testUsers = {
  admin: {
    email: 'admin@test.com',
    password: 'Admin123!',
    role: 'admin',
  },
  regularUser: {
    email: 'user@test.com',
    password: 'User123!',
    role: 'user',
  },
  premiumUser: {
    email: 'premium@test.com',
    password: 'Premium123!',
    role: 'premium',
  },
};

// In tests
import { testUsers } from '../data/test-users';

test('admin can access admin panel', async ({ page }) => {
  await loginPage.login(testUsers.admin.email, testUsers.admin.password);
  await page.goto('/admin');
  await expect(page.getByRole('heading', { name: 'Admin Panel' })).toBeVisible();
});
```

### Tagging and Filtering Tests

```typescript
// Tag tests with annotations
test('user can checkout @smoke @critical', async ({ page }) => {
  // ...
});

test('user can view order history @regression', async ({ page }) => {
  // ...
});

test.describe('Payment tests @payment', () => {
  test('credit card payment', async ({ page }) => {
    // ...
  });
  
  test('paypal payment', async ({ page }) => {
    // ...
  });
});
```

```bash
# Run tests by tag
npx playwright test --grep "@smoke"
npx playwright test --grep "@critical"
npx playwright test --grep "@smoke|@critical"

# Exclude tests by tag
npx playwright test --grep-invert "@slow"
```

---

## Module 10: Building a Complete Framework

### Final Project Structure

```
playwright-framework/
├── playwright.config.ts          # Main configuration
├── package.json
├── .env                          # Environment variables (gitignored)
├── .env.example                  # Template for env vars
│
├── tests/
│   ├── auth.setup.ts            # Authentication setup
│   ├── login/
│   │   └── login.spec.ts
│   ├── products/
│   │   ├── product-list.spec.ts
│   │   └── product-details.spec.ts
│   ├── checkout/
│   │   ├── cart.spec.ts
│   │   └── payment.spec.ts
│   └── api/
│       └── api-tests.spec.ts
│
├── pages/
│   ├── BasePage.ts
│   ├── LoginPage.ts
│   ├── HomePage.ts
│   ├── ProductPage.ts
│   ├── CartPage.ts
│   └── CheckoutPage.ts
│
├── fixtures/
│   └── test-fixtures.ts         # Custom fixtures
│
├── utils/
│   ├── test-data.ts             # Test data helpers
│   ├── api-helpers.ts           # API utilities
│   └── date-helpers.ts          # Date utilities
│
├── data/
│   ├── users.json               # User test data
│   └── products.json            # Product test data
│
├── playwright/
│   └── .auth/                   # Auth state (gitignored)
│       ├── admin.json
│       └── user.json
│
└── reports/                     # Test reports (gitignored)
```

### Complete Test Example

```typescript
// tests/checkout/cart.spec.ts
import { test, expect } from '../../fixtures/test-fixtures';
import { testProducts } from '../../data/products';

test.describe('Shopping Cart', () => {
  test.beforeEach(async ({ productPage }) => {
    await productPage.goto();
  });

  test('user can add product to cart @smoke @cart', async ({ 
    productPage, 
    cartPage 
  }) => {
    // Add first product
    await productPage.addToCart(testProducts.laptop.name);
    
    // Verify cart badge updated
    await expect(productPage.cartBadge).toHaveText('1');
    
    // Go to cart
    await productPage.goToCart();
    
    // Verify product in cart
    await expect(cartPage.getProductRow(testProducts.laptop.name)).toBeVisible();
    await expect(cartPage.subtotal).toHaveText(testProducts.laptop.price);
  });

  test('user can update quantity @cart', async ({ productPage, cartPage }) => {
    await productPage.addToCart(testProducts.laptop.name);
    await productPage.goToCart();
    
    // Update quantity
    await cartPage.updateQuantity(testProducts.laptop.name, 3);
    
    // Verify total updated
    const expectedTotal = testProducts.laptop.numericPrice * 3;
    await expect(cartPage.subtotal).toHaveText(`$${expectedTotal.toFixed(2)}`);
  });

  test('user can remove product from cart @cart', async ({ 
    productPage, 
    cartPage 
  }) => {
    await productPage.addToCart(testProducts.laptop.name);
    await productPage.goToCart();
    
    // Remove product
    await cartPage.removeProduct(testProducts.laptop.name);
    
    // Verify empty cart
    await expect(cartPage.emptyCartMessage).toBeVisible();
  });

  test('guest user sees login prompt at checkout @cart @auth', async ({ 
    page, 
    productPage, 
    cartPage 
  }) => {
    // Clear authentication
    await page.context().clearCookies();
    
    await productPage.addToCart(testProducts.laptop.name);
    await productPage.goToCart();
    await cartPage.proceedToCheckout();
    
    // Verify redirect to login
    await expect(page).toHaveURL(/login/);
    await expect(page.getByText('Please login to continue')).toBeVisible();
  });
});
```

### CI/CD Integration (GitHub Actions)

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          
      - name: Install dependencies
        run: npm ci
        
      - name: Install Playwright Browsers
        run: npx playwright install --with-deps
        
      - name: Run Playwright tests
        run: npx playwright test
        env:
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

---

## Exercises & Hands-On Projects

### Exercise 1: Locator Practice (Beginner)

Given this HTML, write the best locators:

```html
<header>
  <nav>
    <a href="/">Home</a>
    <a href="/products">Products</a>
    <a href="/about">About Us</a>
  </nav>
  <button class="login-btn">Sign In</button>
</header>

<main>
  <h1>Welcome to Our Store</h1>
  
  <form id="search-form">
    <label for="search">Search Products</label>
    <input id="search" type="text" placeholder="Enter product name..." />
    <button type="submit">Search</button>
  </form>
  
  <div class="product-list">
    <article data-testid="product-1">
      <h2>Laptop Pro</h2>
      <p class="price">$999.99</p>
      <button>Add to Cart</button>
    </article>
    <article data-testid="product-2">
      <h2>Wireless Mouse</h2>
      <p class="price">$29.99</p>
      <button>Add to Cart</button>
    </article>
  </div>
</main>
```

**Tasks:**
1. Click the "Products" navigation link
2. Click the "Sign In" button
3. Fill the search input with "laptop"
4. Add "Wireless Mouse" to cart
5. Verify the price of "Laptop Pro"

### Exercise 2: Page Object Model (Intermediate)

Create page objects for a todo application:
- `TodoPage.ts` with:
  - Add new todo
  - Mark todo complete
  - Delete todo
  - Filter by status (all/active/completed)
  - Get count of remaining items

### Exercise 3: Authentication Framework (Advanced)

Build a complete auth system:
1. Create setup for admin and regular user
2. Create fixtures that provide pre-logged-in pages
3. Write tests that use both user types
4. Handle session expiry gracefully

### Exercise 4: API + UI Testing (Advanced)

Create tests that:
1. Create test data via API
2. Verify the data appears in UI
3. Modify data via UI
4. Verify changes via API
5. Clean up test data

---

## Quick Reference Cards

### Locator Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                    LOCATOR QUICK REFERENCE                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ ROLE (Preferred)                                               │
│ page.getByRole('button', { name: 'Submit' })                   │
│ page.getByRole('link', { name: 'Home' })                       │
│ page.getByRole('textbox', { name: 'Email' })                   │
│ page.getByRole('checkbox', { name: 'Remember' })               │
│ page.getByRole('heading', { level: 1 })                        │
│                                                                 │
│ FORM CONTROLS                                                  │
│ page.getByLabel('Email')                                       │
│ page.getByPlaceholder('Enter email')                           │
│                                                                 │
│ TEXT                                                           │
│ page.getByText('Welcome')                                      │
│ page.getByText(/welcome/i)                                     │
│                                                                 │
│ TEST ID                                                        │
│ page.getByTestId('submit-btn')                                 │
│                                                                 │
│ FILTERING                                                      │
│ locator.filter({ hasText: 'Product' })                         │
│ locator.filter({ has: page.getByRole('button') })              │
│                                                                 │
│ CHAINING                                                       │
│ page.getByRole('list').getByRole('listitem')                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Assertion Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                  ASSERTION QUICK REFERENCE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ VISIBILITY                                                      │
│ await expect(locator).toBeVisible();                           │
│ await expect(locator).toBeHidden();                            │
│                                                                 │
│ TEXT                                                           │
│ await expect(locator).toHaveText('exact');                     │
│ await expect(locator).toContainText('partial');                │
│                                                                 │
│ STATE                                                          │
│ await expect(locator).toBeEnabled();                           │
│ await expect(locator).toBeDisabled();                          │
│ await expect(locator).toBeChecked();                           │
│                                                                 │
│ VALUE                                                          │
│ await expect(locator).toHaveValue('input value');              │
│                                                                 │
│ COUNT                                                          │
│ await expect(locator).toHaveCount(5);                          │
│                                                                 │
│ ATTRIBUTE                                                      │
│ await expect(locator).toHaveAttribute('href', '/home');        │
│ await expect(locator).toHaveClass(/active/);                   │
│                                                                 │
│ PAGE                                                           │
│ await expect(page).toHaveURL('/dashboard');                    │
│ await expect(page).toHaveTitle(/Home/);                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Command Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                   COMMAND QUICK REFERENCE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ RUNNING TESTS                                                   │
│ npx playwright test                    # Run all tests          │
│ npx playwright test --headed           # See browser            │
│ npx playwright test --ui               # Interactive mode       │
│ npx playwright test file.spec.ts       # Specific file          │
│ npx playwright test --project=chrome   # Specific browser       │
│ npx playwright test --grep "@smoke"    # By tag                 │
│ npx playwright test --debug            # Debug mode             │
│                                                                 │
│ REPORTS                                                         │
│ npx playwright show-report             # Open HTML report       │
│                                                                 │
│ CODEGEN                                                         │
│ npx playwright codegen example.com     # Record tests           │
│                                                                 │
│ BROWSERS                                                        │
│ npx playwright install                 # Install browsers       │
│ npx playwright install chromium        # Specific browser       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Resources

### Official Documentation
- Playwright Docs: https://playwright.dev/docs/intro
- API Reference: https://playwright.dev/docs/api/class-playwright
- Best Practices: https://playwright.dev/docs/best-practices

### Learning Resources
- Microsoft Playwright Training: https://learn.microsoft.com/en-us/training/modules/build-with-playwright/
- Playwright YouTube: https://www.youtube.com/c/Playwrightdev

### Community
- Discord: https://aka.ms/playwright/discord
- GitHub: https://github.com/microsoft/playwright
- Stack Overflow: https://stackoverflow.com/questions/tagged/playwright

---

*Created for QA Engineers transitioning from manual testing and automation engineers building frameworks from scratch.*

*Version 1.0 | January 2026*
