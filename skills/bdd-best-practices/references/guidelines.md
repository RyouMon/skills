# BDD Coding Guidelines

> Version: 1.0
> Scope: Gherkin / Cucumber / BDD Projects
> Based on: Cucumber Official Documentation, Community Best Practices, Architecture Patterns & CI/CD Research

---

## Table of Contents

- [1. Gherkin Syntax Standards (GL01-GL09)](#1-gherkin-syntax-standards)
- [2. Scenario Authoring (SC01-SC09)](#2-scenario-authoring)
- [3. Step Definitions (SD01-SD09)](#3-step-definitions)
- [4. Data-Driven Testing (DD01-DD06)](#4-data-driven-testing)
- [5. Team Collaboration & Process (TM01-TM06)](#5-team-collaboration--process)
- [6. Test Organization (TO01-TO08)](#6-test-organization)

---

## 1. Gherkin Syntax Standards

---

### GL01: Each `.feature` File Contains Only One Feature

- **Principle Description**: Strictly adhere to one `Feature` keyword per `.feature` file. Do not concatenate multiple independent feature descriptions in the same file.
- **Rationale**: A single Feature corresponds to an independent functional domain, facilitating file navigation, tag management, and report generation. Multiple Features in one file cause report confusion and tag inheritance failures.
- **Code Example**:

```gherkin
# FILE: features/customer-login.feature
@authentication @regression
Feature: Customer Login
  As a registered customer
  I want to log in with my credentials
  So that I can view and manage my orders

  Scenario: Successful login with valid credentials
    Given "John" has a registered account
    When "John" logs in with valid credentials
    Then he should be redirected to the order dashboard
```

```java
// Feature files are named using kebab-case, aligned with domain concepts
// customer-login.feature  <-- correct
// login-and-registration.feature  <-- incorrect (contains two features)
```

- **Anti-pattern Reference**: `anti-patterns.md#feature-coupled-scenario-organization`

---

### GL02: Use Two-Space Indentation for Consistent Formatting

- **Principle Description**: Gherkin files must use two spaces as the indentation unit. Tabs or four-space indentation are prohibited. All `.feature` files in the same project must follow a consistent indentation style.
- **Rationale**: Consistent indentation improves readability and facilitates code review and diff comparison. The Cucumber official documentation recommends two spaces.
- **Code Example**:

```gherkin
Feature: Order Management
  # Two-space indentation
  Background:
    Given the order service is available
    And the inventory is stocked

  Scenario: Place a new order
    Given a customer has selected a product
    When the customer places an order
    Then the order should be confirmed
```

```java
// Configure uniformly in .editorconfig
// indent_size = 2
// indent_style = space
```

- **Anti-pattern Reference**: `anti-patterns.md#inconsistent-formatting-style`

---

### GL03: Feature Description Must Include User Story Three-Part Format

- **Principle Description**: Each `Feature` description must include the `As a... I want... So that...` three-part format, clearly defining the role, behavior, and business value.
- **Rationale**: The User Story three-part format forces the team to think about features from a business value perspective, ensuring each Feature has a clear business purpose and audience. It is the foundation of BDD collaboration.
- **Code Example**:

```gherkin
Feature: Shopping Cart Discount
  As an online shopper
  I want to apply discount codes to my cart
  So that I can save money on my purchases

  Scenario: Apply valid discount code
    Given a cart with total amount $100
    When the customer applies discount code "SAVE20"
    Then the cart total should be $80
```

```gherkin
# Counter-example - missing business value explanation
Feature: Shopping Cart Discount
  # No As a / I want / So that
  Scenario: Apply valid discount code
    ...
```

- **Anti-pattern Reference**: `anti-patterns.md#missing-business-value-in-feature-description`

---

### GL04: Use the Rule Keyword to Organize Business Rules

- **Principle Description**: When a Feature contains multiple independent business rules, use the `Rule` keyword (introduced in Gherkin 6+) to group scenarios by business rule. Place relevant Background and Scenario elements under each Rule.
- **Rationale**: The `Rule` keyword makes business rules explicit in Gherkin, improving readability and living documentation quality. It helps business stakeholders quickly locate scenarios corresponding to specific rules.
- **Code Example**:

```gherkin
Feature: Account Security
  As a system administrator
  I want accounts to be protected against unauthorized access
  So that customer data remains secure

  Rule: Valid credentials allow access
    Background:
      Given the authentication service is running

    Scenario: Login with valid username and password
      Given a user with valid credentials
      When the user logs in
      Then access should be granted

  Rule: Account locks after repeated failures
    Scenario: Account locks after three failed attempts
      Given a user with valid credentials
      When the user attempts to log in with wrong password 3 times
      Then the account should be locked
```

```java
// After Rule grouping, reporting tools (e.g., Serenity BDD) can generate sub-reports by business rule
// Each Rule corresponds to an independent business rule validation area
```

- **Anti-pattern Reference**: `anti-patterns.md#implicit-rules`

---

### GL05: Use And/But to Connect Steps, Avoid Repeating Keywords

- **Principle Description**: In the same scenario, when multiple Given or multiple Then steps are needed, the 2nd and subsequent steps must use `And` or `But` instead of repeating the `Given`/`When`/`Then` keyword.
- **Rationale**: `And` and `But` make scenarios read more naturally and fluently, as they are part of the Gherkin language design. Repeating the same keyword makes scenarios appear verbose.
- **Code Example**:

```gherkin
# Recommended - using And/But
Given the user has a registered account
And the user has verified their email
When the user logs in with valid credentials
Then the user should see the dashboard
And a welcome message should be displayed
But the user should not see admin features
```

```gherkin
# Not recommended - repeating keywords
Given the user has a registered account
Given the user has verified their email
When the user logs in with valid credentials
Then the user should see the dashboard
Then a welcome message should be displayed
```

- **Anti-pattern Reference**: `anti-patterns.md#repeated-keyword-steps`

---

### GL06: Use Tags for Scenario Classification and Organization

- **Principle Description**: Use the `@` symbol to add tags to Features and Scenarios, establishing a unified tag naming convention. Tag names use lowercase, hyphen-separated format. Tags are prohibited on Background or individual steps.
- **Rationale**: Tags are the core mechanism for scenario organization and selective execution. They support Boolean expression filtering (e.g., `@smoke and not @slow`), and are the foundation for CI/CD pipelines and report grouping.
- **Code Example**:

```gherkin
@regression @authentication @priority-high
Feature: Customer Login

  @positive @smoke
  Scenario: Successful login with valid credentials
    Given ...
    When ...
    Then ...

  @negative @security
  Scenario Outline: Unsuccessful login with invalid credentials
    Given ...
    When ...
    Then ...

    Examples:
      | username | password  |
      | user1    | wrongpass |
      | user2    | expired   |
```

```java
// Filter execution by tags
// mvn test -Dcucumber.filter.tags="@smoke and not @wip"
// cucumber-js --tags "@regression and not @slow"
```

- **Anti-pattern Reference**: `anti-patterns.md#tag-abuse`

---

### GL07: Comments Should Be Concise and Meaningful

- **Principle Description**: Use `#` for comments. Comments should explain "why" not "what". Block comments are prohibited (Gherkin does not support them). Feature-level comments can be used to associate requirement IDs (e.g., Jira ID).
- **Rationale**: Comments supplement the living documentation, helping readers understand the business logic behind scenarios. Requirement ID comments establish a traceable link between code and requirement management tools.
- **Code Example**:

```gherkin
# JIRA-1234: Account security requirement - Updated 2025-06
# This Feature covers the core business rules for login authentication
Feature: Customer Login

  # Note: this scenario depends on the authentication service being available; not in test scope
  @smoke
  Scenario: Successful login with valid credentials
    Given "John" has a registered account
    When "John" logs in with valid credentials
    Then he should be redirected to the order dashboard
```

```java
// Comments are not for Cucumber to execute; they are for human readers
// Comments end their scope when encountering an empty line or the next keyword
```

- **Anti-pattern Reference**: `anti-patterns.md#meaningless-comments`

---

### GL08: Doc Strings Use Correct Delimiters and Content Type Annotation

- **Principle Description**: When passing large blocks of text, use `"""` or `` ``` `` delimiters and annotate the content type (e.g., `"""markdown`, `"""json`). The delimiter indentation should align with the parent step; internal content indentation is handled automatically.
- **Rationale**: Correct content type annotation enables step definitions to perform appropriate parsing and processing. Uniform Doc String formatting improves readability and automation code maintainability.
- **Code Example**:

```gherkin
Scenario: Create a blog post with markdown content
  Given "Alice" is logged in as an author
  When she creates a new blog post titled "BDD Best Practices"
    And the body is:
      """markdown
      # BDD Best Practices

      ## Use Declarative Style
      Focus on *what* not *how*.

      ## Keep Scenarios Short
      3-5 steps per scenario is ideal.
      """
  Then the post should be published with formatted markdown
```

```java
@When("the body is:")
public void setBody(String body) {
    // The body parameter automatically receives the Doc String content
    // Indentation has been automatically stripped by Cucumber
    this.body = body;
}
```

- **Anti-pattern Reference**: `anti-patterns.md#docstring-format-error`

---

### GL09: Tense Consistency -- Given Uses Present Perfect, When/Then Uses Present Tense

- **Principle Description**: `Given` steps describe actions that have already occurred to establish the current state, using the present perfect tense. `When` and `Then` steps describe actions and results happening now, using the present tense.
- **Rationale**: Tense consistency makes scenarios read more naturally and conforms to English grammar conventions. It helps readers quickly distinguish preconditions (established state) from the behavior being verified.
- **Code Example**:

```gherkin
# Given - present perfect (state already established)
Given a registered user with email "john@example.com"
Given the customer has added 3 items to the cart
Given the inventory stock has been updated

# When - present tense (action occurring)
When the user submits the login form
When the customer applies discount code "SAVE20"

# Then - present tense (result verification)
Then the order total should be updated
Then links related to "panda" are shown on the results page
```

- **Anti-pattern Reference**: `anti-patterns.md#inconsistent-tense`

---

## 2. Scenario Authoring

---

### SC01: One Scenario Tests Only One Behavior

- **Principle Description**: Each Scenario must verify only one specific behavior. Chaining multiple When-Then sequences in the same scenario to test multiple independent behaviors is prohibited.
- **Rationale**: The single-behavior principle is the primary rule of BDD. When a scenario fails, the team should immediately know which specific behavior is problematic. A scenario testing multiple behaviors leads to difficult failure diagnosis and increased maintenance costs.
- **Code Example**:

```gherkin
# Recommended - one behavior per scenario
Feature: Shopping Cart

  Scenario: Add item to cart increases item count
    Given an empty shopping cart
    When the customer adds "Laptop" to the cart
    Then the cart should contain 1 item

  Scenario: Add item to cart updates total
    Given an empty shopping cart
    When the customer adds "Laptop" priced at $999 to the cart
    Then the cart total should be $999

  Scenario: Remove item from cart decreases count
    Given a cart containing "Laptop"
    When the customer removes "Laptop" from the cart
    Then the cart should be empty
```

```gherkin
# Not recommended - multiple When-Then sequences
Scenario: Cart operations
  Given an empty cart
  When I add "Laptop" to the cart
  Then the cart should contain 1 item
  When I add "Mouse" to the cart      # <-- second When
  Then the cart should contain 2 items # <-- second behavior
```

- **Anti-pattern Reference**: `anti-patterns.md#multiple-behaviors-in-one-scenario`

---

### SC02: Keep Scenarios to 3-5 Steps

- **Principle Description**: Each Scenario should have 3-5 steps (excluding Background). Scenarios exceeding 5 steps should be reviewed; those exceeding 8 steps must be split.
- **Rationale**: Scenarios with too many steps lose their expressive power as specification and documentation, as readers cannot quickly understand the scenario intent. Short scenarios are more readable, easier to maintain, and execute faster. Cucumber officially recommends 3-5 steps.
- **Code Example**:

```gherkin
# Recommended - 4 steps, focused on core behavior
Scenario: Successful login with valid credentials
  Given "John" has a registered account with password "ValidPass123"
  When "John" logs in with valid credentials
  Then he should be redirected to the order dashboard
  And he should see a welcome message with his name
```

```gherkin
# Not recommended - too many steps, obscuring the core intent
Scenario: Successful login with valid credentials
  Given the browser is open
  And the user navigates to the login page
  And the page loads completely
  And the user sees the login form
  When the user enters "john@example.com" in the email field
  And the user enters "password" in the password field
  And the user clicks the login button
  And the form is submitted
  Then the page should redirect
  And the dashboard should display
  And the welcome message should appear
```

- **Anti-pattern Reference**: `anti-patterns.md#scenario-step-bloat`

---

### SC03: Scenarios Must Be Fully Independent, No Cross-Scenario Dependencies

- **Principle Description**: Each scenario must be able to run independently at any time and in any order, without depending on the execution results or state of other scenarios. Each scenario is responsible for setting up its own Given preconditions.
- **Rationale**: Cross-scenario dependencies are one of the main causes of fragile tests. They break parallel execution capability, make debugging difficult, and violate the fundamental principle of testing (isolation).
- **Code Example**:

```gherkin
# Recommended - fully independent scenarios
Scenario: Place order after adding items
  Given "John" has a registered account
  And "John" has "Laptop" in his cart     # self-contained precondition
  When "John" proceeds to checkout
  Then the order should be created

Scenario: Place order with empty cart
  Given "John" has a registered account
  And "John" has an empty cart            # self-contained precondition
  When "John" proceeds to checkout
  Then an error should indicate the cart is empty
```

```gherkin
# Not recommended - Scenario B implicitly depends on Scenario A
Scenario: Create account
  When I register with valid details
  Then my account should be created

Scenario: Place order
  # Assumes account has already been created -- this causes problems!
  When I place an order
  Then order should be confirmed
```

- **Anti-pattern Reference**: `anti-patterns.md#cross-scenario-dependencies`

---

### SC04: Use Declarative Style to Describe Behavior, Not Imperative

- **Principle Description**: Use a declarative style when writing steps, describing "what" (business behavior), not an imperative description of "how" (implementation details). Self-test when writing: "If the implementation changes, would this text need to change?"
- **Rationale**: Declarative scenarios are immune to implementation changes. If the login method changes from form-based to OAuth or fingerprint, declarative scenarios do not need modification. Imperative scenarios are coupled with the UI; any interface tweak requires updating numerous scenarios.
- **Code Example**:

```gherkin
# Recommended - declarative (describes behavior)
Scenario: Free subscribers see only free articles
  Given Free Frieda has a free subscription
  When Free Frieda logs in with her valid credentials
  Then she sees a Free article

# Not recommended - imperative (describes implementation details)
Scenario: Free subscribers see only free articles
  Given I am on the login page
  When I type "freeFrieda@example.com" in the email field
  And I type "validPassword123" in the password field
  And I press the "Submit" button
  Then I see "FreeArticle1" on the home page
  And I do not see "PaidArticle1" on the home page
```

```java
// Declarative steps hide "how to implement login" inside the step definitions
// Implementation changes (e.g., from form login to SSO) do not require modifying Feature files
```

- **Anti-pattern Reference**: `anti-patterns.md#imperative-scenarios`

---

### SC05: Write Steps Using Business Language, Avoid Technical Terms

- **Principle Description**: Step text must use business language that all stakeholders (business, development, testing) can understand. Technical terms such as CSS selectors, API endpoints, database table names, and HTTP methods are prohibited.
- **Rationale**: The core of BDD is collaboration and shared understanding. Technical terms in Gherkin exclude non-technical stakeholders from participating and destroy the value of living documentation. Technical details should be hidden in the step definition layer.
- **Code Example**:

```gherkin
# Recommended - using business language
Scenario: Admin user views dashboard
  When the admin user logs in
  Then the admin dashboard should be displayed
  And the sales summary should be visible
```

```gherkin
# Not recommended - using technical terms
Scenario: Admin user views dashboard
  When I send POST to /api/v1/auth with {"user":"admin"}
  And the users table has role='ADMIN' for user_id=123
  Then I should see div#dashboard-header with color #333
  And the response JSON should contain "$.sales.total"
```

```java
// Technical details are handled in step definitions
@When("the admin user logs in")
public void adminLogin() {
    // This could be an API call, UI operation, or any implementation
    loginService.login("admin", adminPassword);
}
```

- **Anti-pattern Reference**: `anti-patterns.md#overly-technical-language`

---

### SC06: Follow Strict Given-When-Then Order

- **Principle Description**: Scenarios must be written in the order of Given (preconditions) -> When (action) -> Then (result). Repeating When-Then sequences in the same scenario is prohibited, as is placing Then before When.
- **Rationale**: Given-When-Then is the structured syntax of BDD, corresponding to the Arrange-Act-Assert pattern. Out-of-order or repeated When-Then violates the single responsibility principle and makes scenarios difficult to read and maintain.
- **Code Example**:

```gherkin
# Recommended - strict order
Scenario: Purchase with valid payment
  Given a cart with total $100
  And the customer has selected "Credit Card" payment
  When the customer confirms the purchase
  Then the payment should be processed
  And an order confirmation should be sent
```

```gherkin
# Not recommended - Then before When, with repeated When-Then
Scenario: Purchase and review
  Given a cart with items
  Then the cart total should display    # <-- Then before When
  When the customer pays
  Then payment should succeed
  When the customer leaves a review     # <-- second When
  Then review should be saved           # <-- second Then
```

- **Anti-pattern Reference**: `anti-patterns.md#out-of-order-given-when-then`

---

### SC07: Use Specific Role Names Instead of Generic Terms

- **Principle Description**: Use specific role names in scenarios (e.g., "Free Frieda", "Admin Alex") instead of generic terms ("a user", "the admin"). Naming gives roles specific business meaning.
- **Rationale**: Specific role names make scenarios more memorable and easier to distinguish different test paths. They help readers understand why this role is in this scenario and what the scenario's boundary conditions are.
- **Code Example**:

```gherkin
# Recommended - specific role names
Scenario: Free subscriber cannot access premium content
  Given Free Frieda has a free subscription
  When Free Frieda tries to access "Premium Article"
  Then she should see an upgrade prompt

Scenario: Premium subscriber accesses all content
  Given Premium Patty has a paid subscription
  When Premium Patty tries to access "Premium Article"
  Then she should see the full article
```

```gherkin
# Not recommended - generic terms
Scenario: Free subscriber cannot access premium content
  Given a user has a free subscription
  When the user tries to access premium content
  Then the user should see an upgrade prompt
```

- **Anti-pattern Reference**: `anti-patterns.md#generic-role-names`

---

### SC08: Keep Background Short (No More Than 4 Lines)

- **Principle Description**: Place Given steps shared by all scenarios within a Feature in `Background`. Background content must be short (no more than 4 lines) and contain only core preconditions that readers must know.
- **Rationale**: A long Background scrolls off-screen, preventing readers from seeing the complete scenario. Cucumber officially recommends Background be no more than 4 lines. If it exceeds this, unimportant details should be elevated into higher-level steps.
- **Code Example**:

```gherkin
# Recommended - concise Background (3 lines)
Feature: Shopping Cart Management

  Background:
    Given the customer "Jane" is logged in
    And "Jane" has an empty shopping cart

  Scenario: Add item to cart
    When "Jane" adds "Headphones" to the cart
    Then the cart should contain 1 item
```

```gherkin
# Not recommended - Background too long
Feature: Shopping Cart Management

  Background:
    Given the customer "Jane" is logged in
    And "Jane" has verified her email address
    And "Jane" has completed her profile
    And "Jane" has accepted the terms of service
    And "Jane" has selected her preferred currency as USD
    And "Jane" has an empty shopping cart
    And the inventory service is available
```

- **Anti-pattern Reference**: `anti-patterns.md#background-too-long`

---

### SC09: Scenario Names Should Describe Behavior, Not Verification Points

- **Principle Description**: A Scenario name should be a complete behavior description stating "who, under what circumstances, does what, with what result", not a simple list of verification points.
- **Rationale**: A good scenario name is itself part of the living documentation. When a test fails, the scenario name should immediately tell the team which specific behavior is problematic, without needing to read the step contents.
- **Code Example**:

```gherkin
# Recommended - behavior-descriptive names
Scenario: Account locks after three consecutive failed login attempts
  Given "Jane" has a registered account with password "CorrectPass"
  When "Jane" attempts to log in with "WrongPass" 3 consecutive times
  Then her account should be locked
  And she should see a message "Account locked. Contact support."

Scenario: Free subscribers see only free articles
  Given Free Frieda has a free subscription
  When Free Frieda logs in with her valid credentials
  Then she sees a Free article
```

```gherkin
# Not recommended - vague or verification-point-style names
Scenario: Test login          # too vague
Scenario: Login test case 5   # meaningless number
Scenario: Verify error message displays  # only describes a verification point
```

- **Anti-pattern Reference**: `anti-patterns.md#vague-scenario-names`

---

## 3. Step Definitions

---

### SD01: Use Cucumber Expressions Instead of Regular Expressions

- **Principle Description**: Step Definition expression matching should prioritize Cucumber Expressions (e.g., `{int}`, `{string}`, `{word}`), using regular expressions only when complex pattern matching is needed.
- **Rationale**: Cucumber Expressions are more readable than regular expressions and easier for non-technical people to understand. Built-in parameter types (int, float, word, string, etc.) cover the vast majority of scenarios and support custom parameter types.
- **Code Example**:

```gherkin
# Gherkin scenario
Scenario: Product quantity update
  Given there are 12 cucumbers in stock
  When a customer orders 5 cucumbers
  Then there should be 7 cucumbers remaining
```

```java
// Recommended - Cucumber Expressions
@Given("there are {int} cucumbers in stock")
public void setStock(int count) {
    this.stock = count;
}

@When("a customer orders {int} cucumbers")
public void placeOrder(int count) {
    this.stock -= count;
}

@Then("there should be {int} cucumbers remaining")
public void verifyStock(int expected) {
    assertEquals(expected, this.stock);
}
```

```java
// Not recommended - regular expression (unless necessary)
@Given("^there are (\\d+) cucumbers in stock$")
public void setStock(int count) { ... }
```

- **Anti-pattern Reference**: `anti-patterns.md#regular-expression-abuse`

---

### SD02: Organize Step Definition Files by Domain Concept

- **Principle Description**: Step definition files should be organized by domain concept (e.g., `login.steps.js`, `order.steps.js`), not by Feature or Scenario. Step naming should use domain-relevant names.
- **Rationale**: Feature-coupled step definitions lead to step definition explosion, code duplication, and high maintenance costs. Organizing by domain concept promotes cross-Feature step reuse and reduces maintenance costs.
- **Code Example**:

```
# Recommended directory structure
features/
  user-management/
    login.feature
    registration.feature
  shopping-cart/
    add-to-cart.feature
step-definitions/
  auth.steps.js       # by domain concept: authentication
  cart.steps.js       # by domain concept: shopping cart
  checkout.steps.js   # by domain concept: checkout
```

```java
// Not recommended - coupled by Feature
features/
  login.feature
  steps/
    login_steps.java      # only used by login.feature
  registration.feature
  steps/
    registration_steps.java  # only used by registration.feature
```

- **Anti-pattern Reference**: `anti-patterns.md#feature-coupled-step-definitions`

---

### SD03: Keep Step Definition Code Concise, Single Responsibility

- **Principle Description**: Each Step Definition method/function should do only one thing: call domain layer or automation layer code. Complex business logic, SQL queries, or UI location logic in step definitions is prohibited.
- **Rationale**: Step definitions are the thin layer between Gherkin and implementation code, and should not contain business logic. Complex logic should be delegated to Page Objects, API Clients, or domain services.
- **Code Example**:

```java
// Recommended - concise step definitions, delegated to domain layer
@When("the user logs in with {string} and {string}")
public void login(String username, String password) {
    loginPage.login(username, password);  // delegated to Page Object
}

@Then("the order total should be ${double}")
public void verifyOrderTotal(double expected) {
    OrderSummary summary = checkoutPage.getOrderSummary();
    assertEquals(expected, summary.getTotal());
}
```

```java
// Not recommended - complex logic in step definitions
@When("the user logs in with {string} and {string}")
public void login(String username, String password) {
    WebElement emailField = driver.findElement(By.cssSelector("#email"));
    emailField.clear();
    emailField.sendKeys(username);
    WebElement passwordField = driver.findElement(By.cssSelector("#password"));
    passwordField.sendKeys(password);
    WebElement submitBtn = driver.findElement(By.cssSelector(".submit-btn"));
    // complex wait logic...
    WebDriverWait wait = new WebDriverWait(driver, 10);
    wait.until(ExpectedConditions.elementToBeClickable(submitBtn));
    submitBtn.click();
}
```

- **Anti-pattern Reference**: `anti-patterns.md#step-definition-god-object`

---

### SD04: Steps Must Be Atomic, No Conjunction Steps

- **Principle Description**: Each step describes only one atomic action or state. Using "and" to combine multiple different things into a single step text is prohibited.
- **Rationale**: Conjunction steps make steps overly specialized and difficult to reuse. Cucumber's built-in support for `And` and `But` exists for a reason. Keeping steps atomic maximizes reusability.
- **Code Example**:

```gherkin
# Recommended - atomic steps
Given I have shades
And I have a brand new Mustang
When I start the engine
Then I should feel the power
```

```gherkin
# Not recommended - conjunction steps
Given I have shades and a brand new Mustang
When I start the engine and press the accelerator
```

```java
// Conjunction step matching patterns are too specific to reuse in other scenarios
// Atomic step "I have {string}" can match multiple items
```

- **Anti-pattern Reference**: `anti-patterns.md#conjunction-steps`

---

### SD05: Given Steps Are for Setting System State, No User Interaction

- **Principle Description**: The purpose of `Given` steps is to place the system in a known state before the `When` step. State should be set directly via API calls, database initialization, or domain services, avoiding UI operations to set up preconditions.
- **Rationale**: Setting preconditions through the UI makes tests slow and fragile. Given steps should quickly and directly establish system state. User interaction should be reserved for When steps.
- **Code Example**:

```gherkin
# Gherkin - Given describes state
Scenario: Apply discount to cart
  Given the customer has 3 items in their cart
  And the cart total is $150
  When the customer applies discount code "SAVE20"
  Then the cart total should be $120
```

```java
// Recommended - Given directly sets state (not through UI)
@Given("the customer has {int} items in their cart")
public void setupCart(int itemCount) {
    // Directly create cart data via API or database
    cartService.createCart(testCustomerId);
    for (int i = 0; i < itemCount; i++) {
        cartService.addItem(testProductIds.get(i));
    }
}

@Given("the cart total is ${double}")
public void setCartTotal(double total) {
    // Directly set database state
    cartService.setTotal(testCustomerId, total);
}
```

- **Anti-pattern Reference**: `anti-patterns.md#given-steps-setting-state-through-ui`

---

### SD06: Then Steps Should Assert Observable Output, No Direct Database Queries

- **Principle Description**: `Then` steps should compare user-observable outputs (UI display, API response, message notifications), not directly query the database or check internal system state.
- **Rationale**: Database validation is an internal implementation detail that may fail due to schema changes. Then steps should verify observable system behavior from the user perspective, consistent with BDD's "outside-in" philosophy.
- **Code Example**:

```gherkin
# Gherkin - Then describes observable results
Scenario: Successful order placement
  When the customer places an order for "Laptop"
  Then the order confirmation page should display
  And the confirmation number should be visible
  And a confirmation email should be sent
```

```java
// Recommended - assert observable output
@Then("the order confirmation page should display")
public void verifyConfirmationPage() {
    assertTrue(confirmationPage.isDisplayed());
}

@Then("a confirmation email should be sent")
public void verifyEmailSent() {
    // Check via email service API, not direct database query
    Email email = emailService.getLatestEmail(testCustomerId);
    assertTrue(email.getSubject().contains("Order Confirmed"));
}
```

```java
// Not recommended - direct database query
@Then("the order should be saved")
public void verifyOrderInDatabase() {
    String sql = "SELECT * FROM orders WHERE customer_id = ?";
    ResultSet rs = jdbcTemplate.query(sql, customerId);
    // ... database assertions
}
```

- **Anti-pattern Reference**: `anti-patterns.md#then-steps-direct-database-queries`

---

### SD07: Use Page Object Model to Encapsulate UI Interactions

- **Principle Description**: When testing involves a UI, step definitions must interact with the system through Page Object classes. Using locators (CSS selectors, XPath) directly in step definitions is prohibited.
- **Rationale**: The Page Object Model separates page element location from test logic; when the UI changes, only the Page Object needs updating, not the step definitions. It is the standard design pattern for UI automation testing.
- **Code Example**:

```java
// Page Object layer - encapsulates UI interactions
public class LoginPage {
    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "login-button")
    private WebElement loginButton;

    public void login(String username, String password) {
        usernameField.sendKeys(username);
        passwordField.sendKeys(password);
        loginButton.click();
    }
}
```

```java
// Step Definition layer - concise delegation
public class LoginSteps {
    private LoginPage loginPage;

    public LoginSteps(TestContext context) {
        this.loginPage = context.getLoginPage();
    }

    @When("{word} logs in with valid credentials")
    public void login(String username) {
        User user = userRepository.findByName(username);
        loginPage.login(user.getEmail(), user.getPassword());
    }
}
```

- **Anti-pattern Reference**: `anti-patterns.md#ui-locators-leaking-into-step-definitions`

---

### SD08: Use Dependency Injection to Share State Between Steps

- **Principle Description**: Cross-step state sharing should be implemented through dependency injection (JVM: PicoContainer/Spring; JS: World object). Using global or static variables to pass data between steps is prohibited.
- **Rationale**: Global/static variables cause data races and unpredictable results during parallel execution. Dependency injection ensures each scenario gets a clean state instance, supporting thread safety.
- **Code Example**:

```java
// JVM - using PicoContainer dependency injection
// TestContext.java - shared state class
public class TestContext {
    private User currentUser;
    private Order lastOrder;

    public void setCurrentUser(User user) { this.currentUser = user; }
    public User getCurrentUser() { return currentUser; }
    public void setLastOrder(Order order) { this.lastOrder = order; }
    public Order getLastOrder() { return lastOrder; }
}

// Step Definition - injected via constructor
public class OrderSteps {
    private final TestContext context;

    public OrderSteps(TestContext context) {  // PicoContainer auto-injects
        this.context = context;
    }

    @When("the customer places an order for {string}")
    public void placeOrder(String productName) {
        Order order = orderService.place(context.getCurrentUser(), productName);
        context.setLastOrder(order);  // save to shared state
    }

    @Then("the order confirmation should display")
    public void verifyConfirmation() {
        assertNotNull(context.getLastOrder());
    }
}
```

```javascript
// JavaScript - using World object
Given('the customer places an order for {string}', async function(productName) {
    const order = await orderService.place(this.currentUser, productName);
    this.lastOrder = order;  // save to World object
});
```

- **Anti-pattern Reference**: `anti-patterns.md#global-variable-state-sharing`

---

### SD09: Use Hooks for Setup and Teardown; Prefer Background Over Before Hook

- **Principle Description**: `@Before`/`@After` Hooks are for technical setup (launching browsers, cleaning databases). If setup logic should be readable and understood by non-technical people, prefer `Background`. After Hooks must contain cleanup logic (closing browsers, deleting test data).
- **Rationale**: Logic in Before Hooks is invisible to those reading only the Feature file. Business-related setup should be placed in Background for transparency. After Hooks ensure cleanup runs even when scenarios fail.
- **Code Example**:

```gherkin
# Background for business-visible setup
Feature: Shopping Cart

  Background:
    Given "Jane" is logged in as a registered customer
    # Business stakeholders can understand this precondition

  Scenario: Add item to cart
    ...
```

```java
// Hooks for technical setup and teardown
public class TestHooks {
    private TestContext context;

    public TestHooks(TestContext context) {
        this.context = context;
    }

    @Before(order = 1)  // order controls execution sequence
    public void initializeBrowser() {
        context.setDriver(new ChromeDriver());
    }

    @Before(order = 2)
    public void cleanTestData() {
        testDataService.clearTestData();
    }

    @After  // runs even when scenario fails
    public void tearDown(Scenario scenario) {
        if (scenario.isFailed()) {
            takeScreenshot();
        }
        context.getDriver().quit();
    }
}
```

- **Anti-pattern Reference**: `anti-patterns.md#before-hook-hiding-business-preconditions`

---

## 4. Data-Driven Testing

---

### DD01: Use Scenario Outline + Examples to Test Multiple Data Sets

- **Principle Description**: When the same scenario logic needs to be validated with multiple sets of different data, use `Scenario Outline` with `Examples` tables. Each row of data runs a complete scenario once.
- **Rationale**: Scenario Outline avoids scenario copy-paste, making data variants clear at a glance. Each row in the Examples table is a concrete business example, clearly displaying boundary conditions.
- **Code Example**:

```gherkin
# Recommended - Scenario Outline + Examples
Scenario Outline: Login with various credential combinations
  Given "<username>" has a registered account
  When "<username>" attempts to log in with "<password>"
  Then the response should be "<result>"

  Examples:
    | username | password    | result            |
    | john     | correctPass | success           |
    | john     | wrongPass   | invalid credentials |
    | unknown  | anyPass     | user not found    |
    | john     |             | password required |
    |          | somePass    | username required |
```

```java
// Step definitions use Cucumber Expressions to receive parameters
@When("{string} attempts to log in with {string}")
public void attemptLogin(String username, String password) {
    loginPage.login(username, password);
}
```

- **Anti-pattern Reference**: `anti-patterns.md#scenario-copy-paste`

---

### DD02: Use Data Tables to Pass Structured Data to Steps

- **Principle Description**: When structured data (e.g., multiple records, form fields) needs to be passed to a single step, use Data Tables. A Data Table follows immediately after the step, defined using pipe characters `|`.
- **Rationale**: Data Tables are suitable for step-level batch data injection, complementing the scenario-level injection of Scenario Outlines. Data Tables support multiple data type conversions (List, Map, etc.).
- **Code Example**:

```gherkin
Scenario: New users receive welcome emails
  Given the following users have registered:
    | first_name | email              | plan  |
    | Alice      | alice@example.com  | free  |
    | Bob        | bob@example.com    | pro   |
    | Charlie    | charlie@example.com| basic |
  When the daily welcome email job runs
  Then all users should receive a personalized welcome email
```

```java
@Given("the following users have registered:")
public void registerUsers(DataTable dataTable) {
    List<Map<String, String>> users = dataTable.asMaps();
    for (Map<String, String> user : users) {
        String name = user.get("first_name");
        String email = user.get("email");
        String plan = user.get("plan");
        userService.register(name, email, plan);
    }
}
```

- **Anti-pattern Reference**: `anti-patterns.md#datatable-and-scenario-outline-confusion`

---

### DD03: Distinguish Between Data Table and Scenario Outline Use Cases

- **Principle Description**: Data Tables are for passing multiple rows of data in a single execution (step-level); Scenario Outlines are for executing the same logic multiple times (scenario-level). Confusing the two use cases is prohibited.
- **Rationale**: Confusing usage leads to tests running at the wrong level of abstraction, affecting performance and maintainability. Data Tables are suitable for batch data creation; Scenario Outlines are suitable for data combination testing.
- **Code Example**:

```gherkin
# Data Table - single execution, passing multiple rows (suitable for batch operations)
Scenario: Bulk user registration
  Given the following users are registered in batch:
    | name  | email             | role  |
    | Alice | alice@example.com | admin |
    | Bob   | bob@example.com   | user  |
  Then all registrations should succeed

# Scenario Outline - each row executes a complete scenario once (suitable for data combination testing)
Scenario Outline: Login with different roles
  Given a user with "<role>" role exists
  When the user logs in
  Then the "<expected_page>" should be displayed

  Examples:
    | role    | expected_page   |
    | admin   | admin dashboard |
    | user    | user home       |
    | guest   | login page      |
```

```java
// DataTable is suitable for: creating multiple records, filling form fields, configuration data
// Scenario Outline is suitable for: boundary condition testing, role permission testing, input validation testing
```

- **Anti-pattern Reference**: `anti-patterns.md#incorrect-data-driven-level-selection`

---

### DD04: Use Doc Strings to Pass Large Text Data

- **Principle Description**: When a step needs to receive multi-line text content (e.g., JSON, Markdown, long text messages), use Doc Strings (`"""` delimiters) rather than compressing text into a single-line step.
- **Rationale**: Doc Strings preserve the format and readability of text content, suitable for passing complex content. Annotating the content type (e.g., `"""json`) enables step definitions to perform targeted parsing.
- **Code Example**:

```gherkin
Scenario: Create product with description
  When the admin creates a product with the following details:
    """json
    {
      "name": "Premium Wireless Headphones",
      "price": 199.99,
      "category": "Electronics",
      "description": "Noise-cancelling over-ear headphones with 30-hour battery life."
    }
    """
  Then the product should be listed in the catalog
```

```java
@When("the admin creates a product with the following details:")
public void createProduct(String jsonBody) {
    // jsonBody receives the complete JSON text
    ObjectMapper mapper = new ObjectMapper();
    Product product = mapper.readValue(jsonBody, Product.class);
    productService.create(product);
}
```

- **Anti-pattern Reference**: `anti-patterns.md#single-line-long-text-passing`

---

### DD05: Dynamically Generate Test Data, Avoid Hard-Coding

- **Principle Description**: Step definitions should dynamically generate test data (e.g., random usernames, temporary email addresses), avoiding hard-coding specific data values in scenario text. Scenarios use placeholders or data tables; actual data is generated in step definitions.
- **Rationale**: Hard-coded data makes tests fragile and causes scenario failures when environments change. Dynamic data makes tests more robust and supports parallel execution (each test uses unique data to avoid conflicts).
- **Code Example**:

```gherkin
# Scenarios use placeholders, not hard-coded specific values
Scenario: New user registration
  Given a visitor with a unique email address
  When the visitor completes registration
  Then a confirmation email should be sent to the registered address
```

```java
// Dynamic data generation in step definitions
@Given("a visitor with a unique email address")
public void createUniqueVisitor() {
    String uniqueEmail = "test-" + UUID.randomUUID() + "@example.com";
    this.visitor = new Visitor(uniqueEmail);
}

@Given("a user with {string} role exists")
public void createUserWithRole(String role) {
    String username = "user-" + System.currentTimeMillis();
    userService.createUser(username, role);  // dynamic creation
}
```

- **Anti-pattern Reference**: `anti-patterns.md#hard-coded-test-data`

---

### DD06: Keep Examples Tables at a Reasonable Scale

- **Principle Description**: The number of rows in `Examples` tables should be kept at a reasonable scale (no more than 10-15 rows). Edge cases and boundary condition testing should be left to unit tests; do not exhaust all input combinations in Gherkin.
- **Rationale**: Too many Examples rows is a symptom of "scenario bloat". Gherkin scenarios should be reserved for behaviors that business stakeholders care about; technical edge cases should be covered by unit tests (faster and more precise).
- **Code Example**:

```gherkin
# Recommended - Examples focused on business-relevant data combinations
Scenario Outline: Payment processing with different card types
  Given a cart total of $<amount>
  When the customer pays with "<card_type>"
  Then the payment should be "<result>"

  Examples:
    | amount | card_type | result    |
    | 50.00  | Visa      | approved  |
    | 50.00  | Mastercard| approved  |
    | 50.00  | Amex      | approved  |
    | 0.01   | Visa      | declined  |  # minimum amount boundary
    | 10000  | Visa      | review    |  # large amount requires review
```

```gherkin
# Not recommended - 50+ row Examples exhausting all combinations
# These boundary cases should be left to unit tests
Examples:
  | email     | result              |
  | a@b.c     | valid               |
  | a@b       | invalid             |
  | @b.c      | invalid             |
  | a@.c      | invalid             |
  | ...       | ...                 |
  # 50 more rows...
```

- **Anti-pattern Reference**: `anti-patterns.md#scenario-bloat-and-over-coverage`

---

## 5. Team Collaboration & Process

---

### TM01: Follow the Discovery -> Formulation -> Automation Three-Stage Model

- **Principle Description**: BDD work must follow the three-stage model defined by Cucumber: Discovery (exploring requirements), Formulation (writing Gherkin), Automation (implementing step definitions). Skipping any stage to directly write automation code is prohibited.
- **Rationale**: The three-stage model ensures BDD's core objectives (collaboration and shared understanding) are achieved. Skipping the Discovery stage to directly write Gherkin leads to scenarios disconnected from real business needs, reducing them to test scripts.
- **Code Example**:

```
Phase 1: Discovery
  |-- Three Amigos meeting
  |-- Example Mapping workshop
  |-- Output: business rules + concrete examples (cards)

Phase 2: Formulation
  |-- Document examples as Gherkin scenarios
  |-- Business expert review and confirmation
  |-- Output: Feature files

Phase 3: Automation
  |-- Implement step definitions
  |-- Run and pass tests
  |-- Output: executable test suite
```

```gherkin
# Phase 2 output - Feature file after Formulation
Feature: Customer Login
  # After discussion in the Discovery phase, the following scenarios are confirmed as core requirements
  # JIRA-1234

  Scenario: Successful login with valid credentials
    Given "John" has a registered account
    When "John" logs in with valid credentials
    Then he should be redirected to the order dashboard
```

- **Anti-pattern Reference**: `anti-patterns.md#skipping-discovery-to-write-automation`

---

### TM02: Three Amigos Meeting Must Include Business, Development, and Testing

- **Principle Description**: Three Amigos is a collaborative agile framework where the three core roles must conduct structured conversations around user stories before development: Business (clarifying the problem), Development (deciding how to build), Testing (verifying correctness).
- **Rationale**: Participation from all three parties ensures requirements are examined from business, technical, and quality dimensions. Missing any party leads to understanding deviation, with costly rework later. The meeting should be held one Sprint before development.
- **Code Example**:

```
Three Amigos Meeting Flow:
1. Pre-meeting preparation - select a manageable feature/user story
2. Discuss expected behavior - using Given-When-Then format
3. Identify boundary cases - ask lots of "what if" questions
4. Record examples - using Example Mapping cards
5. Convert to Gherkin - transform agreed examples into structured scenarios
6. Confirm automation scope - determine which examples should be automated

Tips for effective sessions:
- Recommended duration: 35-60 minutes (exceeding this means the story is too large)
- Keep the team small (3-5 people is most efficient)
- Is part of Shift Left practice
```

```gherkin
# Meeting output - Feature file confirmed by all three parties
Feature: Shopping Cart Discount
  # Confirmed by: PO(Alice), Dev(Bob), QA(Carol) - 2025-06-15
  # JIRA-5678

  Scenario: Apply valid discount code reduces total
    Given a cart with total amount $100
    When the customer applies discount code "SAVE20"
    Then the cart total should be $80
    And a discount of $20 should be recorded
```

- **Anti-pattern Reference**: `anti-patterns.md#bdd-without-business-participation`

---

### TM03: Use Example Mapping Workshop to Decompose Requirements

- **Principle Description**: Use Example Mapping technique to decompose user stories: yellow cards (user stories), blue cards (business rules), green cards (examples), red cards (questions). Each blue rule card corresponds to at least one green example card.
- **Rationale**: Example Mapping makes the requirements decomposition process visual and structured. It helps teams identify ambiguity in requirements (red cards) and ensures all business rules have corresponding example coverage.
- **Code Example**:

```
Example Mapping Card Layout:
  [Yellow] User Story: As a shopper, I want to apply discount codes
    |-- [Blue] Rule: Valid discount code reduces cart total
    |     |-- [Green] Example: SAVE20 reduces by 20%
    |     |-- [Green] Example: SAVE50 reduces by 50%
    |-- [Blue] Rule: Expired discount codes are rejected
    |     |-- [Green] Example: expired code EXPIRED50
    |     |-- [Red] Question: Is expiration determined by UTC or local time?
    |-- [Blue] Rule: Already used discount codes cannot be reused
          |-- [Green] Example: second use of same code is rejected
```

```gherkin
# Convert green cards into Gherkin scenarios
Feature: Shopping Cart Discount

  Rule: Valid discount code reduces cart total
    Scenario: SAVE20 reduces total by 20%
      Given a cart with total amount $100
      When the customer applies discount code "SAVE20"
      Then the cart total should be $80
```

- **Anti-pattern Reference**: `anti-patterns.md#coding-without-example-mapping`

---

### TM04: Scenarios Must Be Reviewed and Confirmed by Business Experts

- **Principle Description**: All Gherkin scenarios must be reviewed and confirmed by business experts (Product Owner or Business Analyst) before automation. Scenarios are business contracts, not just test code.
- **Rationale**: Business confirmation ensures scenarios truly reflect business needs rather than developer assumptions. As living documentation, the authority and accuracy of scenarios depend on the recognition of business stakeholders.
- **Code Example**:

```gherkin
# Feature file header annotated with review information
# Reviewed by: Alice Wang (Product Owner)
# Reviewed on: 2025-06-15
# Status: Approved
# JIRA-1234

Feature: Order Cancellation
  As a customer
  I want to cancel my order within 24 hours
  So that I can correct mistakes

  Rule: Orders can be cancelled within 24 hours of placement
    Scenario: Cancel order placed 1 hour ago
      Given an order placed 1 hour ago
      When the customer cancels the order
      Then the order status should be "cancelled"
      And a refund should be initiated
```

- **Anti-pattern Reference**: `anti-patterns.md#scenarios-without-business-confirmation`

---

### TM05: Ubiquitous Language Must Be Unified Across the Entire Team

- **Principle Description**: All stakeholders must use unified business terminology (ubiquitous language) when writing and discussing scenarios. Every word in a scenario should be a domain term that everyone on the team understands.
- **Rationale**: Ubiquitous language is the bridge between BDD and DDD. It eliminates terminology differences between business and technology, ensuring "everyone is using the same words to describe the same thing."
- **Code Example**:

```gherkin
# Recommended - using unified domain terminology
Feature: Room Reservation
  As a hotel guest
  I want to reserve a room
  So that I can guarantee my stay

  Scenario: VIP room reservation marks it as unavailable
    Given a VIP room "Ocean Suite" is available
    When I reserve the "Ocean Suite" for 3 nights
    Then the system should mark "Ocean Suite" as unavailable
    And a reservation confirmation should be sent

# Team glossary:
# Room Type: Standard, Deluxe, VIP, Suite
# Status: Available, Reserved, Checked-in, Checked-out
# Action: Reserve, Cancel, Modify
```

```gherkin
# Not recommended - mixed terminology
Feature: Room Booking  # "Booking" vs "Reservation" - inconsistent terms
  ...
    When I book a room  # "book" vs "reserve" - two words for the same concept
```

- **Anti-pattern Reference**: `anti-patterns.md#inconsistent-terminology`

---

### TM06: Regularly Review and Refactor the BDD Scenario Suite

- **Principle Description**: The BDD scenario suite should be reviewed regularly (recommended once per Sprint), removing outdated scenarios, updating scenarios for business changes, and identifying and fixing fragile tests. Outdated scenarios are worse than having no scenarios.
- **Rationale**: Scenarios are living documentation and must remain synchronized with actual system behavior. Outdated scenarios create false confidence, causing the team to lose trust in the test suite. Regular review is a necessary practice to keep the BDD suite healthy.
- **Code Example**:

```
Sprint Review Checklist:
  [ ] All new features have corresponding Gherkin scenarios
  [ ] Related scenarios updated after business changes
  [ ] @wip tagged scenarios completed and tags removed
  [ ] Failed scenarios analyzed and fixed
  [ ] Meaningless scenarios deleted
  [ ] Step definition reuse rate reasonable (no large amount of duplicate steps)
  [ ] Scenario execution time reasonable (no timeout scenarios)
```

```gherkin
# Update scenarios promptly to reflect business changes
# Old scenario - business rule has changed
# @deprecated - 2025-05: 24-hour cancellation window changed to 12 hours
Scenario: Cancel order within 24 hours
  ...

# New scenario - reflecting the latest business rule
Scenario: Cancel order within 12 hours
  Given an order placed 10 hours ago
  When the customer cancels the order
  Then the order should be cancelled
```

- **Anti-pattern Reference**: `anti-patterns.md#outdated-bdd-scenario-suite`

---

## 6. Test Organization

---

### TO01: Feature Files Use kebab-case Naming Aligned with Domain Concepts

- **Principle Description**: `.feature` filenames use lowercase, hyphen-separated format (kebab-case), with filenames directly related to product domain concepts. Generic names (e.g., `test.feature`, `actions.feature`) are prohibited.
- **Rationale**: Domain-aligned filenames make the project structure self-documenting and facilitate quick feature location. kebab-case is a standard convention in the Cucumber community, with good compatibility across operating systems.
- **Code Example**:

```
# Recommended - domain-aligned naming
features/
  authentication/
    customer-login.feature
    password-reset.feature
    social-login.feature
  order-management/
    place-order.feature
    cancel-order.feature
    order-history.feature
  payment-processing/
    credit-card-payment.feature
    discount-codes.feature
    refund-processing.feature

# Not recommended - generic or vague naming
features/
  test.feature
  actions.feature
  scenarios.feature
  login-test.feature
  new-feature.feature
```

- **Anti-pattern Reference**: `anti-patterns.md#generic-feature-filenames`

---

### TO02: Organize Directory Structure by Functional Module

- **Principle Description**: Feature files are organized in subdirectories by functional module/domain boundary. Each module has one directory with a name reflecting the business domain. Step definition files are organized by domain concept rather than Feature file.
- **Rationale**: A modular directory structure reflects the system's domain architecture, facilitating team collaboration (different teams responsible for different modules) and code navigation. It also enables module-level tag execution and report filtering.
- **Code Example**:

```
project/
  features/                          # Feature file root directory
    authentication/                  # Authentication module
      login.feature
      registration.feature
      password-recovery.feature
    shopping-cart/                   # Shopping cart module
      add-to-cart.feature
      apply-discount.feature
      checkout.feature
    user-profile/                    # User profile module
      update-profile.feature
      manage-preferences.feature
  step-definitions/                  # Step definitions (by domain concept)
    auth.steps.js                    # Authentication-related steps
    cart.steps.js                    # Shopping cart-related steps
    user.steps.js                    # User-related steps
    common.steps.js                  # Common/shared steps
  support/                           # Shared infrastructure
    hooks.js                         # Before/After hooks
    world.js                         # Shared state/World
    custom-parameter-types.js        # Custom parameter types
```

- **Anti-pattern Reference**: `anti-patterns.md#flat-directory-structure`

---

### TO03: Step Definition Files Use `.steps.js` Suffix with Consistent Naming Convention

- **Principle Description**: Step definition filenames use domain concept naming with the `.steps.js` suffix (or the corresponding language convention suffix). Use kebab-case or snake_case consistently.
- **Rationale**: A unified suffix makes step definition files easily identifiable in the project, facilitating build tool configuration and code review. Domain naming ensures steps are aggregated by function rather than dispersed by Feature.
- **Code Example**:

```
# Recommended - unified suffix and naming convention
step-definitions/
  auth.steps.js              # Authentication domain
  cart.steps.js              # Shopping cart domain
  checkout-payment.steps.js  # Composite features connected with hyphens
  common.steps.js            # Common/shared steps
  user-management.steps.js   # User management
```

```java
// Java projects use corresponding package structure and naming
src/test/java/stepdefinitions/
  AuthSteps.java
  CartSteps.java
  CheckoutSteps.java
```

- **Anti-pattern Reference**: `anti-patterns.md#inconsistent-step-definition-naming`

---

### TO04: Use Tags to Control Test Execution in CI/CD Pipelines

- **Principle Description**: Use a tag system to control test execution at different CI/CD stages: @smoke (smoke tests), @regression (regression tests), @wip (work in progress), @e2e (end-to-end), @api (API tests). CI configuration uses tag expressions for selective execution.
- **Rationale**: Tag-driven CI/CD integration makes test execution strategy flexible and controllable. Smoke tests can run quickly on commit, full regression tests run at night, and WIP scenarios are excluded during development.
- **Code Example**:

```gherkin
# Tag system example
@regression @authentication @priority-high
Feature: Customer Login

  @positive @smoke
  Scenario: Successful login with valid credentials
    ...

  @negative @security
  Scenario: Account locks after three failed attempts
    ...

  @wip
  Scenario: Social login with Google
    ...
```

```yaml
# GitHub Actions configuration
- name: Run smoke tests
  run: mvn test -Dcucumber.filter.tags="@smoke"

- name: Run regression tests
  run: mvn test -Dcucumber.filter.tags="@regression and not @wip"
  if: github.ref == 'refs/heads/main'
```

- **Anti-pattern Reference**: `anti-patterns.md#chaotic-tag-system`

---

### TO05: Integrate Multi-Format Reporting into CI/CD Pipeline

- **Principle Description**: The CI/CD pipeline must generate at least two report formats: HTML (human-readable) and JSON (machine-processable). Evidence (screenshots, logs) must be collected on failure.
- **Rationale**: Multi-format reports meet the needs of different audiences: HTML for team and business review, JSON for CI tool parsing, JUnit XML for integration with Jenkins and similar tools. Failure evidence accelerates problem diagnosis.
- **Code Example**:

```java
// Cucumber Runner multi-format report configuration
@RunWith(Cucumber.class)
@CucumberOptions(
    plugin = {
        "pretty",                                    // console
        "html:target/cucumber-reports.html",         // HTML
        "json:target/cucumber-reports.json",         // JSON
        "junit:target/cucumber-reports.xml"          // JUnit XML
    }
)
public class TestRunner { }
```

```yaml
# GitHub Actions - collect evidence on failure
- name: Upload test reports
  uses: actions/upload-artifact@v4
  if: always()
  with:
    name: test-reports
    path: |
      target/cucumber-reports.html
      target/cucumber-reports.json
      target/screenshots/
```

- **Anti-pattern Reference**: `anti-patterns.md#single-report-format`

---

### TO06: Ensure Scenarios Can Be Executed in Parallel

- **Principle Description**: All scenarios must be designed for parallel execution. Eliminate shared state, use independent test data, and avoid competitive access to the same resource. Parallel execution is the foundation of efficient CI/CD operation.
- **Rationale**: Parallel execution significantly shortens test feedback time. Shared state or data races between scenarios are the main causes of parallel failures and must be eliminated at the design stage.
- **Code Example**:

```java
// Recommended - each scenario uses independent data
@Given("a customer with a unique account")
public void createUniqueCustomer() {
    String uniqueId = UUID.randomUUID().toString();
    this.customer = customerService.create("customer-" + uniqueId);
    // each parallel thread uses different data
}

// Recommended - Before Hook cleans shared resources
@Before
public void cleanUp() {
    // Clean data that might affect other scenarios before each scenario
    cartService.clearAllCarts();
    orderService.clearTestOrders();
}
```

```gherkin
# Avoid scenarios depending on specific execution order
# Scenario A and Scenario B must be able to pass independently in any order
```

```bash
# cucumber-js parallel execution
cucumber-js --parallel 4

# Maven Surefire parallel execution
mvn test -DforkCount=4 -DreuseForks=true
```

- **Anti-pattern Reference**: `anti-patterns.md#shared-state-between-scenarios-causing-parallel-failures`

---

### TO07: Use Living Documentation Tools to Generate Readable Documentation

- **Principle Description**: Use tools such as Serenity BDD, Cukedoctor, or SpecFlow+ LivingDoc to automatically generate Living Documentation from Feature files. Living documentation should be accessible to all stakeholders.
- **Rationale**: Living Documentation is one of the core values of BDD. It transforms executable scenarios into business-readable documentation, ensuring documentation is always synchronized with code, eliminating the cost of maintaining separate documentation.
- **Code Example**:

```gradle
// build.gradle - Serenity BDD configuration
plugins {
    id 'java'
    id 'net.serenity-bdd.serenity-gradle-plugin' version '4.0.46'
}

dependencies {
    implementation "net.serenity-bdd:serenity-core:4.0.46"
    implementation "net.serenity-bdd:serenity-cucumber:4.0.46"
}

// Task: test -> aggregate -> generate HTML report
// Report path: target/site/serenity/index.html
```

```properties
# serenity.properties
serenity.take.screenshots=AFTER_EACH_STEP
serenity.reports.show.step.details=true
serenity.report.accessibility=true
```

- **Anti-pattern Reference**: `anti-patterns.md#ignoring-living-documentation`

---

### TO08: Use Custom Parameter Types for Domain-Specific Concepts

- **Principle Description**: Define custom parameter types for domain-specific concepts (e.g., currency, dates, status enumerations) to make Gherkin steps more natural and step definitions more concise.
- **Rationale**: Custom parameter types make Gherkin steps read like natural language (e.g., `{money}`, `{iso-date}`), while automatically handling type conversion, reducing parsing code in step definitions.
- **Code Example**:

```java
// Define custom parameter types
@ParameterType("\\d+\\.\\d{2} (USD|EUR|CNY)")
public Money money(String amount, String currency) {
    return new Money(new BigDecimal(amount), Currency.valueOf(currency));
}

@ParameterType("\\d{4}-\\d{2}-\\d{2}")
public LocalDate isoDate(String date) {
    return LocalDate.parse(date);
}
```

```gherkin
# Using custom parameter types in Gherkin - natural and fluent
Scenario: Purchase with discount
  Given a product priced at 199.99 USD
  When the customer applies a 10% discount on 2024-12-01
  Then the final price should be 179.99 USD
```

```java
// Step definitions directly use domain types
@Given("a product priced at {money}")
public void setProductPrice(Money price) {
    this.product = new Product(price);
}

@When("the customer applies a {int}% discount on {iso-date}")
public void applyDiscount(int percentage, LocalDate date) {
    this.product.applyDiscount(percentage, date);
}
```

- **Anti-pattern Reference**: `anti-patterns.md#missing-custom-parameter-types`

---

*This guide contains 47 coding principles covering six dimensions: Gherkin Syntax Standards, Scenario Authoring, Step Definitions, Data-Driven Testing, Team Collaboration & Process, and Test Organization. All principles are derived from Cucumber official documentation, community best practices, and architecture pattern research, and can be used as coding standards and code review checklists for BDD projects.*
