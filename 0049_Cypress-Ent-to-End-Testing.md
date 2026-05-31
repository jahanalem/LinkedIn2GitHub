# Cypress End-to-End Testing: A Complete Guide with LiliShop

> **A practical, step‑by‑step learning resource**  

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
5. [Writing Your First Tests for LiliShop](#writing-your-first-tests-for-lilishop)  
   - [Login Feature Test (`login.cy.ts`)](#login-feature-test-logincyts)  
   - [Registration Feature Test (`register.cy.ts`)](#registration-feature-test-registercyts)  
6. [How to Run Your Tests](#how-to-run-your-tests)  
7. [Next Steps and Best Practices](#next-steps-and-best-practices)

---

## What Is Cypress and Why Is It Important?

Cypress is a **modern, JavaScript‑based end‑to‑end testing framework** designed specifically for web applications. Unlike older tools (like Selenium) that run outside the browser and communicate over the network, Cypress **executes directly inside the browser**. This gives it native access to everything happening on your page – DOM, network requests, timers, local storage – without the need for drivers or browser plugins.

### Why Cypress matters for your project

- **Fast, reliable tests**: Because Cypress runs in the same event loop as your application, tests are less flaky and execute quickly.
- **Developer‑friendly**: It offers an interactive, visual Test Runner that lets you see what happens step by step, time‑travel through snapshots, and debug with familiar browser developer tools.
- **Automatic waiting**: Cypress automatically waits for commands and assertions, so you rarely need to add arbitrary `sleep` or `wait` statements.
- **Powerful network control**: You can stub, spy on, and modify network requests (`cy.intercept()`), making it easy to test edge cases without a real backend.
- **Rich ecosystem**: It comes with built‑in support for screenshots, videos, and integration with continuous integration (CI) services.

In short, Cypress lets you test your application exactly the way your users interact with it, giving you confidence that your features work correctly.

---

## Cypress Architecture in a Nutshell

Understanding how Cypress is put together helps you write better tests.

1. **Test Runner (GUI)** – The interactive application you see when you run `npx cypress open`. It lists test files, launches browsers, and displays live test execution.
2. **Node Process** – Cypress uses a background Node.js process to orchestrate the browser, read configuration, manage files, and communicate with the test runner.
3. **Browser Process** – The actual browser (Chrome, Edge, Electron, etc.) where your application and the test code both run. Cypress injects a script that bridges the test commands and your app.
4. **Plugins / Support Layer** – The `cypress/support/` and `cypress/plugins/` directories contain code that extends Cypress (custom commands, event listeners, etc.). They run before your test files and can modify the behaviour of Cypress itself.

Because the test code and the application code share the same origin and run in the same window, Cypress can directly manipulate and observe the DOM, console logs, and network traffic – no WebDriver protocol or network lag.

---

## Phase 1: Installing Cypress in the LiliShop Angular Project

Let’s add Cypress to the **LiliShop‑frontend‑angular** repository step by step.

### Prerequisites

- Node.js (LTS version recommended, e.g. 18 or 20)
- A working Angular project (yours is already set up)
- A package manager (npm, included with Node.js)

### Installation Command

Open a terminal, navigate to the **root folder** of your project (where `package.json` lives) and install Cypress as a development dependency:

```bash
npm install cypress --save-dev
```

This downloads Cypress and adds it to your `devDependencies`.

### Opening the Interactive Launchpad

After installation, you can launch the Cypress **Launchpad** – the setup wizard that helps you configure your first tests.

```bash
npx cypress open
```

This command starts the Cypress Test Runner. The first time you run it, the Launchpad will appear and ask you to choose a testing type:

- **E2E Testing** – for full end‑to‑end browser tests (what we need).  
- Component Testing – for isolated component tests (not covered here).

Select **E2E Testing**. Cypress then generates the necessary configuration and file structure for your project.

### Key Files and Directories Created

After the initial setup, Cypress adds (or modifies) these items in your project:

```
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

- **`cypress/e2e/`** – The directory where all your end‑to‑end test files live. Each file typically tests one feature or page. By convention, test files have the `.cy.ts` extension (TypeScript) or `.cy.js` (JavaScript). For example: `login.cy.ts`, `register.cy.ts`, etc. Cypress will automatically discover and list all files inside this folder.

- **`cypress/fixtures/`** – A folder for static data that you can load inside tests. For instance, you might put a `users.json` file here with dummy user credentials and then use `cy.fixture('users.json')` to load that data. Fixtures keep your test code clean by separating data from logic.

- **`cypress/support/`** – Contains files that run **before** every test file.  
  - `e2e.ts` is the entry point for E2E tests – it is executed once before all test files. You can add global `beforeEach` hooks, import custom commands, or configure plugins here.  
  - `commands.ts` is where you define your own custom Cypress commands (like `cy.login()`). This helps avoid repeating complex sequences across tests.

Together, these building blocks give you a clean, maintainable testing structure.

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
- **`baseUrl: 'http://localhost:4200'`** – The base URL of your application. When you use `cy.visit('/account/login')`, Cypress automatically prepends this base URL, so you don’t have to type the full address every time. **This must match the URL where your Angular dev server is running.**
- **`setupNodeEvents(on, config)`** – A place to hook into the Node‑side lifecycle of Cypress. You can listen for events (like `before:run`, `after:screenshot`) or modify the configuration dynamically. For basic tests you can leave it empty or use plugins like code coverage.

---

## Essential Cypress Commands – The Building Blocks

Cypress provides a rich set of chainable commands. Understanding these core commands is the key to writing any test.

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
- `cy.get('app-text-input[formControlName="email"]')` – selects the Angular component `app-text-input` that has a `formControlName` attribute equal to “email”

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
cy.get('app-text-input[formControlName="email"]').find('input').type('user@example.com');
```
This first finds the `app-text-input` component for the email control, then drills down to its `<input>` child using `.find()`, and finally types the email.

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

**Example from our tests:**  
`cy.get('button[type="submit"]').should('be.disabled');`

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

**Example from our login test:**
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
cy.location('hash')
```

Gets information about the current URL. Commonly used to verify navigation after an action.

**Examples:**
- `cy.url().should('include', '/shop')` – checks that the URL contains `/shop`
- `cy.location('pathname').should('eq', '/shop')` – strictly compares the pathname

### Narrowing Down Selections: `.find()`

```typescript
cy.get(selector).find(subSelector)
```

Searches for a child element **within** the previously selected element. This is a chained command that helps you traverse the DOM from a known parent.

**Example:**
```typescript
cy.get('app-text-input[formControlName="email"]').find('input').type('user@example.com');
```
`cy.get(...)` gets the custom component, and `.find('input')` looks inside it for the actual `<input>` tag.

### Creating Aliases: `.as()`

```typescript
.alias(name)
```

Gives a custom name to a DOM element, a network intercept, or a route. You can then refer to it later using `cy.get('@alias')`.

**Example:**
```typescript
cy.intercept('POST', '/api/login').as('loginRequest');
cy.wait('@loginRequest');
cy.get('button').as('submitButton');
cy.get('@submitButton').should('be.disabled');
```
Aliases make your tests more readable and reusable.

### Test Structure: `describe()`, `beforeEach()` and `it()`

Cypress uses Mocha’s syntax for organising tests.

- **`describe(description, callback)`** – Groups together several test cases that belong to the same feature or component. Think of it as a “test suite”.
- **`beforeEach(callback)`** – Runs the given callback **before each** `it` block inside the `describe`. Commonly used to navigate to the page under test, reset state, or set up intercepts.
- **`it(description, callback)`** – Defines a single test case (also known as a “spec”). Inside, you write your commands and assertions.

**Basic blueprint:**
```typescript
describe('Feature Name', () => {
  beforeEach(() => {
    // setup code
  });

  it('should do something specific', () => {
    // test steps and assertions
  });

  it('should do another thing', () => {
    // ...
  });
});
```

---

## Writing Your First Tests for LiliShop

With the commands above, let’s examine two real test files from our project: the login and registration features.

### Login Feature Test (`login.cy.ts`)

File location: `cypress/e2e/auth/login.cy.ts`

```typescript
describe('Login Feature', () => {
  beforeEach(() => {
    // Navigate to the login page before every test case
    cy.visit('/account/login'); 
  });

  it('should enforce form validation rules by disabling the sign-in button initially', () => {
    // The form is empty, so the submit button must be disabled
    cy.get('button[type="submit"]').should('be.disabled');
  });

  it('should enable the sign-in button only when valid input data is provided', () => {
    // Target inputs inside your custom <app-text-input> components
    cy.get('app-text-input[formControlName="email"]').find('input').type('user@example.com');
    cy.get('app-text-input[formControlName="password"]').find('input').type('Password123!');

    // The form should now satisfy validation conditions
    cy.get('button[type="submit"]').should('not.be.disabled');
  });

  it('should navigate to the forgot password route when clicking the corresponding link', () => {
    cy.get('.forgot-password a').click();
    
    // Verify that the router successfully changed the URL path
    cy.url().should('include', '/account/forgot-password');
  });

  it('should successfully submit valid credentials and redirect the user to the shop dashboard', () => {
    // Mock the backend API authentication request to keep tests isolated and fast
    cy.intercept('POST', '/api/account/login', {
      statusCode: 200,
      body: { token: 'mock-jwt-token-string', email: 'user@example.com' }
    }).as('loginRequest');

    // Fill out the login form
    cy.get('app-text-input[formControlName="email"]').find('input').type('user@example.com');
    cy.get('app-text-input[formControlName="password"]').find('input').type('Password123!');
    
    // Click the submit button
    cy.get('button[type="submit"]').click();

    // Verify that the exact HTTP request was sent to the .NET backend
    cy.wait('@loginRequest');

    // Confirm that the user is redirected away from the login page upon success
    cy.url().should('include', '/shop');
  });
});
```

**Walkthrough:**

1. The `describe` block groups all tests related to the login page.
2. `beforeEach` visits the login page before every test, so each test starts fresh.
3. **Test 1** – Checks that an empty form disables the submit button.
4. **Test 2** – Types valid email and password, then asserts that the submit button becomes enabled.
5. **Test 3** – Clicks the “Forgot password” link and verifies that the URL changes to the forgot‑password route.
6. **Test 4** – Tests a full login flow by:
   - Stubbing the login API with `cy.intercept()` and giving it an alias.
   - Filling the form and clicking submit.
   - Waiting for the stubbed request with `cy.wait('@loginRequest')` to confirm the frontend actually called the backend.
   - Asserting that the user is redirected to `/shop`.

### Registration Feature Test (`register.cy.ts`)

File location: `cypress/e2e/auth/register.cy.ts`

```typescript
describe('Registration Feature', () => {
  beforeEach(() => {
    // Navigate to the registration route
    cy.visit('/account/register');
  });

  it('should disable the register button by default when the form is empty', () => {
    cy.get('button[type="submit"]').should('be.disabled');
  });

  it('should display the Google Sign-In container element', () => {
    cy.get('#buttonDiv').should('exist');
  });

  it('should keep the register button disabled if passwords do not match', () => {
    cy.get('app-text-input[formControlName="displayName"]').find('input').type('John Doe');
    cy.get('app-text-input[formControlName="email"]').find('input').type('john.doe@example.com');
    cy.get('app-text-input[formControlName="password"]').find('input').type('SecurePass123!');

    // Enter a non-matching password
    cy.get('app-text-input[formControlName="confirmPassword"]').find('input').type('DifferentPass123!');

    cy.get('button[type="submit"]').should('be.disabled');
  });

  it('should enable the register button when all inputs are valid and match', () => {
    cy.get('app-text-input[formControlName="displayName"]').find('input').type('John Doe');
    cy.get('app-text-input[formControlName="email"]').find('input').type('john.doe@example.com');
    cy.get('app-text-input[formControlName="password"]').find('input').type('SecurePass123!');
    cy.get('app-text-input[formControlName="confirmPassword"]').find('input').type('SecurePass123!');

    cy.get('button[type="submit"]').should('not.be.disabled');
  });

  it('should submit registration successfully, handle the confirmation dialog, and redirect to the shop', () => {
    // 1. Intercept the async email availability check
    cy.intercept('GET', '/api/account/emailexists*', {
      statusCode: 200,
      body: false
    }).as('emailExistsCheck');

    // 2. Intercept the successful registration endpoint
    cy.intercept('POST', '/api/account/register', {
      statusCode: 200,
      body: {
        displayName: 'John Doe',
        email: 'newuser@example.com',
        token: 'mock-jwt-token'
      }
    }).as('registerSuccessRequest');

    // 3. Fill out the form fields and trigger micro-validations
    cy.get('app-text-input[formControlName="displayName"]').find('input').type('John Doe').blur();
    cy.get('app-text-input[formControlName="email"]').find('input').type('newuser@example.com').blur();
    cy.wait('@emailExistsCheck');

    cy.get('app-text-input[formControlName="password"]').find('input').type('SecurePass123!').blur();
    cy.get('app-text-input[formControlName="confirmPassword"]').find('input').type('SecurePass123!').blur();

    // 4. Submit the form and wait for the successful network response
    cy.get('button[type="submit"]').should('not.be.disabled').click();
    cy.wait('@registerSuccessRequest');

    // 5. Interact with the Angular Material Dialog
    cy.get('mat-dialog-container', { timeout: 5000 })
      .should('be.visible')
      .and('contain.text', 'Email Confirmation Sent');

    // Click the confirmation button inside the dialog to close it
    cy.get('mat-dialog-container').find('button').click();

    // 6. Assert that the application routes to the shop after the dialog closes
    cy.location('pathname').should('eq', '/shop');
  });
});
```

**Walkthrough of the key test (last one):**

- **Intercepts** – Two network stubs are set up: one for the email‑availability check (`GET /api/account/emailexists*`) and one for the registration itself (`POST /api/account/register`). Both return fake 200 responses.
- **Form filling** – Each field is typed into, followed by a `.blur()` to trigger Angular validation. After typing the email, we `cy.wait('@emailExistsCheck')` to make sure the stub is called before proceeding.
- **Submission** – The button should now be enabled; we click it.
- **Waiting for the register request** – `cy.wait('@registerSuccessRequest')` ensures the frontend actually made the POST request and received our fake response.
- **Dialog interaction** – Cypress waits up to 5 seconds for the Material Dialog to appear. It asserts the dialog is visible and contains “Email Confirmation Sent”, then clicks the button inside it.
- **Route check** – Finally we verify that the pathname is exactly `/shop` after the dialog closes.

---

## How to Run Your Tests

Before running Cypress, your Angular application **must be running locally**.

### 1. Start the Angular development server

Open a terminal and run:

```bash
npm start
```

Wait until you see `Compiled successfully` and the app is available at `http://localhost:4200`.

### 2. Run Cypress in interactive mode (GUI)

Open a **second terminal** and launch the Test Runner:

```bash
npx cypress open
```

The Launchpad will appear. Select **E2E Testing**, choose your preferred browser (Chrome is recommended), and click the **Start E2E Testing** button. You’ll see a list of test files. Click on `login.cy.ts` or any other file to run it.

The interactive runner will execute the test step by step, showing you the application state and letting you debug with snapshots and the browser console.

### 3. Run Cypress in headless mode (command line)

For continuous integration or a quick check, run Cypress without the GUI:

```bash
npx cypress run --spec "cypress/e2e/auth/login.cy.ts"
```

This executes the specified test file in a headless Electron browser and outputs the results to your terminal. Videos and screenshots are saved by default in the `cypress/videos` and `cypress/screenshots` folders.

To run all E2E tests, simply use:

```bash
npx cypress run
```
