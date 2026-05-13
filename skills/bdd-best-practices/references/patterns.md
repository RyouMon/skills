# BDD Design Pattern Code Template Reference Manual

> Compiled from comprehensive research reports: research_core.md, research_cucumber.md, research_patterns.md, research_community.md
> Version: 2025
> Language: English

---

## Table of Contents

1. [Page Object Pattern](#1-page-object-pattern)
2. [Screenplay Pattern](#2-screenplay-pattern)
3. [Context Injection Pattern](#3-context-injection-pattern)
4. [Builder / Factory Test Data Pattern](#4-builder--factory-test-data-pattern)
5. [Scenario Outline Data-Driven Pattern](#5-scenario-outline-data-driven-pattern)
6. [Step Definition Organization Pattern](#6-step-definition-organization-pattern)

---

## 1. Page Object Pattern

### 1.1 Pattern Description

Page Object Model (POM) is the most classic and widely used design pattern in BDD UI automation. It encapsulates page element locators and page operations into separate classes, achieving decoupling between test code and page implementation. When the UI changes, only the locators in the Page Object class need to be modified, without changing Step Definitions or Feature files.

### 1.2 Applicable Scenarios

| Scenario | Description |
|------|------|
| **Web UI Automation** | Web testing using Selenium, Playwright, Cypress, and other tools |
| **Multi-page Applications** | Complex Web applications with multiple independent pages |
| **Frequently Changing UI** | Front-end page structures are often adjusted, requiring reduced maintenance costs |
| **Team Collaboration** | Testers and developers need a clear layered architecture |
| **Large Test Suites** | Long-term projects requiring high reusability and maintainability |

### 1.3 Complete Code Examples

#### 1.3.1 Java + Selenium Implementation

**Directory Structure:**
```
src/test/java/
  pages/
    BasePage.java          # Page base class
    LoginPage.java         # Login page object
    DashboardPage.java     # Dashboard page object
  stepdefinitions/
    LoginSteps.java        # Login step definitions
    CommonSteps.java       # Common steps
  runners/
    TestRunner.java        # Test runner
```

**BasePage.java - Page Base Class:**
```java
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.PageFactory;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

/**
 * Page Object base class providing common operations shared by all pages
 */
public abstract class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        PageFactory.initElements(driver, this);
    }

    /**
     * Wait for element to be visible then click
     */
    protected void clickWhenReady(WebElement element) {
        wait.until(ExpectedConditions.visibilityOf(element));
        element.click();
    }

    /**
     * Wait for element to be visible then enter text
     */
    protected void typeWhenReady(WebElement element, String text) {
        wait.until(ExpectedConditions.visibilityOf(element));
        element.clear();
        element.sendKeys(text);
    }

    /**
     * Get current page URL
     */
    public String getCurrentUrl() {
        return driver.getCurrentUrl();
    }

    /**
     * Open specified URL
     */
    public void navigateTo(String url) {
        driver.get(url);
    }

    /**
     * Wait for element to be visible
     */
    protected boolean isElementDisplayed(WebElement element) {
        try {
            wait.until(ExpectedConditions.visibilityOf(element));
            return element.isDisplayed();
        } catch (Exception e) {
            return false;
        }
    }
}
```

**LoginPage.java - Login Page Object:**
```java
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

/**
 * Login Page - Page Object
 * Encapsulates all elements and operations of the login page
 */
public class LoginPage extends BasePage {

    // Page URL
    private static final String LOGIN_URL = "https://example.com/login";

    // Element locators - using @FindBy annotations
    @FindBy(id = "username")
    private WebElement usernameField;

    @FindBy(id = "password")
    private WebElement passwordField;

    @FindBy(css = "button[type='submit']")
    private WebElement loginButton;

    @FindBy(className = "error-message")
    private WebElement errorMessage;

    @FindBy(className = "welcome-banner")
    private WebElement welcomeBanner;

    @FindBy(linkText = "Forgot password?")
    private WebElement forgotPasswordLink;

    public LoginPage(WebDriver driver) {
        super(driver);
    }

    /**
     * Open login page
     */
    public void open() {
        navigateTo(LOGIN_URL);
    }

    /**
     * Perform login operation
     */
    public void login(String username, String password) {
        typeWhenReady(usernameField, username);
        typeWhenReady(passwordField, password);
        clickWhenReady(loginButton);
    }

    /**
     * Enter username
     */
    public void enterUsername(String username) {
        typeWhenReady(usernameField, username);
    }

    /**
     * Enter password
     */
    public void enterPassword(String password) {
        typeWhenReady(passwordField, password);
    }

    /**
     * Click login button
     */
    public void clickLoginButton() {
        clickWhenReady(loginButton);
    }

    /**
     * Get error message text
     */
    public String getErrorMessage() {
        wait.until(ExpectedConditions.visibilityOf(errorMessage));
        return errorMessage.getText();
    }

    /**
     * Check if welcome banner is displayed
     */
    public boolean isWelcomeBannerDisplayed() {
        return isElementDisplayed(welcomeBanner);
    }

    /**
     * Click "Forgot password" link
     */
    public void clickForgotPassword() {
        clickWhenReady(forgotPasswordLink);
    }
}
```

**DashboardPage.java - Dashboard Page Object:**
```java
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import java.util.List;

/**
 * Dashboard Page - Page Object
 */
public class DashboardPage extends BasePage {

    @FindBy(css = ".user-greeting")
    private WebElement userGreeting;

    @FindBy(css = ".order-count")
    private WebElement orderCount;

    @FindBy(css = ".recent-orders .order-item")
    private List<WebElement> recentOrders;

    @FindBy(linkText = "Logout")
    private WebElement logoutLink;

    @FindBy(css = ".nav-item.orders")
    private WebElement ordersNavItem;

    public DashboardPage(WebDriver driver) {
        super(driver);
    }

    /**
     * Get user greeting
     */
    public String getUserGreeting() {
        wait.until(ExpectedConditions.visibilityOf(userGreeting));
        return userGreeting.getText();
    }

    /**
     * Get order count
     */
    public int getOrderCount() {
        wait.until(ExpectedConditions.visibilityOf(orderCount));
        return Integer.parseInt(orderCount.getText().trim());
    }

    /**
     * Get recent order count
     */
    public int getRecentOrderCount() {
        return recentOrders.size();
    }

    /**
     * Logout
     */
    public void logout() {
        clickWhenReady(logoutLink);
    }

    /**
     * Navigate to orders page
     */
    public void navigateToOrders() {
        clickWhenReady(ordersNavItem);
    }

    /**
     * Verify if on dashboard page
     */
    public boolean isOnDashboard() {
        return getCurrentUrl().contains("/dashboard");
    }
}
```

**LoginSteps.java - Using Page Object in Step Definitions:**
```java
package stepdefinitions;

import io.cucumber.java.en.*;
import pages.LoginPage;
import pages.DashboardPage;
import context.TestContext;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Login feature step definitions
 * Injects TestContext via constructor to obtain Page Object
 */
public class LoginSteps {

    private final LoginPage loginPage;
    private final DashboardPage dashboardPage;
    private final TestContext context;

    // Obtain TestContext and Page Object through dependency injection
    public LoginSteps(TestContext context) {
        this.context = context;
        this.loginPage = new LoginPage(context.getDriver());
        this.dashboardPage = new DashboardPage(context.getDriver());
    }

    @Given("the user is on the login page")
    public void the_user_is_on_the_login_page() {
        loginPage.open();
        assertTrue(loginPage.getCurrentUrl().contains("/login"));
    }

    @When("the user enters username {string}")
    public void the_user_enters_username(String username) {
        loginPage.enterUsername(username);
        context.setUsername(username);  // Save to context for later use
    }

    @When("the user enters password {string}")
    public void the_user_enters_password(String password) {
        loginPage.enterPassword(password);
    }

    @When("the user clicks the login button")
    public void the_user_clicks_the_login_button() {
        loginPage.clickLoginButton();
    }

    @When("the user logs in with {string} and {string}")
    public void the_user_logs_in_with(String username, String password) {
        loginPage.login(username, password);
        context.setUsername(username);
    }

    @Then("the user should be redirected to the dashboard")
    public void the_user_should_be_redirected_to_the_dashboard() {
        assertTrue(dashboardPage.isOnDashboard(),
            "Expected to be on dashboard page");
    }

    @Then("the welcome message should contain {string}")
    public void the_welcome_message_should_contain(String expectedName) {
        String greeting = dashboardPage.getUserGreeting();
        assertTrue(greeting.contains(expectedName),
            "Expected greeting to contain '" + expectedName + "' but was: " + greeting);
    }

    @Then("the user should see an error message {string}")
    public void the_user_should_see_an_error_message(String expectedMessage) {
        String actualMessage = loginPage.getErrorMessage();
        assertEquals(expectedMessage, actualMessage);
    }

    @Then("the user should remain on the login page")
    public void the_user_should_remain_on_the_login_page() {
        assertTrue(loginPage.getCurrentUrl().contains("/login"));
    }
}
```

**login.feature - Feature File:**
```gherkin
@regression @authentication
Feature: Customer Login
  As a registered customer
  I want to log in with my credentials
  So that I can access my account dashboard

  @positive @smoke
  Scenario: Successful login with valid credentials
    Given the user is on the login page
    When the user logs in with "john.doe@example.com" and "ValidPass123"
    Then the user should be redirected to the dashboard
    And the welcome message should contain "John"

  @negative
  Scenario Outline: Unsuccessful login with invalid credentials
    Given the user is on the login page
    When the user logs in with "<username>" and "<password>"
    Then the user should see an error message "<error_message>"
    And the user should remain on the login page

    Examples:
      | username              | password    | error_message           |
      | john.doe@example.com  | wrongpass   | Invalid credentials     |
      | unknown@example.com   | anypassword | Username not found      |
      | john.doe@example.com  |             | Password is required    |
```

#### 1.3.2 JavaScript + Playwright Implementation

**pages/LoginPage.js:**
```javascript
/**
 * Login Page - Page Object (JavaScript + Playwright)
 */
class LoginPage {
  constructor(page) {
    this.page = page;

    // Element locators
    this.usernameField = page.locator('#username');
    this.passwordField = page.locator('#password');
    this.loginButton = page.locator('button[type="submit"]');
    this.errorMessage = page.locator('.error-message');
    this.welcomeBanner = page.locator('.welcome-banner');
    this.forgotPasswordLink = page.locator('text=Forgot password?');
  }

  /**
   * Open login page
   */
  async open() {
    await this.page.goto('https://example.com/login');
  }

  /**
   * Perform login operation
   */
  async login(username, password) {
    await this.usernameField.fill(username);
    await this.passwordField.fill(password);
    await this.loginButton.click();
  }

  /**
   * Enter username
   */
  async enterUsername(username) {
    await this.usernameField.fill(username);
  }

  /**
   * Enter password
   */
  async enterPassword(password) {
    await this.passwordField.fill(password);
  }

  /**
   * Click login button
   */
  async clickLoginButton() {
    await this.loginButton.click();
  }

  /**
   * Get error message text
   */
  async getErrorMessage() {
    await this.errorMessage.waitFor({ state: 'visible' });
    return await this.errorMessage.textContent();
  }

  /**
   * Check if welcome banner is displayed
   */
  async isWelcomeBannerDisplayed() {
    return await this.welcomeBanner.isVisible();
  }

  /**
   * Get current page URL
   */
  async getCurrentUrl() {
    return this.page.url();
  }
}

module.exports = { LoginPage };
```

**pages/DashboardPage.js:**
```javascript
/**
 * Dashboard Page - Page Object (JavaScript + Playwright)
 */
class DashboardPage {
  constructor(page) {
    this.page = page;

    this.userGreeting = page.locator('.user-greeting');
    this.orderCount = page.locator('.order-count');
    this.logoutLink = page.locator('text=Logout');
  }

  /**
   * Get user greeting
   */
  async getUserGreeting() {
    await this.userGreeting.waitFor({ state: 'visible' });
    return await this.userGreeting.textContent();
  }

  /**
   * Logout
   */
  async logout() {
    await this.logoutLink.click();
  }

  /**
   * Verify if on dashboard page
   */
  async isOnDashboard() {
    const url = this.page.url();
    return url.includes('/dashboard');
  }
}

module.exports = { DashboardPage };
```

**step-definitions/login.steps.js:**
```javascript
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('@playwright/test');
const { LoginPage } = require('../pages/LoginPage');
const { DashboardPage } = require('../pages/DashboardPage');

Given('the user is on the login page', async function() {
  this.loginPage = new LoginPage(this.page);
  this.dashboardPage = new DashboardPage(this.page);
  await this.loginPage.open();
});

When('the user logs in with {string} and {string}', async function(username, password) {
  await this.loginPage.login(username, password);
  this.username = username;  // Save to World context
});

Then('the user should be redirected to the dashboard', async function() {
  const isOnDashboard = await this.dashboardPage.isOnDashboard();
  expect(isOnDashboard).toBe(true);
});

Then('the welcome message should contain {string}', async function(expectedName) {
  const greeting = await this.dashboardPage.getUserGreeting();
  expect(greeting).toContain(expectedName);
});

Then('the user should see an error message {string}', async function(expectedMessage) {
  const actualMessage = await this.loginPage.getErrorMessage();
  expect(actualMessage).toBe(expectedMessage);
});
```

#### 1.3.3 Page Object Manager Pattern (Advanced)

To avoid repeatedly creating Page Object instances in different Step Definitions, use Page Object Manager:

```java
package pages;

import org.openqa.selenium.WebDriver;

/**
 * Page Object Manager - manages Page Object instances uniformly
 * Ensures each Page Object is created only once
 */
public class PageObjectManager {
    private final WebDriver driver;

    private LoginPage loginPage;
    private DashboardPage dashboardPage;
    private OrderPage orderPage;
    private ProfilePage profilePage;

    public PageObjectManager(WebDriver driver) {
        this.driver = driver;
    }

    public LoginPage getLoginPage() {
        if (loginPage == null) {
            loginPage = new LoginPage(driver);
        }
        return loginPage;
    }

    public DashboardPage getDashboardPage() {
        if (dashboardPage == null) {
            dashboardPage = new DashboardPage(driver);
        }
        return dashboardPage;
    }

    public OrderPage getOrderPage() {
        if (orderPage == null) {
            orderPage = new OrderPage(driver);
        }
        return orderPage;
    }

    public ProfilePage getProfilePage() {
        if (profilePage == null) {
            profilePage = new ProfilePage(driver);
        }
        return profilePage;
    }
}
```

### 1.4 Pros and Cons

| Pros | Cons |
|------|------|
| **Separation of Concerns** - UI locators are completely separated from test logic | **May Lead to God Class** - Classes become bloated when there are too many page elements |
| **High Maintainability** - UI changes only require modification in one place | **High Initial Investment** - Requires creating Page Objects for each page |
| **Code Reuse** - Same page operations can be reused across multiple scenarios | **Granularity is Hard to Grasp** - Dividing pages vs components requires experience |
| **Improved Readability** - Method names express business intent | **Limited Support for Dynamic Content** - Complex interactions may require additional handling |
| **Reduced Maintenance Cost** - Decreases test failures caused by UI changes | **Learning Curve** - Beginners need time to understand the layered architecture |

---

## 2. Screenplay Pattern

### 2.1 Pattern Description

The Screenplay pattern (also known as the Actor pattern) is a modern alternative to Page Object Model, proposed by the Serenity BDD community and inspired by the Actor model. It is centered around **user behavior**, elevating the test focus from "pages and elements" to "what the user wants to do." Core concepts include Actor (user), Ability (capability), Task, Interaction, and Question.

### 2.2 Applicable Scenarios

| Scenario | Description |
|------|------|
| **Large Complex Projects** | Requires highly modular and composable test architecture |
| **Multi-role Interactions** | Testing scenarios involving multiple different user roles |
| **High Reuse Requirements** | Same operation sequences are reused across multiple scenarios |
| **Serenity BDD Integration** | Works with Serenity reporting system |
| **Mature Team Skills** | Team already has BDD fundamentals and wants to further improve architecture |

### 2.3 Core Concepts

| Concept | English | Description |
|------|------|------|
| **Actor** | Actor | Represents a user or external system interacting with the system |
| **Ability** | Ability | Defines what an Actor can do (e.g., browse the web, call APIs) |
| **Task** | Task | Represents high-level business activities performed by an Actor (composed of multiple Interactions) |
| **Interaction** | Interaction | Low-level operations (click, type, navigate) |
| **Question** | Question | Retrieves information from the system for assertions |

### 2.4 Complete Code Example (Java + Serenity BDD)

#### Directory Structure
```
src/test/java/
  screenplay/
    questions/          # Question classes - query system state
      OrderTotal.java
      WelcomeMessage.java
    tasks/              # Task classes - high-level business operations
      Login.java
      PlaceOrder.java
      SearchProduct.java
    interactions/       # Interaction classes - low-level operations
      ClickElement.java
      EnterText.java
      NavigateTo.java
    actors/             # Actor configuration
      Cast.java
  stepdefinitions/
    OrderSteps.java
  features/
    order.feature
```

#### Actor Definition

```java
package screenplay.actors;

import net.serenitybdd.screenplay.Actor;
import net.serenitybdd.screenplay.abilities.BrowseTheWeb;
import org.openqa.selenium.WebDriver;

/**
 * Actor definition - represents a user interacting with the system
 */
public class OnlineShopper {

    /**
     * Create an actor named 'name' with the ability to browse the web
     */
    public static Actor named(String name, WebDriver driver) {
        Actor actor = Actor.named(name);
        actor.can(BrowseTheWeb.with(driver));
        return actor;
    }

    /**
     * Quick creation methods for common roles
     */
    public static Actor aNewCustomer(WebDriver driver) {
        return named("New Customer", driver);
    }

    public static Actor aReturningCustomer(WebDriver driver) {
        return named("Returning Customer", driver);
    }

    public static Actor anAdminUser(WebDriver driver) {
        return named("Admin", driver);
    }
}
```

#### Ability Definition

```java
package screenplay.abilities;

import net.serenitybdd.screenplay.Ability;
import net.serenitybdd.screenplay.Actor;
import org.openqa.selenium.WebDriver;

/**
 * Custom Ability - capability to call REST APIs
 */
public class CallApi implements Ability {

    private final String baseUrl;
    private final WebDriver driver; // Used for authentication state

    private CallApi(String baseUrl, WebDriver driver) {
        this.baseUrl = baseUrl;
        this.driver = driver;
    }

    public static CallApi at(String baseUrl, WebDriver driver) {
        return new CallApi(baseUrl, driver);
    }

    public static CallApi as(Actor actor) {
        return actor.abilityTo(CallApi.class);
    }

    public String getBaseUrl() {
        return baseUrl;
    }

    public WebDriver getDriver() {
        return driver;
    }
}
```

#### Task Definition

```java
package screenplay.tasks;

import net.serenitybdd.screenplay.Actor;
import net.serenitybdd.screenplay.Task;
import net.serenitybdd.screenplay.actions.Click;
import net.serenitybdd.screenplay.actions.Enter;
import net.serenitybdd.screenplay.actions.Open;
import net.serenitybdd.screenplay.actions.Scroll;
import net.serenitybdd.screenplay.waits.WaitUntil;
import org.openqa.selenium.By;
import static net.serenitybdd.screenplay.matchers.WebElementStateMatchers.isVisible;

/**
 * Login task - represents high-level business activities performed by the user
 */
public class Login implements Task {

    private final String username;
    private final String password;

    // Element locators - using Target objects
    private static final By USERNAME_FIELD = By.id("username");
    private static final By PASSWORD_FIELD = By.id("password");
    private static final By LOGIN_BUTTON = By.cssSelector("button[type='submit']");

    private Login(String username, String password) {
        this.username = username;
        this.password = password;
    }

    /**
     * Factory method - using fluent API style
     */
    public static Login withCredentials(String username, String password) {
        return new Login(username, password);
    }

    public static Login as(String username) {
        return new Login(username, null); // Password will be set later
    }

    public Login andPassword(String password) {
        return new Login(this.username, password);
    }

    /**
     * Core method for executing the task
     */
    @Override
    public <T extends Actor> void performAs(T actor) {
        actor.attemptsTo(
            Open.url("https://example.com/login"),
            WaitUntil.the(USERNAME_FIELD, isVisible()),
            Enter.theValue(username).into(USERNAME_FIELD),
            Enter.theValue(password).into(PASSWORD_FIELD),
            Click.on(LOGIN_BUTTON)
        );
    }
}
```

```java
package screenplay.tasks;

import net.serenitybdd.screenplay.Actor;
import net.serenitybdd.screenplay.Task;
import net.serenitybdd.screenplay.actions.Click;
import net.serenitybdd.screenplay.actions.Enter;
import net.serenitybdd.screenplay.actions.Scroll;
import org.openqa.selenium.By;
import static net.serenitybdd.screenplay.Tasks.instrumented;

/**
 * Place Order task - composite task example
 */
public class PlaceOrder implements Task {

    private final String productName;
    private String discountCode;

    private static final By SEARCH_FIELD = By.id("search");
    private static final By SEARCH_BUTTON = By.cssSelector(".search-button");
    private static final By ADD_TO_CART_BUTTON = By.cssSelector(".add-to-cart");
    private static final By DISCOUNT_FIELD = By.id("discount-code");
    private static final By APPLY_DISCOUNT_BUTTON = By.id("apply-discount");
    private static final By CHECKOUT_BUTTON = By.cssSelector(".checkout");

    public PlaceOrder(String productName) {
        this.productName = productName;
    }

    /**
     * Factory method
     */
    public static PlaceOrder forProduct(String productName) {
        return instrumented(PlaceOrder.class, productName);
    }

    /**
     * Fluent API for using discount codes
     */
    public PlaceOrder withDiscountCode(String code) {
        this.discountCode = code;
        return this;
    }

    @Override
    public <T extends Actor> void performAs(T actor) {
        // Basic order placement flow
        actor.attemptsTo(
            Enter.theValue(productName).into(SEARCH_FIELD),
            Click.on(SEARCH_BUTTON),
            Scroll.to(ADD_TO_CART_BUTTON),
            Click.on(ADD_TO_CART_BUTTON)
        );

        // Optional: apply discount code
        if (discountCode != null) {
            actor.attemptsTo(
                Enter.theValue(discountCode).into(DISCOUNT_FIELD),
                Click.on(APPLY_DISCOUNT_BUTTON)
            );
        }

        // Continue to checkout
        actor.attemptsTo(
            Click.on(CHECKOUT_BUTTON)
        );
    }
}
```

#### Question Definition

```java
package screenplay.questions;

import net.serenitybdd.screenplay.Actor;
import net.serenitybdd.screenplay.Question;
import net.serenitybdd.screenplay.annotations.Subject;
import net.serenitybdd.screenplay.questions.Text;
import org.openqa.selenium.By;

/**
 * Welcome Message question - retrieves information from the system for assertions
 */
@Subject("the welcome message")
public class WelcomeMessage implements Question<String> {

    private static final By GREETING_ELEMENT = By.cssSelector(".user-greeting");

    @Override
    public String answeredBy(Actor actor) {
        return Text.of(GREETING_ELEMENT).answeredBy(actor);
    }

    public static WelcomeMessage displayed() {
        return new WelcomeMessage();
    }
}
```

```java
package screenplay.questions;

import net.serenitybdd.screenplay.Actor;
import net.serenitybdd.screenplay.Question;
import net.serenitybdd.screenplay.annotations.Subject;
import net.serenitybdd.screenplay.questions.Text;
import org.openqa.selenium.By;

/**
 * Order Total query
 */
@Subject("the order total after discount")
public class OrderTotal implements Question<String> {

    private static final By TOTAL_ELEMENT = By.cssSelector(".order-total");

    @Override
    public String answeredBy(Actor actor) {
        return Text.of(TOTAL_ELEMENT).answeredBy(actor);
    }

    public static OrderTotal afterDiscount() {
        return new OrderTotal();
    }
}
```

#### Using Screenplay in Step Definitions

```java
package stepdefinitions;

import io.cucumber.java.en.*;
import net.serenitybdd.screenplay.Actor;
import net.serenitybdd.screenplay.ensure.Ensure;
import screenplay.actors.OnlineShopper;
import screenplay.tasks.Login;
import screenplay.tasks.PlaceOrder;
import screenplay.questions.WelcomeMessage;
import screenplay.questions.OrderTotal;
import org.openqa.selenium.WebDriver;
import context.TestContext;

import static org.hamcrest.Matchers.*;
import static net.serenitybdd.screenplay.GivenWhenThen.*;

/**
 * Step definitions using Screenplay pattern
 */
public class OrderSteps {

    private final TestContext context;
    private Actor actor;

    public OrderSteps(TestContext context) {
        this.context = context;
    }

    @Given("{word} is a registered customer")
    public void is_a_registered_customer(String actorName) {
        this.actor = OnlineShopper.named(actorName, context.getDriver());
    }

    @Given("{word} has logged in with valid credentials")
    public void has_logged_in(String actorName) {
        this.actor = OnlineShopper.named(actorName, context.getDriver());
        actor.attemptsTo(
            Login.withCredentials("john@example.com", "password123")
        );
    }

    @When("{word} places an order for {string}")
    public void places_an_order(String actorName, String product) {
        actor.attemptsTo(
            PlaceOrder.forProduct(product)
        );
    }

    @When("{word} places an order for {string} with discount {string}")
    public void places_an_order_with_discount(String actorName, String product, String discount) {
        actor.attemptsTo(
            PlaceOrder.forProduct(product)
                .withDiscountCode(discount)
        );
    }

    @Then("{word} should see a welcome message containing {string}")
    public void should_see_welcome_message(String actorName, String expectedText) {
        actor.should(
            seeThat(WelcomeMessage.displayed(), containsString(expectedText))
        );
    }

    @Then("the order total should be {string}")
    public void order_total_should_be(String expectedTotal) {
        actor.should(
            seeThat(OrderTotal.afterDiscount(), equalTo(expectedTotal))
        );
    }
}
```

#### Feature File

```gherkin
@regression @shopping
Feature: Place Orders
  As a registered customer
  I want to place orders with or without discount codes
  So that I can purchase products online

  @positive @smoke
  Scenario: Customer places a standard order
    Given Alice is a registered customer
    And Alice has logged in with valid credentials
    When Alice places an order for "Wireless Mouse"
    Then Alice should see a welcome message containing "Alice"

  @positive
  Scenario: Customer uses a discount code
    Given Bob is a registered customer
    And Bob has logged in with valid credentials
    When Bob places an order for "Mechanical Keyboard" with discount "SAVE20"
    Then the order total should be "$79.99"
    And Bob should see a confirmation message
```

### 2.5 Pros and Cons

| Pros | Cons |
|------|------|
| **Highly Modular** - Tasks can be freely combined, avoiding God Class | **Steep Learning Curve** - Requires understanding Actor, Ability, Task, and other concepts |
| **User-Centric** - Code naturally expresses user behavior | **High Initial Development Cost** - Requires creating many small classes |
| **High Reusability** - Tasks and Questions can be combined in any scenario | **Depends on Serenity BDD** - Best experience requires Serenity framework support |
| **Strong Type Safety** - Compile-time checking of task composition | **Complex Debugging** - Long call chains make problem diagnosis difficult |
| **Automatically Generates Excellent Reports** - Serenity auto-generates narrative documentation | **Not Suitable for Small Projects** - May be over-engineering for simple scenarios |
| **Clear Separation of "Do" and "See"** - Tasks and Questions have clear responsibilities | **Team Training Cost** - Requires all team members to learn the new pattern |

### 2.6 Screenplay vs Page Object Comparison

| Feature | Page Object | Screenplay |
|------|-------------|------------|
| **Focus** | Pages and elements | Users and behaviors |
| **Modularity** | Medium - tends to become God Class | High - small and composable task units |
| **Readability** | Medium - focuses on pages rather than user behavior | High - centered around user behavior |
| **Reusability** | Low - page-to-page flows require repeated linking | High - tasks can be freely combined |
| **Learning Curve** | Low | High |
| **Separation** | Low - "do" and "see" are not clearly separated | High - clearly separates Task from Question |
| **Scale Adaptability** | Small to medium projects | Large complex projects |
| **Report Integration** | Requires additional configuration | Native Serenity support |


---

## 3. Context Injection Pattern

### 3.1 Pattern Description

The Context Injection pattern is used to share state between Cucumber step definitions. Since each scenario is independent and Cucumber creates a new instance of the Step Definition class for each scenario, a mechanism is needed to pass data between multiple steps within a scenario. Context Injection achieves this through **Dependency Injection (DI) containers** or the **World object**.

In the JVM ecosystem, Cucumber recommends injecting shared context objects via constructor; in JavaScript, the World object is used (via the `this` keyword); in Ruby, instance variables are used.

### 3.2 Applicable Scenarios

| Scenario | Description |
|------|------|
| **Multi-step Data Sharing** | Objects created in Given need to be verified in Then |
| **Cross Step Definition Class Sharing** | Multiple Step Def classes need access to the same state |
| **WebDriver Sharing** | Browser instance is shared across all steps |
| **Test Data Passing** | Dynamically generated test data needs to be used in multiple steps |
| **Cleanup and Hook Communication** | After Hook needs to access data created during the scenario |

### 3.3 Complete Code Examples

#### 3.3.1 Java (JVM) + PicoContainer Dependency Injection

**PicoContainer is the lightweight DI container officially recommended by Cucumber-JVM.**

**Maven Dependency:**
```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-picocontainer</artifactId>
    <version>7.34.3</version>
    <scope>test</scope>
</dependency>
```

**TestContext.java - Shared Context Class:**
```java
package context;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import pages.PageObjectManager;

/**
 * TestContext - scenario-level shared context
 * Injected into all Step Definition classes via PicoContainer
 */
public class TestContext {

    private WebDriver driver;
    private PageObjectManager pageObjectManager;
    private ScenarioContext scenarioContext;

    public TestContext() {
        this.scenarioContext = new ScenarioContext();
    }

    public WebDriver getDriver() {
        if (driver == null) {
            String browser = System.getProperty("browser", "chrome");
            switch (browser.toLowerCase()) {
                case "firefox":
                    driver = new FirefoxDriver();
                    break;
                case "chrome":
                default:
                    driver = new ChromeDriver();
                    break;
            }
            driver.manage().window().maximize();
        }
        return driver;
    }

    public PageObjectManager getPageObjectManager() {
        if (pageObjectManager == null) {
            pageObjectManager = new PageObjectManager(getDriver());
        }
        return pageObjectManager;
    }

    public ScenarioContext getScenarioContext() {
        return scenarioContext;
    }

    public void quitDriver() {
        if (driver != null) {
            driver.quit();
            driver = null;
        }
    }
}
```

**ScenarioContext.java - Key-Value Data Store:**
```java
package context;

import java.util.HashMap;
import java.util.Map;

/**
 * ScenarioContext - stores temporary data during scenario execution
 * Used for passing data between steps
 */
public class ScenarioContext {

    private Map<String, Object> contextData;

    public ScenarioContext() {
        this.contextData = new HashMap<>();
    }

    public void set(String key, Object value) {
        contextData.put(key, value);
    }

    @SuppressWarnings("unchecked")
    public <T> T get(String key) {
        return (T) contextData.get(key);
    }

    @SuppressWarnings("unchecked")
    public <T> T getOrDefault(String key, T defaultValue) {
        return (T) contextData.getOrDefault(key, defaultValue);
    }

    public boolean contains(String key) {
        return contextData.containsKey(key);
    }

    public void clear() {
        contextData.clear();
    }

    // Quick methods for common business data
    public void setUsername(String username) { set("username", username); }
    public String getUsername() { return get("username"); }
    public void setUserId(String userId) { set("userId", userId); }
    public String getUserId() { return get("userId"); }
    public void setOrderId(String orderId) { set("orderId", orderId); }
    public String getOrderId() { return get("orderId"); }
    public void setProductName(String productName) { set("productName", productName); }
    public String getProductName() { return get("productName"); }
}
```

**LoginSteps.java - Injecting TestContext via Constructor:**
```java
package stepdefinitions;

import io.cucumber.java.en.*;
import context.TestContext;
import pages.LoginPage;
import pages.DashboardPage;

/**
 * Login step definitions - shared context injected via constructor
 * PicoContainer automatically injects the same TestContext instance
 */
public class LoginSteps {

    private final TestContext context;
    private final LoginPage loginPage;
    private final DashboardPage dashboardPage;

    public LoginSteps(TestContext context) {
        this.context = context;
        this.loginPage = context.getPageObjectManager().getLoginPage();
        this.dashboardPage = context.getPageObjectManager().getDashboardPage();
    }

    @Given("the user navigates to the login page")
    public void navigateToLogin() {
        loginPage.open();
    }

    @When("the user enters credentials {string} / {string}")
    public void enterCredentials(String username, String password) {
        loginPage.enterUsername(username);
        loginPage.enterPassword(password);
        context.getScenarioContext().setUsername(username);
    }

    @When("the user clicks login")
    public void clickLogin() {
        loginPage.clickLoginButton();
    }

    @Then("the login should be successful")
    public void verifyLoginSuccessful() {
        boolean isDashboard = dashboardPage.isOnDashboard();
        org.junit.jupiter.api.Assertions.assertTrue(isDashboard);
        String expectedUsername = context.getScenarioContext().getUsername();
        String greeting = dashboardPage.getUserGreeting();
        org.junit.jupiter.api.Assertions.assertTrue(greeting.contains(expectedUsername));
    }
}
```

**OrderSteps.java - Another Step Def Class Sharing the Same TestContext:**
```java
package stepdefinitions;

import io.cucumber.java.en.*;
import context.TestContext;
import pages.OrderPage;
import pages.DashboardPage;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Order step definitions - shared TestContext injected via constructor
 * This TestContext is the same instance as in LoginSteps
 */
public class OrderSteps {

    private final TestContext context;
    private final OrderPage orderPage;
    private final DashboardPage dashboardPage;

    public OrderSteps(TestContext context) {
        this.context = context;
        this.orderPage = context.getPageObjectManager().getOrderPage();
        this.dashboardPage = context.getPageObjectManager().getDashboardPage();
    }

    @Given("the user has an active account")
    public void userHasActiveAccount() {
        String username = context.getScenarioContext().getUsername();
        if (username == null) {
            username = "default@example.com";
            context.getScenarioContext().setUsername(username);
        }
        if (!dashboardPage.isOnDashboard()) {
            dashboardPage.navigateToOrders();
        }
    }

    @When("the user places an order for {string}")
    public void placeOrder(String productName) {
        context.getScenarioContext().setProductName(productName);
        orderPage.searchProduct(productName);
        orderPage.addToCart();
        orderPage.checkout();
        String orderId = orderPage.getOrderId();
        context.getScenarioContext().setOrderId(orderId);
    }

    @Then("the order should be confirmed")
    public void orderShouldBeConfirmed() {
        String orderId = context.getScenarioContext().getOrderId();
        assertNotNull(orderId, "Order ID should be generated");
        assertTrue(orderPage.isOrderConfirmed());
    }

    @Then("the order summary should show the product {string}")
    public void orderSummaryShouldShowProduct(String expectedProduct) {
        String actualProduct = orderPage.getOrderSummaryProduct();
        assertEquals(expectedProduct, actualProduct);
    }
}
```

**Hooks.java - Using TestContext for Resource Management:**
```java
package hooks;

import io.cucumber.java.*;
import context.TestContext;

/**
 * Hooks - managing resources using injected TestContext
 */
public class TestHooks {

    private final TestContext context;

    public TestHooks(TestContext context) {
        this.context = context;
    }

    @Before(order = 1)
    public void beforeScenario(Scenario scenario) {
        System.out.println("=== Starting Scenario: " + scenario.getName() + " ===");
        context.getScenarioContext().clear();
    }

    @Before(order = 2)
    public void setupBrowser() {
        context.getDriver();
    }

    @After(order = 1)
    public void takeScreenshotOnFailure(Scenario scenario) {
        if (scenario.isFailed()) {
            byte[] screenshot = context.getDriver()
                .getScreenshotAs(org.openqa.selenium.OutputType.BYTES);
            scenario.attach(screenshot, "image/png", "failure_screenshot");
        }
    }

    @After(order = 2)
    public void tearDown(Scenario scenario) {
        System.out.println("=== Finished Scenario: " + scenario.getName() +
            " (Status: " + scenario.getStatus() + ") ===");
        context.quitDriver();
    }
}
```

#### 3.3.2 JavaScript - World Object Implementation

**JavaScript does not use traditional DI but shares state through the World object.**

**world.js - Custom World Class:**
```javascript
const { setWorldConstructor, World } = require('@cucumber/cucumber');
const { chromium } = require('playwright');

/**
 * Custom World class - scenario-level shared state container
 * Each scenario gets an independent World instance
 */
class CustomWorld extends World {
  constructor(options) {
    super(options);
    this.context = new Map();
    this._pages = {};
  }

  async getBrowser() {
    if (!this.browser) {
      this.browser = await chromium.launch({
        headless: process.env.HEADLESS !== 'false'
      });
    }
    return this.browser;
  }

  async getPage() {
    if (!this.page) {
      const browser = await this.getBrowser();
      this.context = await browser.newContext();
      this.page = await this.context.newPage();
    }
    return this.page;
  }

  set(key, value) {
    this.context.set(key, value);
  }

  get(key) {
    return this.context.get(key);
  }

  setUsername(username) { this.set('username', username); }
  getUsername() { return this.get('username'); }
  setOrderId(orderId) { this.set('orderId', orderId); }
  getOrderId() { return this.get('orderId'); }

  async closeBrowser() {
    if (this.context) await this.context.close();
    if (this.browser) await this.browser.close();
  }
}

setWorldConstructor(CustomWorld);
module.exports = { CustomWorld };
```

**step-definitions/login.steps.js - Using World to Share State:**
```javascript
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('@playwright/test');
const { LoginPage } = require('../pages/LoginPage');
const { DashboardPage } = require('../pages/DashboardPage');

Given('the user is on the login page', async function() {
  this.page = await this.getPage();
  this.loginPage = new LoginPage(this.page);
  this.dashboardPage = new DashboardPage(this.page);
  await this.loginPage.open();
});

When('the user logs in with {string} and {string}', async function(username, password) {
  await this.loginPage.login(username, password);
  this.setUsername(username);
  this.set('password', password);
});

Then('the user should be redirected to the dashboard', async function() {
  const isOnDashboard = await this.dashboardPage.isOnDashboard();
  expect(isOnDashboard).toBe(true);
});

Then('the welcome message should contain {string}', async function(expectedName) {
  const greeting = await this.dashboardPage.getUserGreeting();
  expect(greeting).toContain(expectedName);
  const savedUsername = this.getUsername();
  console.log(`Verified for user: ${savedUsername}`);
});
```

**step-definitions/order.steps.js - Cross-File World State Sharing:**
```javascript
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('@playwright/test');
const { OrderPage } = require('../pages/OrderPage');

Given('the user is logged in', async function() {
  if (!this.page) {
    this.page = await this.getPage();
  }
  this.orderPage = new OrderPage(this.page);
  const username = this.getUsername() || 'default@example.com';
  console.log(`Using logged-in user: ${username}`);
});

When('the user places an order for {string}', async function(productName) {
  this.set('productName', productName);
  await this.orderPage.searchProduct(productName);
  await this.orderPage.addToCart();
  await this.orderPage.checkout();
  const orderId = await this.orderPage.getOrderId();
  this.setOrderId(orderId);
});

Then('the order should be confirmed', async function() {
  const orderId = this.getOrderId();
  expect(orderId).toBeTruthy();
  const isConfirmed = await this.orderPage.isOrderConfirmed();
  expect(isConfirmed).toBe(true);
});

Then('the order summary should show {string}', async function(expectedProduct) {
  const actualProduct = await this.orderPage.getOrderSummaryProduct();
  expect(actualProduct).toBe(expectedProduct);
  const savedProduct = this.get('productName');
  expect(actualProduct).toBe(savedProduct);
});
```

#### 3.3.3 Ruby - Using World and Instance Variables

```ruby
# features/support/world.rb

module ShoppingWorld
  attr_accessor :current_user, :current_order, :cart_items

  def initialize
    @cart_items = []
  end

  def logged_in?
    !@current_user.nil?
  end

  def add_to_cart(product)
    @cart_items << product
  end

  def cart_total
    @cart_items.sum { |item| item[:price] }
  end
end

World(ShoppingWorld)
```

```ruby
# features/step_definitions/login_steps.rb

Given('a registered user with email {string}') do |email|
  @current_user = UserRepository.find_by_email(email)
  raise "User not found: #{email}" unless @current_user
end

When('the user logs in with password {string}') do |password|
  login_page = LoginPage.new(@driver)
  login_page.login(@current_user.email, password)
end

Then('the user should be on the dashboard') do
  expect(@driver.current_url).to include('/dashboard')
end
```

```ruby
# features/step_definitions/order_steps.rb

Given('the user has items in their cart') do
  raise 'User not logged in' unless logged_in?
  add_to_cart({ name: 'Widget', price: 29.99 })
  add_to_cart({ name: 'Gadget', price: 49.99 })
end

When('the user proceeds to checkout') do
  checkout_page = CheckoutPage.new(@driver)
  @current_order = checkout_page.complete_checkout(@cart_items)
end

Then('the order total should be {float}') do |expected_total|
  expect(cart_total).to eq(expected_total)
  expect(@current_order.total).to eq(expected_total)
end
```

### 3.4 Pros and Cons

| Pros | Cons |
|------|------|
| **Complete Scenario Isolation** - Each scenario is independent and does not affect others | **Requires Additional Configuration** - DI container or World class needs to be set up |
| **Type Safety** - Compile-time dependency checking in JVM | **Dependency Management** - Requires understanding how DI frameworks work |
| **Clean Code** - Constructor injection makes dependencies explicit | **Circular Dependency Risk** - Need to avoid circular references between context classes |
| **Testability** - Can inject mock objects for unit testing | **State Leakage** - Forgetting to clean up may leave residual data |
| **Unified Resource Management** - Resources can be released uniformly in Hooks | **Overuse** - Not all data should be placed in context |
| **Cross Step Def Sharing** - Multiple Step Def classes share the same state | **Concurrency Limitations** - Thread safety must be ensured during parallel execution |

### 3.5 JVM DI Framework Comparison

| DI Framework | Recommendation | Applicable Scenario | Configuration Complexity |
|---------|--------|----------|------------|
| **PicoContainer** | Officially Recommended | Projects not using other DI frameworks | Low |
| **Spring** | Good | Projects with existing Spring ecosystem | Medium |
| **Guice** | Good | Google ecosystem projects | Medium |
| **OpenEJB** | Optional | Java EE projects | High |
| **Weld** | Optional | CDI projects | High |


---

## 4. Builder / Factory Test Data Pattern

### 4.1 Pattern Description

The Builder and Factory patterns in BDD automation are used for **creating test data**. These patterns solve problems such as hard-coded values in test data, scattered data creation logic, and tight coupling between test data and test logic. The Builder pattern is suitable for creating complex domain objects through chained calls; the Factory pattern is suitable for batch-creating preset objects based on type or conditions.

### 4.2 Applicable Scenarios

| Scenario | Description |
|------|------|
| **Complex Object Creation** | Domain objects have multiple optional fields requiring flexible construction |
| **Test Data Deduplication** | Multiple scenarios use test data with the same structure |
| **Data Randomization** | Need to dynamically generate unique values (email, username, etc.) |
| **Environment Isolation** | Different environments use different test data configurations |
| **Database Seeding** | Preset data needs to be created before scenario execution |

### 4.3 Complete Code Examples

#### 4.3.1 Java - Builder Pattern

**User.java - Domain Model:**
```java
package model;

import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

/**
 * User domain model - created via Builder
 */
public class User {
    private String userId;
    private String username;
    private String email;
    private String password;
    private String firstName;
    private String lastName;
    private LocalDate dateOfBirth;
    private String role;           // ADMIN, CUSTOMER, MANAGER
    private String subscription;   // FREE, BASIC, PREMIUM
    private boolean active;
    private List<String> addresses;
    private String phoneNumber;

    private User(Builder builder) {
        this.userId = builder.userId;
        this.username = builder.username;
        this.email = builder.email;
        this.password = builder.password;
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.dateOfBirth = builder.dateOfBirth;
        this.role = builder.role;
        this.subscription = builder.subscription;
        this.active = builder.active;
        this.addresses = builder.addresses;
        this.phoneNumber = builder.phoneNumber;
    }

    // Getters
    public String getUserId() { return userId; }
    public String getUsername() { return username; }
    public String getEmail() { return email; }
    public String getPassword() { return password; }
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public String getFullName() { return firstName + " " + lastName; }
    public LocalDate getDateOfBirth() { return dateOfBirth; }
    public String getRole() { return role; }
    public String getSubscription() { return subscription; }
    public boolean isActive() { return active; }
    public List<String> getAddresses() { return addresses; }
    public String getPhoneNumber() { return phoneNumber; }
    public void setUserId(String userId) { this.userId = userId; }

    @Override
    public String toString() {
        return "User{username='" + username + "', email='" + email + "', role='" + role + "'}";
    }

    /**
     * Builder inner class
     */
    public static class Builder {
        private String userId;
        private String username;
        private String email;
        private String password = "DefaultPass123!";
        private String firstName = "Test";
        private String lastName = "User";
        private LocalDate dateOfBirth;
        private String role = "CUSTOMER";
        private String subscription = "FREE";
        private boolean active = true;
        private List<String> addresses = new ArrayList<>();
        private String phoneNumber;

        public Builder withUsername(String username) {
            this.username = username;
            return this;
        }

        public Builder withEmail(String email) {
            this.email = email;
            return this;
        }

        public Builder withPassword(String password) {
            this.password = password;
            return this;
        }

        public Builder withFirstName(String firstName) {
            this.firstName = firstName;
            return this;
        }

        public Builder withLastName(String lastName) {
            this.lastName = lastName;
            return this;
        }

        public Builder withDateOfBirth(LocalDate dob) {
            this.dateOfBirth = dob;
            return this;
        }

        public Builder withRole(String role) {
            this.role = role;
            return this;
        }

        public Builder withSubscription(String subscription) {
            this.subscription = subscription;
            return this;
        }

        public Builder isActive(boolean active) {
            this.active = active;
            return this;
        }

        public Builder withAddress(String address) {
            this.addresses.add(address);
            return this;
        }

        public Builder withPhoneNumber(String phone) {
            this.phoneNumber = phone;
            return this;
        }

        public User build() {
            if (this.userId == null) {
                this.userId = UUID.randomUUID().toString();
            }
            if (this.email == null && this.username != null) {
                this.email = this.username + "@example.com";
            }
            return new User(this);
        }
    }
}
```

**UserBuilder.java - Enhanced Builder (with Random Data Generation):**
```java
package builders;

import model.User;
import java.time.LocalDate;
import java.util.UUID;

/**
 * Enhanced User Builder - provides preset user templates and random data generation
 */
public class UserBuilder {

    /**
     * Create a new Builder instance
     */
    public static User.Builder aUser() {
        return new User.Builder();
    }

    /**
     * Create a standard customer user
     */
    public static User aStandardCustomer() {
        return new User.Builder()
            .withUsername("customer_" + randomSuffix())
            .withFirstName("John")
            .withLastName("Doe")
            .withRole("CUSTOMER")
            .withSubscription("FREE")
            .isActive(true)
            .build();
    }

    /**
     * Create a premium subscription customer
     */
    public static User aPremiumCustomer() {
        return new User.Builder()
            .withUsername("premium_" + randomSuffix())
            .withFirstName("Jane")
            .withLastName("Smith")
            .withRole("CUSTOMER")
            .withSubscription("PREMIUM")
            .isActive(true)
            .build();
    }

    /**
     * Create an admin user
     */
    public static User anAdmin() {
        return new User.Builder()
            .withUsername("admin_" + randomSuffix())
            .withFirstName("Admin")
            .withLastName("User")
            .withRole("ADMIN")
            .withSubscription("PREMIUM")
            .isActive(true)
            .build();
    }

    /**
     * Create an inactive user
     */
    public static User anInactiveUser() {
        return new User.Builder()
            .withUsername("inactive_" + randomSuffix())
            .withFirstName("Inactive")
            .withLastName("User")
            .withRole("CUSTOMER")
            .isActive(false)
            .build();
    }

    /**
     * Create a minor user
     */
    public static User aMinorUser() {
        return new User.Builder()
            .withUsername("minor_" + randomSuffix())
            .withFirstName("Young")
            .withLastName("User")
            .withDateOfBirth(LocalDate.now().minusYears(16))
            .withRole("CUSTOMER")
            .build();
    }

    private static String randomSuffix() {
        return UUID.randomUUID().toString().substring(0, 8);
    }

    public static String randomEmail() {
        return "test_" + randomSuffix() + "@example.com";
    }

    public static String randomUsername() {
        return "user_" + randomSuffix();
    }
}
```

**Using Builder in Step Definitions:**
```java
package stepdefinitions;

import io.cucumber.java.en.*;
import context.TestContext;
import model.User;
import builders.UserBuilder;
import static org.junit.jupiter.api.Assertions.*;

public class UserManagementSteps {

    private final TestContext context;

    public UserManagementSteps(TestContext context) {
        this.context = context;
    }

    @Given("a standard customer user exists")
    public void a_standard_customer_exists() {
        User user = UserBuilder.aStandardCustomer();
        UserRepository.save(user);
        context.getScenarioContext().set("currentUser", user);
    }

    @Given("a premium customer with name {string} exists")
    public void a_premium_customer_exists(String name) {
        String[] names = name.split(" ");
        User user = UserBuilder.aPremiumCustomer();
        User customizedUser = new User.Builder()
            .withUsername(user.getUsername())
            .withFirstName(names[0])
            .withLastName(names.length > 1 ? names[1] : "")
            .withEmail(user.getEmail())
            .withRole(user.getRole())
            .withSubscription(user.getSubscription())
            .build();
        UserRepository.save(customizedUser);
        context.getScenarioContext().set("currentUser", customizedUser);
    }

    @Given("an inactive user exists")
    public void an_inactive_user_exists() {
        User user = UserBuilder.anInactiveUser();
        UserRepository.save(user);
        context.getScenarioContext().set("currentUser", user);
    }

    @Given("an admin user exists")
    public void an_admin_user_exists() {
        User user = UserBuilder.anAdmin();
        UserRepository.save(user);
        context.getScenarioContext().set("currentUser", user);
    }

    @When("the admin activates the user's account")
    public void admin_activates_account() {
        User user = context.getScenarioContext().get("currentUser");
        UserService.activate(user.getUserId());
    }

    @Then("the user should be active")
    public void user_should_be_active() {
        User user = context.getScenarioContext().get("currentUser");
        User updated = UserRepository.findById(user.getUserId());
        assertTrue(updated.isActive());
    }
}
```

#### 4.3.2 JavaScript - Factory Pattern

```javascript
// builders/UserFactory.js

const { faker } = require('@faker-js/faker');

/**
 * UserFactory - creates test users using Factory pattern
 */
class UserFactory {

  static createBaseUser(overrides = {}) {
    const firstName = overrides.firstName || faker.person.firstName();
    const lastName = overrides.lastName || faker.person.lastName();

    return {
      userId: faker.string.uuid(),
      username: overrides.username || faker.internet.userName({ firstName, lastName }),
      email: overrides.email || faker.internet.email({ firstName, lastName }),
      password: overrides.password || 'TestPass123!',
      firstName,
      lastName,
      dateOfBirth: overrides.dateOfBirth || faker.date.birthdate({ min: 18, max: 65, mode: 'age' }),
      role: overrides.role || 'CUSTOMER',
      subscription: overrides.subscription || 'FREE',
      active: overrides.active !== undefined ? overrides.active : true,
      addresses: overrides.addresses || [],
      phoneNumber: overrides.phoneNumber || faker.phone.number(),
      ...overrides
    };
  }

  static createCustomer(overrides = {}) {
    return this.createBaseUser({
      role: 'CUSTOMER',
      subscription: 'FREE',
      ...overrides
    });
  }

  static createPremiumCustomer(overrides = {}) {
    return this.createBaseUser({
      role: 'CUSTOMER',
      subscription: 'PREMIUM',
      ...overrides
    });
  }

  static createAdmin(overrides = {}) {
    return this.createBaseUser({
      role: 'ADMIN',
      subscription: 'PREMIUM',
      ...overrides
    });
  }

  static createInactiveUser(overrides = {}) {
    return this.createBaseUser({
      active: false,
      ...overrides
    });
  }

  static createMinor(overrides = {}) {
    const birthDate = new Date();
    birthDate.setFullYear(birthDate.getFullYear() - 16);
    return this.createBaseUser({
      dateOfBirth: birthDate,
      ...overrides
    });
  }

  static createUsers(count, template = {}) {
    return Array.from({ length: count }, () =>
      this.createBaseUser(template)
    );
  }
}

/**
 * ProductFactory - product data factory
 */
class ProductFactory {
  static createProduct(overrides = {}) {
    return {
      productId: faker.string.uuid(),
      name: overrides.name || faker.commerce.productName(),
      category: overrides.category || faker.commerce.department(),
      price: overrides.price || parseFloat(faker.commerce.price()),
      stock: overrides.stock !== undefined ? overrides.stock : faker.number.int({ min: 0, max: 1000 }),
      available: overrides.available !== undefined ? overrides.available : true,
      description: overrides.description || faker.commerce.productDescription(),
      ...overrides
    };
  }

  static createOutOfStockProduct(overrides = {}) {
    return this.createProduct({
      stock: 0,
      available: false,
      ...overrides
    });
  }

  static createProducts(count, template = {}) {
    return Array.from({ length: count }, () =>
      this.createProduct(template)
    );
  }
}

module.exports = { UserFactory, ProductFactory };
```

**Using Factory in Step Definitions:**
```javascript
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('@playwright/test');
const { UserFactory, ProductFactory } = require('../builders/UserFactory');

Given('a registered customer exists', async function() {
  this.user = UserFactory.createCustomer();
  await this.api.createUser(this.user);
  this.set('currentUser', this.user);
});

Given('a premium customer named {string} exists', async function(name) {
  const [firstName, lastName] = name.split(' ');
  this.user = UserFactory.createPremiumCustomer({ firstName, lastName });
  await this.api.createUser(this.user);
  this.set('currentUser', this.user);
});

Given('{int} products exist in the catalog', async function(count) {
  this.products = ProductFactory.createProducts(count);
  for (const product of this.products) {
    await this.api.createProduct(product);
  }
  this.set('products', this.products);
});

Given('an out of stock product exists', async function() {
  this.product = ProductFactory.createOutOfStockProduct();
  await this.api.createProduct(this.product);
  this.set('currentProduct', this.product);
});
```

#### 4.3.3 Ruby - Factory Pattern

```ruby
# features/support/factories/user_factory.rb

require 'faker'
require 'securerandom'

module Factories
  class UserFactory
    def self.create_customer(attrs = {})
      defaults = {
        username: "customer_#{SecureRandom.hex(4)}",
        email: Faker::Internet.email,
        password: 'TestPass123!',
        first_name: Faker::Name.first_name,
        last_name: Faker::Name.last_name,
        role: 'CUSTOMER',
        subscription: 'FREE',
        active: true
      }
      User.create!(defaults.merge(attrs))
    end

    def self.create_admin(attrs = {})
      create_customer(attrs.merge(role: 'ADMIN', subscription: 'PREMIUM'))
    end

    def self.create_premium(attrs = {})
      create_customer(attrs.merge(subscription: 'PREMIUM'))
    end

    def self.create_inactive(attrs = {})
      create_customer(attrs.merge(active: false))
    end
  end
end
```

```ruby
# features/step_definitions/user_steps.rb

Given('a registered customer exists') do
  @current_user = Factories::UserFactory.create_customer
end

Given('an admin user exists') do
  @current_user = Factories::UserFactory.create_admin
end

Given('the following users exist:') do |table|
  @users = table.hashes.map do |row|
    Factories::UserFactory.create_customer(
      username: row['username'],
      email: row['email'],
      role: row['role']
    )
  end
end
```

### 4.4 Pros and Cons

| Pros | Cons |
|------|------|
| **Eliminates Hard-coding** - Randomly generates unique values to avoid conflicts | **Additional Code** - Requires maintaining Builder/Factory classes |
| **High Readability** - `UserBuilder.aStandardCustomer()` is semantically clear | **Initialization Overhead** - Creating many objects may affect performance |
| **High Flexibility** - Customize objects by overriding parameters | **Over-abstraction** - Simple objects don't need Builder |
| **Centralized Management** - Test data logic is unified in one place | **Maintenance Cost** - Builder needs to be updated when domain model changes |
| **Preset Templates** - Commonly used object combinations are reusable | **Implicit Dependencies** - May hide dependencies between test data |


---

## 5. Scenario Outline Data-Driven Pattern

### 5.1 Pattern Description

Scenario Outline is the **data-driven testing** mechanism provided by Gherkin. It allows the same scenario to be run multiple times with different data combinations, separating test logic from test data. `Scenario Outline` itself never runs directly; its steps are interpreted as a **template**; each row of data in the `Examples` table triggers a complete scenario execution.

This is the core pattern for implementing data-driven testing in BDD, suitable for scenarios that need to verify how the same business rule behaves under different inputs.

### 5.2 Applicable Scenarios

| Scenario | Description |
|------|------|
| **Form Validation** | Use different inputs to verify multiple validation rules of the same form |
| **Login Testing** | Authentication scenarios with different username/password combinations |
| **Permission Verification** | Access permission differences for the same feature across different roles |
| **Boundary Value Testing** | Behavior of the same feature under different boundary conditions |
| **Calculation Logic** | Input different parameters to verify calculation results |
| **Internationalization** | Use different language/region data to verify multilingual support |

### 5.3 Complete Code Examples

#### 5.3.1 Gherkin Template

**login-validation.feature - Login form validation:**
```gherkin
@regression @authentication @data-driven
Feature: Login Form Validation
  As a security-conscious user
  I want the login form to validate my input
  So that I can be informed of any errors

  Background:
    Given the user is on the login page

  # ============================================
  # Basic Scenario Outline - username and password validation
  # ============================================
  @negative @smoke
  Scenario Outline: Unsuccessful login with various invalid credentials
    When the user enters username "<username>"
    And the user enters password "<password>"
    And the user clicks the login button
    Then the error message should be "<error_message>"
    And the user should remain on the login page

    Examples:
      | username              | password    | error_message            |
      |                       | password123 | Username is required     |
      | john@example.com      |             | Password is required     |
      |                       |             | Both fields are required |

  # ============================================
  # Extended validation - format validation
  # ============================================
  @negative @format-validation
  Scenario Outline: Login with invalid format inputs
    When the user enters username "<username>"
    And the user enters password "<password>"
    And the user clicks the login button
    Then the field "<field>" should show error "<error_message>"

    Examples:
      | username         | password   | field    | error_message                      |
      | invalid-email    | password123| username | Please enter a valid email address |
      | test@example.com | short      | password | Password must be at least 8 chars  |
      | test@example.com | abcdefgh   | password | Password must contain a number     |

  # ============================================
  # Multiple Examples tag sets
  # ============================================
  @positive @smoke
  Scenario Outline: Successful login with different valid user types
    When the user logs in with "<username>" and "<password>"
    Then the user should be redirected to "<landing_page>"
    And the user role should be "<role>"

    @admin
    Examples: Admin Users
      | username        | password    | landing_page    | role  |
      | admin@test.com  | AdminPass1! | /admin/dashboard| ADMIN |

    @customer
    Examples: Regular Customers
      | username          | password    | landing_page    | role     |
      | john@test.com     | JohnPass1!  | /dashboard      | CUSTOMER |
      | jane@test.com     | JanePass1!  | /dashboard      | CUSTOMER |

    @manager
    Examples: Store Managers
      | username          | password     | landing_page    | role    |
      | manager@test.com  | MgrPass1!    | /manager/portal | MANAGER |
```

#### 5.3.2 Java Step Definition Implementation

```java
package stepdefinitions;

import io.cucumber.java.en.*;
import context.TestContext;
import pages.LoginPage;
import pages.DashboardPage;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Login validation step definitions - supports data-driven testing with Scenario Outline
 */
public class LoginValidationSteps {

    private final TestContext context;
    private final LoginPage loginPage;
    private final DashboardPage dashboardPage;

    public LoginValidationSteps(TestContext context) {
        this.context = context;
        this.loginPage = context.getPageObjectManager().getLoginPage();
        this.dashboardPage = context.getPageObjectManager().getDashboardPage();
    }

    // ---- Background steps ----

    @Given("the user is on the login page")
    public void the_user_is_on_the_login_page() {
        loginPage.open();
        assertTrue(loginPage.getCurrentUrl().contains("/login"),
            "Expected to be on login page");
    }

    // ---- Scenario Outline steps ----

    @When("the user enters username {string}")
    public void the_user_enters_username(String username) {
        loginPage.enterUsername(username);
        context.getScenarioContext().set("lastUsername", username);
    }

    @When("the user enters password {string}")
    public void the_user_enters_password(String password) {
        loginPage.enterPassword(password);
        context.getScenarioContext().set("lastPassword", password);
    }

    @When("the user clicks the login button")
    public void the_user_clicks_the_login_button() {
        loginPage.clickLoginButton();
    }

    @When("the user logs in with {string} and {string}")
    public void the_user_logs_in_with(String username, String password) {
        loginPage.login(username, password);
        context.getScenarioContext().setUsername(username);
    }

    // ---- Then steps ----

    @Then("the error message should be {string}")
    public void the_error_message_should_be(String expectedMessage) {
        String actualMessage = loginPage.getErrorMessage();
        assertEquals(expectedMessage, actualMessage,
            "Error message does not match");
    }

    @Then("the user should remain on the login page")
    public void the_user_should_remain_on_the_login_page() {
        String currentUrl = loginPage.getCurrentUrl();
        assertTrue(currentUrl.contains("/login"),
            "Expected to remain on login page but was: " + currentUrl);
    }

    @Then("the field {string} should show error {string}")
    public void the_field_should_show_error(String field, String expectedMessage) {
        String actualMessage;
        if (field.equals("username")) {
            actualMessage = loginPage.getUsernameFieldError();
        } else if (field.equals("password")) {
            actualMessage = loginPage.getPasswordFieldError();
        } else {
            fail("Unknown field: " + field);
            return;
        }
        assertEquals(expectedMessage, actualMessage);
    }

    @Then("the user should be redirected to {string}")
    public void the_user_should_be_redirected_to(String expectedPath) {
        String currentUrl = dashboardPage.getCurrentUrl();
        assertTrue(currentUrl.contains(expectedPath),
            "Expected URL to contain '" + expectedPath + "' but was: " + currentUrl);
    }

    @Then("the user role should be {string}")
    public void the_user_role_should_be(String expectedRole) {
        String actualRole = dashboardPage.getUserRole();
        assertEquals(expectedRole, actualRole);
    }
}
```

#### 5.3.3 JavaScript Step Definition Implementation

```javascript
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('@playwright/test');
const { LoginPage } = require('../pages/LoginPage');
const { DashboardPage } = require('../pages/DashboardPage');

Given('the user is on the login page', async function() {
  if (!this.page) {
    this.page = await this.getPage();
  }
  this.loginPage = new LoginPage(this.page);
  this.dashboardPage = new DashboardPage(this.page);
  await this.loginPage.open();
});

When('the user enters username {string}', async function(username) {
  await this.loginPage.enterUsername(username);
  this.set('lastUsername', username);
});

When('the user enters password {string}', async function(password) {
  await this.loginPage.enterPassword(password);
  this.set('lastPassword', password);
});

When('the user clicks the login button', async function() {
  await this.loginPage.clickLoginButton();
});

When('the user logs in with {string} and {string}', async function(username, password) {
  await this.loginPage.login(username, password);
  this.setUsername(username);
});

Then('the error message should be {string}', async function(expectedMessage) {
  const actualMessage = await this.loginPage.getErrorMessage();
  expect(actualMessage).toBe(expectedMessage);
});

Then('the user should remain on the login page', async function() {
  const currentUrl = await this.loginPage.getCurrentUrl();
  expect(currentUrl).toContain('/login');
});

Then('the field {string} should show error {string}', async function(field, expectedMessage) {
  let actualMessage;
  if (field === 'username') {
    actualMessage = await this.loginPage.getUsernameFieldError();
  } else if (field === 'password') {
    actualMessage = await this.loginPage.getPasswordFieldError();
  }
  expect(actualMessage).toBe(expectedMessage);
});

Then('the user should be redirected to {string}', async function(expectedPath) {
  const currentUrl = await this.dashboardPage.getCurrentUrl();
  expect(currentUrl).toContain(expectedPath);
});

Then('the user role should be {string}', async function(expectedRole) {
  const actualRole = await this.dashboardPage.getUserRole();
  expect(actualRole).toBe(expectedRole);
});
```

#### 5.3.4 Data Table vs Scenario Outline Usage Comparison

```gherkin
@regression @shopping
Feature: Shopping Cart Calculations
  As a customer
  I want the cart total to be calculated correctly
  So that I know exactly how much I'll pay

  # ============================================
  # Data Table - step-level batch data (single execution)
  # ============================================
  @smoke
  Scenario: Calculate total for multiple items
    Given the cart contains the following items:
      | product        | quantity | unit_price |
      | Widget         | 2        | 10.00      |
      | Gadget         | 1        | 25.00      |
      | Thingamajig    | 3        | 5.00       |
    When the cart total is calculated
    Then the subtotal should be 70.00
    And the item count should be 6

  # ============================================
  # Scenario Outline - scenario-level multiple data (multiple executions)
  # ============================================
  @data-driven
  Scenario Outline: Apply discount codes
    Given the cart has a subtotal of <subtotal>
    When the user applies discount code "<code>"
    Then the discount should be <discount>
    And the final total should be <total>

    Examples:
      | subtotal | code    | discount | total  |
      | 100.00   | SAVE10  | 10.00    | 90.00  |
      | 200.00   | SAVE20  | 40.00    | 160.00 |
      | 50.00    | SAVE50  | 25.00    | 25.00  |
      | 100.00   | INVALID | 0.00     | 100.00 |

  # ============================================
  # Multi-column Examples - complex data scenarios
  # ============================================
  @edge-cases
  Scenario Outline: Shipping cost calculation
    Given the user is in "<country>"
    And the order weight is <weight> kg
    And the shipping method is "<method>"
    When the shipping cost is calculated
    Then the shipping cost should be <cost>
    And the delivery estimate should be "<estimate>"

    Examples:
      | country | weight | method     | cost | estimate    |
      | US      | 0.5    | standard   | 5.00 | 5-7 days    |
      | US      | 0.5    | express    | 12.00| 2-3 days    |
      | US      | 5.0    | standard   | 10.00| 5-7 days    |
      | CA      | 0.5    | standard   | 8.00 | 7-10 days   |
      | CA      | 2.0    | express    | 20.00| 3-5 days    |
      | UK      | 1.0    | standard   | 15.00| 10-14 days  |
```

#### 5.3.5 Java - Data Table Handling

```java
package stepdefinitions;

import io.cucumber.java.en.*;
import io.cucumber.datatable.DataTable;
import context.TestContext;
import model.CartItem;
import java.math.BigDecimal;
import java.util.List;
import java.util.Map;
import static org.junit.jupiter.api.Assertions.*;

public class CartSteps {

    private final TestContext context;

    public CartSteps(TestContext context) {
        this.context = context;
    }

    /**
     * Data Table handling - converted to List<Map<String, String>>
     * Headers as keys, each row as a Map
     */
    @Given("the cart contains the following items:")
    public void the_cart_contains_the_following_items(DataTable dataTable) {
        List<Map<String, String>> rows = dataTable.asMaps();

        for (Map<String, String> row : rows) {
            String product = row.get("product");
            int quantity = Integer.parseInt(row.get("quantity"));
            BigDecimal unitPrice = new BigDecimal(row.get("unit_price"));

            CartItem item = new CartItem(product, quantity, unitPrice);
            context.getCart().addItem(item);
        }
    }

    /**
     * Two-column Data Table converted to Map
     */
    @Given("the following user preferences:")
    public void the_following_user_preferences(DataTable dataTable) {
        Map<String, String> preferences = dataTable.asMap();
        String theme = preferences.get("theme");
        String language = preferences.get("language");
        context.getScenarioContext().set("preferences", preferences);
    }

    /**
     * Single-column Data Table converted to List
     */
    @Given("the user has selected the following categories:")
    public void user_selected_categories(DataTable dataTable) {
        List<String> categories = dataTable.asList();
        context.getScenarioContext().set("categories", categories);
    }

    @When("the cart total is calculated")
    public void the_cart_total_is_calculated() {
        context.getCart().calculateTotal();
    }

    @Then("the subtotal should be {bigdecimal}")
    public void the_subtotal_should_be(BigDecimal expectedSubtotal) {
        BigDecimal actualSubtotal = context.getCart().getSubtotal();
        assertEquals(expectedSubtotal, actualSubtotal);
    }

    @Then("the item count should be {int}")
    public void the_item_count_should_be(int expectedCount) {
        int actualCount = context.getCart().getTotalItemCount();
        assertEquals(expectedCount, actualCount);
    }

    // ---- Scenario Outline steps ----

    @Given("the cart has a subtotal of {bigdecimal}")
    public void cart_has_subtotal(BigDecimal subtotal) {
        context.getCart().setSubtotal(subtotal);
    }

    @When("the user applies discount code {string}")
    public void user_applies_discount(String code) {
        context.getCart().applyDiscount(code);
    }

    @Then("the discount should be {bigdecimal}")
    public void discount_should_be(BigDecimal expectedDiscount) {
        BigDecimal actualDiscount = context.getCart().getDiscount();
        assertEquals(expectedDiscount, actualDiscount);
    }

    @Then("the final total should be {bigdecimal}")
    public void final_total_should_be(BigDecimal expectedTotal) {
        BigDecimal actualTotal = context.getCart().getFinalTotal();
        assertEquals(expectedTotal, actualTotal);
    }
}
```

#### 5.3.6 JavaScript - Data Table Handling

```javascript
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('@playwright/test');

/**
 * Data Table handling - JavaScript
 */
Given('the cart contains the following items:', async function(dataTable) {
  const rows = dataTable.hashes();

  for (const row of rows) {
    const product = row.product;
    const quantity = parseInt(row.quantity);
    const unitPrice = parseFloat(row.unit_price);
    await this.cart.addItem({ product, quantity, unitPrice });
  }
});

Given('the following user preferences:', async function(dataTable) {
  const preferences = dataTable.rowsHash();
  this.set('preferences', preferences);
});

Given('the user has selected the following categories:', async function(dataTable) {
  const categories = dataTable.raw().flat();
  this.set('categories', categories);
});
```

#### 5.3.7 Dynamic Examples - Runtime Data

For scenarios requiring dynamically generated test data, Examples can be generated in combination with code:

```java
package stepdefinitions;

import io.cucumber.java.en.*;
import context.TestContext;
import java.util.stream.Stream;

/**
 * Dynamic data generation - for complex data-driven scenarios
 */
public class DynamicDataSteps {

    /**
     * Use custom parameter types to handle dynamic data markers
     * {random} markers are replaced with actual values in step definitions
     */
    @Given("a user with email {string}")
    public void a_user_with_email(String email) {
        String processedEmail = email.replace("{random}",
            "user_" + System.currentTimeMillis() + "@example.com");
        User user = UserBuilder.aUser()
            .withEmail(processedEmail)
            .build();
        UserRepository.save(user);
        context.getScenarioContext().set("currentUser", user);
    }
}
```

```gherkin
# Dynamic data example
Scenario Outline: Registration with unique emails
  Given a user with email "<email>"
  When the user registers
  Then the registration should be successful

  Examples:
    | email                          |
    | {random}@test.com              |
    | user_{random}@example.com      |
```

### 5.4 Data Table vs Scenario Outline Comparison

| Feature | Data Table | Scenario Outline + Examples |
|------|-----------|----------------------------|
| **Data Injection Level** | Step level | Scenario level |
| **Execution Count** | Single execution with multiple rows of data | Each row executes a complete scenario |
| **Applicable Scenarios** | Batch operations (creating multiple records) | Multiple data combination tests for the same logic |
| **Syntax Position** | Immediately after the step | Examples table at the end of the scenario |
| **Failure Isolation** | Entire step fails | Only affects the scenario for the corresponding data row |
| **Report Readability** | Single test line | Multiple independent test reports |
| **Maintainability** | Data volume should not be too large | Data volume can be larger |

### 5.5 Pros and Cons

| Pros | Cons |
|------|------|
| **Data and Logic Separation** - Scenario templates and data are maintained independently | **May Cause Scenario Bloat** - Too many Examples rows reduces readability |
| **Write Once, Run Multiple Times** - Reduces duplicate scenario code | **Complex Debugging** - Need to determine which data row caused the failure |
| **Improved Coverage** - Easy to cover multiple boundary conditions | **Increased Execution Time** - Each data row runs a complete scenario |
| **High Readability** - Tabular data is intuitive and clear | **Overuse** - Not suitable for testing all permutations (unit tests should be used) |
| **Low Maintenance Cost** - Adding test data only requires adding a row | **Shared Background** - All Examples rows share the Background |
| **Tag Granularity Control** - Examples can be grouped with tags | **Limited Parameterization** - Only supports simple string replacement |


---

## 6. Step Definition Organization Pattern

### 6.1 Pattern Description

The Step Definition organization pattern focuses on how to **properly organize and name step definition files**. This is a key factor affecting the long-term maintainability of BDD projects. The core principle is to **organize by Domain Concept**, not by feature or scenario. This maximizes step definition reuse and avoids the Feature-Coupled Step Definitions anti-pattern.

### 6.2 Applicable Scenarios

| Scenario | Description |
|------|------|
| **Large Projects** | Multiple Feature files and large numbers of Step Definitions |
| **Cross-Feature Reuse** | Same steps used across multiple Features |
| **Multi-person Collaboration** | Multiple developers maintaining Step Defs for different modules |
| **Domain-Driven Design** | Aligned with DDD domain concepts |
| **Long-term Maintenance** | Long project lifecycle requiring sustainable code structure |

### 6.3 Organization Strategies

#### 6.3.1 Recommended Directory Structure

```
project/
├── src/test/
│   ├── java/
│   │   ├── runners/                     # Test runner
│   │   │   └── TestRunner.java
│   │   ├── stepdefinitions/             # Step definitions (grouped by domain concept)
│   │   │   ├── AuthenticationSteps.java # Authentication/login domain
│   │   │   ├── UserManagementSteps.java # User management domain
│   │   │   ├── CartSteps.java           # Shopping cart domain
│   │   │   ├── CheckoutSteps.java       # Checkout domain
│   │   │   ├── SearchSteps.java         # Search domain
│   │   │   ├── NavigationSteps.java     # Navigation domain
│   │   │   └── CommonSteps.java         # Common steps
│   │   ├── pages/                       # Page Objects
│   │   │   ├── BasePage.java
│   │   │   ├── LoginPage.java
│   │   │   ├── DashboardPage.java
│   │   │   ├── CartPage.java
│   │   │   └── CheckoutPage.java
│   │   ├── context/                     # Shared context
│   │   │   └── TestContext.java
│   │   ├── builders/                    # Test data builders
│   │   │   ├── UserBuilder.java
│   │   │   └── ProductBuilder.java
│   │   ├── model/                       # Domain models
│   │   │   ├── User.java
│   │   │   ├── Product.java
│   │   │   └── CartItem.java
│   │   └── hooks/                       # Hooks
│   │       └── TestHooks.java
│   └── resources/
│       ├── features/                    # Feature files
│       │   ├── authentication/
│       │   │   ├── login.feature
│       │   │   ├── registration.feature
│       │   │   └── password-reset.feature
│       │   ├── shopping/
│       │   │   ├── product-search.feature
│       │   │   ├── add-to-cart.feature
│       │   │   └── apply-discount.feature
│       │   └── checkout/
│       │       ├── payment.feature
│       │       ├── shipping.feature
│       │       └── order-confirmation.feature
│       └── cucumber.properties
└── pom.xml
```

#### 6.3.2 Java Implementation - Grouped by Domain

**AuthenticationSteps.java - Authentication Domain Steps:**
```java
package stepdefinitions;

import io.cucumber.java.en.*;
import context.TestContext;
import pages.LoginPage;
import pages.RegistrationPage;
import pages.PasswordResetPage;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Authentication domain step definitions
 * Covers login, registration, password reset, and other authentication-related operations
 */
public class AuthenticationSteps {

    private final TestContext context;
    private final LoginPage loginPage;
    private final RegistrationPage registrationPage;
    private final PasswordResetPage passwordResetPage;

    public AuthenticationSteps(TestContext context) {
        this.context = context;
        this.loginPage = context.getPageObjectManager().getLoginPage();
        this.registrationPage = context.getPageObjectManager().getRegistrationPage();
        this.passwordResetPage = context.getPageObjectManager().getPasswordResetPage();
    }

    // ---- Login-related steps ----

    @Given("the user is on the login page")
    public void user_on_login_page() {
        loginPage.open();
    }

    @When("the user logs in with {string} and {string}")
    public void user_logs_in(String username, String password) {
        loginPage.login(username, password);
        context.getScenarioContext().setUsername(username);
    }

    @When("the user enters {string} as username")
    public void enters_username(String username) {
        loginPage.enterUsername(username);
    }

    @When("the user enters {string} as password")
    public void enters_password(String password) {
        loginPage.enterPassword(password);
    }

    @When("the user clicks the login button")
    public void clicks_login_button() {
        loginPage.clickLoginButton();
    }

    @Then("the login should be successful")
    public void login_successful() {
        assertTrue(context.getPageObjectManager()
            .getDashboardPage().isOnDashboard());
    }

    @Then("the login should fail with error {string}")
    public void login_fails(String error) {
        assertEquals(error, loginPage.getErrorMessage());
    }

    // ---- Registration-related steps ----

    @Given("the user is on the registration page")
    public void user_on_registration_page() {
        registrationPage.open();
    }

    @When("the user registers with:")
    public void user_registers_with(io.cucumber.datatable.DataTable data) {
        Map<String, String> details = data.asMap();
        registrationPage.fillForm(details);
        registrationPage.submit();
    }

    @Then("the registration should be successful")
    public void registration_successful() {
        assertTrue(registrationPage.isSuccessMessageDisplayed());
    }

    // ---- Password reset steps ----

    @When("the user requests password reset for {string}")
    public void request_password_reset(String email) {
        passwordResetPage.requestReset(email);
    }

    @Then("a reset email should be sent to {string}")
    public void reset_email_sent(String email) {
        assertTrue(passwordResetPage.isResetEmailSent(email));
    }
}
```

**CartSteps.java - Shopping Cart Domain Steps:**
```java
package stepdefinitions;

import io.cucumber.java.en.*;
import io.cucumber.datatable.DataTable;
import context.TestContext;
import pages.CartPage;
import pages.ProductPage;
import java.math.BigDecimal;
import java.util.List;
import java.util.Map;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Shopping cart domain step definitions
 * Covers adding products, changing quantities, calculating totals, and other cart operations
 */
public class CartSteps {

    private final TestContext context;
    private final CartPage cartPage;
    private final ProductPage productPage;

    public CartSteps(TestContext context) {
        this.context = context;
        this.cartPage = context.getPageObjectManager().getCartPage();
        this.productPage = context.getPageObjectManager().getProductPage();
    }

    // ---- Product search and browsing ----

    @Given("the user searches for {string}")
    public void user_searches_for(String keyword) {
        productPage.search(keyword);
    }

    @When("the user views product {string}")
    public void user_views_product(String productName) {
        productPage.selectProduct(productName);
        context.getScenarioContext().setProductName(productName);
    }

    // ---- Cart operations ----

    @When("the user adds {string} to cart")
    public void user_adds_to_cart(String productName) {
        productPage.addToCart(productName);
        context.getScenarioContext().setProductName(productName);
    }

    @When("the user adds {int} of {string} to cart")
    public void user_adds_quantity_to_cart(int quantity, String productName) {
        productPage.addToCart(productName, quantity);
    }

    @When("the user removes {string} from cart")
    public void user_removes_from_cart(String productName) {
        cartPage.removeItem(productName);
    }

    @When("the user changes quantity of {string} to {int}")
    public void user_changes_quantity(String productName, int quantity) {
        cartPage.updateQuantity(productName, quantity);
    }

    @When("the user clears the cart")
    public void user_clears_cart() {
        cartPage.clearAll();
    }

    @When("the user applies discount code {string}")
    public void user_applies_discount(String code) {
        cartPage.applyDiscount(code);
    }

    // ---- Cart verification ----

    @Then("the cart should contain {int} item(s)")
    public void cart_should_contain(int count) {
        assertEquals(count, cartPage.getItemCount());
    }

    @Then("the cart should contain {string}")
    public void cart_should_contain_product(String productName) {
        assertTrue(cartPage.hasProduct(productName));
    }

    @Then("the cart should not contain {string}")
    public void cart_should_not_contain(String productName) {
        assertFalse(cartPage.hasProduct(productName));
    }

    @Then("the cart subtotal should be {bigdecimal}")
    public void cart_subtotal(BigDecimal expected) {
        assertEquals(expected, cartPage.getSubtotal());
    }

    @Then("the cart total should be {bigdecimal}")
    public void cart_total(BigDecimal expected) {
        assertEquals(expected, cartPage.getTotal());
    }

    @Then("the discount should be {bigdecimal}")
    public void discount_amount(BigDecimal expected) {
        assertEquals(expected, cartPage.getDiscountAmount());
    }

    // ---- Data Table batch operations ----

    @Given("the cart contains:")
    public void cart_contains(DataTable dataTable) {
        List<Map<String, String>> items = dataTable.asMaps();
        for (Map<String, String> item : items) {
            String product = item.get("product");
            int qty = Integer.parseInt(item.get("quantity"));
            productPage.addToCart(product, qty);
        }
    }

    @Then("the cart should display:")
    public void cart_should_display(DataTable dataTable) {
        List<Map<String, String>> expectedItems = dataTable.asMaps();
        for (Map<String, String> expected : expectedItems) {
            String product = expected.get("product");
            int expectedQty = Integer.parseInt(expected.get("quantity"));
            BigDecimal expectedPrice = new BigDecimal(expected.get("price"));
            assertTrue(cartPage.hasProduct(product));
            assertEquals(expectedQty, cartPage.getProductQuantity(product));
            assertEquals(expectedPrice, cartPage.getProductPrice(product));
        }
    }
}
```

**NavigationSteps.java - Navigation Domain Steps:**
```java
package stepdefinitions;

import io.cucumber.java.en.*;
import context.TestContext;
import pages.*;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Navigation domain step definitions
 * Contains all page navigation and general UI operations
 * These steps may be reused across multiple Features
 */
public class NavigationSteps {

    private final TestContext context;

    public NavigationSteps(TestContext context) {
        this.context = context;
    }

    @Given("the user navigates to {string}")
    public void navigates_to(String url) {
        context.getDriver().get(url);
    }

    @When("the user clicks on {string} link")
    public void clicks_link(String linkText) {
        new BasePage(context.getDriver()).clickLink(linkText);
    }

    @When("the user clicks on {string} button")
    public void clicks_button(String buttonText) {
        new BasePage(context.getDriver()).clickButton(buttonText);
    }

    @When("the user goes back")
    public void goes_back() {
        context.getDriver().navigate().back();
    }

    @Then("the page title should contain {string}")
    public void page_title_contains(String expected) {
        String title = context.getDriver().getTitle();
        assertTrue(title.contains(expected),
            "Expected title to contain '" + expected + "' but was: " + title);
    }

    @Then("the current URL should contain {string}")
    public void current_url_contains(String expected) {
        assertTrue(context.getDriver().getCurrentUrl().contains(expected));
    }

    @Then("{string} should be visible")
    public void should_be_visible(String element) {
        assertTrue(new BasePage(context.getDriver()).isElementVisible(element));
    }

    @Then("{string} should not be visible")
    public void should_not_be_visible(String element) {
        assertFalse(new BasePage(context.getDriver()).isElementVisible(element));
    }

    @Then("a message {string} should be displayed")
    public void message_displayed(String message) {
        String pageText = new BasePage(context.getDriver()).getPageText();
        assertTrue(pageText.contains(message));
    }
}
```

**CommonSteps.java - Common Steps:**
```java
package stepdefinitions;

import io.cucumber.java.en.*;
import context.TestContext;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Common step definitions
 * Contains cross-domain basic assertions and helper steps
 */
public class CommonSteps {

    private final TestContext context;

    public CommonSteps(TestContext context) {
        this.context = context;
    }

    @Then("the response should be successful")
    public void response_successful() {
        Integer statusCode = context.getScenarioContext().get("responseStatusCode");
        assertNotNull(statusCode);
        assertTrue(statusCode >= 200 && statusCode < 300,
            "Expected successful response but got: " + statusCode);
    }

    @Then("the response status should be {int}")
    public void response_status(int expectedStatus) {
        Integer actualStatus = context.getScenarioContext().get("responseStatusCode");
        assertEquals(expectedStatus, actualStatus);
    }

    @Then("{string} should equal {string}")
    public void should_equal(String actual, String expected) {
        assertEquals(expected, actual);
    }

    @Then("{string} should contain {string}")
    public void should_contain(String actual, String expected) {
        assertTrue(actual.contains(expected));
    }

    @Then("{int} should be greater than {int}")
    public void should_be_greater_than(int actual, int expected) {
        assertTrue(actual > expected);
    }

    @When("the user waits for {int} seconds")
    public void wait_seconds(int seconds) throws InterruptedException {
        Thread.sleep(seconds * 1000L);
    }

    @When("the user waits for the page to load")
    public void wait_for_page_load() {
        new pages.BasePage(context.getDriver())
            .waitForPageToLoad();
    }
}
```

#### 6.3.3 JavaScript - Grouped by Function

```javascript
// step-definitions/authentication.steps.js
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('@playwright/test');
const { LoginPage } = require('../pages/LoginPage');

Given('the user is on the login page', async function() {
  this.page = await this.getPage();
  this.loginPage = new LoginPage(this.page);
  await this.loginPage.open();
});

When('the user logs in with {string} and {string}', async function(username, password) {
  await this.loginPage.login(username, password);
  this.setUsername(username);
});

Then('the login should be successful', async function() {
  expect(this.page.url()).toContain('/dashboard');
});
```

```javascript
// step-definitions/cart.steps.js
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('@playwright/test');
const { CartPage } = require('../pages/CartPage');

When('the user adds {string} to cart', async function(product) {
  if (!this.cartPage) {
    this.cartPage = new CartPage(this.page);
  }
  await this.cartPage.addProduct(product);
  this.set('lastAddedProduct', product);
});

When('the user removes {string} from cart', async function(product) {
  await this.cartPage.removeProduct(product);
});

Then('the cart should contain {int} item(s)', async function(count) {
  const itemCount = await this.cartPage.getItemCount();
  expect(itemCount).toBe(count);
});

Then('the cart total should be {string}', async function(expectedTotal) {
  const actualTotal = await this.cartPage.getTotal();
  expect(actualTotal).toBe(expectedTotal);
});
```

```javascript
// step-definitions/navigation.steps.js
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('@playwright/test');

Given('the user navigates to {string}', async function(url) {
  if (!this.page) {
    this.page = await this.getPage();
  }
  await this.page.goto(url);
});

When('the user clicks on {string} link', async function(linkText) {
  await this.page.click(`text=${linkText}`);
});

Then('the page title should contain {string}', async function(expected) {
  const title = await this.page.title();
  expect(title).toContain(expected);
});

Then('the current URL should contain {string}', async function(expected) {
  expect(this.page.url()).toContain(expected);
});
```

### 6.4 Organization Principles and Best Practices

#### 6.4.1 Naming Conventions

| Type | Naming Convention | Example |
|------|----------|------|
| **Step Def File** | `{domain}.steps.js` / `{Domain}Steps.java` | `authentication.steps.js` / `CartSteps.java` |
| **Page Object File** | `{PageName}Page.java` / `{page-name}.page.js` | `LoginPage.java` / `login.page.js` |
| **Feature File** | `{feature-name}.feature` | `login.feature`, `add-to-cart.feature` |
| **Context File** | `TestContext.java` / `world.js` | `TestContext.java` |
| **Hook File** | `hooks.js` / `TestHooks.java` | `hooks.js` |

#### 6.4.2 Core Organization Principles

```
1. Group by domain concept, not by Feature
   OK: CartSteps.java (Shopping cart domain)
   BAD: add-to-cart.steps.js (Feature-coupled)

2. One Step Def file covers one business domain
   - AuthenticationSteps: Login, registration, password reset
   - CartSteps: Add to cart, change quantity, clear cart

3. Step naming uses domain language, aligned with UBIQUITOUS LANGUAGE
   - OK: "the user adds {string} to cart"
   - BAD: "the user clicks button with id 'add-to-cart-btn'"

4. Put common steps in CommonSteps / navigation.steps.js
   - Page navigation
   - URL verification
   - General UI assertions

5. Feature files are placed in subdirectories by functional module
   - features/authentication/
   - features/shopping/
   - features/checkout/
```

#### 6.4.3 Anti-pattern Comparison

**Anti-pattern: Feature-Coupled Step Definition**
```
# Not recommended - organized by Feature (code duplication, hard to reuse)
steps/
  login.steps.js          # Contains login steps
  registration.steps.js   # Also contains username input steps
  password-reset.steps.js # Also contains username input steps
```

**Recommended Pattern: Organize by Domain Concept**
```
# Recommended - organized by domain (high reuse, low coupling)
steps/
  authentication.steps.js  # All authentication-related steps
  cart.steps.js            # All shopping cart-related steps
  navigation.steps.js      # All navigation-related steps
```

### 6.5 Pros and Cons

| Pros | Cons |
|------|------|
| **High Reusability** - Same steps are shared across all Features | **Requires Upfront Planning** - Need to identify domain boundaries |
| **Low Coupling** - Step Def does not depend on a specific Feature | **Responsibility Division Challenge** - Some steps may belong to multiple domains |
| **Easy to Maintain** - Change one place, takes effect globally | **Increased File Count** - More files compared to Feature-based organization |
| **Aligned with DDD** - Naturally maps to domain model | **Newcomer Learning Cost** - Need to understand domain concepts to locate code |
| **Avoids Code Duplication** - Common steps are centrally managed | **Over-abstraction** - May create too many tiny Step Def classes |
| **Team Collaboration Friendly** - Different developers maintain different domains | **Refactoring Cost** - Adjusting domain boundaries later requires moving large amounts of code |

---

## Appendix A: Pattern Selection Quick Reference

| Pattern | Applicable Scenario | Complexity | Team Size | Learning Curve |
|------|----------|--------|----------|----------|
| **Page Object** | Web UI automation | Low | Any | Low |
| **Screenplay** | Large complex projects, multi-role interactions | High | Medium-Large | High |
| **Context Injection** | Sharing state between steps | Medium | Any | Medium |
| **Builder/Factory** | Complex test data creation | Medium | Any | Low |
| **Scenario Outline** | Data-driven testing | Low | Any | Low |
| **Step Def Organization** | Large project Step Def management | Low | Medium-Large | Low |

## Appendix B: Multi-Language Syntax Comparison

| Feature | Java (JVM) | JavaScript (Node.js) | Ruby |
|------|-----------|---------------------|------|
| **Step Def Syntax** | Annotation: `@Given("...")` | Function: `Given('...', fn)` | Block: `Given('...') do ... end` |
| **DI Approach** | PicoContainer / Spring | World object (`this`) | World mixin / instance variables |
| **State Sharing** | Constructor injection | `this.key = value` | `@instance_var` |
| **Hook Syntax** | `@Before`, `@After` | `Before`, `After` | `Before`, `After`, `Around` |
| **Data Table** | `DataTable` object | `dataTable.hashes()` | `table.hashes` |
| **Async Support** | Synchronous code | `async/await` | Block/Proc |
| **Page Object** | Class + constructor | Class/constructor function | Module/Class |
| **Builder Pattern** | Inner Builder class | Factory function/class | Factory method |

## Appendix C: Gherkin Data-Driven Syntax Quick Reference

| Syntax | Description | Example |
|------|------|------|
| **Scenario Outline** | Templated scenario | `Scenario Outline: Login with <type>` |
| **Examples** | Provides data table | `Examples: \| type \| ... \|` |
| **Data Table** | Step-level batch data | `Given ... \| col1 \| col2 \|` |
| **Doc String** | Multi-line text parameter | `Given ... """text"""` |
| **Parameter Types** | Automatic type conversion | `{int}`, `{string}`, `{float}`, `{word}` |
| **Optional Text** | Optional words | `cucumber(s)` |
| **Alternative Words** | Synonyms | `belly/stomach` |

---

> **Document Notes**: This patterns.md file is compiled from 4 in-depth research reports, covering BDD core principles, Cucumber framework technical practices, architecture patterns and CI/CD, community practices and tool ecosystems. Each pattern provides complete code examples in Java, JavaScript, and Ruby, ready for direct use in projects.
