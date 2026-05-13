# BDD Anti-Patterns Catalog

> This document summarizes the 12 most common anti-patterns in BDD (Behavior-Driven Development) practice.
> Each anti-pattern includes: symptom identification, consequence analysis, fix solutions, corresponding correct principles, and code examples.
>
> Compiled based on Cucumber official documentation, community best practices, and industry research reports.

---

## Table of Contents

1. [Imperative Scenarios](#ap1-imperative)
2. [Feature-coupled Step Definitions](#ap2-feature-coupled)
3. [Conjunction Steps (And/But Overuse)](#ap3-conjunction)
4. [UI-centric Gherkin](#ap4-ui-centric)
5. [God Step Definition](#ap5-god-step)
6. [Brittle Selectors](#ap6-brittle-selectors)
7. [Testing Through the UI](#ap7-through-ui)
8. [Lack of Ubiquitous Language](#ap8-no-ubiquitous-language)
9. [Scenario Dependence](#ap9-scenario-dependence)
10. [Too Many Details](#ap10-too-many-details)
11. [Sleep/Wait Anti-pattern](#ap11-sleep-wait)
12. [No Clear Given-When-Then](#ap12-no-gwt)

---

## <a id="ap1-imperative"></a>AP-01: Imperative Scenarios

> **One-line description**: Scenarios describe "how to do" (operation steps) rather than "what to do" (business behavior), tightly coupled with implementation details.

### Symptoms (How to Identify)

- Specific UI operation verbs appear in scenarios: `click`, `type`, `press`, `enter`, `select`, `scroll`
- Steps contain page element names: `email field`, `password field`, `Submit button`, `text box`
- Steps contain specific URLs or page paths: `visit "/login"`, `redirected to "/dashboard"`
- Scenarios exceed 6-8 steps, each being a low-level interaction
- Scenarios read like an "operation manual" or "user guide" rather than a "behavior specification"
- Scenarios need frequent modification when the UI changes

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Fragility** | UI minor adjustments (e.g., button renaming, field reordering) cause scenarios to break |
| **High Maintenance Cost** | Every interface redesign requires batch modification of Gherkin files |
| **Poor Readability** | Business stakeholders struggle to understand the actual behavior being verified from lengthy steps |
| **Collaboration Barrier** | Non-technical people cannot understand technically-oriented scenario descriptions |
| **Cannot Serve as Living Documentation** | Scenarios become test scripts, losing their value as a shared understanding vehicle |

### Fix Solutions

1. **Adopt Declarative Style**: Each step conveys a **business concept**, with specific values left unspecified
2. **Raise Abstraction Level**: Encapsulate UI interaction details into the Step Definition layer
3. **Self-Test Question**: Ask yourself "If the implementation changed (e.g., from web to voice interface), would this text need to change?" If the answer is "yes", it needs refactoring
4. **Focus on Business Value**: Scenarios should answer "What does the user want to accomplish?" rather than "How to interact with the interface"

### Corresponding Correct Principles

- **G-01**: Describe behavior, not implementation
- **G-02**: Adopt Declarative style
- **G-04**: Scenarios should focus on a single business behavior
- **G-07**: Use business language, not technical terminology

### Code Examples

**Before Fix (Imperative - Not Recommended):**

```gherkin
Feature: Subscribers see different articles based on their subscription level

  Scenario: Free subscribers see only the free articles
    Given I am on the login page
    When I type "freeFrieda@example.com" in the email field
    And I type "validPassword123" in the password field
    And I press the "Submit" button
    Then I see "FreeArticle1" on the home page
    And I do not see "PaidArticle1" on the home page
```

**After Fix (Declarative - Recommended):**

```gherkin
Feature: Subscribers see different articles based on their subscription level

  Scenario: Free subscribers see only the free articles
    Given Free Frieda has a free subscription
    When Free Frieda logs in with her valid credentials
    Then she sees a Free article

  Scenario: Paid subscribers see both types of articles
    Given Paid Patty has a basic-level paid subscription
    When Paid Patty logs in with her valid credentials
    Then she sees a Free article and a Paid article
```

**Fixed Step Definition (Java):**

```java
@Given("{word} has a free subscription")
public void userHasFreeSubscription(String userName) {
    // Create a free subscription user via API or direct database operation
    userService.createUserWithSubscription(userName, SubscriptionType.FREE);
}

@When("{word} logs in with her valid credentials")
public void userLogsIn(String userName) {
    // UI interaction details hidden in Page Object
    UserCredentials creds = userService.getCredentials(userName);
    loginPage.login(creds.getEmail(), creds.getPassword());
}

@Then("she sees a Free article")
public void seesFreeArticle() {
    assertTrue(articlePage.isFreeArticleVisible());
    assertFalse(articlePage.isPaidArticleVisible());
}
```

---

## <a id="ap2-feature-coupled"></a>AP-02: Feature-coupled Step Definitions

> **One-line description**: Step definition files are organized one-to-one with Feature files, preventing steps from being reused across scenarios.

### Symptoms (How to Identify)

- Directory structure contains Steps files that correspond one-to-one with Feature files:
  ```
  features/
    edit_work_experience.feature
    edit_languages.feature
    edit_education.feature
    steps/
      edit_work_experience_steps.java
      edit_languages_steps.java
      edit_education_steps.java
  ```
- Nearly identical code appears across multiple Step Definition files (e.g., login, logout steps)
- Modifying a common step requires synchronized changes across multiple files
- Step Definition file names highly correspond to Feature file names

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Step Definition Explosion** | As Features increase, Step Definition files grow linearly |
| **Code Duplication** | Common steps (e.g., login, navigation) are repeated in multiple places |
| **Difficult Maintenance** | Modifying common behavior requires changing multiple files, easily leading to omissions |
| **Low Reusability** | Cannot share step definitions across Features |
| **Violates DRY Principle** | The same logic appears multiple times in different files |

### Fix Solutions

1. **Organize by Domain Concept**: Group step definitions by **business domain/concept** rather than by Feature
2. **Extract Common Steps**: Extract steps shared across Features (e.g., authentication, navigation, common assertions) into independent Steps classes
3. **Naming Convention**: Use domain-related names (e.g., `authentication.steps.java`, `cart.steps.java`) rather than Feature-related names
4. **Introduce Shared Libraries**: Use Hooks, World objects, or DI containers to share state between Steps

### Corresponding Correct Principles

- **G-06**: Keep step definitions reusable
- **G-11**: Organize project structure by domain concept
- **G-05**: Follow the DRY principle

### Code Examples

**Before Fix (Feature-coupled - Not Recommended):**

```
features/
  login.feature
  checkout.feature
  order_history.feature
step_definitions/
  LoginSteps.java       # Contains login logic
  CheckoutSteps.java    # Contains another set of login logic (duplicate!)
  OrderHistorySteps.java # Contains login logic yet again (duplicate!)
```

```java
// CheckoutSteps.java - Duplicate login steps
@Given("the user is logged in")
public void userIsLoggedIn() {
    driver.get("/login");
    driver.findElement(By.id("email")).sendKeys("user@test.com");
    driver.findElement(By.id("password")).sendKeys("password");
    driver.findElement(By.id("login-btn")).click();
}
```

**After Fix (Domain-organized - Recommended):**

```
features/
  login.feature
  checkout.feature
  order_history.feature
step_definitions/
  AuthenticationSteps.java  # All authentication-related steps
  CartSteps.java            # Shopping cart-related steps
  NavigationSteps.java      # Navigation-related steps
  OrderSteps.java           # Order-related steps
```

```java
// AuthenticationSteps.java - Unified authentication steps
public class AuthenticationSteps {
    private final TestContext context;
    private final LoginPage loginPage;

    public AuthenticationSteps(TestContext context) {
        this.context = context;
        this.loginPage = new LoginPage(context.getDriver());
    }

    @Given("{word} has logged in with valid credentials")
    public void hasLoggedIn(String userName) {
        UserCredentials creds = context.getCredentials(userName);
        loginPage.login(creds.getEmail(), creds.getPassword());
    }

    @Given("the user is not authenticated")
    public void userIsNotAuthenticated() {
        context.clearSession();
    }
}
```

---

## <a id="ap3-conjunction"></a>AP-03: Conjunction Steps (And/But Overuse / Conjunction Steps)

> **One-line description**: A step text contains conjunctions like "and", "or", effectively cramming multiple operations into a single step.

### Symptoms (How to Identify)

- Gherkin steps contain multiple actions connected by `and`, `or`, commas:
  ```gherkin
  Given I have shades and a brand new Mustang
  When the user enters credentials and clicks login
  ```
- A Step Definition method performs multiple unrelated operations
- Steps are overly specialized and cannot be reused in other scenarios
- Step descriptions read like long sentences rather than atomic actions

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Steps Difficult to Reuse** | Combined steps are too specialized for use in other scenarios |
| **Unclear Intent** | One step doing multiple things makes it hard to pinpoint issues when failing |
| **Violates Atomicity** | Good steps should be atomic (do only one thing) |
| **Vague Reports** | Failure reports cannot precisely locate "which step" caused the problem |
| **Underutilizes Gherkin Structure** | Cucumber's built-in `And`/`But` exists precisely to split combined actions |

### Fix Solutions

1. **Split Conjunction Steps**: Break combined steps into multiple independent atomic steps, connected with `And` or `But`
2. **Maintain Step Atomicity**: Each step does only one thing
3. **Extract Helper Methods**: If multiple steps need to be combined, extract independent helper methods in the Step Definition while keeping independent steps at the Gherkin level
4. **Use Background**: If multiple scenarios share a set of Given steps, use Background to extract the common parts

### Corresponding Correct Principles

- **G-03**: Each step describes only one action
- **G-04**: Scenarios focus on a single business behavior
- **G-06**: Keep step definitions reusable
- **G-14**: Each scenario should have 3-5 steps

### Code Examples

**Before Fix (Conjunction Steps - Not Recommended):**

```gherkin
Scenario: Purchase with discount
  Given I have a shopping cart with 3 items and a valid discount code
  When I apply the discount code and proceed to checkout
  Then the discount should be applied and the total should be updated
```

```java
@Given("I have a shopping cart with {int} items and a valid discount code")
public void setupCartWithDiscount(int itemCount) {
    // Two things mixed together: adding items + obtaining discount code
    for (int i = 0; i < itemCount; i++) {
        catalogPage.addItemToCart("item-" + i);
    }
    String discountCode = promoService.generateDiscountCode("SAVE20");
    context.setDiscountCode(discountCode);
}
```

**After Fix (Atomic Steps - Recommended):**

```gherkin
Scenario: Purchase with discount
  Given I have a shopping cart with 3 items
  And I have a valid discount code "SAVE20"
  When I apply the discount code
  And I proceed to checkout
  Then the discount should be applied
  And the total should be reduced by 20%
```

```java
@Given("I have a shopping cart with {int} items")
public void setupCart(int itemCount) {
    for (int i = 0; i < itemCount; i++) {
        catalogPage.addItemToCart("item-" + i);
    }
}

@Given("I have a valid discount code {string}")
public void haveDiscountCode(String code) {
    String discountCode = promoService.generateDiscountCode(code);
    context.setDiscountCode(discountCode);
}

@When("I apply the discount code")
public void applyDiscount() {
    checkoutPage.applyDiscountCode(context.getDiscountCode());
}

@Then("the discount should be applied")
public void verifyDiscountApplied() {
    assertTrue(checkoutPage.isDiscountApplied());
}

@Then("the total should be reduced by {int}%")
public void verifyDiscountAmount(int percentage) {
    BigDecimal expectedDiscount = context.getOriginalTotal()
        .multiply(BigDecimal.valueOf(percentage / 100.0));
    assertEquals(expectedDiscount, checkoutPage.getDiscountAmount());
}
```

---

## <a id="ap4-ui-centric"></a>AP-04: UI-centric Gherkin

> **One-line description**: Gherkin scenarios expose UI implementation details such as button IDs, CSS class names, field names, and specific URLs.

### Symptoms (How to Identify)

- Steps contain specific UI element identifiers:
  ```gherkin
  When I type "user@example.com" into input field with id "email-input"
  And I type "password" into input field with id "password-input"
  And I click button with class "btn-submit"
  ```
- Steps contain specific URLs:
  ```gherkin
  Given I am on "https://example.com/login"
  Then I should be redirected to "https://example.com/dashboard"
  ```
- Steps contain CSS selectors:
  ```gherkin
  Then I should see div#dashboard-header with color #333
  ```
- Business stakeholders cannot understand what is being verified from the steps

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **UI Change = Breakage** | Button ID changes, CSS class name adjustments cause scenarios to fail |
| **Not Business-Readable** | Business stakeholders cannot understand what `#email-input` is |
| **Violates Abstraction Level** | Gherkin should be at the business layer; UI details belong in the automation layer |
| **Cannot Serve as Living Documentation** | Scenarios become UI test scripts, losing their specification function |
| **Multiple Maintenance Burden** | Front-end refactoring requires synchronously updating large numbers of scenario files |

### Fix Solutions

1. **Elevate to Business Behavior Layer**: Replace UI element references with business concepts
2. **Push UI Details Downward**: Encapsulate element locators, selectors, etc. into Page Object or Screenplay Task
3. **Use Business Role Language**: Describe behavior from the user perspective, not from the developer perspective describing the interface
4. **Separate Concerns**: Gherkin = Business Specification; Step Def = Automation Bridge; Page Object = UI Interaction

### Corresponding Correct Principles

- **G-01**: Describe behavior, not implementation
- **G-02**: Adopt Declarative style
- **G-07**: Use business language, not technical terminology
- **G-10**: UI details belong in the automation layer and should not appear in Gherkin

### Code Examples

**Before Fix (UI-centric - Not Recommended):**

```gherkin
Feature: User Login

  Scenario: Login with valid credentials
    Given I am on "https://app.example.com/login"
    When I type "john@example.com" into the field with id "email"
    And I type "SecurePass123" into the field with id "password"
    And I click the button with id "login-btn"
    Then I should be redirected to "/dashboard"
    And the element with id "welcome-msg" should contain "Welcome, John"
```

**After Fix (Business-centric - Recommended):**

```gherkin
Feature: User Login
  As a registered user
  I want to log in with my credentials
  So that I can access my dashboard

  Scenario: Login with valid credentials
    Given John has a registered account
    When John logs in with his credentials
    Then he should be redirected to the dashboard
    And he should see a welcome message with his name
```

**UI Interaction Encapsulated in Page Object:**

```java
// LoginPage.java - UI details encapsulated here
public class LoginPage {
    @FindBy(id = "email")           // UI locator appears only once
    private WebElement emailField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(id = "login-btn")
    private WebElement loginButton;

    public void login(String email, String password) {
        emailField.sendKeys(email);
        passwordField.sendKeys(password);
        loginButton.click();
    }
}

// LoginSteps.java - Step Definition stays concise
public class LoginSteps {
    private final LoginPage loginPage;
    private final DashboardPage dashboardPage;
    private final TestContext context;

    public LoginSteps(TestContext context) {
        this.context = context;
        this.loginPage = new LoginPage(context.getDriver());
        this.dashboardPage = new DashboardPage(context.getDriver());
    }

    @When("{word} logs in with his credentials")
    public void userLogsIn(String userName) {
        UserCredentials creds = context.getCredentials(userName);
        loginPage.login(creds.getEmail(), creds.getPassword());
    }

    @Then("he should be redirected to the dashboard")
    public void shouldSeeDashboard() {
        assertTrue(dashboardPage.isDisplayed());
        assertTrue(dashboardPage.getCurrentUrl().contains("/dashboard"));
    }
}
```

---

## <a id="ap5-god-step"></a>AP-05: God Step Definition

> **One-line description**: A Step Definition method takes on too many responsibilities, containing large amounts of business logic, data processing, UI operations, and assertions.

### Symptoms (How to Identify)

- A Step Definition method exceeds 30-50 lines
- Method contains multiple `if/else` branches, loops, exception handling
- Method simultaneously handles: data preparation, UI operations, database validation, external API calls
- Step Definition class becomes abnormally large (hundreds or even thousands of lines)
- Method name cannot clearly describe its complete responsibility
- A large amount of "magic" logic is hidden behind a single step

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Poor Readability** | Massive methods are difficult to understand and review |
| **Difficult Debugging** | When a step fails, it's hard to locate the specific cause |
| **Violates Single Responsibility** | One method doing multiple things, violating the SRP principle |
| **Low Reusability** | Logic is coupled in a specific step and cannot be reused by other steps |
| **Maintenance Nightmare** | Any change requires understanding and modifying a massive method |
| **Difficult to Test** | Step Definitions themselves are difficult to unit test |

### Fix Solutions

1. **Extract Helper Methods**: Split large methods into multiple private helper methods, each doing one thing
2. **Introduce Service Layer**: Extract business logic into domain service classes
3. **Use Page Object**: Encapsulate UI operations into Page Objects
4. **Use Dependency Injection**: Inject required services through DI containers, keeping Steps classes lean
5. **Follow the 10-Line Rule**: Try to keep each Step Definition method under 10 lines
6. **Extract Utility Classes**: Extract general data processing, waiting logic into utility classes

### Corresponding Correct Principles

- **G-05**: Follow the DRY principle
- **G-08**: Keep step definitions concise
- **G-11**: Organize code by domain concept
- **G-15**: Follow the Single Responsibility Principle

### Code Examples

**Before Fix (God Step - Not Recommended):**

```java
@When("the user completes checkout with {string}")
public void completeCheckout(String paymentMethod) {
    // Data preparation
    User user = context.getCurrentUser();
    List<CartItem> items = cartService.getItems(user.getId());
    if (items.isEmpty()) {
        throw new IllegalStateException("Cart is empty");
    }

    // Calculate price
    BigDecimal subtotal = BigDecimal.ZERO;
    for (CartItem item : items) {
        subtotal = subtotal.add(item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())));
    }
    BigDecimal tax = subtotal.multiply(BigDecimal.valueOf(0.08));
    BigDecimal total = subtotal.add(tax);

    // UI operations
    checkoutPage.enterShippingAddress(user.getAddress());
    checkoutPage.selectPaymentMethod(paymentMethod);
    checkoutPage.enterCardDetails(user.getCardNumber(), user.getExpiry(), user.getCvv());
    checkoutPage.clickPlaceOrder();

    // Wait for processing
    try {
        Thread.sleep(3000); // Hard-coded wait!
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    // Database validation
    Order order = orderRepository.findLatestOrderByUser(user.getId());
    assertNotNull(order);
    assertEquals(total.doubleValue(), order.getTotalAmount(), 0.01);

    // Send notification
    emailService.sendOrderConfirmation(user.getEmail(), order.getId());
    context.setCurrentOrder(order);
}
```

**After Fix (Responsibility Separation - Recommended):**

```java
// CheckoutSteps.java - Lean Step Definition
public class CheckoutSteps {
    private final CheckoutService checkoutService;
    private final CheckoutPage checkoutPage;
    private final OrderAssertions orderAssertions;
    private final TestContext context;

    public CheckoutSteps(TestContext context, CheckoutService checkoutService) {
        this.context = context;
        this.checkoutService = checkoutService;
        this.checkoutPage = new CheckoutPage(context.getDriver());
        this.orderAssertions = new OrderAssertions();
    }

    @When("the user completes checkout with {string}")
    public void completeCheckout(String paymentMethod) {
        OrderSummary summary = checkoutService.prepareOrder(context.getCurrentUser());
        checkoutPage.completeCheckout(summary, paymentMethod);
    }

    @Then("the order total should include 8% tax")
    public void verifyTaxCalculation() {
        Order order = context.getCurrentOrder();
        orderAssertions.assertTaxApplied(order, new BigDecimal("0.08"));
    }

    @Then("an order confirmation should be sent")
    public void verifyConfirmationEmail() {
        orderAssertions.assertConfirmationSent(context.getCurrentOrder());
    }
}

// CheckoutService.java - Business logic layer
public class CheckoutService {
    public OrderSummary prepareOrder(User user) {
        List<CartItem> items = cartService.getItems(user.getId());
        validateCartNotEmpty(items);
        return calculateOrderSummary(items);
    }

    private void validateCartNotEmpty(List<CartItem> items) {
        if (items.isEmpty()) {
            throw new IllegalStateException("Cart is empty");
        }
    }

    private OrderSummary calculateOrderSummary(List<CartItem> items) {
        BigDecimal subtotal = items.stream()
            .map(i -> i.getPrice().multiply(BigDecimal.valueOf(i.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        BigDecimal tax = subtotal.multiply(new BigDecimal("0.08"));
        return new OrderSummary(subtotal, tax, subtotal.add(tax));
    }
}

// CheckoutPage.java - UI operation layer
public class CheckoutPage {
    public void completeCheckout(OrderSummary summary, String paymentMethod) {
        enterShippingAddress();
        selectPaymentMethod(paymentMethod);
        confirmPaymentDetails();
        submitOrder();
        waitForOrderConfirmation();
    }
}
```

---

## <a id="ap6-brittle-selectors"></a>AP-06: Brittle Selectors

> **One-line description**: Using easily-changed UI selectors (such as absolute XPath, dynamically generated IDs, position-based indexes) to locate elements in automation code.

### Symptoms (How to Identify)

- Using absolute XPath:
  ```java
  @FindBy(xpath = "/html/body/div[2]/div[1]/div[3]/form/input[1]")
  ```
- Relying on dynamically generated IDs:
  ```java
  @FindBy(id = "ext-gen-472")  // Auto-generated ID that may change with each deployment
  ```
- Using index/position-based selectors:
  ```java
  @FindBy(css = ".item:nth-child(3)")
  ```
- Relying on specific CSS class names (often used for styling rather than functional identification):
  ```java
  @FindBy(className = "btn-primary-red-gradient")
  ```
- Using `contains` with unstable text content
- Selectors coupled with front-end framework (React, Vue, Angular) internal implementations

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Extremely Fragile** | Page structure adjustments, front-end framework upgrades cause test failures |
| **High Maintenance Cost** | Selectors need frequent updates |
| **Unreliable** | Unstable in dynamic content, asynchronous loading scenarios |
| **Slow Execution** | Complex XPath or deep DOM traversal is time-consuming |
| **Tightly Coupled with Front-End** | Test code cannot be independent of the front-end technology stack |

### Fix Solutions

1. **Use Stable Attributes**: Prefer test-specific attributes like `data-testid`, `data-cy`, `data-qa`
2. **Use Semantic Selectors**: Locate based on element role and ARIA attributes (e.g., `getByRole`, `getByLabelText`)
3. **Use Relative XPath**: Build stable paths based on element relationships and text content
4. **Collaborate with Front-End Team**: Agree to add stable `data-*` attributes for automation
5. **Use Page Object Encapsulation**: Centralize all selectors in Page Objects, changes only need to be made in one place
6. **Adopt Modern Location Strategies**: Such as Playwright's `getByTestId`, `getByRole`, `getByLabel`, etc.

### Corresponding Correct Principles

- **G-10**: UI details belong in the automation layer
- **G-13**: Use stable element location strategies
- **G-15**: Encapsulate points of change

### Code Examples

**Before Fix (Brittle Selectors - Not Recommended):**

```java
public class ProductPage {
    // Absolute XPath, extremely fragile
    @FindBy(xpath = "/html/body/div[2]/main/div[1]/div[3]/button")
    private WebElement addToCartButton;

    // Dynamically generated ID, unreliable
    @FindBy(id = "react-root-42-abc")
    private WebElement productTitle;

    // CSS class name for styling, may change frequently
    @FindBy(css = ".MuiButtonBase-root.contained.primary")
    private WebElement submitButton;

    // Index-based, fails when data changes
    @FindBy(css = ".product-item:nth-child(3) .price")
    private WebElement thirdProductPrice;
}
```

**After Fix (Stable Selectors - Recommended):**

```java
public class ProductPage {
    // Use data-testid (stable identifier agreed upon with the front-end team)
    @FindBy(css = "[data-testid='add-to-cart-button']")
    private WebElement addToCartButton;

    // Use semantic role location
    @FindBy(css = "[role='heading'][aria-level='1']")
    private WebElement productTitle;

    // Use data-* attributes
    @FindBy(css = "[data-testid='submit-order-button']")
    private WebElement submitButton;

    // Locate based on text content (insensitive to data changes)
    @FindBy(xpath = "//div[contains(@data-testid, 'product-') and .//h3[text()='%s']]//span[@data-testid='price']")
    private WebElement productPriceByName;
}
```

**Front-End Collaboration (HTML Markup Example):**

```html
<!-- Good practice: add data-testid -->
<button data-testid="add-to-cart-button"
        class="MuiButtonBase-root contained primary">
  Add to Cart
</button>

<h1 role="heading" aria-level="1" data-testid="product-title">
  iPhone 15 Pro
</h1>

<!-- List items also have stable identifiers -->
<div data-testid="product-item" data-product-id="IPHONE-15-PRO">
  <h3 data-testid="product-name">iPhone 15 Pro</h3>
  <span data-testid="product-price">$999</span>
  <button data-testid="add-to-cart">Add to Cart</button>
</div>
```

**Using Playwright's Modern Location Strategy (More Recommended):**

```typescript
// Playwright built-in locators - based on user-visible characteristics
const addToCartButton = page.getByTestId('add-to-cart-button');
const submitButton = page.getByRole('button', { name: 'Submit Order' });
const emailInput = page.getByLabel('Email Address');
const productTitle = page.getByRole('heading', { name: 'iPhone 15 Pro' });

// These locators are resilient to changes from the user perspective
await page.getByRole('link', { name: 'Checkout' }).click();
await expect(page.getByText('Order Confirmed')).toBeVisible();
```


---

## <a id="ap7-through-ui"></a>AP-07: Testing Through the UI

> **One-line description**: All business behavior is verified through end-to-end UI layer testing, ignoring opportunities for testing at lower levels.

### Symptoms (How to Identify)

- Project has almost no API layer tests or domain layer unit tests
- All scenarios open a real browser and operate real UI
- Test suite execution time is extremely long (hours)
- Test failures are mainly due to UI rendering timing, browser compatibility, and other unrelated issues
- Even testing pure business rules (e.g., price calculation, discount logic) goes through the complete UI flow
- Test environment requires complete front-end and back-end deployment to run
- Test data preparation also relies on UI operations (e.g., creating test data through web forms)

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Slow Execution** | UI tests are 10-100x slower than API tests, 1000x+ slower than unit tests |
| **Flaky** | UI tests are affected by network latency, rendering timing, browser state, and other factors |
| **Delayed Feedback** | Developers cannot get fast feedback, violating the rapid iteration principle |
| **Difficult Debugging** | When UI fails, it's hard to determine whether it's a UI bug, business logic bug, or environment issue |
| **High Resource Consumption** | Requires Selenium Grid, browser instances, and other extensive infrastructure |
| **Insufficient Coverage** | Complex business rule boundary conditions are difficult to exhaustively test through the UI |

### Fix Solutions

1. **Test Pyramid Strategy**: Put most tests at the lower level (unit tests, integration tests), with only a small number of critical paths going through UI
2. **API Layer BDD**: Use the same Gherkin scenarios, but Step Definitions call APIs instead of operating browsers
3. **Layered Testing**:
   - **Unit Tests**: Pure business logic, algorithms, boundary conditions
   - **API/Integration Tests**: End-to-end business processes without going through UI
   - **UI Tests**: Only verify user-visible end-to-end scenarios and critical paths
4. **Use Test Doubles**: Mock external dependencies to isolate the system under test
5. **Choose Appropriate Test Entry Points**: Select the most efficient test level based on business rules

### Corresponding Correct Principles

- **G-09**: Choose the appropriate test level
- **G-12**: Test the right behavior at the right level
- **G-16**: Follow the test pyramid
- **G-17**: Prefer API/service layer tests for verifying business rules

### Code Examples

**Before Fix (All Through UI - Not Recommended):**

```gherkin
# All scenarios open browser and operate UI
Feature: Discount Calculation

  Scenario: VIP customer gets 20% discount
    Given I open the homepage
    And I click "Sign In"
    And I enter email "vip@example.com"
    And I enter password "test123"
    And I click "Login"
    And I navigate to "Products"
    And I add "Laptop" to cart
    And I go to cart
    When I apply discount code "VIP20"
    Then the total should show "$800"  # Original price $1000 - 20%
    # This scenario takes 30+ seconds and may fail due to UI timing
```

**After Fix (Layered Testing - Recommended):**

**Option A: UI Layer (Only verify critical user paths)**

```gherkin
Feature: Checkout Flow

  @ui @smoke
  Scenario: Customer completes purchase with discount
    Given John has a VIP account
    And he has added a Laptop to his cart
    When he proceeds to checkout
    And he applies discount code "VIP20"
    Then he should see the discounted total of "$800"
    And he should be able to complete the order
    # Only verify the UI experience of critical paths
```

**Option B: API Layer (Verify business rules - faster, more stable)**

```gherkin
Feature: Discount Rules

  @api
  Scenario Outline: Discount calculation for different customer tiers
    Given a <tier> customer with a cart total of $<total>
    When the discount is calculated
    Then the discount should be $<discount>
    And the final total should be $<final>

    Examples:
      | tier     | total | discount | final |
      | VIP      | 1000  | 200      | 800   |
      | VIP      | 500   | 100      | 400   |
      | Standard | 1000  | 50       | 950   |
      | Standard | 500   | 25       | 475   |
      | New      | 1000  | 0        | 1000  |
    # 30+ boundary conditions completed in <1 second
```

```java
// API Step Definition
public class DiscountApiSteps {
    private final DiscountService discountService;
    private DiscountResult result;

    public DiscountApiSteps(DiscountService discountService) {
        this.discoutService = discountService;
    }

    @Given("a {word} customer with a cart total of ${bigdecimal}")
    public void setupCustomer(String tier, BigDecimal total) {
        CustomerTier customerTier = CustomerTier.valueOf(tier.toUpperCase());
        Cart cart = new Cart(total, customerTier);
        context.setCart(cart);
    }

    @When("the discount is calculated")
    public void calculateDiscount() {
        result = discountService.calculateDiscount(context.getCart());
    }

    @Then("the discount should be ${bigdecimal}")
    public void verifyDiscount(BigDecimal expectedDiscount) {
        assertEquals(0, expectedDiscount.compareTo(result.getDiscountAmount()));
    }

    @Then("the final total should be ${bigdecimal}")
    public void verifyFinalTotal(BigDecimal expectedTotal) {
        assertEquals(0, expectedTotal.compareTo(result.getFinalTotal()));
    }
}
```

---

## <a id="ap8-no-ubiquitous-language"></a>AP-08: Lack of Ubiquitous Language

> **One-line description**: Scenarios use technical terminology, developer jargon, and inconsistent business vocabulary rather than the team's collectively defined business language.

### Symptoms (How to Identify)

- Technical terminology appears in scenarios:
  ```gherkin
  When I send a POST request to /api/v1/auth
  Then the JSON response should contain "$.name"
  And the users table should have role='ADMIN'
  ```
- The same business concept uses different names in different scenarios:
  ```gherkin
  # Scenario A
  Given a premium subscriber exists
  # Scenario B
  Given a VIP customer has an account
  # Scenario C
  Given a gold member is registered
  # These three may refer to the same concept!
  ```
- Developer jargon is used: `DTO`, `payload`, `middleware`, `cache`, `serializer`
- Database terminology is used: `users table`, `foreign key`, `column`, `record`
- Business stakeholders need translation to understand scenarios
- New team members don't understand the terminology in scenarios

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Collaboration Barrier** | Business stakeholders cannot understand scenarios, losing the core value of BDD |
| **Terminology Ambiguity** | One concept with multiple names leads to communication chaos and false assumptions |
| **Difficult Maintenance** | Inconsistent terminology causes duplicate step definitions and scenarios that are hard to find |
| **Violates Living Documentation** | Scenarios cannot serve as business specification documents |
| **Tests Detached from Business** | Scenarios verify technical implementation rather than business behavior |

### Fix Solutions

1. **Establish a Glossary**: The team collectively defines and records a business terminology table, used uniformly in all scenarios
2. **Conduct Three Amigos Meetings**: Business, development, and testing jointly determine the ubiquitous language
3. **Use Example Mapping**: Unify terminology through card-based discussions
4. **Review Terminology in Code Reviews**: Check terminology consistency during scenario reviews
5. **DDD Context Mapping**: Introduce the Bounded Context concept from Domain-Driven Design
6. **Naming Convention**: Determine a unique name for each business concept and use it consistently across all scenarios

### Corresponding Correct Principles

- **G-07**: Use business language (ubiquitous language)
- **G-18**: Establish and maintain a team glossary
- **G-19**: Business stakeholders can understand scenarios

### Code Examples

**Before Fix (Terminology Chaos - Not Recommended):**

```gherkin
# Three different terminology systems used in the same business domain
Feature: Purchase Flow

  Scenario: Premium user buys with discount
    Given a premium subscriber exists in the users table
    When the customer sends a POST to /api/checkout
    And the DTO contains discount code "VIP20"
    Then the serializer should output final price as 800
    And the cache should invalidate

  Scenario: VIP member checkout
    Given a gold member is registered in the DB
    When the client sends a checkout payload
    Then the response JSON should have total = 800
    And Redis cache is cleared

  Scenario: Elite customer purchase
    Given an elite user exists
    When the frontend submits the order form
    Then the API returns price 800
    And the session cache refreshes
```

**After Fix (Unified Ubiquitous Language - Recommended):**

```gherkin
# Team glossary definitions:
# - Customer tiers: Free, Premium, VIP, Enterprise
# - Purchase actions: "places an order", "checks out"
# - Discount verification: "discount is applied", "final total reflects discount"

Feature: Purchase Flow

  Scenario: Premium customer places an order with discount
    Given Maria is a Premium customer
    When she places an order for a Laptop with discount code "VIP20"
    Then the discount of 20% should be applied
    And her final total should be $800

  Scenario: VIP customer checks out
    Given John is a VIP customer
    When he checks out with a Laptop in his cart
    Then the VIP discount should be automatically applied
    And his final total should be $800

  Scenario: Enterprise customer places an order
    Given Acme Corp is an Enterprise customer
    When they place an order for 10 Laptops
    Then the enterprise discount of 25% should be applied
    And their final total should be $7,500
```

**Team Glossary (Example):**

```markdown
# E-commerce Ubiquitous Language Glossary

| Term (Term)       | Synonyms (Synonyms)       | Description                    |
|-------------------|---------------------------|--------------------------------|
| Customer          | User, Account Holder      | Registered user of the system  |
| Premium           | Gold, Elite               | Paid membership tier           |
| VIP               | Platinum                  | Premium paid membership tier   |
| Enterprise        | Business, Corporate       | Enterprise customer tier       |
| Places an order   | Purchases, Buys           | Initiates purchase flow        |
| Checks out        | Completes purchase        | Completes shopping cart checkout|
| Cart              | Basket, Shopping Bag      | Virtual shopping cart          |
| Discount code     | Promo code, Coupon        | Discount code / coupon         |
| Final total       | Grand total, Amount due   | Final amount due after discount|
```

---

## <a id="ap9-scenario-dependence"></a>AP-09: Scenario Dependence

> **One-line description**: The execution of a scenario depends on other scenarios having been executed and left state behind; scenarios are not independent.

### Symptoms (How to Identify)

- Scenario A creates data, Scenario B assumes that data already exists:
  ```gherkin
  Scenario: Create a product
    When I create a product named "Laptop"
    Then the product should exist

  Scenario: Add product to cart
    # Implicit assumption: "Laptop" was created in the scenario above
    When I add "Laptop" to my cart
    Then the cart should contain 1 item
  ```
- Using `@order` tags to force scenarios to execute in a specific order
- Scenario failure messages show "data not found", but running the scenario alone passes
- Test suite needs to run in a specific order to pass entirely
- One scenario fails, causing a large number of subsequent scenarios to cascade fail
- Scenarios share database state without cleanup
- Execution order between scenarios affects results

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Cascade Failures** | A prerequisite scenario failure causes multiple subsequent scenarios to fail |
| **Difficult Debugging** | Cannot run a failing scenario alone to locate the problem |
| **Parallelism Blocked** | Scenario dependencies make parallel execution impossible |
| **Unreliable** | Changes in execution order may cause test failures |
| **Violates Independence** | Each scenario should be an independent test unit |
| **Difficult Maintenance** | Modifying one scenario may break other scenarios that depend on it |

### Fix Solutions

1. **Each Scenario Self-Contained**: Each scenario sets its own Given preconditions, not depending on other scenarios
2. **Use Background**: Extract shared Given steps into Background
3. **Data Preparation Steps**: Explicitly create required data in Given
4. **Test Data Factory**: Use Factory/Builder patterns to quickly create test data in Given
5. **Hooks Cleanup**: Use `@After` Hook to ensure data cleanup after scenario ends
6. **World State Isolation**: Use Cucumber's World object to ensure scenarios don't share state

### Corresponding Correct Principles

- **G-20**: Scenarios must be completely independent
- **G-21**: Scenario execution order should not affect results
- **G-22**: Each scenario sets its own preconditions

### Code Examples

**Before Fix (Scenario Dependence - Not Recommended):**

```gherkin
Feature: Shopping Cart

  # Scenario A: Create data
  @order(1)
  Scenario: Create a product
    When I create a product named "Laptop" priced at $1000
    Then the product should be available

  # Scenario B: Depends on data created in Scenario A
  @order(2)
  Scenario: Add product to cart
    When I add "Laptop" to my cart  # Depends on the product above already existing!
    Then the cart should contain 1 item

  # Scenario C: Depends on Scenario B's cart state
  @order(3)
  Scenario: Apply discount
    When I apply discount code "SAVE10"  # Depends on cart already having items!
    Then the total should be $900
```

**After Fix (Independent Scenarios - Recommended):**

```gherkin
Feature: Shopping Cart

  Background:
    Given a product "Laptop" priced at $1000 exists

  Scenario: Add product to cart
    Given I have an empty cart
    When I add "Laptop" to my cart
    Then the cart should contain 1 item
    And the cart total should be $1000

  Scenario: Apply discount to cart
    Given I have "Laptop" in my cart
    When I apply discount code "SAVE10"
    Then the discount should be $100
    And the cart total should be $900

  Scenario: Checkout with multiple items
    Given I have "Laptop" priced at $1000 in my cart
    And I have "Mouse" priced at $50 in my cart
    When I proceed to checkout
    Then the subtotal should be $1050
```

```java
// Use data factory to create test data
public class CartSteps {
    private final ProductFactory productFactory;
    private final CartService cartService;
    private final TestContext context;

    @Given("a product {string} priced at ${bigdecimal} exists")
    public void createProduct(String name, BigDecimal price) {
        Product product = productFactory.create(name, price);
        context.addProduct(product);
    }

    @Given("I have an empty cart")
    public void emptyCart() {
        Cart cart = cartService.createEmptyCart(context.getCurrentUser());
        context.setCart(cart);
    }

    @Given("I have {string} in my cart")
    public void addProductToCart(String productName) {
        Product product = context.findProduct(productName);
        Cart cart = cartService.addToCart(context.getCurrentUser(), product);
        context.setCart(cart);
    }

    @Given("I have {string} priced at ${bigdecimal} in my cart")
    public void addProductWithPrice(String productName, BigDecimal price) {
        Product product = productFactory.create(productName, price);
        context.addProduct(product);
        Cart cart = cartService.addToCart(context.getCurrentUser(), product);
        context.setCart(cart);
    }

    @After  // Ensure cleanup after each scenario
    public void cleanup() {
        cartService.clearCart(context.getCurrentUser());
    }
}

// ProductFactory - Test data factory
public class ProductFactory {
    public Product create(String name, BigDecimal price) {
        return productRepository.save(
            Product.builder()
                .name(name)
                .price(price)
                .sku(generateUniqueSku(name))
                .build()
        );
    }
}
```

---

## <a id="ap10-too-many-details"></a>AP-10: Too Many Details

> **One-line description**: Scenarios are stuffed with large amounts of details unrelated to the current behavior verification, obscuring the truly important information.

### Symptoms (How to Identify)

- Scenario steps exceed 6-8 lines, containing large amounts of unnecessary steps
- Given contains multiple setup steps unrelated to the current scenario
- Scenarios contain unnecessary data values (e.g., complete addresses, long text)
- Scenario Outline is used but the Examples table has dozens of rows
- Large numbers of steps in scenarios are just "navigate to page", "click menu" and other setup actions
- Readers cannot tell at a glance what the scenario is verifying
- Background exceeds 4 lines, containing too much shared setup

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Poor Readability** | Unrelated details drown out the core verification logic |
| **High Maintenance Cost** | More steps means more content to maintain |
| **Blurred Focus** | Unclear scenario objectives, violating "one scenario, one behavior" |
| **Not Business-Readable** | Business stakeholders cannot extract key information from lengthy steps |
| **Slow Execution** | Every extra step increases execution time |

### Fix Solutions

1. **Focus on Core Behavior**: Each scenario verifies only one business behavior, delete unrelated steps
2. **Streamline Given**: Given steps should be streamlined to only include necessary context setup
3. **Use Composite Steps**: Encapsulate multiple related setup steps into a high-level Given step
4. **Externalize Data**: Use Data Tables or Examples to separate data from scenario structure
5. **Delete Navigation Steps**: Set state directly through Given rather than through UI navigation
6. **Follow the 3-5 Step Rule**: Cucumber officially recommends 3-5 steps per scenario

### Corresponding Correct Principles

- **G-14**: Each scenario should have 3-5 steps
- **G-04**: Scenarios focus on a single business behavior
- **G-23**: Background should not exceed 4 lines
- **G-24**: Remove details unrelated to the verification objective

### Code Examples

**Before Fix (Too Many Details - Not Recommended):**

```gherkin
Feature: Order Management

  Background:
    Given the user opens the browser
    And the user navigates to "https://shop.example.com"
    And the user clicks on "Sign In" link
    And the user enters email "admin@example.com"
    And the user enters password "admin123"
    And the user clicks the "Sign In" button
    And the user waits for the dashboard to load
    And the user clicks on "Catalog" menu
    And the user clicks on "Products" submenu
    # Background already has 8 steps! Most are navigation

  Scenario: Admin views product details
    Given the above background steps are completed
    When the user clicks on product "Laptop" in the product list
    And the user waits for the product page to load
    And the user scrolls down to see product details
    Then the user should see product name "Laptop"
    And the user should see product price "$1000"
    And the user should see product description "High-performance laptop..."
    And the user should see product SKU "LAP-001"
    And the user should see product category "Electronics"
    And the user should see product status "In Stock"
    # 14+ step scenario, core verification is drowned out
```

**After Fix (Streamlined and Focused - Recommended):**

```gherkin
Feature: Product Management

  Background:
    Given an admin user is logged in
    # 1 composite step replaces 8 navigation steps

  Scenario: Admin views product details
    Given a product "Laptop" priced at $1000 exists
    When the admin views the product details for "Laptop"
    Then the product details should show:
      | Field       | Value                  |
      | Name        | Laptop                 |
      | Price       | $1000                  |
      | Category    | Electronics            |
      | Status      | In Stock               |
    # 4-step clear scenario, core verification is obvious at a glance

  Scenario: Admin updates product price
    Given a product "Laptop" priced at $1000 exists
    When the admin changes the price to $1200
    Then the product price should be updated to $1200
    # 3-step scenario, extremely clear
```

```java
// Use composite steps to encapsulate complex setup
@Given("an admin user is logged in")
public void adminIsLoggedIn() {
    // All navigation and login logic encapsulated in one method
    User admin = userFactory.createAdmin();
    loginPage.navigateTo();
    loginPage.login(admin.getEmail(), admin.getPassword());
    dashboardPage.waitForLoad();
    context.setCurrentUser(admin);
}

@Given("a product {string} priced at ${bigdecimal} exists")
public void createProduct(String name, BigDecimal price) {
    Product product = productFactory.create(name, price);
    context.addProduct(product);
}

@When("the admin views the product details for {string}")
public void viewProductDetails(String productName) {
    Product product = context.findProduct(productName);
    productPage.viewDetails(product.getId());
}

@Then("the product details should show:")
public void verifyProductDetails(DataTable expectedDetails) {
    Map<String, String> expected = expectedDetails.asMap();
    ProductDetails actual = productPage.getDetails();

    assertEquals(expected.get("Name"), actual.getName());
    assertEquals(expected.get("Price"), actual.getPrice());
    assertEquals(expected.get("Category"), actual.getCategory());
    assertEquals(expected.get("Status"), actual.getStatus());
}
```

---

## <a id="ap11-sleep-wait"></a>AP-11: Sleep/Wait Anti-pattern

> **One-line description**: Using `Thread.sleep()` or fixed-time waits in automation code rather than waiting for a specific condition to be satisfied.

### Symptoms (How to Identify)

- Code contains `Thread.sleep(3000)` or similar fixed waits
- Tests heavily use Implicit Wait
- Tests intermittently fail because "element hasn't loaded yet", "fixed" by increasing wait time
- Wait times are "empirical values" (e.g., 3 seconds, 5 seconds) rather than based on actual conditions
- Test execution time grows linearly with wait time
- Different environments (local/CI) require different wait times

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Slow Execution** | Fixed waits waste time, continuing to wait even when elements are ready |
| **Flaky** | Fixed waits may be insufficient during network fluctuations, causing test failures |
| **Environment Dependent** | Fast locally, slow in CI; the same wait time has different effects in different environments |
| **Difficult Maintenance** | Cannot determine the "correct" wait time, usually can only keep increasing |
| **Resource Waste** | CPU spins waiting, overall test suite time increases dramatically |
| **Masks Real Problems** | "Fixing" tests by increasing wait time may mask performance issues |

### Fix Solutions

1. **Explicit Wait**: Wait for a specific condition to occur (element visible, clickable, text appears, etc.)
2. **Polling Mechanism**: Check the condition at intervals, continue immediately when condition is met
3. **Timeout Protection**: Set maximum wait time to prevent infinite waiting
4. **Use Modern Framework Wait Mechanisms**:
   - Selenium: `WebDriverWait` + `ExpectedConditions`
   - Playwright: Built-in auto-wait, `expect().toBeVisible()`
   - Cypress: Automatic waiting for commands and assertions
5. **Wait for the Right Condition**: Wait for business conditions (e.g., "order status becomes confirmed") rather than UI conditions
6. **Avoid Implicit Waits**: Mixing implicit waits with explicit waits leads to unpredictable behavior

### Corresponding Correct Principles

- **G-25**: Use explicit wait instead of fixed wait
- **G-26**: Wait for conditions, not time
- **G-27**: Leverage framework built-in wait mechanisms

### Code Examples

**Before Fix (Fixed Wait - Not Recommended):**

```java
@When("the user submits the order")
public void submitOrder() {
    checkoutPage.clickSubmitOrder();

    // Fixed wait of 5 seconds - even if order is processed in 500ms, still waits 5 seconds
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    // If CI environment is slower than 5 seconds, test will still fail
    assertTrue(confirmationPage.isDisplayed());
}

@Then("the report should be generated")
public void verifyReportGenerated() {
    reportsPage.clickGenerateReport();

    // Fixed wait of 10 seconds - seriously slows down the test suite
    try {
        Thread.sleep(10000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    assertTrue(reportsPage.isReportReady());
}
```

**After Fix (Explicit Wait - Recommended):**

```java
// Java + Selenium explicit wait
@When("the user submits the order")
public void submitOrder() {
    checkoutPage.clickSubmitOrder();

    // Wait for confirmation page to appear, max 10 seconds, continue immediately when it appears
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until(ExpectedConditions.urlContains("/confirmation"));

    assertTrue(confirmationPage.isDisplayed());
}

@Then("the report should be generated")
public void verifyReportGenerated() {
    reportsPage.clickGenerateReport();

    // Wait for specific condition: report ready indicator is visible
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
    wait.until(ExpectedConditions.visibilityOfElementLocated(
        By.cssSelector("[data-testid='report-ready-indicator']")
    ));

    assertTrue(reportsPage.isReportReady());
}
```

```typescript
// Playwright - Built-in auto-wait (recommended)
When('the user submits the order', async () => {
    await checkoutPage.submitOrder();

    // Playwright automatically waits for element to appear
    await expect(page.getByTestId('order-confirmation'))
        .toBeVisible({ timeout: 10000 });  // Continue immediately when condition is met
});

Then('the report should be generated', async () => {
    await reportsPage.generateReport();

    // Wait for business condition: report ready
    await expect(page.getByTestId('report-status'))
        .toHaveText('Ready', { timeout: 30000 });
});
```

```java
// Custom wait condition (wait for business state)
public class BusinessConditionWait {
    public static void waitForOrderStatus(
            OrderService service,
            String orderId,
            OrderStatus expectedStatus,
            int timeoutSeconds) {

        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(timeoutSeconds));
        wait.until(driver -> {
            OrderStatus current = service.getOrderStatus(orderId);
            return current == expectedStatus;
        });
    }
}

// Usage
@Then("the order status should be {word}")
public void verifyOrderStatus(String expectedStatus) {
    BusinessConditionWait.waitForOrderStatus(
        orderService,
        context.getCurrentOrderId(),
        OrderStatus.valueOf(expectedStatus),
        30  // Wait up to 30 seconds
    );
}
```

---

## <a id="ap12-no-gwt"></a>AP-12: No Clear Given-When-Then

> **One-line description**: Given/When/Then steps in scenarios are out of order, multiple When or Then appear interchangeably, or necessary step types are missing.

### Symptoms (How to Identify)

- Scenario starts with `When`, with no `Given` to set context
- Multiple `When` steps appear in a scenario (multiple actions)
- Given/When/Then appear in chaotic alternation:
  ```gherkin
  Given something
  When action happens
  Then expect result
  When another action happens      # Wrong: When after Then
  Then another result
  ```
- Scenario lacks `Then` (only Given + When, no assertion)
- Scenario is all `Given` (no action and no result)
- `And`/`But` connect mismatched step types (e.g., `When` followed by `And` but is actually a Given-type step)
- Scenario reads like a running account rather than a structured specification

### Consequences (Why It's Bad)

| Consequence | Description |
|-------------|-------------|
| **Unclear Intent** | Cannot distinguish preconditions, actions, and expected results |
| **Violates Structure** | Gherkin's core structure is destroyed, losing readability |
| **Incomplete Tests** | Scenarios lacking Then have no verification, only execution |
| **Difficult Debugging** | Multiple When steps make it impossible to determine which action caused the failure |
| **Violates Single Behavior Principle** | Multiple When steps mean the scenario is testing multiple behaviors |
| **Difficult Maintenance** | Unstructured scenarios are hard to understand and modify |

### Fix Solutions

1. **Strictly Follow Given-When-Then Structure**:
   - **Given** (one or more): Set initial context/state
   - **When** (one): Describe the action/event triggering verification
   - **Then** (one or more): Describe expected results/assertions
2. **One Scenario, One When**: If a scenario needs multiple When steps, split it into multiple scenarios
3. **Ensure There is a Then**: Every scenario must have Then steps to verify results
4. **Use Correct Keywords**: And/But should inherit the keyword meaning of the previous step
5. **Avoid Mixing Actions in Given**: Given should describe state rather than actions
6. **Use Background for Shared Givens**: Keep the scenario body concise

### Corresponding Correct Principles

- **G-28**: Strictly follow Given-When-Then order
- **G-29**: Each scenario has only one When
- **G-30**: Each scenario must have at least one Then
- **G-04**: Scenarios focus on a single business behavior

### Code Examples

**Before Fix (Chaotic Order - Not Recommended):**

```gherkin
Feature: Order Management

  Scenario: Process order and notify customer
    When I create an order for "Laptop"   # Missing Given to set context
    Then the order should be created
    Given the inventory shows 10 items     # Given appears after Then!
    When I process the order               # Second When!
    Then the inventory should show 9 items
    When the system sends notification     # Third When!
    Then the customer should receive email
    # 3 When + 3 Then, testing 3 different behaviors

  Scenario: Cancel order
    Given an order exists
    When I cancel the order
    # Missing Then! No verification!

  Scenario: Complete checkout flow
    Given I am logged in
    And I have items in my cart
    When I proceed to checkout
    And I enter shipping address          # And inherits When - this is another action
    And I select payment method           # Another action
    And I confirm the order               # Yet another action
    Then the order should be placed
    And the confirmation should be sent
    # And after When are actually more When steps
```

**After Fix (Clear Structure - Recommended):**

```gherkin
Feature: Order Management

  Background:
    Given an admin user is logged in
    And the inventory has 10 "Laptop" units

  # Scenario 1: Create order
  Scenario: Create a new order
    Given a customer "John" has requested to buy a "Laptop"
    When an order is created for the customer
    Then the order should be in "Pending" status
    And the customer should see the order in their order list

  # Scenario 2: Processing order reduces inventory
  Scenario: Processing order reduces inventory
    Given an order exists for 1 "Laptop" unit
    When the order is processed
    Then the inventory should show 9 "Laptop" units
    And the order status should be "Processed"

  # Scenario 3: Send notification
  Scenario: Order completion sends notification
    Given an order has been completed for customer "john@example.com"
    When the order confirmation is triggered
    Then "john@example.com" should receive an order confirmation email
    And the email should contain the order details

  # Scenario 4: Cancel order
  Scenario: Cancel an order
    Given an order exists with status "Pending"
    When the order is cancelled
    Then the order status should be "Cancelled"
    And the inventory should be restored to 10 units

  # Scenario 5: Complete checkout flow
  Scenario: Customer completes checkout
    Given a customer has items in their cart
    And the customer has provided shipping details
    And the customer has selected a payment method
    When the customer confirms the order
    Then the order should be placed successfully
    And a confirmation should be displayed
    And an order summary should be sent via email
```

```java
// Before fix: Chaotic Step Definition
public class BadOrderSteps {
    // One method handles create+process+notification, messy responsibilities
    @When("I create and process an order for {string}")
    public void createAndProcess(String product) {
        // Create order
        Order order = orderService.create(product);
        // Process order
        orderService.process(order.getId());
        // Send notification
        notificationService.send(order);
    }
}

// After fix: Clear responsibility separation
public class GoodOrderSteps {
    // Create order - Given step
    @Given("a customer {string} has requested to buy a {string}")
    public void customerRequestsPurchase(String customerName, String product) {
        Customer customer = customerFactory.findOrCreate(customerName);
        Product requestedProduct = productService.find(product);
        context.setPurchaseRequest(new PurchaseRequest(customer, requestedProduct));
    }

    // Create order - When step
    @When("an order is created for the customer")
    public void createOrder() {
        PurchaseRequest request = context.getPurchaseRequest();
        Order order = orderService.create(request);
        context.setCurrentOrder(order);
    }

    // Verify order status - Then step
    @Then("the order should be in {string} status")
    public void verifyOrderStatus(String expectedStatus) {
        OrderStatus actual = orderService.getStatus(context.getCurrentOrder().getId());
        assertEquals(OrderStatus.valueOf(expectedStatus), actual);
    }

    // Process order - Separate When step
    @When("the order is processed")
    public void processOrder() {
        orderService.process(context.getCurrentOrder().getId());
    }

    // Verify inventory - Then step
    @Then("the inventory should show {int} {string} units")
    public void verifyInventory(int expectedCount, String productName) {
        int actual = inventoryService.getAvailableQuantity(productName);
        assertEquals(expectedCount, actual);
    }
}
```

---

## Appendix

### Anti-Patterns Quick Reference

| ID | Anti-Pattern | Core Problem | Severity | Main Fix Strategy |
|----|--------------|--------------|----------|-------------------|
| AP-01 | Imperative Scenarios | Describes "how to do" rather than "what to do" | High | Adopt Declarative style |
| AP-02 | Feature-coupled Steps | Steps coupled one-to-one with Features | High | Organize by domain concept |
| AP-03 | Conjunction Steps | One step contains multiple actions | Medium | Keep steps atomic |
| AP-04 | UI-centric Gherkin | Gherkin exposes UI details | High | Elevate to business behavior layer |
| AP-05 | God Step Definition | One Step Def handles too much logic | High | Extract Service/Page Object |
| AP-06 | Brittle Selectors | Uses fragile UI selectors | High | Use data-testid and other stable locators |
| AP-07 | Testing Through the UI | All tests go through UI | High | Test pyramid layering |
| AP-08 | Lack of Ubiquitous Language | Uses technical terminology | High | Establish ubiquitous language glossary |
| AP-09 | Scenario Dependence | Scenarios depend on each other | High | Each scenario self-contained |
| AP-10 | Too Many Details | Scenarios contain too many details | Medium | Focus on core behavior, streamline steps |
| AP-11 | Sleep/Wait Anti-pattern | Uses fixed waits | Medium | Use explicit waits |
| AP-12 | No Clear Given-When-Then | GWT order is chaotic | High | Strictly follow GWT structure |

### Severity Level Descriptions

| Level | Description |
|-------|-------------|
| **High** | Seriously affects BDD's core values (collaboration, living documentation, maintainability), should be prioritized for fixing |
| **Medium** | Reduces test quality and execution efficiency, leads to maintenance issues in the long term |
| **Low** | Minor impact, more of a style issue |

### Reference Sources

- [Cucumber Official Documentation - Anti-Patterns](https://cucumber.io/docs/guides/anti-patterns/)
- [Cucumber Official Documentation - Better Gherkin](https://cucumber.io/docs/bdd/better-gherkin/)
- [Cucumber Official Documentation - Gherkin Reference](https://cucumber.io/docs/gherkin/reference/)
- [TestQuality - Best Practices for Maintainable Gherkin](https://testquality.com/best-practices-for-writing-maintainable-gherkin-test-cases/)
- [Gherkin Golden Rules](https://jignect.tech/understanding-the-bdd-gherkin-language-main-rules-for-bdd-ui-scenarios/)
- [SmartBear - Test Automation with Gherkin](https://smartbear.com/learn/automated-testing/test-automation-with-gherkin/)
- [Redsauce - Gherkin Best Practices](https://redsauce.com/en/blog/gherkin-best-practices)
- [Cucumber School - BDD Training](https://school.cucumber.io/)

---

> **Document Maintenance Note**: This document should be used in conjunction with `guidelines.md`. The "corresponding correct principles" in the anti-patterns reference principle IDs defined in the `guidelines.md` principle system. When adding or modifying principles, the references in this document should be updated accordingly.
