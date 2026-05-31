# Cypress End‑to‑End Testing: A Complete Beginner’s Guide with LiliShop

> **From zero knowledge to a professional, production‑ready test suite**  
> Learn Cypress from the ground up and apply enterprise‑grade patterns to your Angular application.

---

**Test code availability**  
All test files discussed in this guide are available in the project’s GitHub repository:  
[LiliShop‑frontend‑angular / cypress](https://github.com/jahanalem/LiliShop-frontend-angular/tree/main/cypress)

---


## Table of Contents

1. [What Is Cypress and Why Is It Important?](#what-is-cypress-and-why-is-it-important)  
2. [Cypress Architecture in a Nutshell](#cypress-architecture-in-a-nutshell)  
3. [Phase 1: Installing Cypress in the LiliShop Angular Project](#phase-1-installing-cypress-in-the-lilishop-angular-project)  
   - [Prerequisites](#prerequisites)  
   - [Installation Command](#installation-command)  
   - [Opening the Interactive Launchpad](#opening-the-interactive-launchpad)  
   - [Key Files and Directories Created](#key-files-and-directories-created)  
   - [Understanding `cypress.config.ts`](#understanding-cypressconfigts)  
4. [Essential Cypress Commands – The Building Blocks](#essential-cypress-commands--the-building-blocks)  
   - [Navigation: `cy.visit()`](#navigation-cyvisit)  
   - [Selecting Elements: `cy.get()`](#selecting-elements-cyget)  
   - [Finding by Text: `cy.contains()`](#finding-by-text-cycontains)  
   - [Interacting with Elements: `cy.click()` and `cy.type()`](#interacting-with-elements-cyclick-and-cytype)  
   - [Assertions: `.should()`](#assertions-should)  
   - [Waiting: `cy.wait()`](#waiting-cywait)  
   - [Network Control: `cy.intercept()`](#network-control-cyintercept)  
   - [Working with URLs: `cy.location()`](#working-with-urls-cylocation)  
   - [Narrowing Down Selections: `.find()`](#narrowing-down-selections-find)  
   - [Creating Aliases: `.as()`](#creating-aliases-as)  
   - [Test Structure: `describe()`, `beforeEach()` and `it()`](#test-structure-describe-beforeeach-and-it)  
5. [From Simple Tests to Professional Architecture](#from-simple-tests-to-professional-architecture)  
   - [The problem with raw selectors](#the-problem-with-raw-selectors)  
   - [Step 1: Introduce `data-cy` attributes in your components](#step-1-introduce-data-cy-attributes-in-your-components)  
   - [Step 2: Restructure the directory layout](#step-2-restructure-the-directory-layout)  
   - [Step 3: Isolate test data using fixtures](#step-3-isolate-test-data-using-fixtures)  
   - [Step 4: Implement reusable custom commands](#step-4-implement-reusable-custom-commands)  
   - [Step 5: Refactored test files](#step-5-refactored-test-files)  
6. [How to Run Your Tests](#how-to-run-your-tests)


   

<img width="3068" height="1096" alt="Cypress-End-to-End-Testing_register" src="https://github.com/user-attachments/assets/039eb9d0-fc5d-42c4-bcd8-115038a84fcf" />

<img width="3066" height="1822" alt="Cypress-End-to-End-Testing_login" src="https://github.com/user-attachments/assets/bb650cb6-9769-48fe-8f26-1fc5280836e3" />


---

## What Is Cypress and Why Is It Important?

Cypress is a **modern, JavaScript‑based end‑to‑end testing framework** built specifically for web applications. Unlike older tools (such as Selenium) that run outside the browser and communicate over the network, Cypress **executes directly inside the browser**. This gives it native access to everything happening on your page – DOM elements, network requests, timers, local storage – without the need for external drivers or browser plugins.

### Why Cypress matters for your project

- **Fast, reliable tests** – Because Cypress operates in the same event loop as your application, tests are less flaky and run quickly.
- **Developer‑friendly** – It offers an interactive, visual Test Runner that lets you see exactly what happens step by step. You can time‑travel through DOM snapshots, see before/after states, and debug using familiar browser developer tools.
- **Automatic waiting** – Cypress automatically waits for commands and assertions to become true, so you rarely need to add arbitrary `sleep` or `wait` statements.
- **Powerful network control** – You can stub, spy on, and modify network requests with `cy.intercept()`, making it easy to test edge cases without a real backend.
- **Rich ecosystem** – Built‑in support for screenshots, videos, and seamless integration with continuous integration (CI) services.

In essence, Cypress enables you to test your application exactly the way your users experience it, giving you confidence that your features work correctly at every stage.

---

## Cypress Architecture in a Nutshell

Understanding how Cypress is put together helps you write more effective tests.

1. **Test Runner (GUI)** – The interactive application launched by `npx cypress open`. It lists test files, launches browsers, and displays live test execution with full debug capabilities.
2. **Node Process** – A background Node.js process that orchestrates the browser, reads configuration, manages file system operations, and communicates with the Test Runner.
3. **Browser Process** – The actual browser (Chrome, Edge, Electron, Firefox, etc.) where your application and the test code both run. Cypress injects a script that bridges your test commands and your app.
4. **Plugins / Support Layer** – The `cypress/support/` and `cypress/plugins/` directories contain code that extends Cypress (custom commands, global event listeners, etc.). They run before your test files and can modify the behaviour of Cypress itself.

Because the test code and the application share the same origin and run in the same window, Cypress can directly manipulate and observe the DOM, console logs, and network traffic – no WebDriver protocol or network lag involved.

---

## Phase 1: Installing Cypress in the LiliShop Angular Project

Let’s add Cypress to the **LiliShop‑frontend‑angular** repository step by step.

### Prerequisites

- Node.js (LTS version recommended, e.g. 18 or 20)
- A working Angular project (your LiliShop frontend is already set up)
- A package manager (npm, included with Node.js)

### Installation Command

Open a terminal in the **root folder** of your project (where `package.json` lives) and install Cypress as a development dependency:

```bash
npm install cypress --save-dev
```

This downloads Cypress and adds it to your `devDependencies`.

### Opening the Interactive Launchpad

After installation, launch the Cypress **Launchpad** – the setup wizard that helps you configure your first tests.

```bash
npx cypress open
```

The first time you run this, the Launchpad appears and asks you to choose a testing type:

- **E2E Testing** – for full end‑to‑end browser tests (what we need).  
- Component Testing – for isolated component tests (not covered here).

Select **E2E Testing**. Cypress then generates the necessary configuration and file structure for your project.

### Key Files and Directories Created

After the initial setup, Cypress adds (or modifies) these items in your project:

```text
your-project/
├── cypress/
│   ├── e2e/             <-- Your test files go here
│   ├── fixtures/        <-- Static test data (JSON, images, etc.)
│   ├── support/         <-- Custom commands, global configurations
│   │   ├── commands.ts  <-- Where you add your own reusable functions
│   │   └── e2e.ts       <-- Runs before every test file
│   └── cypress.config.ts <-- Main configuration file
```

Let’s break down each one:

- **`cypress.config.ts`** – The main configuration file. Here you define settings like the base URL, browser, viewport size, environment variables, and how Cypress should handle plugins. It is the single source of truth for your test configuration.

- **`cypress/e2e/`** – The directory where all your end‑to‑end test files live. Each file typically tests one feature or page. By convention, test files have the `.cy.ts` extension (TypeScript) or `.cy.js` (JavaScript). For example: `login.cy.ts`, `register.cy.ts`, etc. Cypress automatically discovers and lists all files inside this folder.

- **`cypress/fixtures/`** – A folder for static data that you can load inside tests. For instance, you might put a `users.json` file here with dummy user credentials and then use `cy.fixture('users.json')` to load that data. Fixtures keep your test code clean by separating data from logic.

- **`cypress/support/`** – Contains files that run **before** every test file.  
  - `e2e.ts` is the entry point for E2E tests – it is executed once before all test files. You can add global `beforeEach` hooks, import custom commands, or configure plugins here.  
  - `commands.ts` is where you define your own custom Cypress commands (like `cy.login()`). This helps avoid repeating complex sequences across tests.

### Understanding `cypress.config.ts`

Here is the configuration file generated for our LiliShop project:

```typescript
import { defineConfig } from "cypress";

export default defineConfig({
  allowCypressEnv: false,

  e2e: {
    baseUrl: 'http://localhost:4200',
    setupNodeEvents(on, config) {
      // implement node event listeners here
    },
  },
});
```

- **`defineConfig()`** – A helper function that provides TypeScript autocompletion for the Cypress configuration object.
- **`allowCypressEnv: false`** – A security setting that prevents certain environment variables from being overwritten in the test environment.
- **`e2e`** – The configuration block specific to end‑to‑end testing.
- **`baseUrl: 'http://localhost:4200'`** – The base URL of your Angular application. When you use `cy.visit('/account/login')`, Cypress automatically prepends this base URL, so you don’t have to type the full address every time. **This must match the URL where your Angular dev server is running.**
- **`setupNodeEvents(on, config)`** – A place to hook into the Node‑side lifecycle of Cypress. You can listen for events (like `before:run`, `after:screenshot`) or modify the configuration dynamically. For basic tests you can leave it empty or use plugins like code coverage.

---

## Essential Cypress Commands – The Building Blocks

Cypress provides a rich set of chainable commands. Mastering these core commands is the foundation of writing any test.

### Navigation: `cy.visit()`

```typescript
cy.visit(url)
```

Visits the given URL in the browser. If you have set a `baseUrl` in the config, you can use a relative path.

**Example:**  
`cy.visit('/account/login')` → loads `http://localhost:4200/account/login`

It automatically waits for the page’s `load` event before continuing, so your app is ready to be tested.

### Selecting Elements: `cy.get()`

```typescript
cy.get(selector)
```

Finds one or more DOM elements using a CSS selector. The command **retries** automatically until the element(s) exist in the DOM (within a default timeout of 4 seconds), making it resilient to slow rendering.

**Examples:**
- `cy.get('button[type="submit"]')` – selects the submit button  
- `cy.get('.forgot-password a')` – selects the anchor inside an element with class “forgot-password”  
- `cy.get('[data-cy="email"]')` – selects an element that has a `data-cy` attribute equal to `"email"` (recommended for stable selectors)

### Finding by Text: `cy.contains()`

```typescript
cy.contains(text)
// or
cy.contains(selector, text)
```

Returns the DOM element that contains the given text. Extremely useful for checking visible content or interacting with buttons/links by their label.

**Examples:**
- `cy.contains('Submit').click()` – clicks the element containing the text “Submit”  
- `cy.contains('mat-dialog-container', 'Email Confirmation Sent')` – asserts that the dialog contains the given text

### Interacting with Elements: `cy.click()` and `cy.type()`

```typescript
cy.get(selector).click()
cy.get(selector).type(text)
```

- **`click()`** – Clicks the element. Cypress automatically waits until the element is visible, not disabled, and not covered by another element.  
- **`type()`** – Types text into an input field. It also simulates keystrokes, triggering Angular’s `input` and `change` events, so two‑way bindings update correctly.

**Example:**
```typescript
cy.get('[data-cy="email"]').find('input').type('user@example.com');
```
This first finds the custom component with `data-cy="email"`, drills down to its `<input>` child using `.find()`, and types the email.

### Assertions: `.should()`

```typescript
cy.get(selector).should(assertion, value)
```

Asserts that something is true about the selected element(s). The command **retries** until the assertion passes or a timeout occurs. This automatic retry mechanism is the heart of Cypress’ stability.

Common assertions:
- `should('be.visible')` – element is visible
- `should('be.disabled')` – element has the `disabled` attribute
- `should('not.be.disabled')` – element is not disabled
- `should('exist')` – element exists in the DOM
- `should('include', text)` – element’s text includes the given string (works with `cy.url()` too)

### Waiting: `cy.wait()`

```typescript
cy.wait(alias)
// or
cy.wait(time)
```

Waits for a specific condition. The most powerful use is waiting for a network request that was aliased with `.as()`.

- `cy.wait('@loginRequest')` – pauses until the request aliased as `loginRequest` is completed. This ensures the backend mock was called before making further assertions.
- `cy.wait(1000)` – a fixed wait in milliseconds (generally avoid this; use aliases or retryable commands instead).

### Network Control: `cy.intercept()`

```typescript
cy.intercept(method, url, response?)
```

Spies on or stubs network requests. You can:
- **Stub a response** – provide a fake response, so your test doesn’t depend on a real backend.
- **Spy on a request** – observe that a request was made, without changing the response.
- **Wait for a request** – using `.as()` and `cy.wait()`.

**Example:**
```typescript
cy.intercept('POST', '/api/account/login', {
  statusCode: 200,
  body: { token: 'mock-jwt-token-string', email: 'user@example.com' }
}).as('loginRequest');
```
This intercepts any POST to `/api/account/login` and immediately returns a fake successful response. The `.as('loginRequest')` gives it an alias for later waiting.

### Working with URLs: `cy.location()`

```typescript
cy.location()
cy.location('pathname')
```

Gets information about the current URL. Commonly used to verify navigation after an action.

**Examples:**
- `cy.url().should('include', '/shop')` – checks that the URL contains `/shop`
- `cy.location('pathname').should('eq', '/shop')` – strictly compares the pathname

### Narrowing Down Selections: `.find()`

```typescript
cy.get(selector).find(subSelector)
```

Searches for a child element **within** the previously selected element.

**Example:**
```typescript
cy.get('[data-cy="email"]').find('input').type('user@example.com');
```

### Creating Aliases: `.as()`

```typescript
.alias(name)
```

Gives a custom name to a DOM element, a network intercept, or a route. You can then refer to it later using `cy.get('@alias')`.

### Test Structure: `describe()`, `beforeEach()` and `it()`

Cypress uses Mocha’s syntax for organising tests.

- **`describe(description, callback)`** – Groups together several test cases that belong to the same feature.
- **`beforeEach(callback)`** – Runs the given callback **before each** `it` block inside the `describe`.
- **`it(description, callback)`** – Defines a single test case (a “spec”).

**Basic blueprint:**
```typescript
describe('Feature Name', () => {
  beforeEach(() => {
    // setup code
  });

  it('should do something specific', () => {
    // test steps and assertions
  });
});
```

---

## From Simple Tests to Professional Architecture

When you first write Cypress tests, it’s natural to target elements using CSS classes, IDs, or tag attributes. While that works, it creates a fragile test suite: any change to the component’s styling or structure can break your tests, even if the functionality remains correct.

Professional test suites follow a few key principles:
- **Selectors are stable and independent** of CSS/JS changes.
- **Test data is separated** from test logic.
- **Repetitive UI actions are abstracted** into reusable commands.
- **Tests read like plain English** and focus on behaviour, not implementation details.

We’ll now transform the basic LiliShop tests into this professional architecture.

### The problem with raw selectors

Look at these typical raw selectors:
```typescript
cy.get('app-text-input[formControlName="email"]').find('input').type(...)
cy.get('button[type="submit"]').should('be.disabled');
```
If the component’s attribute names change, or if you restructure the template, these selectors break. A better approach is to use **dedicated test attributes** – `data-cy` – that you add to your HTML solely for the purpose of testing.

### Step 1: Introduce `data-cy` attributes in your components

In your Angular component templates, add `data-cy` attributes to key interactive elements.

**Login component** (simplified snippet):
```html
<div class="login-container">
  <mat-card>
    <mat-card-title>Login</mat-card-title>
    <mat-card-content>
      <form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
        <app-text-input data-cy="email" formControlName="email" [label]="'Email address'">
        </app-text-input>

        <app-text-input data-cy="password" formControlName="password" [label]="'Password'" [type]="'password'">
        </app-text-input>

        <button data-cy="submit-button" mat-raised-button color="primary"
                [disabled]="!isFormValid()" type="submit">
          <mat-icon>login</mat-icon> Sign in
        </button>

        <div class="forgot-password">
          <a data-cy="forgot-password-link" routerLink="/account/forgot-password">Forgot password?</a>
        </div>
      </form>
    </mat-card-content>
  </mat-card>
</div>
```

**Registration component** (simplified snippet):
```html
<section class="register-container">
  ...
  <form [formGroup]="registerForm" (ngSubmit)="onSubmit($event)">
    <app-text-input data-cy="display-name-container" formControlName="displayName" [label]="'Display Name'"></app-text-input>
    <app-text-input data-cy="email-container" formControlName="email" [label]="'Email address'"></app-text-input>
    <app-text-input data-cy="password-container" formControlName="password" [label]="'Password'" [type]="'password'"></app-text-input>
    <app-text-input data-cy="confirm-password-container" formControlName="confirmPassword" [label]="'Confirm Password'" [type]="'password'"></app-text-input>

    <button data-cy="submit-button" [disabled]="!isFormValid()" type="submit" mat-raised-button color="primary">
      Register
    </button>
  </form>
  ...
  <div id="buttonDiv" data-cy="google-signin-btn"></div>
</section>
```

Now your tests can target `[data-cy="email"]` instead of fragile component selectors. This decouples your test from internal DOM structure and class names.

### Step 2: Restructure the directory layout

Organize your end‑to‑end specs by feature domain.

```text
cypress/
├── e2e/
│   └── auth/
│       ├── login.cy.ts
│       └── register.cy.ts
├── fixtures/
│   └── auth-data.json
└── support/
    └── commands.ts
```

### Step 3: Isolate test data using fixtures

Create `cypress/fixtures/auth-data.json`:

```json
{
  "validUser": {
    "displayName": "John Doe",
    "email": "newuser@example.com",
    "password": "SecurePass123!",
    "token": "mock-jwt-token-string"
  },
  "mismatchedUser": {
    "displayName": "John Doe",
    "email": "john.doe@example.com",
    "password": "SecurePass123!",
    "confirmPassword": "DifferentPass123!"
  },
  "matchingFormUser": {
    "displayName": "John Doe",
    "email": "john.doe@example.com",
    "password": "SecurePass123!"
  },
  "loginUser": {
    "email": "user@example.com",
    "password": "Password123!",
    "token": "mock-jwt-token-string"
  }
}
```

Each test will load the relevant user object with `cy.fixture('auth-data')`.

### Step 4: Implement reusable custom commands

In `cypress/support/commands.ts`, we encapsulate repeated patterns:

```typescript
/// <reference types="cypress" />

/**
 * Types text into an <app-text-input> component identified by its data-cy attribute,
 * then triggers a blur event to activate Angular validations.
 * @returns The input element wrapped in a Cypress chainable.
 */
Cypress.Commands.add('typeInAppInput', (dataCy: string, text: string) => {
  return cy.get(`[data-cy="${dataCy}"]`)
    .find('input')
    .type(text)
    .blur();
});

/**
 * Waits for the Angular Material Dialog to appear, asserts it contains the expected title,
 * and clicks the first button inside the dialog (typically the "OK" button).
 * @returns The clicked button element.
 */
Cypress.Commands.add('handleMaterialDialog', (expectedTitle: string) => {
  cy.get('mat-dialog-container', { timeout: 5000 })
    .should('be.visible')
    .and('contain.text', expectedTitle);

  cy.get('mat-dialog-container').find('button').click();
});

/**
 * Fills out the email and password fields (using data-cy="email" / "password")
 * and clicks the submit button. Assumes the button becomes enabled.
 * @returns The clicked submit button element.
 */
Cypress.Commands.add('loginViaUI', (email: string, password: string) => {
  cy.typeInAppInput('email', email);
  cy.typeInAppInput('password', password);
  cy.get('[data-cy="submit-button"]').should('not.be.disabled').click();
});
```

Now register the TypeScript types so that your IDE provides autocomplete and type checking. Add this inside `cypress/support/e2e.ts` (or a dedicated `index.d.ts`):

```typescript
declare global {
  namespace Cypress {
    interface Chainable {
      typeInAppInput(dataCy: string, text: string): Chainable<JQuery<HTMLInputElement>>;
      handleMaterialDialog(expectedTitle: string): Chainable<JQuery<HTMLButtonElement>>;
      loginViaUI(email: string, password: string): Chainable<JQuery<HTMLButtonElement>>;
    }
  }
}
export {};
```

### Step 5: Refactored test files

Now the test files are clean, declarative, and resilient.

**`cypress/e2e/auth/login.cy.ts`:**
```typescript
describe('Login Feature', () => {
  beforeEach(() => {
    cy.visit('/account/login');
  });

  it('should enforce form validation rules by disabling the sign-in button initially', () => {
    cy.get('[data-cy="submit-button"]').should('be.disabled');
  });

  it('should enable the sign-in button only when valid input data is provided', () => {
    cy.fixture('auth-data').then((data) => {
      const user = data.loginUser;

      cy.typeInAppInput('email', user.email);
      cy.typeInAppInput('password', user.password);

      cy.get('[data-cy="submit-button"]').should('not.be.disabled');
    });
  });

  it('should navigate to the forgot password route when clicking the corresponding link', () => {
    cy.get('[data-cy="forgot-password-link"]').click();
    cy.location('pathname').should('eq', '/account/forgot-password');
  });

  it('should successfully submit valid credentials and redirect the user to the shop dashboard', () => {
    cy.fixture('auth-data').then((data) => {
      const user = data.loginUser;

      cy.intercept('POST', '/api/account/login', {
        statusCode: 200,
        body: { token: user.token, email: user.email }
      }).as('loginRequest');

      cy.loginViaUI(user.email, user.password);
      cy.wait('@loginRequest');

      cy.location('pathname').should('eq', '/shop');
    });
  });
});
```

**`cypress/e2e/auth/register.cy.ts`:**
```typescript
describe('Registration Feature', () => {
  beforeEach(() => {
    cy.visit('/account/register');
  });

  it('should disable the register button by default when the form is empty', () => {
    cy.get('[data-cy="submit-button"]').should('be.disabled');
  });

  it('should display the Google Sign-In container element', () => {
    cy.get('[data-cy="google-signin-btn"]').should('exist');
  });

  it('should keep the register button disabled if passwords do not match', () => {
    cy.fixture('auth-data').then((data) => {
      const user = data.mismatchedUser;

      cy.typeInAppInput('display-name-container', user.displayName);
      cy.typeInAppInput('email-container', user.email);
      cy.typeInAppInput('password-container', user.password);
      cy.typeInAppInput('confirm-password-container', user.confirmPassword);

      cy.get('[data-cy="submit-button"]').should('be.disabled');
    });
  });

  it('should enable the register button when all inputs are valid and match', () => {
    cy.fixture('auth-data').then((data) => {
      const user = data.matchingFormUser;

      cy.typeInAppInput('display-name-container', user.displayName);
      cy.typeInAppInput('email-container', user.email);
      cy.typeInAppInput('password-container', user.password);
      cy.typeInAppInput('confirm-password-container', user.password);

      cy.get('[data-cy="submit-button"]').should('not.be.disabled');
    });
  });

  it('should submit registration successfully, handle the confirmation dialog, and redirect to the shop', () => {
    cy.fixture('auth-data').then((data) => {
      const user = data.validUser;

      cy.intercept('GET', '/api/account/emailexists*', { statusCode: 200, body: false }).as('emailExistsCheck');
      cy.intercept('POST', '/api/account/register', {
        statusCode: 200,
        body: { displayName: user.displayName, email: user.email, token: user.token }
      }).as('registerSuccessRequest');

      cy.typeInAppInput('display-name-container', user.displayName);
      cy.typeInAppInput('email-container', user.email);
      cy.wait('@emailExistsCheck');

      cy.typeInAppInput('password-container', user.password);
      cy.typeInAppInput('confirm-password-container', user.password);

      cy.get('[data-cy="submit-button"]').should('not.be.disabled').click();
      cy.wait('@registerSuccessRequest');

      cy.handleMaterialDialog('Email Confirmation Sent');

      cy.location('pathname').should('eq', '/shop');
    });
  });
});
```

Notice how the tests now read as a clear sequence of high‑level actions. Any changes to the underlying UI implementation only require updating the `data-cy` attributes or the custom commands – not every test case.

---

## How to Run Your Tests

Before running Cypress, your Angular application **must be running locally**.

### 1. Start the Angular development server

```bash
npm start
```

Wait until the app is available at `http://localhost:4200`.

### 2. Run Cypress in interactive mode (GUI)

In a second terminal:

```bash
npx cypress open
```

Select **E2E Testing**, choose your browser, and click on any test file to run it. The interactive runner lets you debug step by step, view DOM snapshots, and inspect network requests.

### 3. Run Cypress in headless mode (CI / terminal)

```bash
npx cypress run --spec "cypress/e2e/auth/login.cy.ts"
```

To run all E2E tests:

```bash
npx cypress run
```

