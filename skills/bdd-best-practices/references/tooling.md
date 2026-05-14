# BDD Tools and Framework Reference Manual

> Version: 1.0 | Based on 2025 BDD Ecosystem Research
> Applicable Scope: Java / JavaScript / TypeScript / Ruby / .NET / Python

---

## Table of Contents

1. [Framework Versions and Selection](#1-framework-versions-and-selection)
2. [IDE Plugins and Configuration](#2-ide-plugins-and-configuration)
3. [Reporting Tools Comparison](#3-reporting-tools-comparison)
4. [Utility Libraries](#4-utility-libraries)
5. [Living Documentation Tools](#5-living-documentation-tools)
6. [Code Quality Tools](#6-code-quality-tools)

---

## 1. Framework Versions and Selection

### 1.1 Cucumber Ecosystem Overview

> Source: Cucumber 2025 Year in Review[^122^], QA Skills[^25^]

| Framework | Primary Language | Syntax | Latest Version | Maintenance Status |
|-----------|-----------------|--------|----------------|-------------------|
| **Cucumber** | Java, JS, Ruby, Go | Gherkin | See table below | Very Active |
| **SpecFlow** | C#/.NET | Gherkin | v4.x+ | Active |
| **Behave** | Python | Gherkin | v1.2.6+ | Maintained |
| **Gauge** | Java, JS, Python, C#, Ruby, Go | Markdown | v1.x | Active |
| **Reqnroll** | C#/.NET | Gherkin | v2.x+ | Active (SpecFlow successor) |

### 1.2 Cucumber Latest Versions by Language Implementation

#### cucumber-js (JavaScript/TypeScript)

> Source: cucumber-js GitHub Releases[^91^]

| Version | Release Date | Key Changes |
|---------|--------------|-------------|
| **v12.8.3** | 2025-12 | Fixed thrown strings handling |
| **v12.8.2** | 2025-11 | Dependency updates |
| **v12.8.0** | 2025-10 | Custom externalising options support |
| **v12.7.0** | 2025-09 | ESM source references support, parallel mode environment variable passing |
| **v12.4.0** | 2025-07 | **TypeScript config file support** |
| **v12.3.0** | 2025-06 | Node.js 25.x support, named BeforeAll/AfterAll hooks |
| **v12.2.0** | 2025-05 | **Execution sharding support** |
| **v12.0.0** | 2025 | Major release: dropped Node.js 18.x/23.x support |

**Key New Features**:
- TypeScript config file support (`cucumber.config.ts`)
- Execution Sharding for distributed testing
- Custom plugin system
- `@cucumber/node` under development (based on Node.js test runner)

**Installation**:

```bash
# npm
npm install --save-dev @cucumber/cucumber

# pnpm
pnpm add -D @cucumber/cucumber

# TypeScript additional dependencies
npm install --save-dev ts-node typescript @types/node
```

**Recommended package.json Configuration**:

```json
{
  "type": "module",
  "scripts": {
    "test:cucumber": "cucumber-js --config cucumber.config.ts",
    "test:cucumber:smoke": "cucumber-js --config cucumber.config.ts --tags @smoke",
    "test:cucumber:parallel": "cucumber-js --config cucumber.config.ts --parallel 4"
  },
  "devDependencies": {
    "@cucumber/cucumber": "^12.8.0",
    "@types/node": "^20.0.0",
    "ts-node": "^10.9.0",
    "typescript": "^5.4.0"
  }
}
```

**TypeScript Configuration**:

```typescript
// cucumber.config.ts
import type { Configuration } from '@cucumber/cucumber';

export default {
  paths: ['features/**/*.feature'],
  require: ['step-definitions/**/*.ts'],
  requireModule: ['ts-node/register'],
  format: [
    'progress',
    'html:reports/cucumber-report.html',
    'json:reports/cucumber-report.json'
  ],
  formatOptions: {
    colorsEnabled: true,
    snippetInterface: 'async-await'
  },
  publishQuiet: true,
  parallel: 2
} satisfies Configuration;
```

#### cucumber-jvm (Java)

> Source: Maven Central[^26^], cucumber-jvm GitHub[^95^]

| Version | Release Date | Usage |
|---------|--------------|-------|
| **7.34.3** | 2026-03 | 19 projects |
| **7.34.2** | 2026-01 | 35 projects |
| **7.34.1** | 2026-01 | 10 projects |
| **7.33.0** | 2025-12 | 49 projects |
| **7.32.0** | 2025-11 | 15 projects |
| **7.31.0** | 2025-10 | 22 projects |

**2025 Highlights**:
- 20 releases (primarily bug fixes and dependency updates)
- Support for locale sensitive parameter transformers
- **cucumber-junit deprecated**, cucumber-junit-platform-engine recommended
- Cucumber JUnit Platform Engine supports rerun files

**Maven Dependencies**:

```xml
<properties>
    <cucumber.version>7.34.3</cucumber.version>
    <junit.version>5.10.0</junit.version>
</properties>

<dependencies>
    <!-- Core dependency -->
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-java</artifactId>
        <version>${cucumber.version}</version>
        <scope>test</scope>
    </dependency>

    <!-- JUnit 5 Platform Engine (recommended) -->
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-junit-platform-engine</artifactId>
        <version>${cucumber.version}</version>
        <scope>test</scope>
    </dependency>

    <!-- JUnit 5 -->
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-suite</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>

    <!-- Dependency Injection: PicoContainer (recommended) -->
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-picocontainer</artifactId>
        <version>${cucumber.version}</version>
        <scope>test</scope>
    </dependency>

    <!-- Optional: Spring integration -->
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-spring</artifactId>
        <version>${cucumber.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

**Gradle Configuration**:

```gradle
ext {
    cucumberVersion = '7.34.3'
}

dependencies {
    testImplementation "io.cucumber:cucumber-java:${cucumberVersion}"
    testImplementation "io.cucumber:cucumber-junit-platform-engine:${cucumberVersion}"
    testImplementation "io.cucumber:cucumber-picocontainer:${cucumberVersion}"
    testImplementation 'org.junit.platform:junit-platform-suite:1.10.0'
}
```

**JUnit 5 Suite Runner**:

```java
// src/test/java/com/example/CucumberTestSuite.java
package com.example;

import org.junit.platform.suite.api.*;

@Suite
@IncludeEngines("cucumber")
@SelectPackages("com.example.steps")
@ConfigurationParameter(key = "cucumber.glue", value = "com.example.steps")
@ConfigurationParameter(key = "cucumber.features", value = "src/test/resources/features")
@ConfigurationParameter(key = "cucumber.plugin", 
    value = "pretty, html:target/cucumber-reports.html, json:target/cucumber-reports.json")
public class CucumberTestSuite {
}
```

#### cucumber-ruby

> Source: cucumber-ruby GitHub[^126^]

| Version | Release Date | Key Changes |
|---------|--------------|-------------|
| **v10.0.0** | 2025 | Architecture change milestone |
| v9.x | 2024 | Stable version |

**2025 Highlights**:
- 5 releases, including v10.0.0
- Removed Ruby 2.7 and 3.0 support, minimum Ruby 3.1
- Added Ruby 4.0+ support
- Improved backtrace filtering

**Gemfile**:

```ruby
source 'https://rubygems.org'

group :test do
  gem 'cucumber', '~> 10.0'
  gem 'rspec', '~> 3.12'
  gem 'selenium-webdriver', '~> 4.15'
  gem 'capybara', '~> 3.39'
end
```

### 1.3 Multi-Language Implementation Comparison

> Source: Knapsack Pro[^165^], QA Skills[^25^]

| Feature | Cucumber-JVM | Cucumber-JS | Cucumber-Ruby |
|---------|-------------|-------------|---------------|
| **Primary Language** | Java | JavaScript/TypeScript | Ruby |
| **Supported Languages** | Java, Kotlin, Scala, Groovy, Clojure | JavaScript, TypeScript | Ruby |
| **Step Matching** | Cucumber Expressions + Regex | Cucumber Expressions + Regex | Regex (native) |
| **DI Support** | PicoContainer, Spring, Guice, etc. | World object | World context |
| **Parallel Execution** | Yes (JUnit 5, TestNG) | Yes (--parallel) | Limited |
| **Report Formats** | JSON, HTML, JUnit, Allure | JSON, HTML | JSON, HTML |
| **IDE Support** | IntelliJ IDEA, Eclipse | VS Code, WebStorm | RubyMine |
| **Around Hook** | Not supported | Not supported | **Ruby only** |

### 1.4 Framework Selection Decision Matrix

| Scenario | Recommended Framework | Reason |
|----------|----------------------|--------|
| Java team, largest ecosystem | **Cucumber-JVM** | Most documentation, rich plugins |
| JavaScript/TypeScript project | **cucumber-js** | Native support, modern features |
| Ruby project | **cucumber-ruby** | Earliest implementation, mature and stable |
| .NET project | **SpecFlow/Reqnroll** | Deep Visual Studio integration |
| Python project | **Behave** | Pythonic API |
| Multi-language team | **Gauge** | Markdown syntax is more flexible |
| Need LivingDoc | **Serenity BDD** | Industry-leading documentation generation |

### 1.5 Step Definition Syntax Comparison

**Java (JVM)**:

```java
import io.cucumber.java.en.*;

public class AuthSteps {
    @Given("a registered user with email {string}")
    public void aRegisteredUser(String email) {
        user = UserFactory.createRegistered(email);
    }

    @When("the user submits the login form")
    public void submitLoginForm() {
        loginPage.login(user.getEmail(), user.getPassword());
    }

    @Then("the user should see the dashboard")
    public void shouldSeeDashboard() {
        assertTrue(new DashboardPage(driver).isDisplayed());
    }
}
```

**JavaScript**:

```javascript
const { Given, When, Then } = require('@cucumber/cucumber');
const assert = require('assert');

Given('a registered user with email {string}', async function(email) {
    this.user = await createRegisteredUser(email);
});

When('the user submits the login form', async function() {
    await this.loginPage.login(this.user.email, this.user.password);
});

Then('the user should see the dashboard', async function() {
    const visible = await this.dashboardPage.isDisplayed();
    assert.strictEqual(visible, true);
});
```

**TypeScript**:

```typescript
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from 'chai';

Given('a registered user with email {string}', async function(email: string) {
    this.user = await createUser(email);
});

When('the user submits the login form', async function() {
    this.result = await this.loginPage.submit(this.user);
});

Then('the user should see the dashboard', function() {
    expect(this.result.page).to.equal('dashboard');
});
```

**Ruby**:

```ruby
Given('a registered user with email {string}') do |email|
    @user = UserFactory.create_registered(email)
end

When('the user submits the login form') do
    @login_page.login(@user.email, @user.password)
end

Then('the user should see the dashboard') do
    expect(@dashboard_page).to be_displayed
end
```

---

## 2. IDE Plugins and Configuration

### 2.1 IntelliJ IDEA / WebStorm

> Source: Cucumber Official Documentation[^37^]

**Recommended Plugins**:

| Plugin Name | Purpose | Installation |
|-------------|---------|--------------|
| **Gherkin** | Gherkin syntax highlighting, formatting, auto-completion | Built-in (IntelliJ Ultimate) / Plugin Marketplace |
| **Cucumber for Java** | Java Step Definition navigation | Plugin Marketplace |
| **Cucumber for Groovy** | Groovy support | Plugin Marketplace |

**Configuration Steps**:

```
1. File -> Settings -> Plugins
2. Search for "Gherkin" and install
3. Search for "Cucumber for Java" and install
4. Restart IDE
5. File -> Settings -> Languages & Frameworks -> Gherkin
   - Configure feature file directory
   - Configure Step definitions directory
```

**Keyboard Shortcuts**:

| Action | Windows/Linux | macOS |
|--------|---------------|-------|
| Jump from Feature to Step Definition | `Ctrl + Click` | `Cmd + Click` |
| Create missing Step Definition | `Alt + Enter` | `Option + Enter` |
| Format Gherkin file | `Ctrl + Alt + L` | `Cmd + Option + L` |
| Find Step Definition usages | `Ctrl + B` | `Cmd + B` |

**IntelliJ Code Templates**:

```java
// Live Template used when creating Step Definitions
// Settings -> Editor -> Live Templates -> Cucumber

@Given("$STEP_TEXT$")
public void $METHOD_NAME$($PARAMETERS$) {
    $BODY$
}
```

### 2.2 VS Code

> Source: Cucumber Official Documentation

**Recommended Plugins**:

| Plugin Name | Purpose | Installation |
|-------------|---------|--------------|
| **Cucumber (Gherkin) Full Support** | Gherkin syntax highlighting, Step Definition navigation | Marketplace search |
| **Cucumber** | Basic Gherkin support | Marketplace search |
| **vscode-cucumber-js** | cucumber-js specific support | Marketplace search |

**settings.json Configuration**:

```json
{
  // Gherkin formatting
  "[gherkin]": {
    "editor.defaultFormatter": "alexkrechik.cucumberautocomplete",
    "editor.formatOnSave": true,
    "editor.tabSize": 2
  },

  // Cucumber auto-completion
  "cucumberautocomplete.steps": [
    "step-definitions/**/*.ts",
    "step-definitions/**/*.js",
    "src/test/java/**/steps/**/*.java"
  ],
  "cucumberautocomplete.syncfeatures": "features/**/*.feature",
  "cucumberautocomplete.strictGherkinCompletion": true,
  "cucumberautocomplete.smartSnippets": true,
  "cucumberautocomplete.stepsInvariants": true,
  "cucumberautocomplete.onTypeFormat": true,

  // File associations
  "files.associations": {
    "*.feature": "gherkin"
  }
}
```

**VS Code Keyboard Shortcuts**:

| Action | Windows/Linux | macOS |
|--------|---------------|-------|
| Jump to Step Definition | `Ctrl + Click` / `F12` | `Cmd + Click` / `F12` |
| View all references | `Shift + F12` | `Shift + F12` |
| Format document | `Shift + Alt + F` | `Shift + Option + F` |

### 2.3 RubyMine

**RubyMine built-in cucumber-ruby support**:

```
1. Open project -> Auto-recognize .feature files
2. Jump from feature step to step definition: Ctrl + Click
3. Run scenario: Right-click -> Run 'Scenario: xxx'
4. Debug: Right-click -> Debug 'Scenario: xxx'
```

### 2.4 Eclipse

**Recommended Plugins**:

| Plugin Name | Purpose |
|-------------|---------|
| **Natural** | Gherkin editor for Eclipse |
| **Cucumber Eclipse Plugin** | Step Definition navigation |

```
Installation steps:
1. Help -> Eclipse Marketplace
2. Search for "Cucumber" or "Natural"
3. Install and restart
```

### 2.5 IDE Feature Comparison

| Feature | IntelliJ Ultimate | VS Code | RubyMine |
|---------|------------------|---------|----------|
| Gherkin syntax highlighting | Built-in | Plugin required | Built-in |
| Step Definition navigation | Built-in | Plugin required | Built-in |
| Auto-completion | Excellent | Good | Excellent |
| Formatting | Excellent | Good | Excellent |
| Run/Debug | Built-in | Terminal | Built-in |
| Refactoring support | Excellent | Limited | Excellent |
| Multi-language support | Java/Kotlin/Scala/Groovy | JS/TS/Java, etc. | Ruby |

---

## 3. Reporting Tools Comparison

### 3.1 Reporting Tools Overview

> Source: Cucumber Official Documentation[^59^], BrowserStack[^71^]

| Tool | Type | Supported Languages | Pros | Cons |
|------|------|--------------------|------|------|
| **Allure** | Reporting Framework | Java, JS, Python, Ruby, .NET | Timeline view, historical trends, beautiful | Requires additional service |
| **Serenity BDD** | Reporting Library | Java, JS | Living Documentation, requirement traceability | Primarily supports Java/JS |
| **ExtentReports** | Reporting Library | Java, JS, Python, .NET | Beautiful, customizable, multiple themes | Complex configuration |
| **Cluecumber** | Maven Plugin | Java | Clean and readable, easy Jenkins integration | Limited to Cucumber JVM |
| **Cucumber HTML** | Built-in | All languages | Zero configuration, out-of-the-box | Basic features |
| **ReportPortal** | Server-side | All languages | AI classification, real-time reporting, multi-platform | Requires service deployment |
| **Pickles** | .NET Tool | .NET | Generates functional documentation | Limited to .NET |
| **Cukedoctor** | Java Tool | Java | Based on AsciiDoc | Limited to JVM |

### 3.2 Allure Report

> Source: QA Skills[^62^]

**Maven Dependencies**:

```xml
<!-- Allure Cucumber7 JVM -->
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-cucumber7-jvm</artifactId>
    <version>2.24.0</version>
</dependency>
```

**Maven Plugin**:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-maven</artifactId>
            <version>2.12.0</version>
            <configuration>
                <reportVersion>2.24.0</reportVersion>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Code Example**:

```java
import io.qameta.allure.*;

@Epic("Authentication")
@Feature("Login")
public class LoginSteps {

    @Step("Login with username {0}")
    public void login(String username, String password) {
        loginPage.enterUsername(username);
        loginPage.enterPassword(password);
        loginPage.clickLogin();
    }

    @Attachment(value = "Screenshot", type = "image/png")
    public byte[] takeScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @TestCaseId("AUTH-001")
    @Severity(SeverityLevel.CRITICAL)
    @Story("Successful login")
    @Given("a registered user with email {string}")
    public void aRegisteredUser(String email) {
        // Step implementation
    }
}
```

**Generate Report**:

```bash
# Run tests
mvn clean test

# Generate Allure report
mvn allure:report

# Start Allure service (auto-opens browser)
mvn allure:serve
```

**JavaScript Configuration**:

```bash
npm install --save-dev allure-cucumberjs
```

```typescript
// cucumber.config.ts
import type { Configuration } from '@cucumber/cucumber';

export default {
  format: ['allure-cucumberjs/reporter'],
  formatOptions: {
    resultsDir: './allure-results'
  }
} satisfies Configuration;
```

### 3.3 Serenity BDD

> Source: Serenity BDD Official[^59^][^62^]

**Maven Dependencies**:

```xml
<properties>
    <serenity.version>4.0.46</serenity.version>
    <cucumber.version>7.34.3</cucumber.version>
</properties>

<dependencies>
    <dependency>
        <groupId>net.serenity-bdd</groupId>
        <artifactId>serenity-core</artifactId>
        <version>${serenity.version}</version>
    </dependency>
    <dependency>
        <groupId>net.serenity-bdd</groupId>
        <artifactId>serenity-cucumber</artifactId>
        <version>${serenity.version}</version>
    </dependency>
    <dependency>
        <groupId>net.serenity-bdd</groupId>
        <artifactId>serenity-screenplay</artifactId>
        <version>${serenity.version}</version>
    </dependency>
    <dependency>
        <groupId>net.serenity-bdd</groupId>
        <artifactId>serenity-screenplay-webdriver</artifactId>
        <version>${serenity.version}</version>
    </dependency>
</dependencies>
```

**Gradle Configuration**:

```gradle
plugins {
    id 'java'
    id 'net.serenity-bdd.serenity-gradle-plugin' version '4.0.46'
}

ext {
    serenityVersion = '4.0.46'
}

dependencies {
    implementation "net.serenity-bdd:serenity-core:${serenityVersion}"
    implementation "net.serenity-bdd:serenity-cucumber:${serenityVersion}"
    implementation "net.serenity-bdd:serenity-screenplay:${serenityVersion}"
}

// Serenity task
tasks.register('aggregate', JavaExec) {
    classpath = sourceSets.test.runtimeClasspath
    mainClass = 'net.serenitybdd.cli.Main'
    args 'aggregate'
}
```

**Serenity Configuration**:

```properties
# serenity.properties
serenity.take.screenshots=AFTER_EACH_STEP
serenity.reports.show.step.details=true
serenity.report.accessibility=true
serenity.console.colors=true
serenity.logging=VERBOSE
serenity.verbose.steps=true
serenity.report.show.manual.tests=false
serenity.outputDirectory=target/site/serenity

# Screenshot strategy options:
# FOR_EACH_ACTION - every UI interaction
# AFTER_EACH_STEP - after each step completes
# FOR_FAILURES - only on failure
# DISABLED - no screenshots
```

**Screenplay Pattern Example**:

```java
// Serenity BDD test in Screenplay pattern
Actor alice = Actor.named("Alice")
    .whoCan(BrowseTheWeb.with(driver));

alice.attemptsTo(
    Open.browserOn().the(LoginPage.class),
    Enter.theValue("alice@example.com").into(LoginPage.EMAIL_FIELD),
    Enter.theValue("password123").into(LoginPage.PASSWORD_FIELD),
    Click.on(LoginPage.LOGIN_BUTTON)
);

alice.should(
    seeThat(TheWebPage.title(), containsString("Dashboard"))
);
```

**Serenity/JS Configuration (WebdriverIO)**:

```typescript
// wdio.conf.ts
import { WebdriverIOConfig } from '@serenity-js/webdriverio';

export const config: WebdriverIOConfig = {
    framework: '@serenity-js/webdriverio',
    serenity: {
        runner: 'cucumber',
        crew: [
            [ '@serenity-js/serenity-bdd', { specDirectory: './specs' } ],
            [ '@serenity-js/core:ArtifactArchiver', { outputDirectory: 'target/site/serenity' } ],
        ],
    },
    cucumberOpts: {
        formatOptions: {
            specDirectory: './features',
        },
    },
    runner: 'local',
};
```

### 3.4 ExtentReports

```xml
<!-- Maven Dependencies -->
<dependency>
    <groupId>com.aventstack</groupId>
    <artifactId>extentreports</artifactId>
    <version>5.1.2</version>
</dependency>
```

**Code Example**:

```java
import com.aventstack.extentreports.*;
import com.aventstack.extentreports.reporter.*;

public class ExtentReportManager {
    private static ExtentReports extent;
    private static ExtentTest test;

    public static void initReport() {
        ExtentSparkReporter spark = new ExtentSparkReporter("target/extent-report.html");
        spark.config().setTheme(Theme.DARK);
        spark.config().setDocumentTitle("BDD Test Report");
        spark.config().setReportName("Cucumber Test Results");

        extent = new ExtentReports();
        extent.attachReporter(spark);
        extent.setSystemInfo("Environment", "Test");
        extent.setSystemInfo("Browser", "Chrome");
    }

    public static void createTest(String name) {
        test = extent.createTest(name);
    }

    public static void logPass(String message) {
        test.pass(message);
    }

    public static void logFail(String message) {
        test.fail(message);
    }

    public static void attachScreenshot(String name, byte[] screenshot) {
        test.addScreenCaptureFromBase64String(
            Base64.getEncoder().encodeToString(screenshot), name
        );
    }

    public static void flushReport() {
        extent.flush();
    }
}
```

### 3.5 Cluecumber (JVM)

> Source: revanab.com[^76^]

```xml
<plugin>
    <groupId>com.trivago.rta</groupId>
    <artifactId>cluecumber-report-plugin</artifactId>
    <version>2.9.4</version>
    <executions>
        <execution>
            <id>report</id>
            <phase>post-integration-test</phase>
            <goals>
                <goal>reporting</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <sourceJsonReportDirectory>target/cucumber-reports</sourceJsonReportDirectory>
        <generatedHtmlReportDirectory>target/generated-report</generatedHtmlReportDirectory>
        <customPageTitle>UI Test Results</customPageTitle>
        <expandBeforeAfterHooks>true</expandBeforeAfterHooks>
        <expandStepHooks>true</expandStepHooks>
        <expandDocStrings>true</expandDocStrings>
        <expandAttachments>true</expandAttachments>
    </configuration>
</plugin>
```

### 3.6 ReportPortal

**Features**[^61^]:
- AI-powered failure classification
- Real-time reporting - no need to wait for execution to finish
- Customizable widgets and dashboards
- One-click access to historical execution data
- Multi-language, multi-platform support

```java
// reportportal.properties
rp.endpoint = http://localhost:8080
rp.uuid = your-api-key
rp.launch = cucumber-tests
rp.project = your-project
rp.enable = true
```

```xml
<!-- Maven Dependencies -->
<dependency>
    <groupId>com.epam.reportportal</groupId>
    <artifactId>agent-java-cucumber6</artifactId>
    <version>5.1.0</version>
</dependency>
```

### 3.7 Cucumber Built-in Reports

```java
// JUnit 4 Runner
@RunWith(Cucumber.class)
@CucumberOptions(
    plugin = {
        "pretty",                                    // Console output
        "progress",                                  // Progress dots
        "html:target/cucumber-reports/cucumber.html", // HTML report
        "json:target/cucumber-reports/cucumber.json", // JSON report
        "junit:target/cucumber-reports/cucumber.xml", // JUnit XML
        "timeline:target/cucumber-reports/timeline",  // Timeline
        "rerun:target/rerun.txt"                     // Failed scenarios
    }
)
public class TestRunner { }
```

**JavaScript Built-in Reports**:

```bash
# Multiple formats simultaneously
npx cucumber-js \
  --format progress \
  --format html:reports/cucumber-report.html \
  --format json:reports/cucumber-report.json \
  --format junit:reports/cucumber-report.xml \
  --format rerun:@rerun.txt
```

### 3.8 Reporting Tool Selection Guide

| Scenario | Recommended Tool | Reason |
|----------|-----------------|--------|
| Need trend analysis | **Allure** | Historical trends, timeline view |
| Need Living Documentation | **Serenity BDD** | Requirement traceability, narrative reports |
| Quick start | **Cucumber HTML** | Zero configuration |
| Jenkins integration | **Cluecumber** | Maven plugin, Jenkins-friendly |
| Team collaboration analysis | **ReportPortal** | Real-time reporting, AI classification |
| .NET project | **Pickles** | Functional documentation generation |
| Aesthetic needs | **ExtentReports** | Modern UI, multiple themes |

---

## 4. Utility Libraries

### 4.1 Cucumber Expressions

> Source: Cucumber Official Documentation[^37^][^151^]

Cucumber Expressions is the recommended expression syntax for Step Definition matching, replacing regular expressions.

#### Built-in Parameter Types

| Parameter Type | Matches | Examples |
|----------------|---------|----------|
| `{int}` | Integer | `42`, `-19` |
| `{float}` | Floating point | `3.6`, `.8`, `-9.2` |
| `{word}` | Single word (no spaces) | `banana` |
| `{string}` | Single or double quoted string | `"hello"`, `'world'` |
| `{}` | Any content | Any text |
| `{bigdecimal}` | High-precision decimal | `3.14159` |
| `{double}` | 64-bit floating point | `3.1415926535` |
| `{long}` | 64-bit integer | `9223372036854775807` |

#### Optional Text and Alternatives

```gherkin
# Optional text (parentheses indicate optional)
I have {int} cucumber(s) in my belly
# Matches: "1 cucumber" and "5 cucumbers"

# Alternative words (slash-separated)
I have {int} cucumber(s) in my belly/stomach
# Matches: "in my belly" and "in my stomach"
```

#### Custom Parameter Types (Java)

```java
// CustomParameterTypes.java
import io.cucumber.java.ParameterType;

public class CustomParameterTypes {

    @ParameterType("red|blue|green|yellow")
    public Color color(String color) {
        return Color.valueOf(color.toUpperCase());
    }

    @ParameterType("\\d{4}-\\d{2}-\\d{2}")
    public LocalDate date(String date) {
        return LocalDate.parse(date);
    }

    @ParameterType("\\$\\d+\\.\\d{2}")
    public Money price(String price) {
        return Money.parse(price);
    }
}
```

**Using Custom Parameters**:

```gherkin
# feature file
Scenario: Purchase with color and date
  Given a product with {color} option
  When the price is {price}
  And the delivery date is {date}
  Then the order should be valid
```

```java
@Given("a product with {color} option")
public void productWithColor(Color color) {
    // color parameter auto-converts to Color enum
}

@When("the price is {price}")
public void setPrice(Money price) {
    // price auto-converts to Money object
}

@When("the delivery date is {date}")
public void setDeliveryDate(LocalDate date) {
    // date auto-converts to LocalDate
}
```

#### Custom Parameter Types (JavaScript)

```javascript
// support/parameter-types.js
const { defineParameterType } = require('@cucumber/cucumber');

defineParameterType({
    name: 'color',
    regexp: /red|blue|green|yellow/,
    transformer: (color) => color.toUpperCase()
});

defineParameterType({
    name: 'date',
    regexp: /\d{4}-\d{2}-\d{2}/,
    transformer: (date) => new Date(date)
});

defineParameterType({
    name: 'currency',
    regexp: /\$([\d.]+)/,
    transformer: (amount) => parseFloat(amount)
});
```

### 4.2 Tag Expressions

Tag Expressions provide powerful boolean expression filtering capabilities[^37^].

#### Syntax Reference

| Expression | Meaning | Example |
|------------|---------|---------|
| `@fast` | Only @fast tag | Basic filter |
| `@wip and not @slow` | @wip and not @slow | Exclude slow tests |
| `@smoke and @fast` | Has both @smoke and @fast | Intersection |
| `@gui or @database` | @gui or @database | Union |
| `(@smoke or @ui) and (not @slow)` | Complex combination | Mixed logic |
| `not @wip and not @ignore` | Not @wip and not @ignore | Multiple exclusions |

#### Usage by Platform

```bash
# Maven
mvn test -Dcucumber.filter.tags="@smoke and not @wip"

# cucumber-js
npx cucumber-js --tags "@regression and not @slow"

# cucumber-ruby
cucumber --tags "@smoke and not @wip"

# JUnit Runner
@CucumberOptions(tags = "@smoke and not @wip")

# Environment variable
CUCUMBER_FILTER_TAGS="@smoke and @fast" mvn test
```

### 4.3 Data Table API

#### Java DataTable Transformation

```java
// List<Map<String, String>> - one Map per row
@Given("the following users:")
public void the_following_users(List<Map<String, String>> users) {
    for (Map<String, String> user : users) {
        String name = user.get("name");
        String email = user.get("email");
    }
}

// List<List<String>> - raw row data
@Given("the following data:")
public void the_following_data(List<List<String>> data) {
    for (List<String> row : data) {
        String firstCol = row.get(0);
        String secondCol = row.get(1);
    }
}

// Map<String, String> - key-value pairs
@Given("the following config:")
public void the_following_config(Map<String, String> config) {
    String timeout = config.get("timeout");
}

// DataTable object - full operations
@Given("the following table:")
public void the_following_table(DataTable dataTable) {
    List<Map<String, String>> maps = dataTable.asMaps();
    DataTable diff = dataTable.diff(otherTable);
    List<String> firstColumn = dataTable.column(0);
}
```

#### JavaScript DataTable Operations

```javascript
// hashes() - Array of { column: value }
Given('the following users:', async function(dataTable) {
    const users = dataTable.hashes();
    for (const user of users) {
        await this.createUser(user.name, user.email);
    }
});

// raw() - raw 2D array
Given('the following data:', async function(dataTable) {
    const rows = dataTable.raw();
    // rows[0] = header, rows[1+] = data
});

// rows() - data rows without header
Given('the following rows:', async function(dataTable) {
    const rows = dataTable.rows();
});

// rowsHash() - two-column table to Map
Given('the following config:', async function(dataTable) {
    const config = dataTable.rowsHash();
    // { key1: value1, key2: value2 }
});
```

### 4.4 Dependency Injection (DI)

#### PicoContainer (JVM Recommended)

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-picocontainer</artifactId>
    <version>7.34.3</version>
    <scope>test</scope>
</dependency>
```

```java
// TestContext.java - shared state class
public class TestContext {
    private WebDriver driver;
    private String scenarioName;
    private Map<String, Object> data = new HashMap<>();

    public WebDriver getDriver() {
        if (driver == null) {
            driver = new ChromeDriver();
        }
        return driver;
    }

    public void set(String key, Object value) {
        data.put(key, value);
    }

    public Object get(String key) {
        return data.get(key);
    }
}

// LoginSteps.java - auto-injected TestContext
public class LoginSteps {
    private TestContext context;

    public LoginSteps(TestContext context) {
        this.context = context;
    }

    @Given("the user logs in with {string} and {string}")
    public void login(String username, String password) {
        LoginPage loginPage = new LoginPage(context.getDriver());
        loginPage.login(username, password);
    }
}

// OrderSteps.java - shares the same TestContext instance
public class OrderSteps {
    private TestContext context;

    public OrderSteps(TestContext context) {
        this.context = context;
    }

    @Then("the order should be created")
    public void verifyOrder() {
        // can access context.getDriver() and other shared resources
    }
}
```

#### Spring Integration

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-spring</artifactId>
    <version>7.34.3</version>
    <scope>test</scope>
</dependency>
```

```java
// Use @ScenarioScope to ensure Beans don't leak across scenarios
@Component
@ScenarioScope
public class TestContext {
    private WebDriver driver;
    // ...
}
```

#### JavaScript World Object

```javascript
// support/world.js
const { setWorldConstructor, World } = require('@cucumber/cucumber');

class CustomWorld extends World {
    constructor(options) {
        super(options);
        this.context = {};
        this.driver = null;
    }

    setContext(key, value) {
        this.context[key] = value;
    }

    getContext(key) {
        return this.context[key];
    }

    async initDriver() {
        if (!this.driver) {
            this.driver = await chromium.launch();
        }
        return this.driver;
    }
}

setWorldConstructor(CustomWorld);

// Usage in steps
Given('I have a product', async function() {
    this.setContext('product', await createProduct());
});
```

---

## 5. Living Documentation Tools

### 5.1 Living Documentation Concept

> Source: Testomat.io[^65^], CucumberStudio[^66^]

Living Documentation is a type of dynamic technical documentation that is:
- **Actual** - truly reflects current system behavior
- **Reliable** - kept reliable through automated verification
- **Minimal work** - minimizes manual maintenance effort
- **Collaborative** - created through multi-role collaboration
- **Insightful** - provides valuable insights

### 5.2 Living Documentation Tools Comparison

| Tool | Language/Platform | Description | Recommended Scenario |
|------|-------------------|-------------|---------------------|
| **Serenity BDD** | Java/JS | Most comprehensive Living Documentation reports | Java/JS projects |
| **Pickles** | .NET | Converts BDD tests into readable documents | .NET projects |
| **Cukedoctor** | Java | Based on Cucumber + Asciidoctor | JVM projects |
| **SpecFlow+ LivingDoc** | .NET | Official SpecFlow solution | SpecFlow projects |
| **CucumberStudio** | Multi-platform | SmartBear commercial BDD platform | Enterprise |
| **Testomat.io** | Multi-platform | TMS supporting Living Documentation | Multi-platform teams |

### 5.3 Serenity BDD Living Documentation

> Source: Serenity BDD Official[^59^][^62^]

Serenity BDD is the industry-leading Living Documentation tool.

**Report Contents Include**:
1. Test results overview - dashboard showing pass/fail rates
2. Feature documentation - each feature presented as a narrative
3. Step-by-step screenshots - screenshot record for each step
4. Requirement coverage - mapping tests to requirements
5. Tag-based filtering

**Gradle Configuration**:

```gradle
plugins {
    id 'java'
    id 'net.serenity-bdd.serenity-gradle-plugin' version '4.0.46'
}

dependencies {
    implementation "net.serenity-bdd:serenity-core:4.0.46"
    implementation "net.serenity-bdd:serenity-cucumber:4.0.46"
}

// Generate report
// ./gradlew test aggregate
```

**Serenity BDD Versions**:

| Version | Key Features |
|---------|-------------|
| **5.3.0** | String annotation support |
| **4.0.46** | Gradle plugin, Screenplay pattern |
| **4.0.18** | JUnit 5 support, Selenium 4 integration |

### 5.4 Pickles (.NET)

> Source: Cucumber Official Tools List[^86^]

Pickles converts BDD tests into documents readable by non-technical stakeholders.

```bash
# Install Pickles
dotnet tool install --global pickles

# Generate documentation
pickles --feature-directory=./features \
        --output-directory=./docs \
        --documentation-format=dhtml

# Supported formats: dhtml, html, word, excel, markdown
```

### 5.5 Cukedoctor (JVM)

> Source: Cucumber Official Tools List[^86^]

Cukedoctor generates readable feature documents based on Cucumber JSON reports and Asciidoctor.

```xml
<!-- Maven Dependencies -->
<dependency>
    <groupId>com.github.cukedoctor</groupId>
    <artifactId>cukedoctor-maven-plugin</artifactId>
    <version>3.7.0</version>
</dependency>
```

```xml
<!-- Maven Plugin -->
<plugin>
    <groupId>com.github.cukedoctor</groupId>
    <artifactId>cukedoctor-maven-plugin</artifactId>
    <version>3.7.0</version>
    <configuration>
        <outputFileName>living-documentation</outputFileName>
        <outputDir>target/docs</outputDir>
        <format>html</format>
        <hideSummarySection>false</hideSummarySection>
        <hideScenarioKeyword>true</hideScenarioKeyword>
        <hideStepTime>true</hideStepTime>
    </configuration>
</plugin>
```

```bash
# Generate documentation
mvn cukedoctor:execute
```

### 5.6 SpecFlow+ LivingDoc

> Source: SmartBear[^66^]

```yaml
# Azure DevOps Pipeline Configuration
steps:
- task: SpecFlowPlus@0
  inputs:
    generatorFolder: '$(Build.SourcesDirectory)'
    projectFilePath: 'MyProject.csproj'
    outputType: 'Html'
    outputFile: '$(Build.ArtifactStagingDirectory)/livingdoc.html'
```

### 5.7 Testomat.io Living Documentation

```bash
# Enable Living Documentation to get a public link
# Can be embedded in Confluence, viewable without authentication
```

### 5.8 Living Documentation vs Test Reports

| Dimension | Test Report | Living Documentation |
|-----------|-------------|---------------------|
| Focus | Test execution results | Functional behavior description |
| Audience | Primarily QA team | Whole team + stakeholders |
| Content | Test-focused | More detailed feature descriptions |
| Consumers | Test engineers | Everyone |
| Purpose | Verify test completion | Shared understanding + traceability |

---

## 6. Code Quality Tools

### 6.1 Gherkin Lint Tools

> Source: GitHub - gherkin-best-practices[^118^]

| Tool | Platform | Purpose |
|------|----------|---------|
| **gherkin-lint** | Node.js | Gherkin syntax and style checking |
| **cuke_linter** | Ruby | Cucumber feature file lint |
| **Gherkin Code Style** | IntelliJ | IDE built-in formatting rules |

**gherkin-lint Configuration (.gherkin-lintrc)**:

```json
{
  "no-files-without-scenarios": "on",
  "no-unnamed-features": "on",
  "no-unnamed-scenarios": "on",
  "no-dupe-scenario-names": ["on", "in-feature"],
  "no-dupe-feature-names": "on",
  "no-partially-commented-tag-lines": "on",
  "no-trailing-spaces": "on",
  "no-multiple-empty-lines": "on",
  "no-empty-file": "on",
  "no-scenario-outlines-without-examples": "on",
  "no-restricted-tags": ["on", {"tags": ["@ignore", "@todo"]}],
  "use-and": "warn",
  "no-duplicate-tags": "on",
  "no-superfluous-tags": "on",
  "no-homogenous-tags": "on",
  "one-space-between-tags": "on",
  "indentation": ["on", {"Feature": 0, "Background": 2, "Scenario": 2, "Step": 4, "Examples": 4, "example": 6, "Tag": 0}],
  "no-unchecked-background": "warn",
  "no-background-only-scenario": "warn",
  "max-scenarios-per-file": ["on", {"maxScenarios": 15, "countOutlineExamples": true}],
  "file-name": ["on", {"style": "kebab-case"}],
  "new-line-at-eof": ["on", "yes"],
  "no-step-out-of-context": "on",
  "max-steps-per-scenario": ["warn", 10],
  "scenario-size": ["on", {"steps": 15, "examples": 10}]
}
```

**package.json Configuration**:

```json
{
  "scripts": {
    "lint:gherkin": "gherkin-lint features/**/*.feature",
    "lint:gherkin:fix": "gherkin-lint --fix features/**/*.feature"
  },
  "devDependencies": {
    "gherkin-lint": "^4.2.2"
  }
}
```

### 6.2 ESLint/Prettier Integration (JavaScript)

```json
// .eslintrc.json
{
  "extends": ["eslint:recommended"],
  "overrides": [
    {
      "files": ["step-definitions/**/*.js", "support/**/*.js"],
      "env": {
        "node": true
      }
    }
  ]
}
```

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

### 6.3 Checkstyle/SpotBugs (Java)

```xml
<!-- pom.xml - Checkstyle -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <configLocation>checkstyle.xml</configLocation>
        <includeTestSourceDirectory>true</includeTestSourceDirectory>
    </configuration>
</plugin>
```

### 6.4 SonarQube Integration

```yaml
# sonar-project.properties
sonar.projectKey=my-bdd-project
sonar.sources=src/main/java
sonar.tests=src/test/java
sonar.test.inclusions=**/*Test.java,**/*Steps.java
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
sonar.junit.reportPaths=target/cucumber-reports

# Include feature files
sonar.sources=features,src/main/java
sonar.inclusions=**/*.feature,**/*.java
```

```yaml
# GitHub Actions - SonarQube Scan
- name: SonarQube Scan
  uses: SonarSource/sonarqube-scan-action@master
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

### 6.5 Code Quality Checklist

| # | Check Item | Tool | Priority |
|---|-----------|------|----------|
| 1 | Gherkin syntax correctness | gherkin-lint | High |
| 2 | Step definition matching | IDE / cucumber --dry-run | High |
| 3 | Unused step definitions | cucumber --dry-run | Medium |
| 4 | Feature file formatting | Prettier / IDE | Medium |
| 5 | Code coverage | JaCoCo / Istanbul | High |
| 6 | Duplicate steps | SonarQube | Medium |
| 7 | Code complexity | SonarQube / Checkstyle | Medium |
| 8 | Dependency security check | OWASP Dependency Check | High |

### 6.6 Pre-commit Hook

```bash
#!/bin/sh
# .git/hooks/pre-commit

echo "Running pre-commit checks..."

# 1. Gherkin lint
npx gherkin-lint features/**/*.feature
if [ $? -ne 0 ]; then
    echo "Gherkin lint failed!"
    exit 1
fi

# 2. Dry run cucumber
cucumber-js --dry-run
if [ $? -ne 0 ]; then
    echo "Cucumber dry run failed! Check for undefined steps."
    exit 1
fi

# 3. Run smoke tests
cucumber-js --tags "@smoke"
if [ $? -ne 0 ]; then
    echo "Smoke tests failed!"
    exit 1
fi

echo "Pre-commit checks passed!"
```

### 6.7 Husky + lint-staged Configuration

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "pre-push": "npm run test:cucumber:smoke"
    }
  },
  "lint-staged": {
    "*.feature": ["gherkin-lint"],
    "*.{js,ts}": ["eslint --fix", "prettier --write"],
    "*.java": ["checkstyle"]
  }
}
```

---

## Source Index

### Tier 1 - Official Documentation and Source Code

| # | Source | URL | Referenced Content |
|---|--------|-----|-------------------|
| [^37^] | Cucumber Official API | https://cucumber.io/docs/cucumber/api/ | Cucumber Expressions, Tags, DI |
| [^59^] | Cucumber Reporting | https://cucumber.io/docs/cucumber/reporting/ | Built-in plugins, third-party tools |
| [^91^] | cucumber-js Releases | https://github.com/cucumber/cucumber-js/releases | v12.x versions and features |
| [^95^] | cucumber-jvm Releases | https://github.com/cucumber/cucumber-jvm/releases | 7.34.x versions |
| [^122^] | Cucumber 2025 Review | https://news.cucumber.io/archive/2025-year-in-review/ | Year in review |
| [^126^] | cucumber-ruby Releases | https://github.com/cucumber/cucumber-ruby/releases | v10.x versions |
| [^151^] | behave Expressions | https://behave.readthedocs.io/en/stable/appendix.cucumber_expressions/ | Parameter type definitions |

### Tier 2 - Authoritative Technical Sources

| # | Source | URL | Referenced Content |
|---|--------|-----|-------------------|
| [^25^] | QA Skills BDD Comparison | https://qaskills.sh/blog/bdd-frameworks-comparison-2026 | In-depth BDD framework comparison |
| [^59^] | Mintlify Serenity | https://mintlify.com/Petzega/mkrs-btg-tests/framework/serenity-bdd | Serenity BDD integration |
| [^62^] | QA Skills Serenity Guide | https://qaskills.sh/blog/serenity-bdd-testing-guide | Serenity BDD detailed guide |
| [^71^] | BrowserStack | https://www.browserstack.com/guide/test-management-reporting-tools | Reporting tools comparison |
| [^86^] | Cucumber Related Tools | https://cucumber.io/docs/tools/related-tools/ | Official tools list |

### Tier 3 - Community and Tools

| # | Source | URL | Referenced Content |
|---|--------|-----|-------------------|
| [^165^] | Knapsack Pro | https://knapsackpro.com/testing_frameworks/difference_between/cucumber-jvm/vs/cucumber | JVM vs Ruby comparison |
| [^26^] | Maven Central | https://mvnrepository.com/artifact/io.cucumber/cucumber-java | cucumber-jvm versions |
| [^118^] | gherkin-best-practices | https://github.com/andredesousa/gherkin-best-practices | Gherkin best practices |

---

> This reference manual is based on comprehensive research from 25+ authoritative sources, covering Cucumber official documentation (Tier 1), authoritative technical blogs (Tier 2), and community best practices (Tier 3). Tool version information is current as of 2025; it is recommended to consult the official latest documentation before actual use.
