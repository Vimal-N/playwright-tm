# Mastering Page Object Model (POM) in Playwright

## A Complete Strategy Guide with Real-Life Analogies

---

## Table of Contents

1. [What is Page Object Model?](#what-is-page-object-model)
2. [Real-Life Analogies to Understand POM](#real-life-analogies-to-understand-pom)
3. [Why POM Matters - The Problems It Solves](#why-pom-matters---the-problems-it-solves)
4. [POM Architecture & Principles](#pom-architecture--principles)
5. [Folder Structure & Organization](#folder-structure--organization)
6. [Step-by-Step Implementation Guide](#step-by-step-implementation-guide)
7. [Progressive Examples: Beginner to Advanced](#progressive-examples-beginner-to-advanced)
8. [Common Patterns & Best Practices](#common-patterns--best-practices)
9. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
10. [Exercises & Practice Projects](#exercises--practice-projects)
11. [Mastery Checklist](#mastery-checklist)

---

## What is Page Object Model?

Page Object Model (POM) is a **design pattern** that creates a separation between:
- **WHAT** you want to test (test logic)
- **HOW** you interact with the page (page interactions)

Each web page (or significant component) in your application gets its own "Page Object" class that contains:
- **Locators** - How to find elements
- **Actions** - What you can do on that page
- **Navigation** - How to get to that page

```
┌─────────────────────────────────────────────────────────────────┐
│                    WITHOUT PAGE OBJECT MODEL                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Test File: "Find email input, fill it, find password input,   │
│              fill it, find button, click it, verify URL..."    │
│                                                                 │
│  ❌ Test knows too much about HOW to do things                  │
│  ❌ Same locators repeated in many tests                        │
│  ❌ If UI changes, update ALL tests                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                     WITH PAGE OBJECT MODEL                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Test File: "loginPage.login(email, password)"                  │
│                                                                 │
│  Page Object: Knows HOW to find elements and interact           │
│                                                                 │
│  ✅ Test only knows WHAT to do                                  │
│  ✅ Locators defined once                                       │
│  ✅ If UI changes, update ONE file                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Real-Life Analogies to Understand POM

### Analogy 1: The Restaurant Kitchen 🍳

Imagine you're a **customer at a restaurant** (the Test) and the **kitchen** is the Page Object.

**Without POM (No Kitchen):**
```
You want a sandwich, so you:
1. Walk into the storage room
2. Find the bread (locator)
3. Find the cheese (locator)
4. Find the knife (locator)
5. Slice the bread yourself
6. Assemble the sandwich
7. Clean up

Tomorrow, they move the bread to a different shelf.
Now you need to learn the new location!
```

**With POM (With Kitchen):**
```
You want a sandwich, so you:
1. Tell the waiter: "One cheese sandwich, please"

The kitchen (Page Object) knows:
- Where the bread is stored
- Where the cheese is kept
- How to assemble a sandwich

Tomorrow, they move the bread? Kitchen handles it.
You still just say: "One cheese sandwich, please"
```

```typescript
// Without POM - You (test) do everything
test('make sandwich', async ({ page }) => {
  await page.goto('/storage');
  await page.locator('#bread-shelf-3').click();
  await page.locator('#bread-whole-wheat').click();
  await page.locator('#cheese-section').click();
  await page.locator('#cheese-cheddar').click();
  // ... 20 more steps
});

// With POM - Kitchen handles details
test('make sandwich', async ({ page }) => {
  const kitchen = new KitchenPage(page);
  await kitchen.makeSandwich('whole-wheat', 'cheddar');
});
```

---

### Analogy 2: TV Remote Control 📺

Your **TV remote** is like a Page Object for your TV.

**The Remote (Page Object) provides:**
- Buttons (locators) - Power, Volume, Channel
- Actions (methods) - turnOn(), changeChannel(), increaseVolume()

**You (the Test) just say:**
- "Turn on the TV" → `remote.turnOn()`
- "Go to channel 5" → `remote.changeChannel(5)`

**You don't need to know:**
- Which infrared frequency to use
- How the TV processes signals
- Internal wiring of the remote

```typescript
// RemotePage.ts - The "Page Object" for your TV
class TVRemote {
  readonly powerButton: Locator;
  readonly volumeUp: Locator;
  readonly volumeDown: Locator;
  readonly channelInput: Locator;

  async turnOn() {
    await this.powerButton.click();
  }

  async changeChannel(channel: number) {
    await this.channelInput.fill(channel.toString());
    await this.channelInput.press('Enter');
  }

  async setVolume(level: number) {
    // Complex logic hidden from the user
    for (let i = 0; i < level; i++) {
      await this.volumeUp.click();
    }
  }
}

// Test - Simple and readable
test('watch favorite show', async () => {
  await remote.turnOn();
  await remote.changeChannel(42);
  await remote.setVolume(15);
});
```

---

### Analogy 3: ATM Machine 🏧

An **ATM** is a perfect Page Object example!

```
┌─────────────────────────────────────────────────────────────────┐
│                         ATM MACHINE                             │
│                      (Page Object)                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LOCATORS (Elements):                                           │
│  ├── Card Slot                                                  │
│  ├── PIN Pad                                                    │
│  ├── Screen                                                     │
│  ├── Cash Dispenser                                             │
│  └── Receipt Printer                                            │
│                                                                 │
│  ACTIONS (Methods):                                             │
│  ├── insertCard(cardNumber)                                     │
│  ├── enterPIN(pin)                                              │
│  ├── selectWithdraw()                                           │
│  ├── enterAmount(amount)                                        │
│  ├── confirmTransaction()                                       │
│  └── takeReceipt()                                              │
│                                                                 │
│  NAVIGATION:                                                    │
│  └── goto() - Walk to the ATM                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
// ATMPage.ts
class ATMPage {
  readonly cardSlot: Locator;
  readonly pinPad: Locator;
  readonly screen: Locator;
  readonly cashDispenser: Locator;

  async withdrawCash(pin: string, amount: number) {
    await this.enterPIN(pin);
    await this.selectWithdraw();
    await this.enterAmount(amount);
    await this.confirmTransaction();
  }

  private async enterPIN(pin: string) {
    for (const digit of pin) {
      await this.pinPad.getByRole('button', { name: digit }).click();
    }
    await this.pinPad.getByRole('button', { name: 'Enter' }).click();
  }
}

// Test - Clear intention
test('withdraw money for groceries', async () => {
  const atm = new ATMPage(page);
  await atm.insertCard('1234-5678-9012-3456');
  await atm.withdrawCash('1234', 100);
  await expect(atm.cashDispenser).toContainText('Please take your cash');
});
```

---

### Analogy 4: Recipe Book 📖

Think of Page Objects as **recipes** in a cookbook.

| Cookbook Concept | POM Equivalent |
|------------------|----------------|
| Recipe Name | Page Object Class Name |
| Ingredients List | Locators |
| Step-by-step Instructions | Methods |
| "Prep the kitchen" | `goto()` method |
| Chef's Notes | Comments |

**Without Recipe (No POM):**
Every time you cook, you figure it out from scratch. Different results each time.

**With Recipe (With POM):**
Follow the recipe → Consistent results every time!

---

### Analogy 5: Smartphone Apps 📱

Each **app** on your phone is like a Page Object!

```
┌──────────────────────────────────────────────────────────────────┐
│                        YOUR PHONE                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  📷 CameraApp (Page Object)                                      │
│     ├── shutterButton                                            │
│     ├── switchCameraButton                                       │
│     ├── takePhoto()                                              │
│     └── switchToVideo()                                          │
│                                                                  │
│  💬 MessagesApp (Page Object)                                    │
│     ├── composeButton                                            │
│     ├── messageInput                                             │
│     ├── sendMessage(to, text)                                    │
│     └── openConversation(contact)                                │
│                                                                  │
│  🛒 ShoppingApp (Page Object)                                    │
│     ├── searchBar                                                │
│     ├── cartIcon                                                 │
│     ├── searchProduct(query)                                     │
│     ├── addToCart(product)                                       │
│     └── checkout()                                               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

Each app:
- Has its own **interface** (locators)
- Has its own **functions** (methods)
- Is **independent** from other apps
- Can be **updated** without affecting how you use it

---

## Why POM Matters - The Problems It Solves

### Problem 1: The Maintenance Nightmare

```
SCENARIO: Your app has 100 tests. The login button ID changes from 
"#login-btn" to "#sign-in-button"

WITHOUT POM:
┌─────────────────────────────────────────────────────────────────┐
│  ❌ Find and replace in 100 test files                          │
│  ❌ Risk missing some occurrences                                │
│  ❌ Hours of work                                                │
│  ❌ High chance of errors                                        │
└─────────────────────────────────────────────────────────────────┘

WITH POM:
┌─────────────────────────────────────────────────────────────────┐
│  ✅ Change ONE line in LoginPage.ts                             │
│  ✅ All 100 tests automatically use the new locator             │
│  ✅ 30 seconds of work                                          │
│  ✅ Zero risk of missing anything                               │
└─────────────────────────────────────────────────────────────────┘
```

### Problem 2: Code Duplication

```typescript
// WITHOUT POM - Same code repeated everywhere

// test1.spec.ts
await page.getByLabel('Email').fill('user@test.com');
await page.getByLabel('Password').fill('pass123');
await page.getByRole('button', { name: 'Sign In' }).click();

// test2.spec.ts
await page.getByLabel('Email').fill('admin@test.com');
await page.getByLabel('Password').fill('admin123');
await page.getByRole('button', { name: 'Sign In' }).click();

// test3.spec.ts
await page.getByLabel('Email').fill('guest@test.com');
await page.getByLabel('Password').fill('guest123');
await page.getByRole('button', { name: 'Sign In' }).click();

// ... repeated 50 more times
```

```typescript
// WITH POM - Write once, use everywhere

// test1.spec.ts
await loginPage.login('user@test.com', 'pass123');

// test2.spec.ts
await loginPage.login('admin@test.com', 'admin123');

// test3.spec.ts
await loginPage.login('guest@test.com', 'guest123');
```

### Problem 3: Readability

```typescript
// WITHOUT POM - What is this test doing? 🤔
test('test case 1', async ({ page }) => {
  await page.goto('/app');
  await page.locator('#nav-menu').click();
  await page.locator('.dropdown-item:nth-child(3)').click();
  await page.locator('input[name="q"]').fill('laptop');
  await page.locator('form').locator('button').click();
  await page.locator('.product-card').first().locator('.add-btn').click();
  await page.locator('#cart-icon').click();
  await page.locator('.checkout-btn').click();
  // ... what is happening?!
});

// WITH POM - Crystal clear! ✨
test('user can search and purchase a product', async ({ page }) => {
  await homePage.goto();
  await homePage.navigateToProducts();
  await productsPage.searchFor('laptop');
  await productsPage.addFirstProductToCart();
  await cartPage.proceedToCheckout();
});
```

---

## POM Architecture & Principles

### The Three Pillars of POM

```
┌─────────────────────────────────────────────────────────────────┐
│                    THREE PILLARS OF POM                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│         ┌───────────┐                                           │
│         │ LOCATORS  │                                           │
│         │           │                                           │
│         │ WHERE are │                                           │
│         │ elements? │                                           │
│         └─────┬─────┘                                           │
│               │                                                 │
│    ┌──────────┴──────────┐                                      │
│    │                     │                                      │
│    ▼                     ▼                                      │
│ ┌───────────┐     ┌───────────┐                                 │
│ │  ACTIONS  │     │NAVIGATION │                                 │
│ │           │     │           │                                 │
│ │ WHAT can  │     │ HOW to    │                                 │
│ │ you do?   │     │ get here? │                                 │
│ └───────────┘     └───────────┘                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Core Principles

#### Principle 1: Single Responsibility
Each page object handles **ONE page or component only**.

```
✅ CORRECT:
LoginPage     → Only login-related elements and actions
DashboardPage → Only dashboard-related elements and actions
ProfilePage   → Only profile-related elements and actions

❌ WRONG:
AppPage → Contains login, dashboard, profile, settings, everything...
```

#### Principle 2: Encapsulation
Hide the "how" (locators, complex logic) and expose the "what" (simple methods).

```typescript
// Page Object HIDES complexity
class CheckoutPage {
  // Private - hidden from tests
  private readonly cardNumberInput: Locator;
  private readonly expiryInput: Locator;
  private readonly cvvInput: Locator;
  private readonly billingForm: Locator;

  // Public - exposed to tests
  async fillPaymentDetails(card: string, expiry: string, cvv: string) {
    await this.cardNumberInput.fill(card);
    await this.expiryInput.fill(expiry);
    await this.cvvInput.fill(cvv);
  }
}

// Test doesn't know about individual inputs
test('complete payment', async () => {
  await checkoutPage.fillPaymentDetails('4111111111111111', '12/25', '123');
});
```

#### Principle 3: Reusability
Write methods that can be used across multiple tests.

```typescript
class ProductPage {
  // Reusable across many tests
  async addToCart(productName: string) { }
  async removeFromCart(productName: string) { }
  async getProductPrice(productName: string): Promise<string> { }
  async filterByCategory(category: string) { }
  async sortBy(option: string) { }
}
```

#### Principle 4: No Assertions in Page Objects
Keep assertions in tests, not in page objects (with rare exceptions).

```typescript
// ❌ WRONG - Assertion inside page object
class LoginPage {
  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
    await expect(this.page).toHaveURL('/dashboard'); // ❌ Don't do this!
  }
}

// ✅ CORRECT - Assertion in test
class LoginPage {
  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}

test('successful login', async ({ page }) => {
  await loginPage.login('user@test.com', 'password');
  await expect(page).toHaveURL('/dashboard'); // ✅ Assertion in test
});
```

---

## Folder Structure & Organization

### Recommended Project Structure

```
playwright-framework/
│
├── 📁 tests/                          # All test files
│   ├── 📁 auth/
│   │   ├── login.spec.ts
│   │   ├── logout.spec.ts
│   │   └── registration.spec.ts
│   │
│   ├── 📁 products/
│   │   ├── search.spec.ts
│   │   ├── filters.spec.ts
│   │   └── product-details.spec.ts
│   │
│   ├── 📁 checkout/
│   │   ├── cart.spec.ts
│   │   ├── payment.spec.ts
│   │   └── order-confirmation.spec.ts
│   │
│   └── auth.setup.ts                  # Authentication setup
│
├── 📁 pages/                          # All page objects
│   ├── BasePage.ts                    # Common functionality
│   ├── LoginPage.ts
│   ├── HomePage.ts
│   ├── ProductListPage.ts
│   ├── ProductDetailPage.ts
│   ├── CartPage.ts
│   ├── CheckoutPage.ts
│   └── OrderConfirmationPage.ts
│
├── 📁 components/                     # Reusable component objects
│   ├── HeaderComponent.ts
│   ├── FooterComponent.ts
│   ├── NavigationComponent.ts
│   ├── SearchComponent.ts
│   └── ModalComponent.ts
│
├── 📁 fixtures/                       # Custom test fixtures
│   └── test-fixtures.ts
│
├── 📁 utils/                          # Helper utilities
│   ├── test-data.ts
│   ├── api-helpers.ts
│   └── date-helpers.ts
│
├── 📁 data/                           # Test data files
│   ├── users.json
│   ├── products.json
│   └── addresses.json
│
├── 📁 playwright/
│   └── 📁 .auth/                      # Auth state (gitignored)
│       └── user.json
│
├── playwright.config.ts               # Playwright configuration
├── package.json
├── tsconfig.json
└── .gitignore
```

### Visual Representation of Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROJECT ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                      ┌─────────────┐                            │
│                      │   TESTS     │                            │
│                      │ (spec files)│                            │
│                      └──────┬──────┘                            │
│                             │                                   │
│                             │ uses                              │
│                             ▼                                   │
│     ┌───────────────────────────────────────────┐               │
│     │              PAGE OBJECTS                  │               │
│     │  ┌─────────┐ ┌─────────┐ ┌─────────┐     │               │
│     │  │ Login   │ │ Product │ │ Cart    │     │               │
│     │  │ Page    │ │ Page    │ │ Page    │ ... │               │
│     │  └────┬────┘ └────┬────┘ └────┬────┘     │               │
│     │       │           │           │           │               │
│     │       └───────────┴───────────┘           │               │
│     │                   │                       │               │
│     │                   ▼                       │               │
│     │            ┌───────────┐                  │               │
│     │            │ BasePage  │                  │               │
│     │            └───────────┘                  │               │
│     └───────────────────────────────────────────┘               │
│                             │                                   │
│                             │ uses                              │
│                             ▼                                   │
│     ┌───────────────────────────────────────────┐               │
│     │              COMPONENTS                    │               │
│     │  ┌─────────┐ ┌─────────┐ ┌─────────┐     │               │
│     │  │ Header  │ │ Search  │ │ Modal   │ ... │               │
│     │  └─────────┘ └─────────┘ └─────────┘     │               │
│     └───────────────────────────────────────────┘               │
│                             │                                   │
│                             │ uses                              │
│                             ▼                                   │
│     ┌───────────────────────────────────────────┐               │
│     │         UTILITIES & TEST DATA             │               │
│     │  ┌─────────┐ ┌─────────┐ ┌─────────┐     │               │
│     │  │ helpers │ │  data   │ │ fixtures│     │               │
│     │  └─────────┘ └─────────┘ └─────────┘     │               │
│     └───────────────────────────────────────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### When to Create Separate Files

```
┌─────────────────────────────────────────────────────────────────┐
│              DECISION GUIDE: WHEN TO SPLIT FILES                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Create a NEW PAGE OBJECT when:                                 │
│  ├── URL changes (different page)                               │
│  ├── Major context switch (login → dashboard)                   │
│  └── Page has 10+ unique interactions                           │
│                                                                 │
│  Create a COMPONENT when:                                       │
│  ├── Element appears on multiple pages (header, footer)         │
│  ├── Reusable widget (modal, dropdown, datepicker)              │
│  └── Complex nested structure used in many places               │
│                                                                 │
│  Keep in SAME file when:                                        │
│  ├── Small, related functionality                               │
│  ├── Less than 5 methods                                        │
│  └── Only used by one page                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Implementation Guide

### Step 1: Analyze the Page

Before writing any code, **study the page** you're automating.

```
┌─────────────────────────────────────────────────────────────────┐
│                   PAGE ANALYSIS CHECKLIST                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  □ What is the URL?                                             │
│  □ What elements can users interact with?                       │
│  □ What actions can users perform?                              │
│  □ What are the expected outcomes?                              │
│  □ Are there any reusable components (header, footer)?          │
│  □ Does this page navigate to other pages?                      │
│  □ What data does this page display?                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Example Analysis - Login Page:**

| Question | Answer |
|----------|--------|
| URL | `/login` |
| Interactive Elements | Email input, Password input, Submit button, Remember me checkbox, Forgot password link |
| User Actions | Fill credentials, Submit form, Check remember me, Navigate to forgot password |
| Expected Outcomes | Successful login → Dashboard, Failed login → Error message |
| Reusable Components | Header (logo, nav links) |
| Navigation | Goes to Dashboard, Forgot Password, or Registration |

### Step 2: Create the Base Page (Optional but Recommended)

```typescript
// pages/BasePage.ts
import { Page, Locator } from '@playwright/test';

export class BasePage {
  readonly page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // Common navigation helper
  async navigateTo(path: string) {
    await this.page.goto(path);
  }

  // Wait for page to be fully loaded
  async waitForPageLoad() {
    await this.page.waitForLoadState('networkidle');
  }

  // Get current URL
  async getCurrentUrl(): Promise<string> {
    return this.page.url();
  }

  // Take screenshot (useful for debugging)
  async takeScreenshot(name: string) {
    await this.page.screenshot({ path: `screenshots/${name}.png` });
  }

  // Common wait helper
  async waitForElement(locator: Locator, timeout = 5000) {
    await locator.waitFor({ state: 'visible', timeout });
  }
}
```

### Step 3: Create the Page Object

```typescript
// pages/LoginPage.ts
import { Page, Locator, expect } from '@playwright/test';
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  // ============================================
  // LOCATORS - Define all elements
  // ============================================
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly rememberMeCheckbox: Locator;
  readonly forgotPasswordLink: Locator;
  readonly signUpLink: Locator;
  readonly errorMessage: Locator;
  readonly pageHeading: Locator;

  // ============================================
  // CONSTRUCTOR - Initialize locators
  // ============================================
  constructor(page: Page) {
    super(page);
    
    // Use semantic locators (getByRole, getByLabel)
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Sign In' });
    this.rememberMeCheckbox = page.getByRole('checkbox', { name: 'Remember me' });
    this.forgotPasswordLink = page.getByRole('link', { name: 'Forgot password' });
    this.signUpLink = page.getByRole('link', { name: 'Sign up' });
    this.errorMessage = page.getByRole('alert');
    this.pageHeading = page.getByRole('heading', { name: 'Login' });
  }

  // ============================================
  // NAVIGATION - How to get to this page
  // ============================================
  async goto() {
    await this.navigateTo('/login');
    await this.waitForElement(this.submitButton);
  }

  // ============================================
  // ACTIONS - What users can do
  // ============================================
  
  /**
   * Fill the email field
   */
  async fillEmail(email: string) {
    await this.emailInput.fill(email);
  }

  /**
   * Fill the password field
   */
  async fillPassword(password: string) {
    await this.passwordInput.fill(password);
  }

  /**
   * Click the submit button
   */
  async clickSubmit() {
    await this.submitButton.click();
  }

  /**
   * Complete login with email and password
   */
  async login(email: string, password: string) {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.clickSubmit();
  }

  /**
   * Login with Remember Me checked
   */
  async loginWithRememberMe(email: string, password: string) {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.rememberMeCheckbox.check();
    await this.clickSubmit();
  }

  /**
   * Click forgot password link
   */
  async clickForgotPassword() {
    await this.forgotPasswordLink.click();
  }

  /**
   * Click sign up link
   */
  async clickSignUp() {
    await this.signUpLink.click();
  }

  // ============================================
  // GETTERS - Retrieve information
  // ============================================

  /**
   * Get the error message text
   */
  async getErrorMessageText(): Promise<string> {
    return (await this.errorMessage.textContent()) || '';
  }

  /**
   * Check if login form is displayed
   */
  async isLoginFormVisible(): Promise<boolean> {
    return await this.submitButton.isVisible();
  }
}
```

### Step 4: Create the Test File

```typescript
// tests/auth/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../../pages/LoginPage';
import { DashboardPage } from '../../pages/DashboardPage';

test.describe('Login Page', () => {
  let loginPage: LoginPage;
  let dashboardPage: DashboardPage;

  test.beforeEach(async ({ page }) => {
    // Initialize page objects
    loginPage = new LoginPage(page);
    dashboardPage = new DashboardPage(page);
    
    // Navigate to login page before each test
    await loginPage.goto();
  });

  test('should display login form', async () => {
    // Verify all elements are visible
    await expect(loginPage.emailInput).toBeVisible();
    await expect(loginPage.passwordInput).toBeVisible();
    await expect(loginPage.submitButton).toBeVisible();
    await expect(loginPage.rememberMeCheckbox).toBeVisible();
  });

  test('should login successfully with valid credentials', async ({ page }) => {
    // Perform login
    await loginPage.login('valid@user.com', 'validPassword123');
    
    // Verify redirect to dashboard
    await expect(page).toHaveURL('/dashboard');
    await expect(dashboardPage.welcomeMessage).toBeVisible();
  });

  test('should show error with invalid credentials', async () => {
    // Attempt login with wrong password
    await loginPage.login('valid@user.com', 'wrongPassword');
    
    // Verify error message
    await expect(loginPage.errorMessage).toBeVisible();
    await expect(loginPage.errorMessage).toContainText('Invalid credentials');
  });

  test('should navigate to forgot password page', async ({ page }) => {
    await loginPage.clickForgotPassword();
    
    await expect(page).toHaveURL('/forgot-password');
  });

  test('should navigate to registration page', async ({ page }) => {
    await loginPage.clickSignUp();
    
    await expect(page).toHaveURL('/register');
  });
});
```

### Step 5: Integrate with Fixtures (Advanced)

```typescript
// fixtures/test-fixtures.ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';
import { ProductPage } from '../pages/ProductPage';
import { CartPage } from '../pages/CartPage';

// Define fixture types
type PageFixtures = {
  loginPage: LoginPage;
  dashboardPage: DashboardPage;
  productPage: ProductPage;
  cartPage: CartPage;
};

// Extend base test with fixtures
export const test = base.extend<PageFixtures>({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },
  
  dashboardPage: async ({ page }, use) => {
    await use(new DashboardPage(page));
  },
  
  productPage: async ({ page }, use) => {
    await use(new ProductPage(page));
  },
  
  cartPage: async ({ page }, use) => {
    await use(new CartPage(page));
  },
});

export { expect } from '@playwright/test';
```

```typescript
// tests/auth/login.spec.ts (with fixtures)
import { test, expect } from '../../fixtures/test-fixtures';

test.describe('Login Page', () => {
  test.beforeEach(async ({ loginPage }) => {
    await loginPage.goto();
  });

  test('should login successfully', async ({ loginPage, dashboardPage, page }) => {
    await loginPage.login('user@test.com', 'password');
    
    await expect(page).toHaveURL('/dashboard');
    await expect(dashboardPage.welcomeMessage).toBeVisible();
  });
});
```

---

## Progressive Examples: Beginner to Advanced

### Level 1: Basic Page Object (Beginner)

A simple page object with essential elements and one main action.

```typescript
// pages/SearchPage.ts - BEGINNER LEVEL
import { Page, Locator } from '@playwright/test';

export class SearchPage {
  readonly page: Page;
  readonly searchInput: Locator;
  readonly searchButton: Locator;
  readonly results: Locator;

  constructor(page: Page) {
    this.page = page;
    this.searchInput = page.getByPlaceholder('Search...');
    this.searchButton = page.getByRole('button', { name: 'Search' });
    this.results = page.locator('.search-results');
  }

  async search(query: string) {
    await this.searchInput.fill(query);
    await this.searchButton.click();
  }
}
```

### Level 2: Page Object with Multiple Actions (Intermediate)

```typescript
// pages/ProductPage.ts - INTERMEDIATE LEVEL
import { Page, Locator } from '@playwright/test';
import { BasePage } from './BasePage';

export class ProductPage extends BasePage {
  readonly productList: Locator;
  readonly categoryFilter: Locator;
  readonly sortDropdown: Locator;
  readonly priceRangeMin: Locator;
  readonly priceRangeMax: Locator;
  readonly applyFiltersButton: Locator;
  readonly cartBadge: Locator;

  constructor(page: Page) {
    super(page);
    this.productList = page.locator('[data-testid="product-list"]');
    this.categoryFilter = page.getByRole('combobox', { name: 'Category' });
    this.sortDropdown = page.getByRole('combobox', { name: 'Sort by' });
    this.priceRangeMin = page.getByLabel('Min Price');
    this.priceRangeMax = page.getByLabel('Max Price');
    this.applyFiltersButton = page.getByRole('button', { name: 'Apply Filters' });
    this.cartBadge = page.locator('[data-testid="cart-badge"]');
  }

  async goto() {
    await this.navigateTo('/products');
  }

  // Get a specific product card
  getProductCard(productName: string): Locator {
    return this.productList
      .locator('article')
      .filter({ hasText: productName });
  }

  async addToCart(productName: string) {
    const product = this.getProductCard(productName);
    await product.getByRole('button', { name: 'Add to Cart' }).click();
  }

  async filterByCategory(category: string) {
    await this.categoryFilter.selectOption(category);
  }

  async sortBy(option: string) {
    await this.sortDropdown.selectOption(option);
  }

  async filterByPriceRange(min: number, max: number) {
    await this.priceRangeMin.fill(min.toString());
    await this.priceRangeMax.fill(max.toString());
    await this.applyFiltersButton.click();
  }

  async getProductCount(): Promise<number> {
    return await this.productList.locator('article').count();
  }

  async getProductPrice(productName: string): Promise<string> {
    const product = this.getProductCard(productName);
    return (await product.locator('.price').textContent()) || '';
  }

  async getCartItemCount(): Promise<number> {
    const text = await this.cartBadge.textContent();
    return parseInt(text || '0', 10);
  }
}
```

### Level 3: Page Object with Components (Advanced)

```typescript
// components/HeaderComponent.ts
import { Page, Locator } from '@playwright/test';

export class HeaderComponent {
  readonly page: Page;
  readonly logo: Locator;
  readonly searchInput: Locator;
  readonly searchButton: Locator;
  readonly cartIcon: Locator;
  readonly cartBadge: Locator;
  readonly userMenu: Locator;
  readonly logoutButton: Locator;

  constructor(page: Page) {
    this.page = page;
    const header = page.getByRole('banner');
    
    this.logo = header.getByRole('img', { name: 'Logo' });
    this.searchInput = header.getByPlaceholder('Search products');
    this.searchButton = header.getByRole('button', { name: 'Search' });
    this.cartIcon = header.getByRole('link', { name: 'Cart' });
    this.cartBadge = header.locator('[data-testid="cart-count"]');
    this.userMenu = header.getByRole('button', { name: 'User menu' });
    this.logoutButton = header.getByRole('menuitem', { name: 'Logout' });
  }

  async search(query: string) {
    await this.searchInput.fill(query);
    await this.searchButton.click();
  }

  async goToCart() {
    await this.cartIcon.click();
  }

  async getCartCount(): Promise<number> {
    const text = await this.cartBadge.textContent();
    return parseInt(text || '0', 10);
  }

  async logout() {
    await this.userMenu.click();
    await this.logoutButton.click();
  }

  async goToHome() {
    await this.logo.click();
  }
}
```

```typescript
// components/ModalComponent.ts
import { Page, Locator } from '@playwright/test';

export class ModalComponent {
  readonly page: Page;
  readonly overlay: Locator;
  readonly content: Locator;
  readonly closeButton: Locator;
  readonly title: Locator;

  constructor(page: Page) {
    this.page = page;
    this.overlay = page.locator('[data-testid="modal-overlay"]');
    this.content = page.getByRole('dialog');
    this.closeButton = this.content.getByRole('button', { name: 'Close' });
    this.title = this.content.getByRole('heading');
  }

  async isOpen(): Promise<boolean> {
    return await this.content.isVisible();
  }

  async close() {
    await this.closeButton.click();
  }

  async closeByClickingOutside() {
    await this.overlay.click({ position: { x: 10, y: 10 } });
  }

  async getTitle(): Promise<string> {
    return (await this.title.textContent()) || '';
  }

  // Generic method to get content
  getContent(): Locator {
    return this.content;
  }
}
```

```typescript
// pages/DashboardPage.ts - ADVANCED LEVEL (with components)
import { Page, Locator } from '@playwright/test';
import { BasePage } from './BasePage';
import { HeaderComponent } from '../components/HeaderComponent';
import { ModalComponent } from '../components/ModalComponent';

export class DashboardPage extends BasePage {
  // Components
  readonly header: HeaderComponent;
  readonly modal: ModalComponent;

  // Page-specific locators
  readonly welcomeMessage: Locator;
  readonly recentOrders: Locator;
  readonly quickActions: Locator;
  readonly notifications: Locator;

  constructor(page: Page) {
    super(page);
    
    // Initialize components
    this.header = new HeaderComponent(page);
    this.modal = new ModalComponent(page);

    // Page-specific elements
    this.welcomeMessage = page.getByRole('heading', { name: /welcome/i });
    this.recentOrders = page.locator('[data-testid="recent-orders"]');
    this.quickActions = page.locator('[data-testid="quick-actions"]');
    this.notifications = page.locator('[data-testid="notifications"]');
  }

  async goto() {
    await this.navigateTo('/dashboard');
  }

  async getUserName(): Promise<string> {
    const text = await this.welcomeMessage.textContent();
    const match = text?.match(/Welcome,\s+(.+?)!/);
    return match ? match[1] : '';
  }

  async getRecentOrderCount(): Promise<number> {
    return await this.recentOrders.locator('.order-item').count();
  }

  async clickQuickAction(actionName: string) {
    await this.quickActions
      .getByRole('button', { name: actionName })
      .click();
  }

  async getNotificationCount(): Promise<number> {
    return await this.notifications.locator('.notification-item').count();
  }

  // Use header component methods
  async searchProducts(query: string) {
    await this.header.search(query);
  }

  async logout() {
    await this.header.logout();
  }
}
```

### Level 4: Page Object with Page Transitions (Expert)

```typescript
// pages/CheckoutPage.ts - EXPERT LEVEL (with transitions)
import { Page, Locator } from '@playwright/test';
import { BasePage } from './BasePage';
import { OrderConfirmationPage } from './OrderConfirmationPage';

export class CheckoutPage extends BasePage {
  // Step indicators
  readonly stepIndicator: Locator;
  
  // Shipping step
  readonly shippingForm: Locator;
  readonly firstNameInput: Locator;
  readonly lastNameInput: Locator;
  readonly addressInput: Locator;
  readonly cityInput: Locator;
  readonly zipInput: Locator;
  readonly countrySelect: Locator;
  readonly continueToPaymentButton: Locator;

  // Payment step
  readonly paymentForm: Locator;
  readonly cardNumberInput: Locator;
  readonly expiryInput: Locator;
  readonly cvvInput: Locator;
  readonly placeOrderButton: Locator;

  // Order summary
  readonly orderSummary: Locator;
  readonly subtotal: Locator;
  readonly shipping: Locator;
  readonly tax: Locator;
  readonly total: Locator;

  constructor(page: Page) {
    super(page);
    
    this.stepIndicator = page.locator('[data-testid="checkout-steps"]');
    
    // Shipping
    this.shippingForm = page.locator('[data-testid="shipping-form"]');
    this.firstNameInput = page.getByLabel('First Name');
    this.lastNameInput = page.getByLabel('Last Name');
    this.addressInput = page.getByLabel('Address');
    this.cityInput = page.getByLabel('City');
    this.zipInput = page.getByLabel('ZIP Code');
    this.countrySelect = page.getByRole('combobox', { name: 'Country' });
    this.continueToPaymentButton = page.getByRole('button', { name: 'Continue to Payment' });

    // Payment
    this.paymentForm = page.locator('[data-testid="payment-form"]');
    this.cardNumberInput = page.getByLabel('Card Number');
    this.expiryInput = page.getByLabel('Expiry Date');
    this.cvvInput = page.getByLabel('CVV');
    this.placeOrderButton = page.getByRole('button', { name: 'Place Order' });

    // Summary
    this.orderSummary = page.locator('[data-testid="order-summary"]');
    this.subtotal = page.locator('[data-testid="subtotal"]');
    this.shipping = page.locator('[data-testid="shipping-cost"]');
    this.tax = page.locator('[data-testid="tax"]');
    this.total = page.locator('[data-testid="total"]');
  }

  async goto() {
    await this.navigateTo('/checkout');
  }

  // Shipping methods
  async fillShippingInfo(info: {
    firstName: string;
    lastName: string;
    address: string;
    city: string;
    zip: string;
    country: string;
  }) {
    await this.firstNameInput.fill(info.firstName);
    await this.lastNameInput.fill(info.lastName);
    await this.addressInput.fill(info.address);
    await this.cityInput.fill(info.city);
    await this.zipInput.fill(info.zip);
    await this.countrySelect.selectOption(info.country);
  }

  async continueToPayment() {
    await this.continueToPaymentButton.click();
    await this.paymentForm.waitFor({ state: 'visible' });
  }

  // Payment methods
  async fillPaymentInfo(info: {
    cardNumber: string;
    expiry: string;
    cvv: string;
  }) {
    await this.cardNumberInput.fill(info.cardNumber);
    await this.expiryInput.fill(info.expiry);
    await this.cvvInput.fill(info.cvv);
  }

  /**
   * Places the order and returns the OrderConfirmationPage
   * This is an example of a page transition method
   */
  async placeOrder(): Promise<OrderConfirmationPage> {
    await this.placeOrderButton.click();
    
    // Wait for navigation to confirmation page
    await this.page.waitForURL(/\/order-confirmation/);
    
    // Return the new page object
    return new OrderConfirmationPage(this.page);
  }

  // Complete checkout flow in one method
  async completeCheckout(
    shippingInfo: Parameters<typeof this.fillShippingInfo>[0],
    paymentInfo: Parameters<typeof this.fillPaymentInfo>[0]
  ): Promise<OrderConfirmationPage> {
    await this.fillShippingInfo(shippingInfo);
    await this.continueToPayment();
    await this.fillPaymentInfo(paymentInfo);
    return await this.placeOrder();
  }

  // Order summary methods
  async getTotal(): Promise<string> {
    return (await this.total.textContent()) || '';
  }

  async getCurrentStep(): Promise<string> {
    const activeStep = this.stepIndicator.locator('.active');
    return (await activeStep.textContent()) || '';
  }
}
```

---

## Common Patterns & Best Practices

### Pattern 1: Method Chaining (Fluent Interface)

```typescript
class FormPage extends BasePage {
  async fillName(name: string): Promise<FormPage> {
    await this.nameInput.fill(name);
    return this;
  }

  async fillEmail(email: string): Promise<FormPage> {
    await this.emailInput.fill(email);
    return this;
  }

  async selectCountry(country: string): Promise<FormPage> {
    await this.countrySelect.selectOption(country);
    return this;
  }

  async submit() {
    await this.submitButton.click();
  }
}

// Usage - fluent chain
await formPage
  .fillName('John')
  .then(p => p.fillEmail('john@test.com'))
  .then(p => p.selectCountry('USA'))
  .then(p => p.submit());

// Or simpler:
await formPage.fillName('John');
await formPage.fillEmail('john@test.com');
await formPage.selectCountry('USA');
await formPage.submit();
```

### Pattern 2: Dynamic Element Locators

```typescript
class DataTablePage extends BasePage {
  // Dynamic locator methods for tables
  getRowByText(text: string): Locator {
    return this.page.getByRole('row', { name: new RegExp(text, 'i') });
  }

  getRowById(id: string): Locator {
    return this.page.locator(`tr[data-id="${id}"]`);
  }

  getCellInRow(rowText: string, columnName: string): Locator {
    const columnIndex = this.getColumnIndex(columnName);
    return this.getRowByText(rowText).locator(`td:nth-child(${columnIndex})`);
  }

  async getColumnIndex(columnName: string): Promise<number> {
    const headers = this.page.getByRole('columnheader');
    const count = await headers.count();
    
    for (let i = 0; i < count; i++) {
      const text = await headers.nth(i).textContent();
      if (text?.includes(columnName)) {
        return i + 1;
      }
    }
    throw new Error(`Column ${columnName} not found`);
  }
}
```

### Pattern 3: Wait Helpers

```typescript
class BasePage {
  // Wait for specific conditions
  async waitForLoading() {
    // Wait for loading spinner to appear and disappear
    const spinner = this.page.locator('.loading-spinner');
    await spinner.waitFor({ state: 'visible' }).catch(() => {});
    await spinner.waitFor({ state: 'hidden' });
  }

  async waitForToast(message?: string) {
    const toast = this.page.getByRole('alert');
    await toast.waitFor({ state: 'visible' });
    if (message) {
      await expect(toast).toContainText(message);
    }
    return toast;
  }

  async waitForTableLoad() {
    // Wait for table skeleton/loading to finish
    await this.page.locator('table tbody tr').first().waitFor();
  }
}
```

### Pattern 4: Test Data Objects

```typescript
// data/test-users.ts
export interface UserCredentials {
  email: string;
  password: string;
}

export interface ShippingAddress {
  firstName: string;
  lastName: string;
  address: string;
  city: string;
  zip: string;
  country: string;
}

export const testUsers = {
  admin: {
    email: 'admin@test.com',
    password: 'Admin123!',
  },
  standard: {
    email: 'user@test.com',
    password: 'User123!',
  },
} as const;

export const testAddresses: Record<string, ShippingAddress> = {
  us: {
    firstName: 'John',
    lastName: 'Doe',
    address: '123 Main St',
    city: 'New York',
    zip: '10001',
    country: 'USA',
  },
  uk: {
    firstName: 'Jane',
    lastName: 'Smith',
    address: '456 High Street',
    city: 'London',
    zip: 'SW1A 1AA',
    country: 'UK',
  },
};
```

```typescript
// Usage in tests
import { testUsers, testAddresses } from '../data/test-users';

test('checkout with US address', async ({ checkoutPage }) => {
  await checkoutPage.fillShippingInfo(testAddresses.us);
});
```

---

## Anti-Patterns to Avoid

### ❌ Anti-Pattern 1: God Page Object

```typescript
// ❌ BAD - One massive page object for everything
class AppPage {
  // Login elements
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  
  // Dashboard elements
  readonly welcomeMessage: Locator;
  readonly recentOrders: Locator;
  
  // Product elements
  readonly productList: Locator;
  readonly addToCartButton: Locator;
  
  // Cart elements
  readonly cartItems: Locator;
  readonly checkoutButton: Locator;
  
  // ... 200 more locators
  
  async login() { }
  async addToCart() { }
  async checkout() { }
  // ... 100 more methods
}
```

```typescript
// ✅ GOOD - Separate page objects
class LoginPage { /* login-specific */ }
class DashboardPage { /* dashboard-specific */ }
class ProductPage { /* product-specific */ }
class CartPage { /* cart-specific */ }
```

### ❌ Anti-Pattern 2: Assertions in Page Objects

```typescript
// ❌ BAD - Assertions inside page object
class LoginPage {
  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
    
    // ❌ Don't do this!
    await expect(this.page).toHaveURL('/dashboard');
    await expect(this.welcomeMessage).toBeVisible();
  }
}
```

```typescript
// ✅ GOOD - Assertions in tests only
class LoginPage {
  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}

// Test file
test('login redirects to dashboard', async ({ loginPage, page }) => {
  await loginPage.login('user@test.com', 'password');
  
  // ✅ Assertions belong in tests
  await expect(page).toHaveURL('/dashboard');
});
```

### ❌ Anti-Pattern 3: Storing State in Page Objects

```typescript
// ❌ BAD - Storing state
class CartPage {
  private itemCount: number = 0;  // ❌ Don't store state!
  
  async addItem(product: string) {
    await this.addToCartButton.click();
    this.itemCount++;  // ❌ This will cause bugs!
  }
  
  getItemCount(): number {
    return this.itemCount;  // ❌ Returns stale data
  }
}
```

```typescript
// ✅ GOOD - Always read from the page
class CartPage {
  async addItem(product: string) {
    await this.getProductCard(product)
      .getByRole('button', { name: 'Add to Cart' })
      .click();
  }
  
  async getItemCount(): Promise<number> {
    // ✅ Always get fresh data from the page
    const text = await this.cartBadge.textContent();
    return parseInt(text || '0', 10);
  }
}
```

### ❌ Anti-Pattern 4: Hard-coded Waits

```typescript
// ❌ BAD - Fixed timeouts
class ProductPage {
  async addToCart(product: string) {
    await this.addButton.click();
    await this.page.waitForTimeout(2000);  // ❌ Never do this!
  }
}
```

```typescript
// ✅ GOOD - Wait for specific conditions
class ProductPage {
  async addToCart(product: string) {
    const currentCount = await this.getCartCount();
    await this.addButton.click();
    
    // ✅ Wait for observable change
    await expect(this.cartBadge).toHaveText((currentCount + 1).toString());
  }
}
```

### ❌ Anti-Pattern 5: Exposing Locators Directly in Tests

```typescript
// ❌ BAD - Test accesses internal locator details
test('add product', async ({ productPage }) => {
  // ❌ Test knows too much about internal implementation
  await productPage.productList.locator('article').first().locator('.add-btn').click();
});
```

```typescript
// ✅ GOOD - Test uses high-level methods
test('add product', async ({ productPage }) => {
  // ✅ Test only knows the action, not the implementation
  await productPage.addFirstProductToCart();
  // or
  await productPage.addToCart('Laptop');
});
```

---

## Exercises & Practice Projects

### Exercise 1: Create a Login Page Object (Beginner)

**Task:** Create a page object for a login page with:
- Email and password inputs
- Submit button
- Error message display
- "Forgot password" link

**Acceptance Criteria:**
- [ ] All locators use semantic selectors (getByRole, getByLabel)
- [ ] `login(email, password)` method works
- [ ] `getErrorMessage()` returns error text
- [ ] Extends BasePage

---

### Exercise 2: Create a Product Listing Page (Intermediate)

**Task:** Create a page object for an e-commerce product listing with:
- Product cards with name, price, and "Add to Cart" button
- Category filter dropdown
- Sort by dropdown
- Search functionality
- Cart badge showing item count

**Acceptance Criteria:**
- [ ] Can add any product by name
- [ ] Can filter by category
- [ ] Can sort products
- [ ] Can get price of any product
- [ ] Can get cart count

---

### Exercise 3: Full Checkout Flow (Advanced)

**Task:** Create page objects for a complete checkout flow:
1. CartPage - View items, update quantities, proceed to checkout
2. CheckoutPage - Shipping info, payment info, place order
3. OrderConfirmationPage - Order number, order details

**Acceptance Criteria:**
- [ ] All three pages implemented
- [ ] CheckoutPage.placeOrder() returns OrderConfirmationPage
- [ ] Complete checkout can be done with one method call
- [ ] Use TypeScript interfaces for shipping/payment data

---

### Exercise 4: Refactor Existing Tests (Expert)

**Task:** Take the following test and refactor it using POM:

```typescript
// Before - messy test
test('complete purchase', async ({ page }) => {
  await page.goto('/login');
  await page.locator('#email').fill('user@test.com');
  await page.locator('#password').fill('password123');
  await page.locator('button[type="submit"]').click();
  await page.waitForURL('/dashboard');
  
  await page.locator('.nav-link:has-text("Products")').click();
  await page.locator('.product-card:has-text("Laptop") button').click();
  await page.locator('#cart-icon').click();
  await page.locator('.checkout-btn').click();
  
  await page.locator('#first-name').fill('John');
  await page.locator('#last-name').fill('Doe');
  await page.locator('#address').fill('123 Main St');
  await page.locator('#city').fill('NYC');
  await page.locator('#zip').fill('10001');
  await page.locator('.continue-btn').click();
  
  await page.locator('#card-number').fill('4111111111111111');
  await page.locator('#expiry').fill('12/25');
  await page.locator('#cvv').fill('123');
  await page.locator('.place-order-btn').click();
  
  await expect(page.locator('.confirmation-message')).toBeVisible();
});
```

**Acceptance Criteria:**
- [ ] Create LoginPage, ProductPage, CartPage, CheckoutPage, ConfirmationPage
- [ ] Test should be under 15 lines
- [ ] Test should be readable without comments

---

## Mastery Checklist

### Beginner Level ✅
- [ ] Understand what POM is and why it's useful
- [ ] Can create a basic page object with locators
- [ ] Can write simple methods like `login()`, `search()`
- [ ] Use semantic locators (getByRole, getByLabel)
- [ ] Understand constructor and locator initialization

### Intermediate Level ✅
- [ ] Can create BasePage with common functionality
- [ ] Can use page objects in tests with fixtures
- [ ] Understand the difference between locators and actions
- [ ] Can create dynamic locator methods
- [ ] Can handle forms with multiple inputs
- [ ] Follow single responsibility principle

### Advanced Level ✅
- [ ] Can create reusable component objects
- [ ] Can compose page objects from components
- [ ] Understand page transitions (returning new page objects)
- [ ] Can use TypeScript interfaces for test data
- [ ] Can refactor existing tests to use POM
- [ ] Know all anti-patterns and avoid them

### Expert Level ✅
- [ ] Can design POM architecture for large applications
- [ ] Can implement method chaining (fluent interface)
- [ ] Can create custom fixtures that provide page objects
- [ ] Can handle complex scenarios (modals, dynamic content)
- [ ] Can mentor others on POM best practices
- [ ] Can evaluate trade-offs in POM design decisions

---

## Summary: The POM Philosophy

```
┌─────────────────────────────────────────────────────────────────┐
│                    THE POM PHILOSOPHY                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  "Write tests that describe WHAT you want to test,             │
│   not HOW to interact with the page."                          │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │   TEST says:    "Log in as admin"                       │   │
│  │   PAGE knows:   Where inputs are, how to submit         │   │
│  │                                                         │   │
│  │   TEST says:    "Add laptop to cart"                    │   │
│  │   PAGE knows:   How to find product, click button       │   │
│  │                                                         │   │
│  │   TEST says:    "Complete checkout"                     │   │
│  │   PAGE knows:   All the steps involved                  │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Remember:                                                      │
│  • One page = One class                                         │
│  • Locators in page, assertions in tests                        │
│  • Methods should be named like user actions                    │
│  • When UI changes, only page objects change                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

*This guide is part of the Playwright Training Curriculum. Practice each level before moving to the next. Mastery comes through repetition and real-world application.*
