# BDD Project Architecture Design Specification

> This specification is based on Cucumber official documentation, community best practices, and multi-language implementation comparisons, systematically defining architecture design standards for BDD test automation projects.
>
> Scope: JVM (Java/Kotlin) / JavaScript (Node.js/TypeScript) / Ruby technology stacks

---

## Table of Contents

1. [Directory Structure Specification](#1-directory-structure-specification)
2. [Layered Architecture Patterns](#2-layered-architecture-patterns)
3. [Module Dependency Rules](#3-module-dependency-rules)
4. [Naming Conventions](#4-naming-conventions)
5. [Page Object Pattern](#5-page-object-pattern)
6. [Screenplay Pattern](#6-screenplay-pattern)
7. [Dependency Injection Configuration](#7-dependency-injection-configuration)

---

## 1. Directory Structure Specification

### 1.1 Directory Structure Design Principles

BDD project directory structure should follow these core principles:

| Principle | Description |
|------|------|
| **Separation of Concerns** | Feature files, step definitions, page objects, and utility classes should be stored independently |
| **Organize by Domain** | Group by business domain modules, not by technical layer types |
| **Consistency** | Maintain uniform naming and organization style within the same project |
| **Scalability** | Structure should support natural growth from small to large project scale |
| **Language Adaptation** | Follow conventions of each language ecosystem |

### 1.2 JVM (Java/Kotlin) Directory Structure

#### Maven Project Recommended Structure

```
project-root/
├── pom.xml                                          # Maven build configuration
├── cucumber.properties                              # Cucumber global configuration
├── serenity.properties                              # Serenity BDD configuration (optional)
│
├── src/
│   ├── main/
│   │   └── java/                                    # Application code under test (omit if standalone test project)
│   │
│   └── test/
│       ├── java/                                    # Test source code
│       │   └── com/company/project/
│       │       ├── runners/                         # Test Runner classes
│       │       │   ├── CucumberTestSuite.java       # JUnit 5 Suite main entry point
│       │       │   ├── SmokeTestRunner.java         # Smoke test Runner
│       │       │   └── RegressionTestRunner.java    # Regression test Runner
│       │       │
│       │       ├── stepdefinitions/                 # Step definition layer
│       │       │   ├── auth/
│       │       │   │   ├── LoginSteps.java          # Login-related steps
│       │       │   │   └── RegistrationSteps.java   # Registration-related steps
│       │       │   ├── cart/
│       │       │   │   ├── CartSteps.java           # Cart steps
│       │       │   │   └── CheckoutSteps.java       # Checkout steps
│       │       │   ├── common/
│       │       │   │   ├── CommonSteps.java         # Common steps (navigation/waiting, etc.)
│       │       │   │   └── HookSteps.java           # Global Hooks
│       │       │   └── api/
│       │       │       └── ApiSteps.java            # API test steps
│       │       │
│       │       ├── pages/                           # Page Object layer
│       │       │   ├── BasePage.java                # Page object abstract base class
│       │       │   ├── LoginPage.java               # Login page
│       │       │   ├── HomePage.java                # Home page
│       │       │   ├── CartPage.java                # Cart page
│       │       │   └── CheckoutPage.java            # Checkout page
│       │       │
│       │       ├── screenplay/                      # Screenplay pattern (optional)
│       │       │   ├── abilities/                   # Ability definitions
│       │       │   │   └── BrowseTheWeb.java
│       │       │   ├── tasks/                       # Task definitions
│       │       │   │   ├── Login.java
│       │       │   │   ├── AddToCart.java
│       │       │   │   └── NavigateTo.java
│       │       │   ├── questions/                   # Question definitions
│       │       │   │   ├── TheCartTotal.java
│       │       │   │   └── TheWelcomeMessage.java
│       │       │   └── interactions/                # Interaction definitions
│       │       │       ├── ClickOn.java
│       │       │       └── EnterValue.java
│       │       │
│       │       ├── context/                         # Shared context/state management
│       │       │   ├── TestContext.java             # Test context (PicoContainer)
│       │       │   └── ScenarioContext.java         # Scenario-level context
│       │       │
│       │       ├── api/                             # API client layer
│       │       │   ├── ApiClient.java               # Generic API client
│       │       │   ├── AuthApi.java                 # Authentication API
│       │       │   └── OrderApi.java                # Order API
│       │       │
│       │       ├── model/                           # Domain models/data transfer objects
│       │       │   ├── User.java                    # User entity
│       │       │   ├── Product.java                 # Product entity
│       │       │   └── Order.java                   # Order entity
│       │       │
│       │       ├── utils/                           # Utility classes
│       │       │   ├── DriverFactory.java           # WebDriver factory
│       │       │   ├── ConfigReader.java            # Config reader
│       │       │   ├── WaitUtils.java               # Wait utilities
│       │       │   └── TestDataGenerator.java       # Test data generator
│       │       │
│       │       └── transformers/                    # Custom parameter transformers
│       │           ├── DateTransformer.java         # Date transformation
│       │           └── MoneyTransformer.java        # Money transformation
│       │
│       └── resources/
│           ├── features/                            # Feature files
│           │   ├── auth/
│           │   │   ├── login.feature                # Login feature
│           │   │   └── registration.feature         # Registration feature
│           │   ├── cart/
│           │   │   ├── add-to-cart.feature          # Add to cart
│           │   │   └── checkout.feature             # Checkout flow
│           │   └── api/
│           │       └── order-api.feature            # Order API
│           │
│           ├── junit-platform.properties            # JUnit 5 configuration
│           ├── logback-test.xml                     # Log configuration
│           └── testdata/                            # Test data files
│               ├── users.json                       # User test data
│               └── products.csv                     # Product test data
│
└── target/                                          # Build output
    ├── cucumber-reports/                            # Cucumber reports
    └── serenity/                                    # Serenity reports
```

#### Gradle Project Recommended Structure

```
project-root/
├── build.gradle.kts                                 # Gradle build configuration (Kotlin DSL)
├── settings.gradle.kts
├── serenity.properties
│
├── src/
│   ├── test/
│   │   ├── java/                                    # Java test code (same as Maven)
│   │   └── resources/
│   │       └── features/                            # Feature files (same as Maven)
│   └── main/                                        # For shared code if needed
└── build/                                           # Build output
```

#### Kotlin Project Adaptation

```kotlin
// build.gradle.kts (Kotlin DSL)
plugins {
    java
    id("net.serenity-bdd.serenity-gradle-plugin") version "4.0.46"
}

dependencies {
    testImplementation("io.cucumber:cucumber-java:7.34.3")
    testImplementation("io.cucumber:cucumber-junit-platform-engine:7.34.3")
    testImplementation("io.cucumber:cucumber-picocontainer:7.34.3")
    testImplementation("net.serenity-bdd:serenity-core:4.0.46")
    testImplementation("net.serenity-bdd:serenity-cucumber:4.0.46")
    testImplementation("org.junit.platform:junit-platform-suite:1.10.0")
    testImplementation("org.assertj:assertj-core:3.24.2")
}
```

### 1.3 JavaScript / TypeScript Directory Structure

```
project-root/
├── package.json                                     # npm configuration
├── tsconfig.json                                    # TypeScript configuration
├── cucumber.config.ts                               # Cucumber configuration (v12.4.0+)
│
├── features/                                        # Feature files directory
│   ├── auth/
│   │   ├── login.feature                            # Login feature
│   │   └── registration.feature                     # Registration feature
│   ├── cart/
│   │   ├── add-to-cart.feature                      # Add to cart
│   │   └── checkout.feature                         # Checkout flow
│   └── search/
│       └── product-search.feature                   # Product search
│
├── src/                                             # Source code
│   ├── step-definitions/                            # Step definitions
│   │   ├── auth/
│   │   │   ├── login.steps.ts                       # Login steps
│   │   │   └── registration.steps.ts                # Registration steps
│   │   ├── cart/
│   │   │   ├── cart.steps.ts                        # Cart steps
│ │   │   └── checkout.steps.ts                      # Checkout steps
│   │   └── hooks/                                   # Global Hooks
│   │       └── world.hooks.ts                       # World extensions
│   │
│   ├── pages/                                       # Page Object layer
│   │   ├── base.page.ts                             # Page base class
│   │   ├── login.page.ts                            # Login page
│   │   ├── home.page.ts                             # Home page
│   │   └── cart.page.ts                             # Cart page
│   │
│   ├── screenplay/                                  # Screenplay pattern (optional)
│   │   ├── actor.ts                                 # Actor definition
│   │   ├── abilities/
│   │   │   └── browse-the-web.ts                    # Browse the web ability
│   │   ├── tasks/
│   │   │   ├── login.ts                             # Login task
│   │   │   └── add-to-cart.ts                       # Add to cart task
│   │   ├── questions/
│   │   │   └── cart-total.ts                        # Cart total query
│   │   └── interactions/
│   │       ├── click.ts                             # Click interaction
│   │       └── enter.ts                             # Enter interaction
│   │
│   ├── world/                                       # Custom World
│   │   └── custom-world.ts                          # Custom World class
│   │
│   ├── api/                                         # API clients
│   │   ├── client.ts                                # Generic API client
│   │   └── endpoints/                               # Endpoint definitions
│   │       ├── auth.endpoint.ts                     # Authentication endpoint
│   │       └── order.endpoint.ts                    # Order endpoint
│   │
│   ├── utils/                                       # Utility functions
│   │   ├── config.ts                                # Configuration management
│   │   ├── waits.ts                                 # Wait utilities
│   │   ├── selectors.ts                             # Selector utilities
│   │   └── test-data.ts                             # Test data generation
│   │
│   └── types/                                       # Type definitions
│       └── index.ts                                 # Common types
│
├── reports/                                         # Report output
├── test-results/                                    # Test results
└── playwright.config.ts                             # Playwright configuration (if used)
```

#### JavaScript Project (No TypeScript)

```
project-root/
├── package.json
├── cucumber.js                                      # Cucumber configuration file
│
├── features/
│   ├── auth/
│   │   ├── login.feature
│   │   └── registration.feature
│   └── support/                                     # Support files
│       ├── hooks.js                                 # Before/After Hooks
│       ├── world.js                                 # World extensions
│       └── parameter-types.js                       # Custom parameter types
│
├── step-definitions/
│   ├── auth/
│   │   ├── login.steps.js
│   │   └── registration.steps.js
│   └── hooks.js                                     # Global Hooks
│
├── pages/                                           # Page Object layer
│   ├── base.page.js
│   ├── login.page.js
│   └── home.page.js
│
├── utils/
│   ├── driver.js                                    # Driver management
│   ├── config.js                                    # Config reader
│   └── waits.js                                     # Wait utilities
│
└── reports/
```

### 1.4 Ruby Directory Structure

```
project-root/
├── Gemfile                                          # Gem dependencies
├── Gemfile.lock
├── cucumber.yml                                     # Cucumber configuration file
├── Rakefile
│
├── features/                                        # Feature files directory
│   ├── auth/
│   │   ├── login.feature                            # Login feature
│   │   └── registration.feature                     # Registration feature
│   ├── cart/
│   │   ├── add_to_cart.feature                      # Add to cart
│   │   └── checkout.feature                         # Checkout flow
│   └── support/                                     # Support files
│       ├── env.rb                                   # Environment configuration (most important)
│       ├── hooks.rb                                 # Before/After/Around Hooks
│       ├── world.rb                                 # World extensions
│       ├── parameter_types.rb                       # Custom parameter types
│       ├── selectors.rb                             # Selector definitions
│       └── pages/                                   # Page Object layer
│           ├── base_page.rb                         # Page base class
│           ├── login_page.rb                        # Login page
│           ├── home_page.rb                         # Home page
│           └── cart_page.rb                         # Cart page
│
├── step_definitions/                                # Step definitions (must be at same level or subdirectory as features)
│   ├── auth_steps.rb                                # Authentication-related steps
│   ├── cart_steps.rb                                # Cart steps
│   └── common_steps.rb                              # Common steps
│
├── lib/                                             # Custom library code
│   ├── api_clients/                                 # API clients
│   │   ├── base_client.rb                           # Generic client
│   │   └── order_client.rb                          # Order client
│   ├── models/                                      # Domain models
│   │   ├── user.rb                                  # User model
│   │   └── product.rb                               # Product model
│   ├── helpers/                                     # Helper modules
│   │   ├── data_helper.rb                           # Data helper
│   │   └── wait_helper.rb                           # Wait helper
│   └── screenplay/                                  # Screenplay pattern (optional)
│       ├── actor.rb                                 # Actor
│       ├── ability.rb                               # Ability base class
│       ├── abilities/
│       │   └── browse_the_web.rb                    # Browse the web ability
│       ├── task.rb                                  # Task base class
│       ├── tasks/
│       │   ├── login.rb                             # Login task
│       │   └── add_to_cart.rb                       # Add to cart task
│       ├── question.rb                              # Question base class
│       └── questions/
│           └── cart_total.rb                        # Cart total query
│
├── config/
│   ├── environments.yml                             # Environment configuration
│   └── browsers.yml                                 # Browser configuration
│
└── reports/                                         # Report output
    ├── cucumber.json                                # JSON report
    └── cucumber.html                                # HTML report
```

#### Ruby's `features/support/env.rb` Core Configuration

```ruby
# features/support/env.rb - the most important configuration file in Ruby projects

require 'capybara'
require 'capybara/cucumber'
require 'selenium-webdriver'
require 'rspec/expectations'

# Capybara configuration
Capybara.configure do |config|
  config.default_driver = :selenium_chrome
  config.javascript_driver = :selenium_chrome_headless
  config.default_max_wait_time = 10
  config.app_host = ENV.fetch('APP_HOST', 'http://localhost:3000')
end

# Register remote driver (Selenium Grid)
Capybara.register_driver :selenium_remote do |app|
  Capybara::Selenium::Driver.new(
    app,
    browser: :remote,
    url: ENV.fetch('GRID_URL', 'http://localhost:4444/wd/hub'),
    options: Selenium::WebDriver::Chrome::Options.new
  )
end

# Auto-load lib directory
$LOAD_PATH.unshift(File.expand_path('../../lib', __dir__))
Dir.glob('lib/**/*.rb').each { |f| require f.sub(%r{^lib/}, '').sub(/\.rb$/, '') }
```

### 1.5 Three Technology Stack Directory Structure Comparison

| Layer | JVM (Java) | JavaScript/TypeScript | Ruby |
|------|-----------|----------------------|------|
| **Configuration** | `pom.xml` / `build.gradle` | `package.json` | `Gemfile` / `cucumber.yml` |
| **Feature files** | `src/test/resources/features/` | `features/` | `features/` |
| **Step Definitions** | `src/test/java/stepdefinitions/` | `src/step-definitions/` | `step_definitions/` (fixed name) |
| **Page objects** | `src/test/java/pages/` | `src/pages/` | `features/support/pages/` |
| **Hooks** | Annotations `@Before`/`@After` | `src/step-definitions/hooks/` | `features/support/hooks.rb` |
| **World/Context** | DI container (PicoContainer) | `world/` custom World | `features/support/world.rb` |
| **API clients** | `src/test/java/api/` | `src/api/` | `lib/api_clients/` |
| **Utility classes** | `src/test/java/utils/` | `src/utils/` | `lib/helpers/` |
| **Config reader** | `ConfigReader.java` | `config.ts` | `config/environments.yml` |

---

## 2. Layered Architecture Patterns

### 2.1 Four-Layer Architecture Overview

BDD test automation should adopt a strict layered architecture to ensure separation of concerns and maintainability:

```
┌──────────────────────────────────────────────────────────────┐
│                    Layer 1: Gherkin Layer                        │
│                   (Business Specification Layer)                              │
│  .feature files - Describe system behavior using business language                      │
│  Feature / Scenario / Given-When-Then                         │
├──────────────────────────────────────────────────────────────┤
│                    Layer 2: Step Definitions Layer               │
│                   (Step Definition Layer / Glue Layer)                          │
│  Step definition classes - Bridge between Gherkin and code                              │
│  Given() / When() / Then() method implementation                           │
├──────────────────────────────────────────────────────────────┤
│              Layer 3: Automation Abstraction Layer                             │
│     (Page Objects / API Clients / Task Classes)              │
│  Encapsulate specific system interactions - UI operations, API calls, database operations             │
├──────────────────────────────────────────────────────────────┤
│              Layer 4: System Under Test Layer (SUT)                         │
│     (Application Under Test)                                  │
│  Web application / API service / Mobile application / Desktop application                     │
└──────────────────────────────────────────────────────────────┘
```

### 2.2 Layer Responsibility Definitions

#### Layer 1: Gherkin Layer (Business Specification Layer)

| Attribute | Description |
|------|------|
| **Responsibility** | Describe what the system should **do (What)**, not **how it does it (How)** |
| **Language** | Pure business language, using Ubiquitous Language |
| **Audience** | Business experts, product managers, QA, developers |
| **Technical Constraints** | **Zero technical details** - No CSS selectors, no API endpoints, no database operations |
| **File Format** | `.feature` files, Gherkin syntax |

```gherkin
# Layer 1 Example - Pure business language, zero technical details
Feature: Customer Login
  As a registered customer
  I want to log in with my credentials
  So that I can access my order dashboard

  @smoke @positive
  Scenario: Successful login with valid credentials
    Given "Jane" has a registered account
    When "Jane" logs in with her valid credentials
    Then she should be redirected to the order dashboard
    And she should see a personalized welcome message
```

#### Layer 2: Step Definitions Layer (Step Definition Layer)

| Attribute | Description |
|------|------|
| **Responsibility** | Map Gherkin steps to code calls, orchestrate automation operations |
| **Principle** | **Keep it concise**，Only do "translation" and "orchestration", do not implement specific interaction logic |
| **Code Volume** | 1-3 lines of code per step is appropriate |
| **Dependency Direction** | Call Layer 3 (Page Objects / API Clients / Tasks) |
| **Prohibitions** | Directly operate WebDriver / HTTP Client / Database |

```java
// Layer 2 Example - JVM (Java)
public class LoginSteps {
    private final LoginPage loginPage;      // Layer 3 dependency
    private final TestContext context;       // Shared state

    // Constructor injection (PicoContainer)
    public LoginSteps(LoginPage loginPage, TestContext context) {
        this.loginPage = loginPage;
        this.context = context;
    }

    @Given("{string} has a registered account")
    public void hasRegisteredAccount(String username) {
        User user = TestDataGenerator.createUser(username);
        context.setCurrentUser(user);           // Store in context
    }

    @When("{string} logs in with her valid credentials")
    public void logsInWithValidCredentials(String username) {
        User user = context.getCurrentUser();
        loginPage.login(user.getEmail(), user.getPassword());  // calls Layer 3
    }

    @Then("she should be redirected to the order dashboard")
    public void shouldSeeOrderDashboard() {
        assertThat(loginPage.getCurrentUrl())
            .contains("/dashboard");
    }
}
```

```typescript
// Layer 2 Example - JavaScript/TypeScript
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';

Given('{string} has a registered account', async function(username: string) {
    this.user = await this.testData.createUser(username);
});

When('{string} logs in with her valid credentials', async function(username: string) {
    await this.loginPage.login(this.user.email, this.user.password);
});

Then('she should be redirected to the order dashboard', async function() {
    await expect(this.page).toHaveURL(/.*dashboard/);
});
```

```ruby
# Layer 2 Example - Ruby
Given('{string} has a registered account') do |username|
  @user = TestDataGenerator.create_user(username)
end

When('{string} logs in with her valid credentials') do |username|
  @login_page.login(@user.email, @user.password)
end

Then('she should be redirected to the order dashboard') do
  expect(@login_page).to have_current_path(%r{/dashboard}, only_path: false)
end
```

#### Layer 3: Automation Abstraction Layer (Page Objects / API Clients / Tasks)

| Sub-layer | Responsibility | Applicable Scenario |
|------|------|----------|
| **Page Objects** | Encapsulate UI element locating and page operations | Web UI automation |
| **API Clients** | Encapsulate HTTP request and response handling | API/service layer testing |
| **Tasks (Screenplay)** | Encapsulate user-centric business activities | Complex UI workflows |
| **Database Helpers** | Encapsulate data preparation and cleanup operations | Scenarios requiring direct data manipulation |

```java
// Layer 3 Example - Page Object (JVM)
public class LoginPage extends BasePage {
    @FindBy(id = "email")
    private WebElement emailField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(css = "[data-testid='login-button']")
    private WebElement loginButton;

    @FindBy(css = "[data-testid='welcome-message']")
    private WebElement welcomeMessage;

    public void login(String email, String password) {
        waitForVisible(emailField);
        emailField.sendKeys(email);
        passwordField.sendKeys(password);
        loginButton.click();
    }

    public String getWelcomeMessage() {
        return waitForVisible(welcomeMessage).getText();
    }
}
```

```typescript
// Layer 3 Example - API Client (JavaScript)
export class AuthApiClient {
    private baseUrl: string;

    constructor(baseUrl: string) {
        this.baseUrl = baseUrl;
    }

    async login(email: string, password: string): Promise<AuthResponse> {
        const response = await fetch(`${this.baseUrl}/api/auth/login`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password }),
        });
        return response.json() as Promise<AuthResponse>;
    }

    async register(userData: RegisterRequest): Promise<User> {
        const response = await fetch(`${this.baseUrl}/api/auth/register`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(userData),
        });
        return response.json() as Promise<User>;
    }
}
```

#### Layer 4: System Under Test Layer (SUT)

| Type | Description | Test Entry Point |
|------|------|----------|
| **Web application** | Frontend application running in browser | URL access |
| **API service** | RESTful / GraphQL service | HTTP endpoint |
| **Mobile application** | iOS/Android native or hybrid application | Appium |
| **Desktop application** | Windows/macOS/Linux application | WinAppDriver/Selenium |
| **Database** | Relational/NoSQL database | JDBC/ORM |

### 2.3 Layered Architecture Code Example (Complete Flow)

The following demonstrates a complete login scenario implementation across four layers:

```
Layer 1 (Gherkin)           Layer 2 (Step Def)           Layer 3 (Page Object)        Layer 4 (SUT)
─────────────────────────────────────────────────────────────────────────────────────────────────
Given "Jane" has a     -->  hasRegisteredAccount()  -->  TestDataGenerator          -->  DB
  registered account                                      .createUser()                  insert
                                                          
When "Jane" logs in    -->  logsInWithValidCreds()  -->  LoginPage.login()          -->  Web
  with her valid                                          (emailField.sendKeys)         App
  credentials                                             (loginButton.click())
                                                          
Then she should see    -->  shouldSeeWelcomeMsg()   -->  LoginPage.getWelcomeMsg()  -->  DOM
  welcome message                                         (welcomeMessage.getText())
```

### 2.4 Hybrid Architecture Pattern (UI + API)

In real projects, BDD scenarios often need to use both UI and API:

```
┌─────────────────────────────────────────────────────────┐
│              Layer 1: Gherkin Scenario                       │
│  Given API: Create test user (API layer)                        │
│  When UI: User login (UI layer)                               │
│  Then UI: Verify page display (UI layer)                           │
│  And API: Verify session state (API layer)                          │
├─────────────────────────────────────────────────────────┤
│              Layer 2: Step Definitions                   │
│  - AuthApiSteps (API-related Given/Then)                   │
│  - LoginPageSteps (UI-related When/Then)                   │
├─────────────────────────────────────────────────────────┤
│              Layer 3: Automation Abstraction Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐   │
│  │ Page Objects │  │ API Clients  │  │  DB Helpers │   │
│  │  LoginPage   │  │  AuthApi     │  │  UserRepo   │   │
│  │  HomePage    │  │  OrderApi    │  │  OrderRepo  │   │
│  └──────────────┘  └──────────────┘  └─────────────┘   │
├─────────────────────────────────────────────────────────┤
│              Layer 4: System Under Test                            │
│         Web App  <--->  API Server  <--->  Database      │
└─────────────────────────────────────────────────────────┘
```

```gherkin
# Layer 1: Hybrid Scenario Example
Feature: Order Management

  @e2e
  Scenario: Customer places an order successfully
    # API layer data preparation
    Given a registered customer "Alice" with a saved address
    And the product "Wireless Mouse" is in stock with quantity 10

    # UI layer execute operations
    When "Alice" logs in to the storefront
    And she adds "Wireless Mouse" to her cart
    And she proceeds to checkout with her saved address
    And she completes the payment

    # UI layer verification
    Then she should see an order confirmation page
    And her order total should be $29.99

    # API layer verification
    And the inventory for "Wireless Mouse" should show quantity 9
    And the order API should return a confirmed order for "Alice"
```

---

## 3. Module Dependency Rules

### 3.1 Core Dependency Rules

BDD projects follow **strict unidirectional dependency** rules: upper layers can call lower layers, lower layers must never reverse-depend on upper layers.

```
Dependency direction (only downward, never upward):

    Layer 1: Gherkin Feature Files
         │
         ▼  (Gherkin steps match through regex/expression)
    Layer 2: Step Definitions
         │
         ▼  (Injected through constructor/method parameters)
    Layer 3: Page Objects / API Clients / Tasks
         │
         ▼  (Through WebDriver / HTTP Client / DB Driver)
    Layer 4: System Under Test
```

### 3.2 Dependency Rules Matrix

| Caller \ Callee | Layer 1 Gherkin | Layer 2 Step Def | Layer 3 Page Object/API | Layer 4 SUT |
|------------------|-----------------|------------------|-------------------------|-------------|
| **Layer 1 Gherkin** | - | Indirect association (step matching) | **Forbidden** | **Forbidden** |
| **Layer 2 Step Def** | **Forbidden** | - (same-layer callable) | **Allowed** | **Forbidden** |
| **Layer 3 Page Object** | **Forbidden** | **Forbidden** | - (same-layer composable) | **Allowed** |
| **Layer 4 SUT** | **Forbidden** | **Forbidden** | **Forbidden** | - |

### 3.3 Same-Layer Dependency Rules

| Layer | Same-layer Dependency Rules | Example |
|------|-------------|------|
| **Layer 2** | Different step definition classes **can** call each other, but circular dependencies should be avoided | `CommonSteps` called by `LoginSteps` |
| **Layer 3** | Page Objects **can** reference each other for navigation, but should remain loosely coupled | `HomePage.navigateToLogin()` returns `LoginPage` |
| **Layer 3** | API Clients **should not** call each other, should be orchestrated through a Service layer | `OrderService` calls `AuthApi` + `OrderApi` |

### 3.4 Dependency Injection Container Scopes

```
Scenario-level scope (new instance per scenario):
┌─────────────────────────────────────┐
│  TestContext / ScenarioContext      │  ◀── Scenario state sharing
│  Page Objects                       │  ◀── New instance per scenario
│  API Clients                        │  ◀── New instance per scenario
│  WebDriver (optional shared)        │  ◀── per scenario or global
└─────────────────────────────────────┘

Global scope (shared across all scenarios):
┌─────────────────────────────────────┐
│  ConfigReader                       │  ◀── Configuration unchanged
│  DriverFactory                      │  ◀── Driver factory
│  TestDataGenerator                  │  ◀── Data generator
│  Database Connection Pool           │  ◀── Connection pool
└─────────────────────────────────────┘
```

### 3.5 Anti-Patterns: Typical Errors Violating Dependency Rules

```java
// Anti-Pattern 1: Layer 3 (Page Object) calls Layer 2 (Step Definition)
// PageObject.java - Critical error!
public class LoginPage {
    public void login(String email, String password) {
        // Page Object should NEVER call Step Definition
        loginSteps.performLogin(email, password);  // Violates dependency rules!
    }
}

// Anti-pattern 2: Layer 2 directly operates Layer 4 (bypassing Layer 3)
// LoginSteps.java - Not recommended
@When("the user enters {string} in the email field")
public void enterEmail(String email) {
    // Using WebDriver directly in Step Definition - violates layering!
    driver.findElement(By.id("email")).sendKeys(email);  // Should go through LoginPage
}

// Anti-pattern 3: Circular dependency
// LoginSteps depends on HomePage, HomePage depends on LoginSteps - fatal error
public class LoginSteps {
    private HomePage homePage;  // OK
}
public class HomePage {
    private LoginSteps steps;   // Circular dependency! Critical error!
}

// Anti-pattern 4: Layer 1 (Gherkin) contains technical implementation details
// login.feature - Critical error!
Scenario: User login
  When I send POST to /api/v1/auth/login        # Technical details in Gherkin!
  And I set header Content-Type to application/json
  Then response status should be 200
```

### 3.6 Correct Dependency Direction Examples

```java
// Correct: Layer 2 -> Layer 3 -> Layer 4

// Layer 2: Step Definition (Orchestrator)
public class CheckoutSteps {
    private final CartPage cartPage;        // Layer 3
    private final OrderApiClient orderApi;  // Layer 3

    public CheckoutSteps(CartPage cartPage, OrderApiClient orderApi) {
        this.cartPage = cartPage;
        this.orderApi = orderApi;
    }

    @When("the user proceeds to checkout")
    public void proceedToCheckout() {
        cartPage.clickCheckoutButton();     // calls Layer 3 (Page Object)
    }

    @Then("the order should be created via API")
    public void verifyOrderCreated() {
        Order order = orderApi.getLatestOrder();  // calls Layer 3 (API Client)
        assertThat(order.getStatus()).isEqualTo("CONFIRMED");
    }
}

// Layer 3: Page Object (Implementer) - Only calls Layer 4
public class CartPage extends BasePage {
    @FindBy(css = "[data-testid='checkout-button']")
    private WebElement checkoutButton;

    public CheckoutPage clickCheckoutButton() {
        waitForClickable(checkoutButton).click();  // Calls Selenium (Layer 4)
        return new CheckoutPage(driver);            // Returns another Page Object
    }
}

// Layer 3: API Client (Implementer) - Only calls Layer 4
public class OrderApiClient {
    private final RestTemplate restTemplate;

    public Order getLatestOrder() {
        return restTemplate.getForObject(
            "/api/orders/latest", Order.class);  // Calls HTTP (Layer 4)
    }
}
```



---

## 4. Naming Conventions

### 4.1 Naming Conventions Overview

| Category | Convention | Example |
|------|------|------|
| **Feature Files** | `kebab-case.feature` | `user-registration.feature` |
| **Step Definition Files** | `kebab-case.steps.java|ts|rb` | `login.steps.java` |
| **Page Object Class** | `PascalCase` + `Page` suffix | `LoginPage.java` |
| **Step Definition Class** | `PascalCase` + `Steps` suffix | `LoginSteps.java` |
| **Task Class (Screenplay)** | `PascalCase` verb-noun | `LogIn.java`, `AddToCart.java` |
| **Question Class (Screenplay)** | `PascalCase` + `The` prefix | `TheCartTotal.java` |
| **Ability Class (Screenplay)** | `PascalCase` + ability description | `BrowseTheWeb.java` |
| **Method Names (JVM)** | `camelCase` descriptive behavior | `enterUsername()` |
| **Method Names (JS)** | `camelCase` descriptive behavior | `enterUsername()` |
| **Method Names (Ruby)** | `snake_case` descriptive behavior | `enter_username` |
| **Directory Names** | `kebab-case` or `snake_case` | `step-definitions/` (JS) / `step_definitions/` (Ruby) |
| **Constants/Config Keys** | `SCREAMING_SNAKE_CASE` | `MAX_WAIT_TIME` |
| **CSS Selectors** | `kebab-case` with `data-testid` | `[data-testid='login-button']` |
| **Variable Names** | Language convention | `usernameField` (JVM/JS) / `username_field` (Ruby) |

### 4.2 Feature File Naming Conventions

#### Basic Rules

```
Format:    <domain-concept>[-<sub-concept>].feature
Example:    customer-login.feature
         checkout-payment.feature
         user-registration-email-validation.feature
```

#### Naming Patterns

| Pattern | Description | Example |
|------|------|------|
| **Domain concept** | Named using business domain concepts | `customer-login.feature` |
| **Verb-noun** | Action + object | `place-order.feature` |
| **Feature-sub-feature** | Main feature-sub-feature | `checkout-payment.feature` |
| **Module-feature** | Module name-feature name | `auth-password-reset.feature` |

#### Feature File Naming Examples

```
features/
├── auth/
│   ├── login.feature                          # Login (simple feature)
│   ├── logout.feature                         # Logout
│   ├── user-registration.feature              # User registration
│   ├── password-reset.feature                 # Password reset
│   ├── social-login.feature                   # Social login
│   └── account-lockout.feature                # Account lockout
├── cart/
│   ├── add-to-cart.feature                    # Add to cart
│   ├── remove-from-cart.feature               # Remove from cart
│   ├── update-cart-quantity.feature           # Update quantity
│   ├── apply-promo-code.feature               # Apply promo code
│   └── cart-calculation.feature               # Cart calculation
├── checkout/
│   ├── checkout-flow.feature                  # Checkout flow
│   ├── payment-processing.feature             # Payment processing
│   ├── shipping-options.feature               # Shipping options
│   └── order-confirmation.feature             # Order confirmation
└── api/
    ├── order-api.feature                      # Order API
    ├── payment-api.feature                    # Payment API
    └── inventory-api.feature                  # Inventory API
```

#### Naming Not Recommended

```
features/
├── test1.feature              # Meaningless name
├── actions.feature            # Too generic
├── scenarios.feature          # Too generic
├── login_test.feature         # Should not contain _test suffix
├── Login.feature              # Should not be capitalized
├── login_steps.feature        # Should not be confused with step definitions
└── new_feature.feature        # Not descriptive
```

### 4.3 Step Definition Class/File Naming

#### JVM (Java/Kotlin)

```
Format:    <Domain><Action>Steps.java
Example:    LoginSteps.java
         CartManagementSteps.java
         OrderCheckoutSteps.java

Anti-examples:    login_steps.java        # Should not use snake_case
         Login.java              # Missing Steps suffix
         login.java              # Lowercase without suffix
         LoginStepDef.java       # Unclear abbreviation
```

#### JavaScript/TypeScript

```
Format:    <domain>-<action>.steps.ts
Example:    login.steps.ts
         cart-management.steps.ts
         order-checkout.steps.ts

Anti-examples:    loginSteps.ts           # Missing .steps marker
         Login.steps.ts          # Should not be capitalized
         login_step.ts           # Should not use snake_case
```

#### Ruby

```
Format:    <domain>_<action>_steps.rb
Example:    auth_steps.rb
         cart_management_steps.rb
         order_checkout_steps.rb

Anti-examples:    LoginSteps.rb           # Should not use PascalCase
         login-steps.rb          # Should not use kebab-case
         steps.rb                # Too generic
```

### 4.4 Page Object Class Naming

```
Format:    <PageName>Page
Example:    LoginPage
         ProductDetailPage
         ShoppingCartPage
         OrderConfirmationPage

Anti-examples:    login_page              # Should use PascalCase
         Login                   # Missing Page suffix
         LoginPageObject         # Object suffix not needed
         TheLoginPage            # The prefix not needed (only for Screenplay Question)
```

### 4.5 Screenplay Pattern Naming

#### Actor Naming

```java
// Actors use character names - third person descriptive names
Actor alice = Actor.named("Alice");                              // Regular user
Actor admin = Actor.named("Admin").whoCan(UseAdminPanel.with(app)); // Administrator
Actor guest = Actor.named("Guest");                              // Guest
```

#### Task Naming

```java
// Tasks use verb phrases - describing what the user wants to do
public class LogIn implements Task { }
public class AddToCart implements Task { }
public class NavigateTo implements Task { }
public class CompleteCheckout implements Task { }
public class SearchFor implements Task { }

// Anti-examples:
public class LoginButtonClick implements Task { }     // Technical details
public class DoLogin implements Task { }               // Not precise enough
public class LogInAction implements Task { }           // Action suffix not needed
```

#### Ability Naming

```java
// Abilities use "BrowseTheWeb" / "CallAnApi" / "UseAdminPanel" pattern
public class BrowseTheWeb implements Ability { }
public class CallAnApi implements Ability { }
public class QueryADatabase implements Ability { }
public class UseAdminPanel implements Ability { }

// Anti-examples:
public class WebDriverAbility implements Ability { }   // Too technical
public class Browser implements Ability { }             // Not descriptive enough
```

#### Question Naming

```java
// Questions use "The" prefix + content to query
public class TheCartTotal implements Question<BigDecimal> { }
public class TheWelcomeMessage implements Question<String> { }
public class TheOrderStatus implements Question<OrderStatus> { }
public class TheNumberOfItemsInCart implements Question<Integer> { }

// Anti-examples:
public class CartTotal implements Question { }         // Missing The prefix
public class GetCartTotal implements Question { }       // Should not start with a verb
public class CartTotalCheck implements Question { }     // Check suffix not needed
```

### 4.6 Gherkin Step Naming Conventions

#### Step Text Naming Principles

| Principle | Description | Example |
|------|------|------|
| **Use third person** | Describe user behavior | `the user enters credentials` |
| **Use present tense** | Describe currently occurring behavior | `the cart contains 2 items` |
| **Use business language** | Do not use technical terms | `the user logs in` (not `clicks login button`) |
| **Parameterize variable parts** | Use Cucumber Expressions | `the user enters {string}` |
| **Keep it concise** | One step per line, avoid conjunctions | Split into multiple `And` |

#### Given/When/Then Naming Patterns

```gherkin
# Given: System state / Precondition (present perfect)
Given "Alice" has a registered account
Given the shopping cart contains 3 items
Given the product "Wireless Mouse" is out of stock
Given the user is on the login page

# When: User action / Trigger event (present tense)
When "Alice" logs in with her valid credentials
When the user clicks the "Add to Cart" button
When the user enters "SAVE20" in the promo code field
When the user submits the checkout form

# Then: Expected result / Assertion (present tense)
Then the cart total should be $59.99
Then "Alice" should see a confirmation message
Then the inventory should show 9 items remaining
Then the user should be redirected to the order confirmation page
```

### 4.7 Tag Naming Conventions

```gherkin
# Tag naming - all lowercase, kebab-case, @ symbol prefix

# By test type
@smoke                    # Smoke test
@regression               # Regression test
@sanity                   # Sanity test
@e2e                      # End-to-end test
@api                      # API test
@ui                       # UI test

# By feature module
@auth                     # Authentication module
@cart                     # Cart module
@checkout                 # Checkout module
@inventory                # Inventory module

# By test nature
@positive                 # Positive test
@negative                 # Negative test
@boundary                 # Boundary value test
@security                 # Security test
@performance              # Performance test

# By priority
@priority-high            # High priority
@priority-medium          # Medium priority
@priority-low             # Low priority

# By status
@wip                      # Work in Progress
@ready                    # Ready to execute
@flaky                    # Flaky test
@ignore                   # Temporarily ignored
@manual                   # Manual test

# By environment
@staging                  # Staging environment only
@production               # Production environment only
@local                     # Local environment only

# By browser/platform
@chrome
@firefox
@mobile
@desktop
```

### 4.8 CSS Selector Naming Conventions (data-testid)

```html
<!-- Recommended: use data-testid attribute for element locating -->
<!-- Avoid CSS class names or XPath (class names may change due to style changes) -->

<!-- Recommended -->
<button data-testid="login-button">Login</button>
<input data-testid="email-field" type="email" />
<div data-testid="cart-total">$59.99</div>
<div data-testid="welcome-message">Welcome, Alice!</div>

<!-- Usage in Page Object -->
```

```java
// Referencing data-testid in Page Object
public class LoginPage {
    @FindBy(css = "[data-testid='login-button']")
    private WebElement loginButton;

    @FindBy(css = "[data-testid='email-field']")
    private WebElement emailField;

    @FindBy(css = "[data-testid='password-field']")
    private WebElement passwordField;
}
```

### 4.9 Complete Naming Example (Across Three Languages)

```
Scenario: User login

JVM (Java):
  File: features/auth/login.feature
  Step class: src/test/java/stepdefinitions/auth/LoginSteps.java
  Page object: src/test/java/pages/LoginPage.java
  Methods: enterUsername(String), clickLoginButton()

JavaScript/TypeScript:
  File: features/auth/login.feature
  Step file: src/step-definitions/auth/login.steps.ts
  Page object: src/pages/login.page.ts
  Methods: enterUsername(), clickLoginButton()

Ruby:
  File: features/auth/login.feature
  Step file: step_definitions/auth_steps.rb
  Page object: features/support/pages/login_page.rb
  Methods: enter_username, click_login_button
```

---

## 5. Page Object Pattern

### 5.1 Page Object Pattern Overview

Page Object Model (POM) is the most widely used design pattern in BDD UI automation, proposed by the Selenium team. It encapsulates page element locating and page operations into independent classes, decoupling test code from page implementation.

**Core advantages:**

| Advantage | Description |
|------|------|
| **Separation of Concerns** | UI location logic separated from test logic |
| **Code reuse** | Same page operations can be reused across multiple scenarios |
| **Maintainability** | Page changes only need modification in one place |
| **Readability** | Test code reads like business operations |
| **Reduced fragility** | Minimize the impact of UI changes |

### 5.2 Page Object Design Principles

| Principle | Description |
|------|------|
| **Single responsibility** | Each Page Object only encapsulates elements and operations of a single page |
| **Encapsulate locators** | All element locators as private fields |
| **Return type** | Navigation operations return target page objects for chainable calls |
| **Do not expose WebDriver** | Page Object uses driver internally, not exposed to callers |
| **Use explicit waits** | All operations use explicit waits instead of fixed waits |
| **Use stable locators** | Prefer using `data-testid` Attribute |
| **Do not place assertions** | Assertions belong in step definitions or test methods |

### 5.3 Standard Page Object Implementation

#### JVM (Java) Complete Implementation

```java
// ==================== BasePage.java - Page Base Class ====================
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public abstract class BasePage {
    protected final WebDriver driver;
    protected final WebDriverWait wait;
    private final String pageUrl;

    public BasePage(WebDriver driver, String pageUrl) {
        this.driver = driver;
        this.pageUrl = pageUrl;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }

    public void open() {
        driver.get(pageUrl);
    }

    public boolean isCurrentPage() {
        return driver.getCurrentUrl().contains(pageUrl);
    }

    public String getCurrentUrl() {
        return driver.getCurrentUrl();
    }

    public String getPageTitle() {
        return driver.getTitle();
    }

    // Explicit wait encapsulation
    protected WebElement waitForVisible(WebElement element) {
        return wait.until(ExpectedConditions.visibilityOf(element));
    }

    protected WebElement waitForClickable(WebElement element) {
        return wait.until(ExpectedConditions.elementToBeClickable(element));
    }

    protected boolean waitForInvisible(WebElement element) {
        return wait.until(ExpectedConditions.invisibilityOf(element));
    }

    protected void waitForPageLoaded() {
        wait.until(driver ->
            ((org.openqa.selenium.JavascriptExecutor) driver)
                .executeScript("return document.readyState").equals("complete"));
    }
}
```

```java
// ==================== LoginPage.java - Login Page Object ====================
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class LoginPage extends BasePage {

    @FindBy(css = "[data-testid='email-field']")
    private WebElement emailField;

    @FindBy(css = "[data-testid='password-field']")
    private WebElement passwordField;

    @FindBy(css = "[data-testid='login-button']")
    private WebElement loginButton;

    @FindBy(css = "[data-testid='welcome-message']")
    private WebElement welcomeMessage;

    @FindBy(css = "[data-testid='error-message']")
    private WebElement errorMessage;

    @FindBy(css = "[data-testid='forgot-password-link']")
    private WebElement forgotPasswordLink;

    public LoginPage(WebDriver driver) {
        super(driver, "/login");
    }

    // Page operations - each operation does one thing
    public LoginPage enterEmail(String email) {
        waitForVisible(emailField).clear();
        emailField.sendKeys(email);
        return this;  // Returns self, supports chainable calls
    }

    public LoginPage enterPassword(String password) {
        waitForVisible(passwordField).clear();
        passwordField.sendKeys(password);
        return this;
    }

    public HomePage clickLoginButton() {
        waitForClickable(loginButton).click();
        return new HomePage(driver);  // Navigate to home page, return new page object
    }

    // Composite operations - encapsulate common workflow
    public HomePage login(String email, String password) {
        enterEmail(email)
            .enterPassword(password)
            .clickLoginButton();
        return new HomePage(driver);
    }

    // Information retrieval - for assertions
    public String getWelcomeMessage() {
        return waitForVisible(welcomeMessage).getText();
    }

    public String getErrorMessage() {
        return waitForVisible(errorMessage).getText();
    }

    public boolean isErrorMessageDisplayed() {
        try {
            return errorMessage.isDisplayed();
        } catch (org.openqa.selenium.NoSuchElementException e) {
            return false;
        }
    }

    // Navigation operations
    public ForgotPasswordPage clickForgotPassword() {
        waitForClickable(forgotPasswordLink).click();
        return new ForgotPasswordPage(driver);
    }
}
```

```java
// ==================== HomePage.java - Home Page Object ====================
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import java.util.List;

public class HomePage extends BasePage {

    @FindBy(css = "[data-testid='search-field']")
    private WebElement searchField;

    @FindBy(css = "[data-testid='search-button']")
    private WebElement searchButton;

    @FindBy(css = "[data-testid='cart-icon']")
    private WebElement cartIcon;

    @FindBy(css = "[data-testid='user-menu']")
    private WebElement userMenu;

    @FindBy(css = "[data-testid='product-card']")
    private List<WebElement> productCards;

    public HomePage(WebDriver driver) {
        super(driver, "/home");
    }

    public HomePage enterSearchTerm(String term) {
        waitForVisible(searchField).sendKeys(term);
        return this;
    }

    public SearchResultsPage clickSearch() {
        waitForClickable(searchButton).click();
        return new SearchResultsPage(driver);
    }

    public SearchResultsPage searchFor(String term) {
        enterSearchTerm(term);
        return clickSearch();
    }

    public CartPage clickCart() {
        waitForClickable(cartIcon).click();
        return new CartPage(driver);
    }

    public int getProductCount() {
        return productCards.size();
    }

    public boolean isDisplayed() {
        return waitForVisible(searchField).isDisplayed();
    }
}
```

```java
// ==================== PageObjectManager.java - Page Object Manager ====================
package pages;

import org.openqa.selenium.WebDriver;

public class PageObjectManager {
    private final WebDriver driver;

    // Use lazy-loading cache
    private LoginPage loginPage;
    private HomePage homePage;
    private CartPage cartPage;

    public PageObjectManager(WebDriver driver) {
        this.driver = driver;
    }

    public LoginPage getLoginPage() {
        if (loginPage == null) {
            loginPage = new LoginPage(driver);
        }
        return loginPage;
    }

    public HomePage getHomePage() {
        if (homePage == null) {
            homePage = new HomePage(driver);
        }
        return homePage;
    }

    public CartPage getCartPage() {
        if (cartPage == null) {
            cartPage = new CartPage(driver);
        }
        return cartPage;
    }
}
```

#### JavaScript/TypeScript Complete Implementation

```typescript
// ==================== base.page.ts - Page Base Class ====================
import { Page, Locator, expect } from '@playwright/test';

export abstract class BasePage {
    protected readonly page: Page;
    protected readonly baseUrl: string;

    constructor(page: Page, baseUrl: string) {
        this.page = page;
        this.baseUrl = baseUrl;
    }

    async open(): Promise<void> {
        await this.page.goto(this.baseUrl);
    }

    async isCurrentPage(): Promise<boolean> {
        const url = this.page.url();
        return url.includes(this.baseUrl);
    }

    async getCurrentUrl(): Promise<string> {
        return this.page.url();
    }

    async getPageTitle(): Promise<string> {
        return this.page.title();
    }

    // Explicit wait encapsulation
    protected async waitForVisible(locator: Locator): Promise<Locator> {
        await expect(locator).toBeVisible();
        return locator;
    }

    protected async waitForClickable(locator: Locator): Promise<Locator> {
        await expect(locator).toBeEnabled();
        return locator;
    }

    protected async waitForPageLoaded(): Promise<void> {
        await this.page.waitForLoadState('networkidle');
    }

    // Helper method to get elements by data-testid
    protected getByTestId(testId: string): Locator {
        return this.page.locator(`[data-testid='${testId}']`);
    }
}
```

```typescript
// ==================== login.page.ts - Login Page Object ====================
import { Page, Locator } from '@playwright/test';
import { BasePage } from './base.page';
import { HomePage } from './home.page';
import { ForgotPasswordPage } from './forgot-password.page';

export class LoginPage extends BasePage {
    // Element locators
    readonly emailField: Locator;
    readonly passwordField: Locator;
    readonly loginButton: Locator;
    readonly welcomeMessage: Locator;
    readonly errorMessage: Locator;
    readonly forgotPasswordLink: Locator;

    constructor(page: Page) {
        super(page, '/login');
        this.emailField = this.getByTestId('email-field');
        this.passwordField = this.getByTestId('password-field');
        this.loginButton = this.getByTestId('login-button');
        this.welcomeMessage = this.getByTestId('welcome-message');
        this.errorMessage = this.getByTestId('error-message');
        this.forgotPasswordLink = this.getByTestId('forgot-password-link');
    }

    // Page operations
    async enterEmail(email: string): Promise<LoginPage> {
        await this.emailField.clear();
        await this.emailField.fill(email);
        return this;
    }

    async enterPassword(password: string): Promise<LoginPage> {
        await this.passwordField.clear();
        await this.passwordField.fill(password);
        return this;
    }

    async clickLoginButton(): Promise<HomePage> {
        await this.loginButton.click();
        return new HomePage(this.page);
    }

    // Composite operations
    async login(email: string, password: string): Promise<HomePage> {
        await this.enterEmail(email);
        await this.enterPassword(password);
        return this.clickLoginButton();
    }

    // Information retrieval
    async getWelcomeMessage(): Promise<string> {
        return this.welcomeMessage.textContent() ?? '';
    }

    async getErrorMessage(): Promise<string> {
        return this.errorMessage.textContent() ?? '';
    }

    async isErrorMessageDisplayed(): Promise<boolean> {
        return this.errorMessage.isVisible();
    }

    // Navigation operations
    async clickForgotPassword(): Promise<ForgotPasswordPage> {
        await this.forgotPasswordLink.click();
        return new ForgotPasswordPage(this.page);
    }
}
```

```typescript
// ==================== home.page.ts - Home Page Object ====================
import { Page, Locator } from '@playwright/test';
import { BasePage } from './base.page';
import { CartPage } from './cart.page';
import { SearchResultsPage } from './search-results.page';

export class HomePage extends BasePage {
    readonly searchField: Locator;
    readonly searchButton: Locator;
    readonly cartIcon: Locator;
    readonly productCards: Locator;

    constructor(page: Page) {
        super(page, '/home');
        this.searchField = this.getByTestId('search-field');
        this.searchButton = this.getByTestId('search-button');
        this.cartIcon = this.getByTestId('cart-icon');
        this.productCards = this.getByTestId('product-card');
    }

    async enterSearchTerm(term: string): Promise<HomePage> {
        await this.searchField.fill(term);
        return this;
    }

    async clickSearch(): Promise<SearchResultsPage> {
        await this.searchButton.click();
        return new SearchResultsPage(this.page);
    }

    async searchFor(term: string): Promise<SearchResultsPage> {
        await this.enterSearchTerm(term);
        return this.clickSearch();
    }

    async clickCart(): Promise<CartPage> {
        await this.cartIcon.click();
        return new CartPage(this.page);
    }

    async getProductCount(): Promise<number> {
        return this.productCards.count();
    }

    async isDisplayed(): Promise<boolean> {
        return this.searchField.isVisible();
    }
}
```

#### Ruby Complete Implementation

```ruby
# ==================== base_page.rb - Page Base Class ====================
# features/support/pages/base_page.rb

class BasePage
    include Capybara::DSL
    include RSpec::Matchers

    attr_reader :page

    def initialize
        @page = Capybara.current_session
    end

    def visit_page(path)
        visit(path)
    end

    def current_url
        page.current_url
    end

    def current_path
        page.current_path
    end

    def page_title
        page.title
    end

    # Explicit wait encapsulation
    def wait_for_visible(selector, timeout: 10)
        page.has_css?(selector, visible: true, wait: timeout)
        page.find(selector)
    end

    def wait_for_clickable(selector, timeout: 10)
        page.has_css?(selector, visible: true, wait: timeout)
        page.find(selector)
    end

    def wait_for_page_loaded
        page.has_no_css?('body.loading', wait: 10)
    end

    # Helper method to get elements by data-testid
    def element_by_testid(testid)
        page.find("[data-testid='#{testid}']")
    end

    def elements_by_testid(testid)
        page.all("[data-testid='#{testid}']")
    end
end
```

```ruby
# ==================== login_page.rb - Login Page Object ====================
# features/support/pages/login_page.rb

require_relative 'base_page'
require_relative 'home_page'
require_relative 'forgot_password_page'

class LoginPage < BasePage
    LOGIN_URL = '/login'.freeze

    def email_field
        element_by_testid('email-field')
    end

    def password_field
        element_by_testid('password-field')
    end

    def login_button
        element_by_testid('login-button')
    end

    def welcome_message
        element_by_testid('welcome-message')
    end

    def error_message
        element_by_testid('error-message')
    end

    def forgot_password_link
        element_by_testid('forgot-password-link')
    end

    # Page operations
    def enter_email(email)
        wait_for_visible("[data-testid='email-field']")
        email_field.set(email)
        self  # Returns self, supports chainable calls
    end

    def enter_password(password)
        password_field.set(password)
        self
    end

    def click_login_button
        login_button.click
        HomePage.new  # Navigate to home page, return new page object
    end

    # Composite operations
    def login(email, password)
        enter_email(email)
        enter_password(password)
        click_login_button
    end

    # Information retrieval
    def get_welcome_message
        welcome_message.text
    end

    def get_error_message
        error_message.text
    end

    def error_message_displayed?
        page.has_css?("[data-testid='error-message']", visible: true)
    end

    # Navigation operations
    def click_forgot_password
        forgot_password_link.click
        ForgotPasswordPage.new
    end
end
```

### 5.4 Using Page Objects in Step Definitions

```java
// ==================== LoginSteps.java - Step Definitions Using Page Object ====================
package stepdefinitions.auth;

import io.cucumber.java.en.*;
import pages.LoginPage;
import pages.HomePage;
import context.TestContext;
import static org.assertj.core.api.Assertions.*;

public class LoginSteps {
    private final LoginPage loginPage;
    private final TestContext context;
    private HomePage homePage;

    // Dependency injection (PicoContainer)
    public LoginSteps(LoginPage loginPage, TestContext context) {
        this.loginPage = loginPage;
        this.context = context;
    }

    @Given("the user is on the login page")
    public void theUserIsOnTheLoginPage() {
        loginPage.open();
    }

    @When("the user enters email {string}")
    public void theUserEntersEmail(String email) {
        loginPage.enterEmail(email);
    }

    @When("the user enters password {string}")
    public void theUserEntersPassword(String password) {
        loginPage.enterPassword(password);
    }

    @When("the user clicks the login button")
    public void theUserClicksTheLoginButton() {
        homePage = loginPage.clickLoginButton();  // Page Object returns next page
    }

    // Composite step - call Page Object composite method
    @When("the user logs in with email {string} and password {string}")
    public void theUserLogsIn(String email, String password) {
        homePage = loginPage.login(email, password);
    }

    @Then("the user should see the welcome message {string}")
    public void theUserShouldSeeWelcomeMessage(String expectedMessage) {
        assertThat(homePage.getWelcomeMessage()).contains(expectedMessage);
    }

    @Then("the user should see an error message {string}")
    public void theUserShouldSeeErrorMessage(String expectedError) {
        assertThat(loginPage.getErrorMessage()).isEqualTo(expectedError);
    }
}
```

### 5.5 Page Object Advanced Patterns

#### Component Pattern (Page Components / Page Fragments)

For components that repeatedly appear on pages (such as navigation bars, footers, search boxes), they can be extracted into independent component classes:

```java
// NavigationBar.java - Reusable Navigation Bar Component
package pages.components;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import pages.BasePage;
import pages.CartPage;
import pages.SearchResultsPage;

public class NavigationBar extends BasePage {

    @FindBy(css = "[data-testid='nav-search-field']")
    private WebElement searchField;

    @FindBy(css = "[data-testid='nav-cart-icon']")
    private WebElement cartIcon;

    @FindBy(css = "[data-testid='nav-user-menu']")
    private WebElement userMenu;

    @FindBy(css = "[data-testid='nav-home-link']")
    private WebElement homeLink;

    public NavigationBar(WebDriver driver) {
        super(driver, "");  // No independent URL
    }

    public SearchResultsPage searchFor(String term) {
        waitForVisible(searchField).sendKeys(term);
        searchField.submit();
        return new SearchResultsPage(driver);
    }

    public CartPage clickCart() {
        waitForClickable(cartIcon).click();
        return new CartPage(driver);
    }
}

// Using navigation bar component in multiple pages
public class HomePage extends BasePage {
    private final NavigationBar navigationBar;

    public HomePage(WebDriver driver) {
        super(driver, "/home");
        this.navigationBar = new NavigationBar(driver);
    }

    public NavigationBar nav() {
        return navigationBar;
    }
}

// Using components in Step Definitions
@When("the user searches for {string} from the navigation bar")
public void searchFromNav(String term) {
    homePage.nav().searchFor(term);
}
```

#### Loadable Component Pattern

Ensure the page is fully loaded before performing operations:

```java
// LoadablePage.java - Loadable Page Interface
package pages;

public interface LoadablePage<T> {
    T waitForPageToLoad();
    boolean isPageLoaded();
}

// CheckoutPage.java - Implementing Loadable Page
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

public class CheckoutPage extends BasePage implements LoadablePage<CheckoutPage> {

    @FindBy(css = "[data-testid='checkout-form']")
    private WebElement checkoutForm;

    @FindBy(css = "[data-testid='loading-spinner']")
    private WebElement loadingSpinner;

    public CheckoutPage(WebDriver driver) {
        super(driver, "/checkout");
    }

    @Override
    public CheckoutPage waitForPageToLoad() {
        waitForInvisible(loadingSpinner);   // Wait for loading to complete
        waitForVisible(checkoutForm);       // Wait for form to appear
        return this;
    }

    @Override
    public boolean isPageLoaded() {
        return checkoutForm.isDisplayed() && !loadingSpinner.isDisplayed();
    }
}

// Usage in Step Definitions
@When("the user proceeds to checkout")
public void proceedToCheckout() {
    checkoutPage.open()
                .waitForPageToLoad();  // Ensure page is fully loaded
}
```

### 5.6 Page Object Best Practices Checklist

| # | Best Practice | Description |
|---|---------|------|
| 1 | **Use `data-testid` attribute** | More stable than CSS class names and XPath |
| 2 | **One class per page** | Maintain single responsibility |
| 3 | **Encapsulate locators as private** | Do not expose externally |
| 4 | **Navigation methods return target page** | Support fluent API |
| 5 | **Use explicit waits** | Do not use `Thread.sleep` |
| 6 | **Do not place assertions in Page Objects** | Assertions belong to the test layer |
| 7 | **Encapsulate components as independent classes** | Navigation bars, search boxes, etc. |
| 8 | **Composite operation methods** | Encapsulate common operation sequences |
| 9 | **Inherit base class for common functionality** | Waiting, navigation, etc. |
| 10 | **Use PageFactory** | Initialize elements in JVM projects |
| 11 | **Handle dynamic content** | Use parameterized methods |
| 12 | **Record page URL** | Facilitate navigation verification |

### 5.7 Page Object Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------|------|----------|
| **God Page Object** | One class containing hundreds of lines, managing multiple pages | Split by page |
| **Exposing WebDriver** | Page Object returns WebElement to caller | Encapsulate all operations |
| **Assertions in PO** | `assertEquals(expected, element.getText())` | Only return data |
| **Using Thread.sleep** | `Thread.sleep(3000)` fixed wait | Use explicit waits |
| **Hard-coded locators** | Selectors scattered everywhere | Centralize as constants |
| **PO mutual dependency** | Page A depends on Page B, B depends on A | Coordinate through Step Definitions |
| **PO calls Step Def** | Reverse dependency | Strict layering |
| **Missing error handling** | Throwing raw exceptions when element not found | Wrap as business exceptions |



---

## 6. Screenplay Pattern

### 6.1 Screenplay Pattern Overview

The Screenplay Pattern (also known as "Screenplay Pattern") is a modern evolution of the Page Object Model, proposed by the Serenity BDD team. It organizes test code around **user behavior**, inspired by the Actor model.

**Core philosophy:**

```
Traditional POM thinking:          Screenplay thinking:
"The user is on the login page"  →   "User (Actor) attempts (Ability) login (Task)"
"Click the login button"        →   "The user performs the login operation"
"Enter username"          →   "The user logs in with her credentials"
```

**Screenplay vs Page Object comparison:**

| Characteristic | Page Object | Screenplay |
|------|-------------|------------|
| **Focus** | Page structure and elements | User behavior and goals |
| **Organization** | Organized by page | Organized by task |
| **Reusability** | Page-level reuse | Task-level reuse (finer granularity) |
| **Readability** | Code focuses on pages | Code reads like user stories |
| **Learning curve** | Lower | Higher |
| **Applicable scale** | Small to medium projects | Large complex projects |
| **God class risk** | High (pages grow and bloat) | Low (small tasks freely composable) |
| **BDD alignment** | Medium | **Extremely high** (Both focus on behavior) |

### 6.2 Screenplay Core Concepts

The Screenplay pattern consists of four core concepts:

```
┌─────────────────────────────────────────────────────────────────┐
│                       Screenplay Pattern                            │
│                                                                  │
│   Actor (User/Role)                                              │
│     ├── can (Ability)                                      │
│     │      └── BrowseTheWeb (Use browser)                        │
│     │      └── CallAnApi (Call API)                             │
│     │      └── QueryDatabase (Query database)                       │
│     │                                                           │
│     ├── attemptsTo (Task)                                  │
│     │      └── LogIn (Login)                                     │
│     │      └── AddToCart (Add to cart)                         │
│     │      └── CompleteCheckout (Complete checkout)                      │
│     │                                                           │
│     └── should seeThat (Question)                          │
│            └── TheWelcomeMessage (Welcome message)                     │
│            └── TheCartTotal (Cart total)                        │
│            └── TheOrderStatus (Order status)                        │
│                                                                  │
│   Tasks are composed of Interactions:                                      │
│   LogIn = Enter(email) + Enter(password) + Click(button)       │
└─────────────────────────────────────────────────────────────────┘
```

#### Concept Details

| Concept | Description | Analogy |
|------|------|------|
| **Actor (Actor/Role)** | User or external system interacting with the system | "Protagonist" in the test scenario |
| **Ability (Capability)** | Capabilities/tools an Actor possesses | User can "browse the web" or "call APIs" |
| **Task (Task)** | High-level business activity the Actor wants to complete | "Login", "Checkout" are tasks |
| **Interaction (Interaction)** | Low-level atomic operations | Click, input, scroll |
| **Question (Question)** | Query information from the system for assertions | "What is the cart total?" |
| **Target (Target)** | Locator description for UI elements | Encapsulation of element locators |

### 6.3 Standard Screenplay Implementation

#### JVM (Java) - Serenity BDD Implementation

```java
// ==================== Actor.java - Actor/Role ====================
package screenplay;

import net.serenitybdd.screenplay.Actor;
import net.serenitybdd.screenplay.abilities.BrowseTheWeb;
import org.openqa.selenium.WebDriver;

public class ActorFactory {

    public static Actor named(String name) {
        return Actor.named(name);
    }

    public static Actor aStandardUser(WebDriver driver) {
        return Actor.named("Standard User")
                .whoCan(BrowseTheWeb.with(driver));
    }

    public static Actor anAdminUser(WebDriver driver) {
        return Actor.named("Admin User")
                .whoCan(BrowseTheWeb.with(driver));
    }
}
```

```java
// ==================== Target.java - Element Target Definitions ====================
package screenplay.targets;

import net.serenitybdd.screenplay.targets.Target;

public class LoginPageTargets {
    public static final Target EMAIL_FIELD = Target.the("email field")
            .locatedBy("[data-testid='email-field']");

    public static final Target PASSWORD_FIELD = Target.the("password field")
            .locatedBy("[data-testid='password-field']");

    public static final Target LOGIN_BUTTON = Target.the("login button")
            .locatedBy("[data-testid='login-button']");

    public static final Target WELCOME_MESSAGE = Target.the("welcome message")
            .locatedBy("[data-testid='welcome-message']");

    public static final Target ERROR_MESSAGE = Target.the("error message")
            .locatedBy("[data-testid='error-message']");
}
```

```java
// ==================== LogIn.java - Login Task ====================
package screenplay.tasks;

import net.serenitybdd.screenplay.Task;
import net.serenitybdd.screenplay.actions.Click;
import net.serenitybdd.screenplay.actions.Enter;
import screenplay.targets.LoginPageTargets;

import static net.serenitybdd.screenplay.Tasks.instrumented;

public class LogIn implements Task {
    private final String email;
    private final String password;

    // Private constructor - force use of factory methods
    private LogIn(String email, String password) {
        this.email = email;
        this.password = password;
    }

    // Factory method - better readability
    public static LogIn withCredentials(String email, String password) {
        return instrumented(LogIn.class, email, password);
    }

    public static LogIn as(User user) {
        return instrumented(LogIn.class, user.getEmail(), user.getPassword());
    }

    @Override
    public <T extends Actor> void performAs(T actor) {
        actor.attemptsTo(
            Enter.theValue(email).into(LoginPageTargets.EMAIL_FIELD),
            Enter.theValue(password).into(LoginPageTargets.PASSWORD_FIELD),
            Click.on(LoginPageTargets.LOGIN_BUTTON)
        );
    }
}
```

```java
// ==================== AddToCart.java - Add to Cart Task ====================
package screenplay.tasks;

import net.serenitybdd.screenplay.Task;
import net.serenitybdd.screenplay.actions.Click;
import net.serenitybdd.screenplay.actions.Scroll;
import net.serenitybdd.screenplay.targets.Target;

import static net.serenitybdd.screenplay.Tasks.instrumented;

public class AddToCart implements Task {
    private final String productName;

    private AddToCart(String productName) {
        this.productName = productName;
    }

    public static AddToCart theProduct(String productName) {
        return instrumented(AddToCart.class, productName);
    }

    @Override
    public <T extends Actor> void performAs(T actor) {
        Target productCard = Target.the(productName + " card")
            .locatedBy("[data-testid='product-card']:has-text('{0}')", productName);

        Target addToCartButton = Target.the("Add to Cart button for " + productName)
            .locatedBy("//[data-testid='product-card'][contains(.,'{0}')]//button[@data-testid='add-to-cart']", productName);

        actor.attemptsTo(
            Scroll.to(productCard),
            Click.on(addToCartButton)
        );
    }
}
```

```java
// ==================== NavigateTo.java - Navigation Task ====================
package screenplay.tasks;

import net.serenitybdd.screenplay.Task;
import net.serenitybdd.screenplay.actions.Open;

import static net.serenitybdd.screenplay.Tasks.instrumented;

public class NavigateTo implements Task {
    private final String pageUrl;

    private NavigateTo(String pageUrl) {
        this.pageUrl = pageUrl;
    }

    public static NavigateTo theLoginPage() {
        return instrumented(NavigateTo.class, "/login");
    }

    public static NavigateTo theHomePage() {
        return instrumented(NavigateTo.class, "/home");
    }

    public static NavigateTo theCartPage() {
        return instrumented(NavigateTo.class, "/cart");
    }

    @Override
    public <T extends Actor> void performAs(T actor) {
        actor.attemptsTo(
            Open.url(pageUrl)
        );
    }
}
```

```java
// ==================== TheWelcomeMessage.java - Query Question ====================
package screenplay.questions;

import net.serenitybdd.screenplay.Question;
import net.serenitybdd.screenplay.questions.Text;
import screenplay.targets.LoginPageTargets;

public class TheWelcomeMessage implements Question<String> {

    // Singleton pattern - stateless questions can be reused
    private static final TheWelcomeMessage INSTANCE = new TheWelcomeMessage();

    public static TheWelcomeMessage text() {
        return INSTANCE;
    }

    @Override
    public String answeredBy(net.serenitybdd.screenplay.Actor actor) {
        return Text.of(LoginPageTargets.WELCOME_MESSAGE).answeredBy(actor);
    }

    @Override
    public String toString() {
        return "the welcome message";
    }
}
```

```java
// ==================== TheCartTotal.java - Cart Total Query ====================
package screenplay.questions;

import net.serenitybdd.screenplay.Question;
import net.serenitybdd.screenplay.questions.Text;
import net.serenitybdd.screenplay.targets.Target;

import java.math.BigDecimal;

public class TheCartTotal implements Question<BigDecimal> {

    private static final Target CART_TOTAL = Target.the("cart total")
            .locatedBy("[data-testid='cart-total']");

    public static TheCartTotal value() {
        return new TheCartTotal();
    }

    @Override
    public BigDecimal answeredBy(net.serenitybdd.screenplay.Actor actor) {
        String text = Text.of(CART_TOTAL).answeredBy(actor);
        // Parse "$29.99" -> 29.99
        return new BigDecimal(text.replace("$", "").trim());
    }

    @Override
    public String toString() {
        return "the cart total";
    }
}
```

```java
// ==================== ScreenplaySteps.java - Using Screenplay in Step Definitions ====================
package stepdefinitions;

import io.cucumber.java.en.*;
import net.serenitybdd.screenplay.Actor;
import net.serenitybdd.screenplay.abilities.BrowseTheWeb;
import screenplay.questions.TheWelcomeMessage;
import screenplay.tasks.LogIn;
import screenplay.tasks.NavigateTo;
import screenplay.tasks.AddToCart;
import model.User;

import static net.serenitybdd.screenplay.GivenWhenThen.*;
import static org.assertj.core.api.Assertions.*;

public class ScreenplaySteps {

    private Actor currentActor;

    @Given("{actor} is on the login page")
    public void actorIsOnTheLoginPage(Actor actor) {
        this.currentActor = actor;
        actor.wasAbleTo(NavigateTo.theLoginPage());
    }

    @When("{actor} logs in with email {string} and password {string}")
    public void actorLogsIn(Actor actor, String email, String password) {
        actor.attemptsTo(LogIn.withCredentials(email, password));
    }

    @When("{actor} logs in as {user}")
    public void actorLogsInAsUser(Actor actor, User user) {
        actor.attemptsTo(LogIn.as(user));
    }

    @Then("{actor} should see the welcome message containing {string}")
    public void actorShouldSeeWelcomeMessage(Actor actor, String expectedMessage) {
        String actualMessage = actor.asksFor(TheWelcomeMessage.text());
        assertThat(actualMessage).contains(expectedMessage);

        // Or use Serenity seeThat syntax
        actor.should(
            seeThat(
                TheWelcomeMessage.text(),
                org.hamcrest.Matchers.containsString(expectedMessage)
            )
        );
    }

    @When("{actor} adds {string} to her cart")
    public void actorAddsProductToCart(Actor actor, String productName) {
        actor.attemptsTo(AddToCart.theProduct(productName));
    }
}
```

```java
// ==================== ParameterType.java - Custom Parameter Types ====================
package stepdefinitions;

import io.cucumber.java.ParameterType;
import model.User;
import net.serenitybdd.screenplay.Actor;
import net.serenitybdd.screenplay.actors.OnStage;

public class CustomParameterTypes {

    @ParameterType(".*")
    public Actor actor(String actorName) {
        return OnStage.theActorCalled(actorName);
    }

    @ParameterType(".*")
    public User user(String username) {
        return TestDataGenerator.createUser(username);
    }
}
```

#### JavaScript/TypeScript - Custom Screenplay Implementation

The JavaScript ecosystem does not have an official Screenplay library (other than Serenity/JS), but you can implement it yourself:

```typescript
// ==================== actor.ts - Actor Core Class ====================
export interface Ability {
    name: string;
}

export interface Task {
    performAs(actor: Actor): Promise<void>;
    toString(): string;
}

export interface Interaction {
    performAs(actor: Actor): Promise<void>;
}

export interface Question<T> {
    answeredBy(actor: Actor): Promise<T>;
}

export class Actor {
    private readonly name: string;
    private readonly abilities: Map<string, Ability> = new Map();
    private readonly remembered: Map<string, any> = new Map();

    constructor(name: string) {
        this.name = name;
    }

    static named(name: string): Actor {
        return new Actor(name);
    }

    whoCan(ability: Ability): Actor {
        this.abilities.set(ability.name, ability);
        return this;
    }

    abilityTo<T extends Ability>(abilityName: string): T {
        const ability = this.abilities.get(abilityName);
        if (!ability) {
            throw new Error(`${this.name} does not have the ability to ${abilityName}`);
        }
        return ability as T;
    }

    async attemptsTo(...tasks: Task[]): Promise<void> {
        for (const task of tasks) {
            await task.performAs(this);
        }
    }

    async asksFor<T>(question: Question<T>): Promise<T> {
        return question.answeredBy(this);
    }

    shouldSee<T>(question: Question<T>, assertion: (actual: T) => void): Promise<void> {
        return this.asksFor(question).then(assertion);
    }

    remembers(key: string, value: any): Actor {
        this.remembered.set(key, value);
        return this;
    }

    recalls<T>(key: string): T {
        return this.remembered.get(key);
    }

    toString(): string {
        return this.name;
    }
}
```

```typescript
// ==================== browse-the-web.ts - Browse the Web Ability ====================
import { Page } from '@playwright/test';
import { Ability } from './actor';

export class BrowseTheWeb implements Ability {
    readonly name = 'browseTheWeb';

    constructor(private readonly page: Page) {}

    static using(page: Page): BrowseTheWeb {
        return new BrowseTheWeb(page);
    }

    getPage(): Page {
        return this.page;
    }
}
```

```typescript
// ==================== interactions.ts - Basic Interactions ====================
import { Actor, Interaction } from './actor';
import { BrowseTheWeb } from './browse-the-web';

export class Click {
    static on(selector: string): Interaction {
        return {
            async performAs(actor: Actor): Promise<void> {
                const page = actor.abilityTo<BrowseTheWeb>('browseTheWeb').getPage();
                await page.click(selector);
            },
            toString(): string {
                return `click on ${selector}`;
            }
        };
    }
}

export class Enter {
    private constructor(private readonly value: string) {}

    static theValue(value: string): Enter {
        return new Enter(value);
    }

    into(selector: string): Interaction {
        return {
            performAs: async (actor: Actor): Promise<void> => {
                const page = actor.abilityTo<BrowseTheWeb>('browseTheWeb').getPage();
                await page.fill(selector, this.value);
            },
            toString: (): string => `enter "${this.value}" into ${selector}`
        };
    }
}

export class Navigate {
    static to(url: string): Interaction {
        return {
            async performAs(actor: Actor): Promise<void> {
                const page = actor.abilityTo<BrowseTheWeb>('browseTheWeb').getPage();
                await page.goto(url);
            },
            toString(): string {
                return `navigate to ${url}`;
            }
        };
    }
}

export class Read {
    static textOf(selector: string): Interaction {
        return {
            async performAs(actor: Actor): Promise<void> {
                // This interaction does not actually modify state
                // In actual use, typically read directly in Questions
            },
            toString(): string {
                return `read text of ${selector}`;
            }
        };
    }
}
```

```typescript
// ==================== tasks.ts - Business Tasks ====================
import { Actor, Task, Question } from './actor';
import { Click, Enter, Navigate } from './interactions';
import { BrowseTheWeb } from './browse-the-web';

// Task: Login
export class LogIn implements Task {
    private constructor(
        private readonly email: string,
        private readonly password: string
    ) {}

    static withCredentials(email: string, password: string): LogIn {
        return new LogIn(email, password);
    }

    async performAs(actor: Actor): Promise<void> {
        await actor.attemptsTo(
            Navigate.to('/login'),
            Enter.theValue(this.email).into("[data-testid='email-field']"),
            Enter.theValue(this.password).into("[data-testid='password-field']"),
            Click.on("[data-testid='login-button']")
        );
    }

    toString(): string {
        return `log in as ${this.email}`;
    }
}

// Task: Add to cart
export class AddToCart implements Task {
    private constructor(private readonly productName: string) {}

    static theProduct(productName: string): AddToCart {
        return new AddToCart(productName);
    }

    async performAs(actor: Actor): Promise<void> {
        const productSelector = `[data-testid='product-card']:has-text("${this.productName}")`;
        const addButton = `${productSelector} [data-testid='add-to-cart']`;

        await actor.attemptsTo(
            Click.on(addButton)
        );
    }

    toString(): string {
        return `add "${this.productName}" to cart`;
    }
}

// Question: Cart total
export class TheCartTotal implements Question<string> {
    static displayed(): TheCartTotal {
        return new TheCartTotal();
    }

    async answeredBy(actor: Actor): Promise<string> {
        const page = actor.abilityTo<BrowseTheWeb>('browseTheWeb').getPage();
        return page.textContent("[data-testid='cart-total']") ?? '';
    }

    toString(): string {
        return 'the cart total';
    }
}

// Question: Welcome message
export class TheWelcomeMessage implements Question<string> {
    static displayed(): TheWelcomeMessage {
        return new TheWelcomeMessage();
    }

    async answeredBy(actor: Actor): Promise<string> {
        const page = actor.abilityTo<BrowseTheWeb>('browseTheWeb').getPage();
        return page.textContent("[data-testid='welcome-message']") ?? '';
    }

    toString(): string {
        return 'the welcome message';
    }
}
```

```typescript
// ==================== login.steps.ts - Using Screenplay in Step Definitions ====================
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { Actor } from '../../screenplay/actor';
import { BrowseTheWeb } from '../../screenplay/browse-the-web';
import { LogIn, AddToCart } from '../../screenplay/tasks';
import { TheWelcomeMessage, TheCartTotal } from '../../screenplay/tasks';

Given('{string} is on the login page', async function(actorName: string) {
    this.actor = Actor.named(actorName)
        .whoCan(BrowseTheWeb.using(this.page));
});

When('{string} logs in with email {string} and password {string}',
    async function(actorName: string, email: string, password: string) {
        await this.actor.attemptTo(LogIn.withCredentials(email, password));
    }
);

Then('{string} should see the welcome message containing {string}',
    async function(actorName: string, expectedMessage: string) {
        const actualMessage = await this.actor.asksFor(TheWelcomeMessage.displayed());
        expect(actualMessage).toContain(expectedMessage);
    }
);

When('{string} adds {string} to her cart', async function(actorName: string, product: string) {
    await this.actor.attemptsTo(AddToCart.theProduct(product));
});

Then('the cart total should be {string}', async function(expectedTotal: string) {
    const actualTotal = await this.actor.asksFor(TheCartTotal.displayed());
    expect(actualTotal.trim()).toBe(expectedTotal);
});
```

#### Ruby - Custom Screenplay Implementation

```ruby
# ==================== actor.rb - Actor Core Class ====================
# lib/screenplay/actor.rb

module Screenplay
    class Actor
        attr_reader :name

        def initialize(name)
            @name = name
            @abilities = {}
            @remembered = {}
        end

        def self.named(name)
            new(name)
        end

        def who_can(ability)
            @abilities[ability.name] = ability
            self
        end

        def ability_to(ability_name)
            ability = @abilities[ability_name]
            raise "#{@name} does not have the ability to #{ability_name}" unless ability
            ability
        end

        def attempts_to(*tasks)
            tasks.each { |task| task.perform_as(self) }
        end

        def asks_for(question)
            question.answered_by(self)
        end

        def should_see_that(question, &matcher)
            actual = asks_for(question)
            matcher.call(actual)
        end

        def remembers(key, value)
            @remembered[key] = value
            self
        end

        def recalls(key)
            @remembered[key]
        end

        def to_s
            @name
        end
    end
end
```

```ruby
# ==================== ability.rb - Ability Base Class ====================
# lib/screenplay/ability.rb

module Screenplay
    class Ability
        attr_reader :name

        def initialize(name)
            @name = name
        end
    end
end
```

```ruby
# ==================== browse_the_web.rb - Browse the Web Ability ====================
# lib/screenplay/abilities/browse_the_web.rb

require_relative '../ability'

module Screenplay
    class BrowseTheWeb < Ability
        attr_reader :session

        def initialize(session = Capybara.current_session)
            super('browseTheWeb')
            @session = session
        end

        def self.with(session = Capybara.current_session)
            new(session)
        end
    end
end
```

```ruby
# ==================== task.rb - Task Base Class ====================
# lib/screenplay/task.rb

module Screenplay
    class Task
        def perform_as(actor)
            raise NotImplementedError, "Subclasses must implement perform_as"
        end

        def to_s
            self.class.name.split('::').last
        end
    end
end
```

```ruby
# ==================== log_in.rb - Login Task ====================
# lib/screenplay/tasks/log_in.rb

require_relative '../task'

module Screenplay
    class LogIn < Task
        def initialize(email, password)
            @email = email
            @password = password
        end

        def self.with_credentials(email, password)
            new(email, password)
        end

        def perform_as(actor)
            session = actor.ability_to('browseTheWeb').session
            session.visit('/login')
            session.fill_in('email-field', with: @email)
            session.fill_in('password-field', with: @password)
            session.click_button('login-button')
        end

        def to_s
            "log in as #{@email}"
        end
    end
end
```

```ruby
# ==================== question.rb - Question Base Class ====================
# lib/screenplay/question.rb

module Screenplay
    class Question
        def answered_by(actor)
            raise NotImplementedError, "Subclasses must implement answered_by"
        end

        def to_s
            self.class.name.split('::').last
        end
    end
end
```

```ruby
# ==================== the_welcome_message.rb - Welcome Message Query ====================
# lib/screenplay/questions/the_welcome_message.rb

require_relative '../question'

module Screenplay
    class TheWelcomeMessage < Question
        def self.displayed
            new
        end

        def answered_by(actor)
            session = actor.ability_to('browseTheWeb').session
            session.find("[data-testid='welcome-message']").text
        end

        def to_s
            'the welcome message'
        end
    end
end
```

```ruby
# ==================== auth_steps.rb - Using Screenplay in Step Definitions ====================
# step_definitions/auth_steps.rb

Given('{string} is on the login page') do |actor_name|
    @actor = Screenplay::Actor.named(actor_name)
        .who_can(Screenplay::BrowseTheWeb.with)
end

When('{string} logs in with email {string} and password {string}') do |actor_name, email, password|
    @actor.attempts_to(Screenplay::LogIn.with_credentials(email, password))
end

Then('{string} should see the welcome message containing {string}') do |actor_name, expected_message|
    actual_message = @actor.asks_for(Screenplay::TheWelcomeMessage.displayed)
    expect(actual_message).to include(expected_message)
end
```

### 6.4 Screenplay Best Practices

| # | Best Practice | Description |
|---|---------|------|
| 1 | **Tasks use business language naming** | `LogIn` not `ClickLoginButton` |
| 2 | **Tasks are composed of Interactions** | A Task contains multiple atomic interactions |
| 3 | **Questions return pure data** | No assertions in Questions |
| 4 | **Actors named using character names** | `Actor.named("Alice")` not `Actor.named("User1")` |
| 5 | **Use factory methods to create Tasks** | `LogIn.withCredentials()` not `new LogIn()` |
| 6 | **Abilities named using verb phrases** | `BrowseTheWeb` not `WebDriverAbility` |
| 7 | **Stateless Tasks use singleton** | Reusable instances avoid repeated creation |
| 8 | **Targets use data-testid** | Stable element locating strategy |
| 9 | **Use seeThat for assertions** | Serenity declarative assertions |
| 10 | **Compose Tasks to build complex workflows** | `CompleteCheckout = EnterAddress + SelectPayment + Confirm` |

### 6.5 Screenplay Anti-Patterns

| Anti-Pattern | Problem | Solution |
|--------|------|----------|
| **Tasks contain assertions** | Mixing "do" and "verify" | Tasks only do operations, Questions are responsible for queries |
| **Tasks too coarse-grained** | One Task does everything | Split into composable fine-grained Tasks |
| **Tasks too fine-grained** | Every click is a Task | Handle atomic operations at the Interaction layer |
| **Hard-coded selectors** | Selectors written in Tasks | Extract into Target classes |
| **Ignoring Ability** | Actor directly operates low-level | Always access resources through Abilities |
| **Question does too much** | Query + assertion + formatting | Questions only return raw data |

### 6.6 Screenplay Pattern Decision Tree

```
Is Screenplay pattern suitable for your project?

1. Is project scale > 50 scenarios?
   No → Page Object may be sufficient
   Yes → Continue

2. Are there multiple roles (user/admin/guest)?
   No → Page Object may be sufficient
   Yes → Screenplay recommended

3. Are there complex cross-page workflows?
   No → Page Object may be sufficient
   Yes → Screenplay recommended

4. Is the team willing to invest learning cost?
   No → Page Object
   Yes → Screenplay strongly recommended

5. Are you using Serenity BDD?
   Yes → Screenplay native support, strongly recommended
   No → Can implement yourself or choose Page Object
```



---

## 7. Dependency Injection Configuration

### 7.1 Dependency Injection Overview

Dependency Injection (DI) is a core mechanism in BDD test architecture, used to share state between step definition classes (such as WebDriver, API clients, Page Objects, etc.) while maintaining isolation between scenarios.

**Why dependency injection is needed:**

| Problem | Solution |
|------|----------|
| State leakage between scenarios | Each scenario gets an independent context instance |
| Step definition classes cannot share data | DI container automatically injects shared objects |
| Manually managing object lifecycle | Container automatically creates and destroys |
| Tightly coupled test code | Program to interfaces, loosely coupled |
| Repeatedly creating expensive resources | Share instances (such as database connections) |

**DI solution comparison across platforms:**

| Platform | Recommended DI Solution | Alternative | Description |
|------|-------------|----------|------|
| **JVM** | **PicoContainer** (officially recommended) | Spring, Guice, Weld | PicoContainer is lightest, zero config |
| **JavaScript** | **World object** | Custom World class | Context sharing based on this |
| **Ruby** | **World module** | Custom module mixin | Context sharing based on modules |

### 7.2 JVM Dependency Injection

#### PicoContainer (Officially Recommended)

PicoContainer is the officially recommended DI container by Cucumber-JVM because it is **zero configuration**, **no annotations**, and **lightweight**.

**Maven dependencies:**

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Cucumber core -->
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-java</artifactId>
        <version>7.34.3</version>
        <scope>test</scope>
    </dependency>

    <!-- Cucumber JUnit Platform Engine -->
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-junit-platform-engine</artifactId>
        <version>7.34.3</version>
        <scope>test</scope>
    </dependency>

    <!-- PicoContainer - officially recommended DI -->
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-picocontainer</artifactId>
        <version>7.34.3</version>
        <scope>test</scope>
    </dependency>

    <!-- JUnit 5 Suite -->
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-suite</artifactId>
        <version>1.10.0</version>
        <scope>test</scope>
    </dependency>

    <!-- Assertion library -->
    <dependency>
        <groupId>org.assertj</groupId>
        <artifactId>assertj-core</artifactId>
        <version>3.24.2</version>
        <scope>test</scope>
    </dependency>

    <!-- Selenium -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.15.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

**How PicoContainer works:**

```
PicoContainer auto-discovery rules:
1. Scan constructors of all step definition classes
2. If constructor parameter type exists in the container, auto-inject
3. If parameter type does not exist, recursively create instance then inject
4. Create new container instance before each scenario executes (scenario isolation)
5. Step definitions within the same scenario share the same instance
```

**Shared state class (TestContext):**

```java
// ==================== TestContext.java - Scenario-level Shared Context ====================
package context;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import model.User;

/**
 * TestContext is managed by PicoContainer, each scenario gets an independent instance.
 * All step definition classes share the same TestContext instance through constructor injection.
 */
public class TestContext {
    private WebDriver driver;
    private User currentUser;
    private String scenarioName;

    // Lazy-initialize WebDriver - only create when needed
    public WebDriver getDriver() {
        if (driver == null) {
            ChromeOptions options = new ChromeOptions();
            options.addArguments("--start-maximized");
            options.addArguments("--disable-notifications");
            // Headless mode (CI environment)
            if (Boolean.parseBoolean(System.getProperty("headless", "false"))) {
                options.addArguments("--headless=new");
            }
            driver = new ChromeDriver(options);
        }
        return driver;
    }

    public void setCurrentUser(User user) {
        this.currentUser = user;
    }

    public User getCurrentUser() {
        return currentUser;
    }

    public void setScenarioName(String name) {
        this.scenarioName = name;
    }

    public String getScenarioName() {
        return scenarioName;
    }

    // Cleanup resources when scenario ends
    public void closeDriver() {
        if (driver != null) {
            driver.quit();
            driver = null;
        }
    }
}
```

**Config reader class (ConfigReader):**

```java
// ==================== ConfigReader.java - Global Configuration ====================
package utils;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

/**
 * ConfigReader is stateless, PicoContainer can create singleton or new instance each time.
 * Recommended to use as singleton (configuration does not change at runtime).
 */
public class ConfigReader {
    private static final Properties properties = new Properties();
    private static final ConfigReader INSTANCE = new ConfigReader();

    private ConfigReader() {
        loadProperties();
    }

    public static ConfigReader getInstance() {
        return INSTANCE;
    }

    private void loadProperties() {
        try (InputStream stream = getClass().getClassLoader()
                .getResourceAsStream("config.properties")) {
            if (stream != null) {
                properties.load(stream);
            }
        } catch (IOException e) {
            System.err.println("Failed to load config.properties: " + e.getMessage());
        }
    }

    public String getBaseUrl() {
        return properties.getProperty("base.url", "http://localhost:8080");
    }

    public String getBrowser() {
        return properties.getProperty("browser", "chrome");
    }

    public int getImplicitWaitSeconds() {
        return Integer.parseInt(properties.getProperty("implicit.wait.seconds", "10"));
    }
}
```

**Page Object class (auto-inject TestContext):**

```java
// ==================== LoginPage.java - Auto-injected Page Object ====================
package pages;

import context.TestContext;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import utils.ConfigReader;

/**
 * Page Object injects TestContext through constructor,
 * PicoContainer automatically resolves and injects required dependencies.
 */
public class LoginPage extends BasePage {

    @FindBy(css = "[data-testid='email-field']")
    private WebElement emailField;

    @FindBy(css = "[data-testid='password-field']")
    private WebElement passwordField;

    @FindBy(css = "[data-testid='login-button']")
    private WebElement loginButton;

    // PicoContainer auto-injects TestContext
    public LoginPage(TestContext context) {
        super(context.getDriver(), ConfigReader.getInstance().getBaseUrl() + "/login");
    }

    public LoginPage enterEmail(String email) {
        waitForVisible(emailField).clear();
        emailField.sendKeys(email);
        return this;
    }

    public LoginPage enterPassword(String password) {
        waitForVisible(passwordField).clear();
        passwordField.sendKeys(password);
        return this;
    }

    public HomePage clickLoginButton() {
        waitForClickable(loginButton).click();
        return new HomePage(driver);
    }

    public HomePage login(String email, String password) {
        enterEmail(email);
        enterPassword(password);
        return clickLoginButton();
    }
}
```

**HomePage class:**

```java
// ==================== HomePage.java ====================
package pages;

import context.TestContext;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import utils.ConfigReader;

public class HomePage extends BasePage {

    @FindBy(css = "[data-testid='welcome-message']")
    private WebElement welcomeMessage;

    @FindBy(css = "[data-testid='user-menu']")
    private WebElement userMenu;

    // Also needs TestContext to get driver
    public HomePage(TestContext context) {
        super(context.getDriver(), ConfigReader.getInstance().getBaseUrl() + "/home");
    }

    // Package-level constructor - used internally by LoginPage.clickLoginButton()
    HomePage(WebDriver driver) {
        super(driver, "");
    }

    public String getWelcomeMessage() {
        return waitForVisible(welcomeMessage).getText();
    }

    public boolean isUserMenuDisplayed() {
        return waitForVisible(userMenu).isDisplayed();
    }
}
```

**Step definition class (constructor injection):**

```java
// ==================== LoginSteps.java - Step Definitions with Injected Dependencies ====================
package stepdefinitions.auth;

import io.cucumber.java.en.*;
import pages.LoginPage;
import pages.HomePage;
import context.TestContext;
import model.User;
import utils.TestDataGenerator;

import static org.assertj.core.api.Assertions.*;

/**
 * LoginSteps declares required dependencies through constructor.
 * PicoContainer automatically creates and injects these dependencies.
 * All step definition classes within the same scenario share the same TestContext instance.
 */
public class LoginSteps {

    // PicoContainer auto-injection - same instance per scenario
    private final TestContext context;
    private final LoginPage loginPage;

    // HomePage lazily created - only available after successful login
    private HomePage homePage;

    /**
     * Constructor injection - PicoContainer will automatically:
     * 1. Create TestContext instance (scenario-level)
     * 2. Create LoginPage instance (needs TestContext)
     * 3. Inject both into this constructor
     */
    public LoginSteps(TestContext context, LoginPage loginPage) {
        this.context = context;
        this.loginPage = loginPage;
    }

    @Given("{string} has a registered account")
    public void hasRegisteredAccount(String username) {
        User user = TestDataGenerator.createUser(username);
        context.setCurrentUser(user);
    }

    @Given("the user is on the login page")
    public void theUserIsOnTheLoginPage() {
        loginPage.open();
    }

    @When("{string} logs in with her valid credentials")
    public void logsInWithValidCredentials(String username) {
        User user = context.getCurrentUser();
        homePage = loginPage.login(user.getEmail(), user.getPassword());
    }

    @When("the user enters email {string} and password {string}")
    public void entersCredentials(String email, String password) {
        loginPage.enterEmail(email)
                 .enterPassword(password);
    }

    @When("the user clicks the login button")
    public void clicksLoginButton() {
        homePage = loginPage.clickLoginButton();
    }

    @Then("{string} should be redirected to the home page")
    public void shouldBeRedirectedToHomePage(String username) {
        assertThat(homePage.isCurrentPage()).isTrue();
    }

    @Then("{string} should see a welcome message")
    public void shouldSeeWelcomeMessage(String username) {
        String message = homePage.getWelcomeMessage();
        assertThat(message).contains(username);
    }
}
```

```java
// ==================== CartSteps.java - Another Step Definition Injecting TestContext ====================
package stepdefinitions.cart;

import io.cucumber.java.en.*;
import pages.CartPage;
import pages.HomePage;
import context.TestContext;
import model.Product;
import utils.TestDataGenerator;

import static org.assertj.core.api.Assertions.*;

/**
 * CartSteps also injects TestContext,
 * PicoContainer ensures it shares the same instance with LoginSteps.
 */
public class CartSteps {

    private final TestContext context;
    private final HomePage homePage;
    private final CartPage cartPage;

    public CartSteps(TestContext context, HomePage homePage, CartPage cartPage) {
        this.context = context;
        this.homePage = homePage;
        this.cartPage = cartPage;
    }

    @Given("the product {string} is in stock with quantity {int}")
    public void productIsInStock(String productName, int quantity) {
        Product product = TestDataGenerator.createProduct(productName, quantity);
        context.setCurrentProduct(product);
    }

    @When("the user adds {string} to the cart")
    public void addsProductToCart(String productName) {
        homePage.searchFor(productName)
                .addFirstResultToCart();
    }

    @Then("the cart should contain {int} item(s)")
    public void cartShouldContain(int count) {
        assertThat(cartPage.getItemCount()).isEqualTo(count);
    }

    @Then("the cart total should be {string}")
    public void cartTotalShouldBe(String expectedTotal) {
        assertThat(cartPage.getTotal()).isEqualTo(expectedTotal);
    }
}
```

**Hooks class (inject TestContext):**

```java
// ==================== TestHooks.java - Global Hooks ====================
package stepdefinitions.hooks;

import io.cucumber.java.*;
import context.TestContext;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;

/**
 * Hooks can also inject dependencies through constructor.
 * PicoContainer ensures Hooks share the same TestContext with step definitions.
 */
public class TestHooks {

    private final TestContext context;

    public TestHooks(TestContext context) {
        this.context = context;
    }

    @Before(order = 0)
    public void beforeScenario(Scenario scenario) {
        context.setScenarioName(scenario.getName());
        System.out.println("Starting scenario: " + scenario.getName());
    }

    @After(order = 0)
    public void afterScenario(Scenario scenario) {
        // Screenshot on failure
        if (scenario.isFailed()) {
            try {
                byte[] screenshot = ((TakesScreenshot) context.getDriver())
                    .getScreenshotAs(OutputType.BYTES);
                scenario.attach(screenshot, "image/png", "Failure Screenshot");
            } catch (Exception e) {
                System.err.println("Failed to take screenshot: " + e.getMessage());
            }
        }

        // Cleanup resources
        context.closeDriver();
        System.out.println("Finished scenario: " + scenario.getName());
    }

    @BeforeStep
    public void beforeStep() {
        // Step-level logging or waiting
    }

    @AfterStep
    public void afterStep(Scenario scenario) {
        // Step-level screenshot (optional)
    }
}
```

**Test Runner (JUnit 5):**

```java
// ==================== CucumberTestSuite.java - JUnit 5 Suite Runner ====================
package runners;

import org.junit.platform.suite.api.*;

@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("features")
@ConfigurationParameter(key = "cucumber.glue", value = "stepdefinitions,context")
@ConfigurationParameter(key = "cucumber.plugin", value = "pretty, html:target/cucumber-reports.html, json:target/cucumber-reports.json")
@ConfigurationParameter(key = "cucumber.publish.enabled", value = "false")
public class CucumberTestSuite {
    // PicoContainer auto-scans constructors and injects dependencies
}
```

**PicoContainer complete dependency graph:**

```
When a scenario executes, PicoContainer automatically builds the following dependency graph:

                    CucumberTestSuite
                         │
                    [Scenario starts]
                         │
                    PicoContainer
                    (New instance)
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
    TestContext    LoginPage        HomePage
         │               │               │
         │         (Needs TestContext)  (Needs TestContext)
         │               │               │
         └───────────────┴───────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
    LoginSteps      CartSteps       TestHooks
    (injects context,  (injects context,  (injects context)
     loginPage)      homePage,
                     cartPage)

    Key: LoginSteps and CartSteps share the same TestContext instance
```

#### Spring Integration

For projects with existing Spring ecosystem, you can use `cucumber-spring`:

**Maven dependencies:**

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-spring</artifactId>
    <version>7.34.3</version>
    <scope>test</scope>
</dependency>
```

**Spring Configuration:**

```java
// ==================== CucumberSpringConfiguration.java ====================
package config;

import io.cucumber.spring.CucumberContextConfiguration;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.ComponentScan;

@CucumberContextConfiguration
@SpringBootTest
@ComponentScan(basePackages = {"pages", "context", "utils"})
public class CucumberSpringConfiguration {
}
```

```java
// ==================== TestContext.java - @ScenarioScope Ensures Scenario Isolation ====================
package context;

import org.springframework.context.annotation.ScopedProxyMode;
import org.springframework.stereotype.Component;
import io.cucumber.spring.ScenarioScope;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

@Component
@ScenarioScope  // Key annotation - new instance per scenario
public class TestContext {
    private WebDriver driver;
    private String currentUser;

    public WebDriver getDriver() {
        if (driver == null) {
            ChromeOptions options = new ChromeOptions();
            options.addArguments("--start-maximized");
            driver = new ChromeDriver(options);
        }
        return driver;
    }

    // ... getters/setters
}
```

```java
// ==================== LoginSteps.java - Spring Injection ====================
package stepdefinitions;

import io.cucumber.java.en.*;
import org.springframework.beans.factory.annotation.Autowired;
import pages.LoginPage;
import context.TestContext;

public class LoginSteps {

    @Autowired  // Spring auto-injection
    private TestContext context;

    @Autowired
    private LoginPage loginPage;

    // Step implementation...
}
```

**PicoContainer vs Spring comparison:**

| Characteristic | PicoContainer | Spring |
|------|--------------|--------|
| **Configuration complexity** | Zero configuration | Requires @CucumberContextConfiguration |
| **Annotation dependency** | None | @Autowired, @Component, @ScenarioScope |
| **Startup speed** | Fast | Slower (needs to start Spring context) |
| **Applicable Scenario** | Pure test projects | Projects with existing Spring ecosystem |
| **Scenario isolation** | Automatic (new container per scenario) | Requires @ScenarioScope annotation |
| **Integration with main application** | Not directly integrated | Natural integration |

### 7.3 JavaScript / TypeScript World Configuration

JavaScript Cucumber does not use traditional DI containers, but shares state through the **World object**. Each scenario gets an independent World instance.

#### Default World

```typescript
// ==================== Default World (Built-in) ====================
// Cucumber provides default World, accessed through this

import { Given, When, Then } from '@cucumber/cucumber';

Given('my color is {string}', function(color: string) {
    // this points to the current scenario's World instance
    this.color = color;  // Store in World
});

Then('my color should not be red', function() {
    // Read from World
    if (this.color === 'red') {
        throw new Error('Wrong Color');
    }
});
```

#### Custom World Class

```typescript
// ==================== custom-world.ts - Custom World ====================
// src/world/custom-world.ts

import { World, IWorldOptions, setWorldConstructor } from '@cucumber/cucumber';
import { Page, Browser, chromium, BrowserContext } from '@playwright/test';
import { LoginPage } from '../pages/login.page';
import { HomePage } from '../pages/home.page';
import { CartPage } from '../pages/cart.page';
import { AuthApiClient } from '../api/client';

/**
 * CustomWorld extends Cucumber's World class for sharing state between steps.
 * Each scenario gets an independent CustomWorld instance.
 */
export class CustomWorld extends World {
    // Playwright related
    private _browser!: Browser;
    private _context!: BrowserContext;
    private _page!: Page;

    // Shared state
    private context: Map<string, any> = new Map();
    public user: any;
    public authToken: string = '';

    // Page Objects (lazy-loaded)
    private _loginPage!: LoginPage;
    private _homePage!: HomePage;
    private _cartPage!: CartPage;

    // API Client
    private _apiClient!: AuthApiClient;

    constructor(options: IWorldOptions) {
        super(options);
    }

    // ========== Playwright Lifecycle ==========

    async initializeBrowser(): Promise<void> {
        this._browser = await chromium.launch({
            headless: process.env.CI === 'true',
            args: ['--disable-notifications']
        });
        this._context = await this._browser.newContext({
            viewport: { width: 1920, height: 1080 },
            recordVideo: process.env.RECORD_VIDEO === 'true' ? {
                dir: './test-results/videos/'
            } : undefined
        });
        this._page = await this._context.newPage();

        // Set default timeout
        this._page.setDefaultTimeout(10000);
    }

    async closeBrowser(): Promise<void> {
        if (this._browser) {
            await this._context?.close();
            await this._browser.close();
        }
    }

    // ========== Property Access ==========

    get page(): Page {
        return this._page;
    }

    // ========== Page Objects (Lazy-loaded) ==========

    get loginPage(): LoginPage {
        if (!this._loginPage) {
            this._loginPage = new LoginPage(this._page);
        }
        return this._loginPage;
    }

    get homePage(): HomePage {
        if (!this._homePage) {
            this._homePage = new HomePage(this._page);
        }
        return this._homePage;
    }

    get cartPage(): CartPage {
        if (!this._cartPage) {
            this._cartPage = new CartPage(this._page);
        }
        return this._cartPage;
    }

    // ========== API Client ==========

    get apiClient(): AuthApiClient {
        if (!this._apiClient) {
            this._apiClient = new AuthApiClient(process.env.BASE_URL || 'http://localhost:3000');
        }
        return this._apiClient;
    }

    // ========== Generic Context Storage ==========

    setContext<T>(key: string, value: T): void {
        this.context.set(key, value);
    }

    getContext<T>(key: string): T | undefined {
        return this.context.get(key);
    }

    // ========== Attachment Methods (Reports) ==========

    async attachScreenshot(name: string): Promise<void> {
        const screenshot = await this._page.screenshot();
        this.attach(screenshot, 'image/png', name);
    }

    async attachPageSource(): Promise<void> {
        const source = await this._page.content();
        this.attach(source, 'text/html', 'page-source.html');
    }
}

// Register custom World as global constructor
setWorldConstructor(CustomWorld);
```

#### World Hooks Configuration

```typescript
// ==================== world.hooks.ts - World Lifecycle Hooks ====================
// src/step-definitions/hooks/world.hooks.ts

import { Before, After, Status } from '@cucumber/cucumber';
import { CustomWorld } from '../../world/custom-world';

/**
 * Before/After Hooks use CustomWorld.
 * Note: this in Hooks points to the current scenario's CustomWorld instance.
 */

// Before scenario: initialize browser
Before(async function(this: CustomWorld) {
    await this.initializeBrowser();
});

// After scenario: cleanup resources
After(async function(this: CustomWorld, scenario) {
    // Screenshot on failure
    if (scenario.result?.status === Status.FAILED) {
        await this.attachScreenshot(`failure-${scenario.pickle.name}`);
        await this.attachPageSource();
    }

    // Record video on failure (Playwright auto-saves)
    if (scenario.result?.status === Status.FAILED && process.env.RECORD_VIDEO === 'true') {
        const videoPath = await this.page.video()?.path();
        if (videoPath) {
            this.attach(videoPath, 'text/plain', 'video-path.txt');
        }
    }

    await this.closeBrowser();
});

// Conditional Hook with tags
Before({ tags: '@mobile' }, async function(this: CustomWorld) {
    // Mobile-specific settings
    await this.page.setViewportSize({ width: 375, height: 812 });
});

// Hook with order
Before({ order: 1 }, async function(this: CustomWorld) {
    console.log('First hook executed');
});
```

#### Using Custom World in Step Definitions

```typescript
// ==================== login.steps.ts - Using CustomWorld ====================
// src/step-definitions/auth/login.steps.ts

import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';
import { CustomWorld } from '../../world/custom-world';

Given('the user is on the login page', async function(this: CustomWorld) {
    await this.loginPage.open();  // Access Page Object through World
});

When('the user logs in with email {string} and password {string}',
    async function(this: CustomWorld, email: string, password: string) {
        this.user = { email, password };  // Store in World
        await this.loginPage.login(email, password);
    }
);

Then('the user should see the welcome message {string}',
    async function(this: CustomWorld, expectedMessage: string) {
        const message = await this.homePage.getWelcomeMessage();  // Access through World
        expect(message).toContain(expectedMessage);
    }
);

// API + UI hybrid testing
Given('a registered customer {string} via API', async function(this: CustomWorld, name: string) {
    const user = await this.apiClient.registerUser({ name, email: `${name}@test.com` });
    this.setContext('apiUser', user);  // Use generic context storage
    this.user = user;
});
```

#### Cucumber Configuration File

```typescript
// ==================== cucumber.config.ts - Cucumber TypeScript Configuration ====================
import { Configuration } from '@cucumber/cucumber';

const config: Configuration = {
    // Feature file paths
    paths: ['features/**/*.feature'],

    // Step definitions and support files
    require: [
        'src/step-definitions/**/*.ts',
        'src/world/*.ts',
        'src/support/*.ts'
    ],

    // Compile TypeScript
    requireModule: ['ts-node/register'],

    // Output format
    format: [
        'progress-bar',
        '@cucumber/pretty-formatter',
        'html:reports/cucumber-report.html',
        'json:reports/cucumber-report.json',
        'junit:reports/cucumber-report.xml'
    ],

    formatOptions: {
        colorsEnabled: true,
        snippetInterface: 'async-await'
    },

    // Parallel execution
    parallel: process.env.CI === 'true' ? 4 : 1,

    // Retry on failure (use with caution)
    retry: process.env.RETRY ? parseInt(process.env.RETRY, 10) : 0,

    // Publish settings
    publishQuiet: true,

    // Custom World constructor (already registered in custom-world.ts)
    // worldParameters can be used to pass parameters
    worldParameters: {
        baseUrl: process.env.BASE_URL || 'http://localhost:3000',
        headless: process.env.CI === 'true'
    }
};

export default config;
```

### 7.4 Ruby World Configuration

Ruby Cucumber uses **World module mixin** mechanism to share state. Methods from modules are injected into each scenario's World instance via `World(ModuleName)`.

#### Basic World Configuration

```ruby
# ==================== env.rb - Core Environment Configuration File ====================
# features/support/env.rb

require 'capybara'
require 'capybara/cucumber'
require 'selenium-webdriver'
require 'rspec/expectations'
require 'site_prism'  # Page Object library (optional)

# ========== Capybara Configuration ==========
Capybara.configure do |config|
    config.default_driver = :selenium_chrome
    config.javascript_driver = :selenium_chrome_headless
    config.default_max_wait_time = 10
    config.app_host = ENV.fetch('APP_HOST', 'http://localhost:3000')
    config.save_path = 'reports/screenshots/'
end

# Register browser driver
Capybara.register_driver :selenium_chrome do |app|
    options = Selenium::WebDriver::Chrome::Options.new
    options.add_argument('--start-maximized')
    options.add_argument('--disable-notifications')
    options.add_argument('--headless') if ENV['CI'] == 'true'
    Capybara::Selenium::Driver.new(app, browser: :chrome, options: options)
end

# Register remote driver (Selenium Grid)
Capybara.register_driver :selenium_remote do |app|
    Capybara::Selenium::Driver.new(
        app,
        browser: :remote,
        url: ENV.fetch('GRID_URL', 'http://localhost:4444/wd/hub'),
        options: Selenium::WebDriver::Chrome::Options.new
    )
end

# ========== Auto-load lib directory ==========
$LOAD_PATH.unshift(File.expand_path('../../lib', __dir__))
Dir.glob(File.join(__dir__, '../../lib/**/*.rb')).sort.each { |f| require f }

# ========== Custom World Module ==========
# Inject utility methods into each scenario's World

module WorldExtensions
    def current_user
        @current_user
    end

    def current_user=(user)
        @current_user = user
    end

    def current_product
        @current_product
    end

    def current_product=(product)
        @current_product = product
    end

    def api_client
        @api_client ||= Api::BaseClient.new(ENV.fetch('API_URL', 'http://localhost:3000/api'))
    end

    def screenshot(name = 'screenshot')
        page.save_screenshot("reports/screenshots/#{name}_#{Time.now.to_i}.png")
    end

    def log(message)
        puts "[#{Time.now}] #{message}"
    end
end

# Inject module into World - every scenario can use these methods
World(WorldExtensions)
```

#### Custom World Class

```ruby
# ==================== world.rb - Advanced World Configuration ====================
# features/support/world.rb

require_relative 'pages/base_page'

# Custom World class - for managing Page Objects and state
class CustomWorld
    # Each scenario gets an instance of this class

    attr_accessor :current_user, :current_order

    def initialize
        @page_cache = {}
    end

    # Lazy-load Page Objects
    def login_page
        @page_cache[:login] ||= Pages::LoginPage.new
    end

    def home_page
        @page_cache[:home] ||= Pages::HomePage.new
    end

    def cart_page
        @page_cache[:cart] ||= Pages::CartPage.new
    end

    def search_page
        @page_cache[:search] ||= Pages::SearchPage.new
    end

    # API clients
    def auth_api
        @auth_api ||= Api::AuthClient.new
    end

    def order_api
        @order_api ||= Api::OrderClient.new
    end

    # Generic context storage
    def remember(key, value)
        @memory ||= {}
        @memory[key] = value
    end

    def recall(key)
        @memory ||= {}
        @memory[key]
    end

    # Cleanup
    def cleanup
        @page_cache.clear
        @memory = {}
    end
end

# Inject CustomWorld into Cucumber World
World { CustomWorld.new }
```

#### World Module Mixin Pattern

```ruby
# ==================== page_object_world.rb - Page Object Dedicated Module ====================
# features/support/page_object_world.rb

# Domain-specific World modules
module AuthWorld
    def login_as(user_type)
        user = create_user(user_type)
        @current_user = user
        login_page.login(user[:email], user[:password])
    end

    def create_user(user_type)
        case user_type
        when 'standard'
            { email: 'user@test.com', password: 'password123', role: 'user' }
        when 'admin'
            { email: 'admin@test.com', password: 'admin123', role: 'admin' }
        when 'guest'
            { email: nil, password: nil, role: 'guest' }
        else
            raise "Unknown user type: #{user_type}"
        end
    end
end

module CartWorld
    def add_product_to_cart(product_name)
        search_page.search_for(product_name)
        search_page.add_first_result_to_cart
    end

    def cart_should_contain(expected_count)
        visit cart_page.path
        expect(cart_page.item_count).to eq(expected_count)
    end

    def cart_total
        cart_page.total_text
    end
end

module ApiWorld
    def create_order_via_api(order_data)
        response = api_client.post('/orders', order_data)
        @current_order = JSON.parse(response.body)
    end

    def get_order_status(order_id)
        response = api_client.get("/orders/#{order_id}")
        JSON.parse(response.body)['status']
    end
end

# Mix all modules into World
World(AuthWorld, CartWorld, ApiWorld)
```

#### Hooks Configuration

```ruby
# ==================== hooks.rb - Global Hooks ====================
# features/support/hooks.rb

# Before scenario
Before do |scenario|
    log("Starting scenario: #{scenario.name}")
    @scenario_name = scenario.name

    # Clear cookies (unless tagged @preserve_session)
    page.driver.browser.manage.delete_all_cookies unless scenario.source_tag_names.include?('@preserve_session')
end

# After scenario
After do |scenario|
    if scenario.failed?
        # Screenshot on failure
        screenshot_name = "failure_#{scenario.name.gsub(/\s+/, '_').downcase}_#{Time.now.to_i}"
        screenshot(screenshot_name)
        embed("reports/screenshots/#{screenshot_name}.png", 'image/png', 'Failure Screenshot')

        # Attach page source
        page_source = page.source
        embed(page_source, 'text/html', 'Page Source')
    end

    # Cleanup custom World
    cleanup if respond_to?(:cleanup)

    log("Finished scenario: #{scenario.name} - #{scenario.failed? ? 'FAILED' : 'PASSED'}")
end

# Conditional Hook with tags
Before('@mobile') do
    page.driver.browser.manage.window.resize_to(375, 812)
end

Before('@desktop') do
    page.driver.browser.manage.window.resize_to(1920, 1080)
end

# Skip tagged scenarios
Around('@wip') do |scenario, block|
    if ENV['RUN_WIP'] == 'true'
        block.call
    else
        skip_this_scenario
    end
end

# Retry Hook (use with caution)
Around('@flaky') do |scenario, block|
    attempts = 0
    max_attempts = 3
    begin
        block.call
    rescue => e
        attempts += 1
        if attempts < max_attempts
            log("Retrying #{scenario.name} (attempt #{attempts + 1}/#{max_attempts})")
            retry
        else
            raise e
        end
    end
end

# Step-level Hook
BeforeStep do |step|
    # Execute before step starts
end

AfterStep do |step|
    # Execute after step ends (can be used for step-level screenshots)
    if ENV['STEP_SCREENSHOTS'] == 'true'
        screenshot("step_#{step.text.gsub(/\s+/, '_').downcase[0..50]}")
    end
end
```

#### Using World in Step Definitions

```ruby
# ==================== auth_steps.rb - Step Definitions Using World ====================
# step_definitions/auth_steps.rb

Given('{string} has a registered account') do |username|
    # Use methods mixed into World
    self.current_user = create_user('standard')
    self.current_user[:name] = username
end

When('{string} logs in with her valid credentials') do |username|
    user = current_user
    login_page.login(user[:email], user[:password])
end

Then('{string} should see the welcome message') do |username|
    expect(home_page.welcome_message_text).to include("Welcome, #{username}")
end

# Use API World module
Given('an order exists for {string} via API') do |customer_name|
    create_order_via_api(
        customer: customer_name,
        items: [{ product: 'Widget', quantity: 2 }]
    )
end

Then('the order status should be {string}') do |expected_status|
    status = get_order_status(@current_order['id'])
    expect(status).to eq(expected_status)
end
```

### 7.5 Three Platform Dependency Injection Comparison

| Characteristic | JVM (PicoContainer) | JavaScript (World) | Ruby (World) |
|------|---------------------|-------------------|--------------|
| **Injection Method** | Constructor injection | `this` object properties | Module mixin |
| **Configuration** | Zero config (auto-discovery) | `setWorldConstructor()` | `World()` method |
| **Scenario isolation** | New container per scenario | New World instance per scenario | New World instance per scenario |
| **Type safety** | Compile-time checking | Runtime (TypeScript partial support) | Runtime |
| **Lifecycle management** | Automatic (constructor/destructor) | `Before`/`After` Hooks | `Before`/`After` Hooks |
| **Object creation** | Constructor recursive resolution | Manual or lazy-loaded | Manual or lazy-loaded |
| **Applicable Scenario** | Strongly typed projects | Flexible dynamic projects | Flexible dynamic projects |
| **Learning curve** | Low | Low | Low |
| **IDE support** | Excellent auto-completion | Good (TypeScript) | Average |

### 7.6 Dependency Injection Best Practices

| # | Best Practice | Description |
|---|---------|------|
| 1 | **Scenario-level isolation** | Each scenario gets an independent context instance, mutable state sharing between scenarios is prohibited |
| 2 | **Constructor injection** | Use constructor injection in JVM instead of field injection |
| 3 | **Lazy-load expensive resources** | WebDriver, database connections, etc. are created lazily |
| 4 | **Single responsibility** | Each shared class only manages one type of state |
| 5 | **Cleanup in Hooks** | Close browser, clear data after scenario ends |
| 6 | **Avoid global variables** | Use DI container management, do not use static variables |
| 7 | **Program to interfaces** | Step definitions depend on interfaces/abstract classes rather than concrete implementations |
| 8 | **Externalize configuration** | Place URLs, timeouts, etc. in configuration files/environment variables |
| 9 | **Collect evidence on failure** | Take screenshots, save page source in After Hook |
| 10 | **Parallel safety** | Ensure shared resources (such as database connection pools) are thread-safe |

---

## Appendix A: Complete Project Quick Start Templates

### A.1 JVM (Maven + PicoContainer + Selenium) Minimal Template

```xml
<!-- pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>bdd-automation</artifactId>
    <version>1.0.0</version>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <cucumber.version>7.34.3</cucumber.version>
    </properties>
    <dependencies>
        <dependency><groupId>io.cucumber</groupId><artifactId>cucumber-java</artifactId><version>${cucumber.version}</version><scope>test</scope></dependency>
        <dependency><groupId>io.cucumber</groupId><artifactId>cucumber-junit-platform-engine</artifactId><version>${cucumber.version}</version><scope>test</scope></dependency>
        <dependency><groupId>io.cucumber</groupId><artifactId>cucumber-picocontainer</artifactId><version>${cucumber.version}</version><scope>test</scope></dependency>
        <dependency><groupId>org.junit.platform</groupId><artifactId>junit-platform-suite</artifactId><version>1.10.0</version><scope>test</scope></dependency>
        <dependency><groupId>org.assertj</groupId><artifactId>assertj-core</artifactId><version>3.24.2</version><scope>test</scope></dependency>
        <dependency><groupId>org.seleniumhq.selenium</groupId><artifactId>selenium-java</artifactId><version>4.15.2</version><scope>test</scope></dependency>
    </dependencies>
</project>
```

### A.2 JavaScript (TypeScript + Playwright) Minimal Template

```json
// package.json
{
  "name": "bdd-automation",
  "version": "1.0.0",
  "scripts": {
    "test": "cucumber-js --config cucumber.config.ts"
  },
  "devDependencies": {
    "@cucumber/cucumber": "^11.0.0",
    "@playwright/test": "^1.40.0",
    "typescript": "^5.3.0",
    "ts-node": "^10.9.0"
  }
}
```

### A.3 Ruby (Capybara + Selenium) Minimal Template

```ruby
# Gemfile
source 'https://rubygems.org'

gem 'cucumber', '~> 9.0'
gem 'capybara', '~> 3.40'
gem 'selenium-webdriver', '~> 4.15'
gem 'rspec-expectations', '~> 3.12'
```

---

## Appendix B: Architecture Decision Checklist

| Decision Item | Option | Recommended Scenario |
|--------|------|----------|
| **Page Object vs Screenplay** | Page Object | Small to medium projects, quick start |
| | Screenplay | Large complex projects, multi-role scenarios |
| **PicoContainer vs Spring** | PicoContainer | Pure test projects |
| | Spring | Projects with existing Spring ecosystem |
| **UI locating strategy** | `data-testid` | Frontend team collaboration available |
| | CSS selectors | Cannot modify frontend code |
| | XPath | Last resort |
| **Browser management** | New browser per scenario | Default recommendation, complete isolation |
| | Shared browser instance | Performance priority, cleanup required |
| **Parallel execution** | Maven Surefire fork | JVM projects |
| | cucumber-js --parallel | JS projects |
| | Process-level parallel | Ruby projects |

---

> **Document Version**: v1.0
> **Scope**: JVM / JavaScript / Ruby technology stacks
> **References**: Cucumber official documentation, Serenity BDD documentation, community best practices

