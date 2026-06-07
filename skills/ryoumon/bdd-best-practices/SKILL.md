---
name: bdd-best-practices
description: >
  BDD (Behavior-Driven Development) coding standards and best practices. Covers Gherkin syntax,
  Cucumber framework, scenario writing, Step Definition design, layered architecture, data-driven
  testing, CI/CD integration, and team collaboration.
  Applies to three tech stacks: Java/JVM, JavaScript/TypeScript, and Ruby. Based on Cucumber
  official documentation and community practices.
  When encountering Feature file writing, step definition design, or test architecture decisions,
  load the corresponding reference files for this skill.
---

# BDD Best Practices - SKILL Quick Reference

> Version: 1.0 | Scope: BDD / Gherkin / Cucumber | Language: English

---

## Quick Decision Table

| Scenario | Load Reference |
|----------|----------------|
| Design Feature file structure | `references/guidelines.md` (GL01-GL09) + `references/architecture.md` (directory structure) |
| Write Gherkin scenarios | `references/guidelines.md` (SC01-SC09) |
| Write Step Definitions | `references/guidelines.md` (SD01-SD09) |
| Data-driven testing (tables/parameterization) | `references/guidelines.md` (DD01-DD06) |
| Project architecture design | `references/architecture.md` (layered architecture + dependency rules) |
| Directory structure selection | `references/architecture.md` (JVM/JS/Ruby three variants) |
| Naming conventions | `references/architecture.md` (naming conventions quick ref) |
| Troubleshoot common pitfalls | `references/anti-patterns.md` (AP-01 ~ AP-12) |
| Team collaboration workflow | `references/guidelines.md` (TM01-TM06) |
| CI/CD integration configuration | `references/guidelines.md` (TO04-TO07) |
| UI automation design | `references/architecture.md` (POM/Screenplay) |

---

## Architecture Quick Reference

### Directory Structure (Three Tech Stacks)

**JVM (Java/Kotlin):**
```
src/test/
  java/
    runners/              # Test Runner
    stepdefinitions/      # Grouped by domain concept (auth/, cart/)
    pages/                # Page Object layer
    context/              # TestContext (PicoContainer DI)
    utils/                # DriverFactory, ConfigReader
  resources/
    features/             # Grouped by module (auth/, cart/)
```

**JavaScript/TypeScript:**
```
features/                 # Feature files (grouped by domain)
src/
  step-definitions/       # Grouped by domain
  pages/                  # Page Object layer
  world/                  # Custom World
  api/                    # API Client
  utils/                  # Utility functions
cucumber.config.ts        # TypeScript config (v12.4.0+)
```

**Ruby:**
```
features/                 # Feature files
  support/
    env.rb                # Core environment configuration
    hooks.rb              # Before/After/Around
    pages/                # Page Object
step_definitions/         # Step definitions (fixed name)
lib/                      # API Clients, Models
```

### Naming Conventions Quick Reference

| Category | JVM | JS/TS | Ruby |
|----------|-----|-------|------|
| Feature file | `user-login.feature` | `user-login.feature` | `user_login.feature` |
| Step Def file | `LoginSteps.java` | `login.steps.ts` | `auth_steps.rb` |
| Page Object class | `LoginPage.java` | `login.page.ts` | `login_page.rb` |
| Method name | `camelCase` | `camelCase` | `snake_case` |
| Directory name | `stepdefinitions/` | `step-definitions/` | `step_definitions/` |
| Custom parameter type | `DateTransformer.java` | `date.transformer.ts` | `date_transformer.rb` |

### Layered Architecture (Four-Layer Unidirectional Dependencies)

```
Layer 1: Gherkin (.feature)  -- Business specification, zero technical details
       |  Step text matching
       v
Layer 2: Step Definitions    -- Thin glue layer between Gherkin and code
       |  Calls Layer 3
       v
Layer 3: Page Objects / API Clients / Tasks  -- Encapsulates specific interactions
       |  WebDriver / HTTP Client
       v
Layer 4: System Under Test   -- System being tested
```

**Core Dependency Rule**: Only downward calls allowed; reverse dependencies are strictly prohibited.

---

## Coding Principles Essentials (10 Core Rules)

The 10 most important rules selected from 47 complete principles:

| # | Principle | One-Sentence Summary |
|---|-----------|----------------------|
| 1 | **Single Feature per File** | Each `.feature` contains only one Feature; filename uses kebab-case aligned with domain concept |
| 2 | **One Scenario = One Behavior** | Do not chain multiple When-Then sequences in the same scenario |
| 3 | **3-5 Step Rule** | 3-5 steps per scenario (excluding Background); must split if exceeding 8 steps |
| 4 | **Declarative > Imperative** | Describe "what" not "how"; self-test: does the text need to change when implementation changes? |
| 5 | **Scenarios Completely Independent** | Each scenario is self-contained with its own Given; does not depend on execution results or order of other scenarios |
| 6 | **Business Language, Zero Technical Terms** | CSS selectors, API endpoints, and database table names are prohibited in Gherkin |
| 7 | **Background <= 4 Lines** | Only place core preconditions truly shared by all scenarios |
| 8 | **Step Definition <= 10 Lines** | Thin glue code; delegate complex logic to Page Object / Service |
| 9 | **Cucumber Expressions > Regex** | Use `{int}`/`{string}`/`{word}` instead of regex; readability first |
| 10 | **Organize Step Defs by Domain** | `auth.steps.js` instead of `login.steps.js`; promote cross-Feature reuse |

---

## Gotchas (Most Common Pitfalls)

> Full version: see `references/anti-patterns.md` (AP-01 ~ AP-12)

| # | Pitfall | One-Sentence Summary |
|---|---------|----------------------|
| 1 | **Imperative Scenarios** | Scenarios filled with `click`/`type`/`enter`, reading like an instruction manual rather than a behavior specification |
| 2 | **Feature-coupled Steps** | Step definition files correspond one-to-one with Features, leading to massive step duplication |
| 3 | **UI Detail Leakage** | UI elements like `#email-input`, `btn-submit` appear in Gherkin |
| 4 | **God Step Def** | A single method with 50+ lines, handling data prep, UI operations, and database verification |
| 5 | **Inter-scenario Dependencies** | Scenario B assumes Scenario A has already created data; cannot run in isolation |
| 6 | **Hard-coded Waits** | Using `Thread.sleep(3000)` instead of explicit waits; unreliable timing |
| 7 | **Conjunction Steps** | `Given I have shades and a brand new Mustang` cannot be split or reused |
| 8 | **Then Queries Database** | Then step directly writes SQL queries, verifying internal implementation rather than user-visible behavior |
| 9 | **GWT Order Confusion** | When appearing after Then, or scenario missing Then verification |
| 10 | **Missing Ubiquitous Language** | The same business concept uses three different names across scenarios; the team cannot reach a shared understanding |

---

## Key Code Templates

### Template 1: Complete Feature + Step Definition (JVM Style)

```gherkin
# FILE: features/auth/customer-login.feature
@regression @authentication
Feature: Customer Login
  As a registered customer
  I want to log in with my credentials
  So that I can view and manage my orders

  Rule: Valid credentials allow access to the dashboard

    @positive @smoke
    Scenario: Successful login with valid credentials
      Given "Jane" has a registered account
      When "Jane" logs in with her valid credentials
      Then she should be redirected to the order dashboard
      And she should see a welcome message with her name

    @negative
    Scenario Outline: Unsuccessful login with invalid credentials
      Given "<username>" has a registered account
      When "<username>" attempts to log in with "<password>"
      Then an error message "<error>" should be displayed

      Examples:
        | username | password  | error               |
        | Jane     | wrongpass | Invalid credentials |
        | Unknown  | anypass   | User not found      |
```

```java
// FILE: src/test/java/stepdefinitions/auth/LoginSteps.java
package stepdefinitions.auth;

import io.cucumber.java.en.*;
import static org.assertj.core.api.Assertions.*;
import pages.LoginPage;
import pages.DashboardPage;
import context.TestContext;

public class LoginSteps {
    private final TestContext ctx;
    private final LoginPage loginPage;
    private final DashboardPage dashboardPage;

    // PicoContainer auto-injects dependencies
    public LoginSteps(TestContext ctx) {
        this.ctx = ctx;
        this.loginPage = new LoginPage(ctx.getDriver());
        this.dashboardPage = new DashboardPage(ctx.getDriver());
    }

    @Given("{string} has a registered account")
    public void hasRegisteredAccount(String username) {
        User user = TestDataGenerator.createUser(username);
        ctx.setCurrentUser(user);
    }

    @When("{string} logs in with her valid credentials")
    public void logsIn(String username) {
        User user = ctx.getCurrentUser();
        loginPage.login(user.getEmail(), user.getPassword());
    }

    @When("{string} attempts to log in with {string}")
    public void attemptLogin(String username, String password) {
        loginPage.login(username, password);
    }

    @Then("she should be redirected to the order dashboard")
    public void shouldSeeDashboard() {
        assertThat(dashboardPage.isDisplayed()).isTrue();
    }

    @Then("an error message {string} should be displayed")
    public void shouldSeeError(String expectedError) {
        assertThat(loginPage.getErrorMessage()).isEqualTo(expectedError);
    }
}
```

### Template 2: Cucumber Expressions + Custom Parameter Types

```gherkin
# Use natural, fluent expressions in Gherkin
Feature: Product Purchase

  Scenario: Apply discount on a product
    Given a product priced at 199.99 USD
    When the customer applies a 10% discount on 2024-12-01
    Then the final price should be 179.99 USD
```

```java
// Custom parameter type definitions
@ParameterType("\\d+\\.\\d{2} (USD|EUR|CNY)")
public Money money(String amount, String currency) {
    return new Money(new BigDecimal(amount), Currency.valueOf(currency));
}

@ParameterType("\\d{4}-\\d{2}-\\d{2}")
public LocalDate isoDate(String date) {
    return LocalDate.parse(date);
}

// Step Definition uses domain types directly
@Given("a product priced at {money}")
public void setProductPrice(Money price) {
    this.product = new Product(price);
}

@When("the customer applies a {int}% discount on {iso-date}")
public void applyDiscount(int percentage, LocalDate date) {
    this.product.applyDiscount(percentage, date);
}

@Then("the final price should be {money}")
public void verifyFinalPrice(Money expected) {
    assertEquals(expected, product.getFinalPrice());
}
```

### Template 3: Scenario Outline + Data Tables (Data-Driven)

```gherkin
# Scenario Outline: same logic x multiple data sets, each row executes one full scenario
Feature: Payment Processing

  Scenario Outline: Payment with different card types
    Given a cart total of $<amount>
    When the customer pays with "<card_type>"
    Then the payment should be "<result>"

    Examples:
      | amount | card_type  | result   |
      | 50.00  | Visa       | approved |
      | 50.00  | Mastercard | approved |
      | 0.01   | Visa       | declined |
      | 10000  | Visa       | review   |

  # Data Table: single execution, structured data passed to a single step
  Scenario: Bulk registration of new users
    Given the following users have registered:
      | first_name | email              | plan  |
      | Alice      | alice@example.com  | free  |
      | Bob        | bob@example.com    | pro   |
      | Charlie    | charlie@example.com| basic |
    When the daily welcome email job runs
    Then all users should receive a personalized welcome email
```

```java
// Scenario Outline step definitions -- parameters auto-injected
@Given("a cart total of ${bigdecimal}")
public void setCartTotal(BigDecimal total) {
    ctx.setCartTotal(total);
}

@When("the customer pays with {string}")
public void processPayment(String cardType) {
    PaymentResult result = paymentService.pay(ctx.getCartTotal(), cardType);
    ctx.setPaymentResult(result);
}

@Then("the payment should be {string}")
public void verifyPaymentResult(String expected) {
    assertThat(ctx.getPaymentResult().getStatus()).isEqualTo(expected);
}

// Data Table step definition -- receives List<Map<>>
@Given("the following users have registered:")
public void registerUsers(DataTable dataTable) {
    List<Map<String, String>> users = dataTable.asMaps();
    for (Map<String, String> row : users) {
        userService.register(
            row.get("first_name"),
            row.get("email"),
            row.get("plan")
        );
    }
}
```

**Data Table vs Scenario Outline Selection Guide:**

| Characteristic | Data Table | Scenario Outline |
|----------------|-----------|-----------------|
| Data level | Step level | Scenario level |
| Execution count | Single execution | Once per row |
| Applicable scenarios | Batch operations, structured input | Data combination testing, boundary conditions |

---

## Reference File Index

| File | One-Sentence Description |
|------|-------------------------|
| `references/guidelines.md` | 47 coding principles full version, covering six dimensions: Gherkin syntax, scenario writing, Step Definitions, data-driven testing, team collaboration, and test organization |
| `references/architecture.md` | BDD project architecture design specifications, including JVM/JS/Ruby directory structures, four-layer architecture, dependency rules, naming conventions, and POM/Screenplay patterns |
| `references/anti-patterns.md` | 12 common anti-patterns catalog, each including symptom identification, impact analysis, remediation plan, and code examples |

---

> **Usage Note**: This file is a quick reference guide for rapid decision-making and core rule review. For detailed principles, complete code examples, and in-depth anti-pattern analysis, please load the corresponding reference files.
