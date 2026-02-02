# Playwright Quick Reference Card

## Essential Locators (Priority Order)

```
1. getByRole()      → page.getByRole('button', { name: 'Submit' })
2. getByLabel()     → page.getByLabel('Email')
3. getByPlaceholder() → page.getByPlaceholder('Enter email')
4. getByText()      → page.getByText('Welcome')
5. getByTestId()    → page.getByTestId('submit-btn')
6. getByAltText()   → page.getByAltText('Logo')
7. getByTitle()     → page.getByTitle('Close')
```

## Common Role Values

| Role | HTML Elements |
|------|---------------|
| button | `<button>`, `<input type="button/submit">` |
| checkbox | `<input type="checkbox">` |
| heading | `<h1>` - `<h6>` |
| link | `<a href="...">` |
| listitem | `<li>` |
| textbox | `<input type="text">`, `<textarea>` |
| combobox | `<select>` |
| radio | `<input type="radio">` |
| row | `<tr>` |
| table | `<table>` |
| navigation | `<nav>` |

## Locator Filtering & Chaining

```typescript
// Filter by text
page.getByRole('listitem').filter({ hasText: 'Product A' })

// Filter by child element
page.getByRole('listitem').filter({ has: page.getByRole('button') })

// Filter by NOT having
page.getByRole('listitem').filter({ hasNot: page.getByText('Sold Out') })

// Chaining
page.getByRole('navigation').getByRole('link', { name: 'Home' })
```

## Assertions (Auto-Retrying)

```typescript
// Visibility
await expect(locator).toBeVisible();
await expect(locator).toBeHidden();

// Text
await expect(locator).toHaveText('exact text');
await expect(locator).toContainText('partial');

// State
await expect(locator).toBeEnabled();
await expect(locator).toBeDisabled();
await expect(locator).toBeChecked();

// Value
await expect(locator).toHaveValue('input value');

// Count
await expect(locator).toHaveCount(5);

// Attributes
await expect(locator).toHaveAttribute('href', '/home');
await expect(locator).toHaveClass(/active/);

// Page
await expect(page).toHaveURL('/dashboard');
await expect(page).toHaveTitle(/Home/);
```

## Actions

```typescript
// Click
await locator.click();
await locator.dblclick();
await locator.click({ button: 'right' });

// Type
await locator.fill('text');
await locator.type('text');     // Character by character
await locator.clear();

// Checkbox/Radio
await locator.check();
await locator.uncheck();
await locator.setChecked(true);

// Select
await locator.selectOption('value');
await locator.selectOption({ label: 'Label' });

// Hover
await locator.hover();

// Focus
await locator.focus();

// Press keys
await locator.press('Enter');
await page.keyboard.press('Control+A');
```

## Page Object Pattern

```typescript
// pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign In' });
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}
```

## Authentication Setup

```typescript
// tests/auth.setup.ts
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('user@test.com');
  await page.getByLabel('Password').fill('password');
  await page.getByRole('button', { name: 'Sign In' }).click();
  await page.waitForURL('/dashboard');
  await page.context().storageState({ path: 'playwright/.auth/user.json' });
});

// playwright.config.ts
projects: [
  { name: 'setup', testMatch: /.*\.setup\.ts/ },
  {
    name: 'chromium',
    use: { storageState: 'playwright/.auth/user.json' },
    dependencies: ['setup'],
  },
]
```

## Custom Fixtures

```typescript
// fixtures/test-fixtures.ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

export const test = base.extend<{ loginPage: LoginPage }>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },
});

export { expect } from '@playwright/test';
```

## Common Commands

```bash
# Run all tests
npx playwright test

# Run headed (see browser)
npx playwright test --headed

# Run specific file
npx playwright test tests/login.spec.ts

# Run specific project/browser
npx playwright test --project=chromium

# Run with UI Mode (interactive)
npx playwright test --ui

# Run by tag
npx playwright test --grep "@smoke"

# Debug mode
npx playwright test --debug

# Generate test (record actions)
npx playwright codegen example.com

# View report
npx playwright show-report

# View trace
npx playwright show-trace trace.zip
```

## Configuration Essentials

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  reporter: 'html',
  
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
});
```

## Test Hooks

```typescript
test.describe('Suite', () => {
  test.beforeAll(async () => { /* Once before all */ });
  test.beforeEach(async ({ page }) => { /* Before each test */ });
  test.afterEach(async ({ page }) => { /* After each test */ });
  test.afterAll(async () => { /* Once after all */ });
  
  test('test case', async ({ page }) => { });
});
```

## Network Mocking

```typescript
await page.route('**/api/users', async (route) => {
  await route.fulfill({
    status: 200,
    body: JSON.stringify([{ id: 1, name: 'John' }]),
  });
});
```

## Waiting

```typescript
// Wait for element (prefer assertions instead)
await page.waitForSelector('.loaded');

// Wait for URL
await page.waitForURL('/dashboard');

// Wait for network
await page.waitForResponse('**/api/data');

// Wait for load state
await page.waitForLoadState('networkidle');

// AVOID: Fixed waits
// await page.waitForTimeout(2000); ❌
```

---

*Print this card for quick reference during test development!*
