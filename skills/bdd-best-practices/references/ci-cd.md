# BDD CI/CD Pipeline Integration Reference Manual

> Version: 1.0 | Based on Cucumber Official Documentation, Community Best Practices, and Industry Research
> Applicable Scope: GitHub Actions / Jenkins / GitLab CI / Azure DevOps

---

## Table of Contents

1. [CI/CD Pipeline Integration Overview](#1-cicd-pipeline-integration-overview)
2. [GitHub Actions Configuration Examples](#2-github-actions-configuration-examples)
3. [Jenkins Pipeline Configuration Examples](#3-jenkins-pipeline-configuration-examples)
4. [GitLab CI Configuration Examples](#4-gitlab-ci-configuration-examples)
5. [Parallel Execution and Sharding](#5-parallel-execution-and-sharding)
6. [Failure Retry Strategies](#6-failure-retry-strategies)
7. [Reporting and Artifact Management](#7-reporting-and-artifact-management)
8. [Quality Gates](#8-quality-gates)

---

## 1. CI/CD Pipeline Integration Overview

### 1.1 Why Integrate CI/CD

> Source: Cucumber Official Documentation[^85^], TestMu[^7^]

- **Automation Efficiency**: Automatically run tests after code commits
- **Enhanced Visibility**: Generate detailed reports
- **Collaboration**: Team members can easily monitor, review, and share test results
- **Faster Releases**: Serve as quality gates to ensure code quality

### 1.2 BDD's Position in CI/CD

```
Code Commit
    |
    v
+-------------------+     +-------------------+     +-------------------+
| Code Compilation/ | --> | Unit Tests (TDD)  | --> | BDD Scenario      |
| Build             |     | (JUnit/pytest/    |     | Execution         |
| (Maven/Gradle/    |     |  Jest)            |     | (Cucumber Scenarios)|
|  npm/webpack)     |     |                   |     |                   |
+-------------------+     +-------------------+     +-------------------+
                                                           |
                     +-------------------------------------+
                     |
                     v
            +-------------------+     +-------------------+
            | Quality Gate      | --> | Reporting &       |
            | (Pass Rate/       |     | Artifacts         |
            |  Coverage/        |     | (HTML/JSON/Allure)|
            |  Flaky Detection) |     |                   |
            +-------------------+     +-------------------+
                     |
                     v
            +-------------------+
            | Deployment        |
            | Decision          |
            | (Proceed if pass) |
            +-------------------+
```

### 1.3 CI/CD Best Practices Overview

> Source: Cucumber Official Documentation[^85^], TestQuality[^107^]

| # | Practice | Description |
|---|------|------|
| 1 | **Run unit tests first, then BDD scenarios** | Fast feedback loop |
| 2 | **Use tags to control execution** | `@smoke`, `@regression`, `@wip` |
| 3 | **Collect evidence on failure** | Screenshots, logs, HTML reports |
| 4 | **Trend analysis** | Track test health over time |
| 5 | **Parallel execution** | Use matrix strategy across browsers/environments |
| 6 | **Quality gates** | Block merge/deployment on failure |
| 7 | **Environment consistency** | Use containerization to ensure dev/test/prod parity |

### 1.4 Recommended Pipeline Stages

```yaml
# Conceptual pipeline
stages:
  - build              # Compilation/Build
  - unit-test          # Unit Tests
  - bdd-smoke          # BDD Smoke Tests
  - bdd-api            # API Layer BDD Tests
  - bdd-ui             # UI Layer BDD Tests
  - bdd-regression     # Full Regression Tests
  - report             # Report Generation
  - quality-gate       # Quality Gate
```

---

## 2. GitHub Actions Configuration Examples

### 2.1 Basic Configuration (JVM + Maven)

> Source: Daniela Baron[^105^], TestQuality[^107^]

```yaml
# .github/workflows/bdd-tests.yml
name: BDD Cucumber Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  # ==========================================
  # Stage 1: Build & Unit Tests
  # ==========================================
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cache Maven dependencies
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}

      - name: Compile
        run: mvn clean compile test-compile

  # ==========================================
  # Stage 2: BDD Smoke Tests
  # ==========================================
  bdd-smoke:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cache Maven dependencies
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}

      - name: Run Smoke Tests
        run: mvn test -Dcucumber.filter.tags="@smoke"

      - name: Upload Smoke Test Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: smoke-test-results
          path: |
            target/cucumber-reports/
            target/cucumber-*.json
            target/cucumber-*.html

  # ==========================================
  # Stage 3: BDD API Tests
  # ==========================================
  bdd-api:
    needs: build
    runs-on: ubuntu-latest
    services:
      # Start the service under test
      app:
        image: myapp:latest
        ports:
          - 8080:8080
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Run API BDD Tests
        run: mvn test -Dcucumber.filter.tags="@api"
        env:
          BASE_URL: http://localhost:8080

      - name: Upload API Test Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: api-test-results
          path: target/cucumber-reports/

  # ==========================================
  # Stage 4: BDD UI Tests (Matrix Strategy)
  # ==========================================
  bdd-ui:
    needs: build
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        browser: [chrome, firefox]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Set up ${{ matrix.browser }}
        uses: browser-actions/setup-${{ matrix.browser }}@latest

      - name: Run UI BDD Tests
        run: mvn test -Dcucumber.filter.tags="@ui"
        env:
          BROWSER: ${{ matrix.browser }}
          HEADLESS: "true"

      - name: Upload UI Test Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: ui-test-results-${{ matrix.browser }}
          path: |
            target/cucumber-reports/
            target/screenshots/

  # ==========================================
  # Stage 5: Full Regression Tests
  # ==========================================
  bdd-regression:
    needs: [bdd-smoke, bdd-api]
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Run Regression Tests
        run: mvn test -Dcucumber.filter.tags="@regression and not @wip and not @flaky"

      - name: Publish Test Results
        uses: EnricoMi/publish-unit-test-result-action@v2
        if: always()
        with:
          files: 'target/cucumber-reports/*.xml'

      - name: Upload Regression Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: regression-report
          path: target/cucumber-reports/

  # ==========================================
  # Stage 6: Allure Report Generation
  # ==========================================
  generate-report:
    needs: [bdd-smoke, bdd-api, bdd-ui, bdd-regression]
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Download all artifacts
        uses: actions/download-artifact@v4

      - name: Generate Allure Report
        uses: simple-elf/allure-report-action@master
        if: always()
        with:
          allure_results: target/allure-results
          allure_history: allure-history

      - name: Deploy Allure Report to GitHub Pages
        if: always()
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: allure-history
```

### 2.2 JavaScript/TypeScript Configuration

```yaml
# .github/workflows/bdd-tests-js.yml
name: BDD Tests (JavaScript)

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  bdd-tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [20, 22]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run Smoke Tests
        run: npx cucumber-js --config cucumber.config.ts --tags "@smoke"

      - name: Run API Tests
        run: npx cucumber-js --config cucumber.config.ts --tags "@api"

      - name: Run UI Tests
        run: npx cucumber-js --config cucumber.config.ts --tags "@ui"
        env:
          HEADLESS: "true"

      - name: Upload Test Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: cucumber-reports-node${{ matrix.node-version }}
          path: |
            reports/
            *.html
            *.json

  # Sharded execution example (cucumber-js v12.2.0+)
  bdd-sharded:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        shard: [1, 2, 3]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run BDD Tests (Shard ${{ matrix.shard }}/3)
        run: npx cucumber-js --config cucumber.config.ts --shard ${{ matrix.shard }}/3

      - name: Upload Shard Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: shard-results-${{ matrix.shard }}
          path: reports/
```

### 2.3 Publishing Allure Reports to GitHub Pages

```yaml
# .github/workflows/allure-report.yml
name: Allure Report

on:
  workflow_run:
    workflows: ["BDD Cucumber Tests"]
    types:
      - completed

jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - name: Download Artifacts
        uses: actions/download-artifact@v4

      - name: Build Allure Report
        uses: simple-elf/allure-report-action@master
        with:
          allure_results: target/allure-results
          allure_history: allure-history

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: allure-history
```

---

## 3. Jenkins Pipeline Configuration Examples

### 3.1 Basic Jenkins Pipeline

> Source: TestMu[^7^], TestQuality[^107^]

```groovy
// Jenkinsfile
pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        MAVEN_OPTS = '-Xmx1024m'
        CUCUMBER_PUBLISH_ENABLED = 'false'
    }

    stages {
        // ==========================================
        // Stage 1: Checkout Code
        // ==========================================
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ==========================================
        // Stage 2: Build
        // ==========================================
        stage('Build') {
            steps {
                sh 'mvn clean compile test-compile -DskipTests'
            }
        }

        // ==========================================
        // Stage 3: Unit Tests
        // ==========================================
        stage('Unit Tests') {
            steps {
                sh 'mvn test -Dtest="*UnitTest"'
            }
            post {
                always {
                    junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }

        // ==========================================
        // Stage 4: BDD Smoke Tests
        // ==========================================
        stage('BDD Smoke Tests') {
            steps {
                sh 'mvn test -Dcucumber.filter.tags="@smoke"'
            }
            post {
                always {
                    cucumber(
                        buildStatus: 'UNSTABLE',
                        fileIncludePattern: '**/cucumber.json',
                        trendsLimit: 10,
                        reportTitle: 'BDD Smoke Test Results'
                    )
                }
            }
        }

        // ==========================================
        // Stage 5: BDD API Tests
        // ==========================================
        stage('BDD API Tests') {
            when {
                expression { currentBuild.result != 'FAILURE' }
            }
            steps {
                sh 'mvn test -Dcucumber.filter.tags="@api"'
            }
            post {
                always {
                    cucumber(
                        buildStatus: 'UNSTABLE',
                        fileIncludePattern: '**/cucumber.json',
                        trendsLimit: 10,
                        reportTitle: 'BDD API Test Results'
                    )
                }
            }
        }

        // ==========================================
        // Stage 6: BDD UI Tests
        // ==========================================
        stage('BDD UI Tests') {
            when {
                expression { currentBuild.result != 'FAILURE' }
            }
            steps {
                sh 'mvn test -Dcucumber.filter.tags="@ui" -Dheadless=true'
            }
            post {
                always {
                    cucumber(
                        buildStatus: 'UNSTABLE',
                        fileIncludePattern: '**/cucumber.json',
                        trendsLimit: 10,
                        reportTitle: 'BDD UI Test Results'
                    )
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'target/cucumber-html-report',
                        reportFiles: 'index.html',
                        reportName: 'Cucumber HTML Report'
                    ])
                }
            }
        }

        // ==========================================
        // Stage 7: Regression Tests
        // ==========================================
        stage('BDD Regression Tests') {
            when {
                allOf {
                    branch 'main'
                    expression { currentBuild.result != 'FAILURE' }
                }
            }
            steps {
                sh '''
                    mvn test \
                        -Dcucumber.filter.tags="@regression and not @wip and not @flaky" \
                        -Dcucumber.execution.parallel.enabled=true
                '''
            }
            post {
                always {
                    cucumber(
                        buildStatus: 'UNSTABLE',
                        fileIncludePattern: '**/cucumber.json',
                        trendsLimit: 10,
                        reportTitle: 'BDD Regression Test Results'
                    )
                }
            }
        }
    }

    // ==========================================
    // Post Stage: Reports & Notifications
    // ==========================================
    post {
        always {
            // Archive artifacts
            archiveArtifacts artifacts: 'target/cucumber-reports/**/*', allowEmptyArchive: true
            archiveArtifacts artifacts: 'target/screenshots/**/*', allowEmptyArchive: true

            // Clean workspace
            cleanWs(deleteDirs: true, notFailBuild: true)
        }

        success {
            // Send success notification
            slackSend(
                color: 'good',
                message: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }

        failure {
            // Send failure notification
            slackSend(
                color: 'danger',
                message: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }

        unstable {
            slackSend(
                color: 'warning',
                message: "Build Unstable (Tests Failed): ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
    }
}
```

### 3.2 Jenkins Parallel Stages

```groovy
// Jenkinsfile - Parallel Execution
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean compile test-compile'
            }
        }

        stage('BDD Tests') {
            parallel {
                stage('Smoke') {
                    steps {
                        sh 'mvn test -Dcucumber.filter.tags="@smoke"'
                    }
                    post {
                        always {
                            cucumber(
                                fileIncludePattern: '**/smoke-cucumber.json',
                                reportTitle: 'Smoke Tests'
                            )
                        }
                    }
                }
                stage('API') {
                    steps {
                        sh 'mvn test -Dcucumber.filter.tags="@api"'
                    }
                    post {
                        always {
                            cucumber(
                                fileIncludePattern: '**/api-cucumber.json',
                                reportTitle: 'API Tests'
                            )
                        }
                    }
                }
                stage('UI') {
                    steps {
                        sh 'mvn test -Dcucumber.filter.tags="@ui"'
                    }
                    post {
                        always {
                            cucumber(
                                fileIncludePattern: '**/ui-cucumber.json',
                                reportTitle: 'UI Tests'
                            )
                        }
                    }
                }
            }
        }
    }
}
```

### 3.3 Required Jenkins Plugins

| Plugin | Purpose |
|------|------|
| **Cucumber Reports Plugin** | Parse and display Cucumber JSON reports |
| **HTML Publisher Plugin** | Publish HTML reports |
| **JUnit Plugin** | Publish JUnit XML test results |
| **Slack Notification Plugin** | Slack notifications |
| **Pipeline Stage View Plugin** | Visualize pipeline stages |
| **Workspace Cleanup Plugin** | Workspace cleanup |

---

## 4. GitLab CI Configuration Examples

### 4.1 Basic GitLab CI Configuration

> Source: TestQuality[^107^]

```yaml
# .gitlab-ci.yml
stages:
  - build
  - unit-test
  - bdd-smoke
  - bdd-api
  - bdd-ui
  - bdd-regression
  - report

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  MAVEN_CLI_OPTS: "--batch-mode --errors --fail-at-end --show-version"
  CUCUMBER_TAGS: "not @wip"

# Cache Maven dependencies
cache:
  paths:
    - .m2/repository/

# ==========================================
# Stage 1: Build
# ==========================================
build:
  stage: build
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn $MAVEN_CLI_OPTS clean compile test-compile -DskipTests
  artifacts:
    paths:
      - target/
    expire_in: 1 hour

# ==========================================
# Stage 2: Unit Tests
# ==========================================
unit-test:
  stage: unit-test
  image: maven:3.9-eclipse-temurin-17
  needs: [build]
  script:
    - mvn $MAVEN_CLI_OPTS test -Dtest="*UnitTest"
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml
    paths:
      - target/surefire-reports/
    expire_in: 1 week

# ==========================================
# Stage 3: BDD Smoke Tests
# ==========================================
bdd-smoke:
  stage: bdd-smoke
  image: maven:3.9-eclipse-temurin-17
  needs: [build]
  script:
    - mvn $MAVEN_CLI_OPTS test -Dcucumber.filter.tags="@smoke and $CUCUMBER_TAGS"
    - mvn $MAVEN_CLI_OPTS cluecumber-report:reporting
  artifacts:
    paths:
      - target/cucumber-reports/
      - target/generated-report/
      - target/cucumber.json
    reports:
      junit: target/cucumber.xml
    expire_in: 1 week
    when: always

# ==========================================
# Stage 4: BDD API Tests
# ==========================================
bdd-api:
  stage: bdd-api
  image: maven:3.9-eclipse-temurin-17
  needs: [build]
  services:
    - name: myapp:latest
      alias: app
  script:
    - mvn $MAVEN_CLI_OPTS test -Dcucumber.filter.tags="@api and $CUCUMBER_TAGS"
    - mvn $MAVEN_CLI_OPTS cluecumber-report:reporting
  artifacts:
    paths:
      - target/cucumber-reports/
      - target/cucumber.json
    reports:
      junit: target/cucumber.xml
    expire_in: 1 week
    when: always

# ==========================================
# Stage 5: BDD UI Tests (Matrix)
# ==========================================
bdd-ui-chrome:
  stage: bdd-ui
  image: maven:3.9-eclipse-temurin-17
  needs: [build]
  variables:
    BROWSER: chrome
    HEADLESS: "true"
  script:
    - mvn $MAVEN_CLI_OPTS test -Dcucumber.filter.tags="@ui and $CUCUMBER_TAGS"
    - mvn $MAVEN_CLI_OPTS cluecumber-report:reporting
  artifacts:
    paths:
      - target/cucumber-reports/
      - target/screenshots/
      - target/cucumber.json
    reports:
      junit: target/cucumber.xml
    expire_in: 1 week
    when: always

bdd-ui-firefox:
  stage: bdd-ui
  image: maven:3.9-eclipse-temurin-17
  needs: [build]
  variables:
    BROWSER: firefox
    HEADLESS: "true"
  script:
    - mvn $MAVEN_CLI_OPTS test -Dcucumber.filter.tags="@ui and $CUCUMBER_TAGS"
  artifacts:
    paths:
      - target/cucumber-reports/
      - target/screenshots/
    reports:
      junit: target/cucumber.xml
    expire_in: 1 week
    when: always

# ==========================================
# Stage 6: Full Regression Tests
# ==========================================
bdd-regression:
  stage: bdd-regression
  image: maven:3.9-eclipse-temurin-17
  needs: [bdd-smoke, bdd-api]
  only:
    - main
    - develop
  script:
    - |
      mvn $MAVEN_CLI_OPTS test \
        -Dcucumber.filter.tags="@regression and not @wip and not @flaky"
    - mvn $MAVEN_CLI_OPTS cluecumber-report:reporting
  artifacts:
    paths:
      - target/cucumber-reports/
      - target/generated-report/
      - target/cucumber.json
    reports:
      junit: target/cucumber.xml
    expire_in: 1 week
    when: always

# ==========================================
# Stage 7: Report Generation
# ==========================================
generate-report:
  stage: report
  image: maven:3.9-eclipse-temurin-17
  needs:
    - bdd-smoke
    - bdd-api
    - bdd-ui-chrome
    - bdd-regression
  dependencies:
    - bdd-smoke
    - bdd-api
    - bdd-ui-chrome
    - bdd-regression
  script:
    - echo "Aggregating test reports..."
  artifacts:
    paths:
      - target/generated-report/
    expire_in: 1 month
    when: always
  allow_failure: true
```

### 4.2 GitLab CI for JavaScript Projects

```yaml
# .gitlab-ci.yml (JavaScript/TypeScript)
image: node:20

stages:
  - install
  - test
  - report

cache:
  paths:
    - node_modules/

variables:
  CUCUMBER_TAGS: "not @wip"

install:
  stage: install
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

bdd-smoke:
  stage: test
  needs: [install]
  script:
    - npx cucumber-js --config cucumber.config.ts --tags "@smoke and $CUCUMBER_TAGS"
  artifacts:
    paths:
      - reports/
    expire_in: 1 week
    when: always

bdd-regression:
  stage: test
  needs: [install]
  script:
    - npx cucumber-js --config cucumber.config.ts --tags "@regression and $CUCUMBER_TAGS"
  artifacts:
    paths:
      - reports/
    expire_in: 1 week
    when: always

bdd-ui:
  stage: test
  image: mcr.microsoft.com/playwright:v1.40.0-jammy
  needs: [install]
  script:
    - npx cucumber-js --config cucumber.config.ts --tags "@ui"
    - npx cucumber-js --config cucumber.config.ts --tags "@ui" --parallel 4
  artifacts:
    paths:
      - reports/
      - test-results/
    expire_in: 1 week
    when: always
```

---

## 5. Parallel Execution and Sharding

### 5.1 Test Sharding Strategies

> Source: cucumber-js Official[^91^], Cucumber Official Documentation[^58^]

```yaml
# ==========================================
# GitHub Actions - Matrix Sharding
# ==========================================
bdd-sharded:
  runs-on: ubuntu-latest
  strategy:
    fail-fast: false
    matrix:
      shard: [1, 2, 3, 4]
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'

    # Maven sharding: shard by feature file
    - name: Run BDD Tests (Shard ${{ matrix.shard }}/4)
      run: |
        mvn test \
          -Dcucumber.features="src/test/resources/features/shard${{ matrix.shard }}" \
          -Dcucumber.filter.tags="@regression and not @wip"

    - name: Upload Shard Results
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: shard-${{ matrix.shard }}
        path: target/cucumber.json

# ==========================================
# cucumber-js built-in sharding (v12.2.0+)
# ==========================================
cucumber-js-shard:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      shard: [1, 2, 3]
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: '20'

    - run: npm ci

    # Use --shard parameter
    - run: npx cucumber-js --shard ${{ matrix.shard }}/3

    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: cucumber-shard-${{ matrix.shard }}
        path: reports/
```

### 5.2 Maven Surefire Parallel Configuration

```xml
<!-- pom.xml -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.2.2</version>
    <configuration>
        <!-- Parallel by class (feature file level) -->
        <parallel>classes</parallel>
        <!-- Threads per class -->
        <threadCountClasses>4</threadCountClasses>
        <!-- JVM fork count -->
        <forkCount>4</forkCount>
        <!-- Reuse forks to speed up execution -->
        <reuseForks>true</reuseForks>
        <!-- Number of reruns on failure -->
        <rerunFailingTestsCount>2</rerunFailingTestsCount>
        <systemPropertyVariables>
            <cucumber.execution.parallel.enabled>true</cucumber.execution.parallel.enabled>
            <cucumber.plugin>pretty,json:target/cucumber.json,html:target/cucumber-html-report</cucumber.plugin>
        </systemPropertyVariables>
    </configuration>
</plugin>
```

### 5.3 TestNG Parallel Configuration

```xml
<!-- testng.xml -->
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Cucumber Suite" parallel="tests" thread-count="4">
    <test name="Smoke Tests">
        <classes>
            <class name="com.your.project.SmokeTestSuite"/>
        </classes>
    </test>
    <test name="API Tests">
        <classes>
            <class name="com.your.project.ApiTestSuite"/>
        </classes>
    </test>
    <test name="UI Tests - Chrome">
        <parameter name="browser" value="chrome"/>
        <classes>
            <class name="com.your.project.UiTestSuite"/>
        </classes>
    </test>
    <test name="UI Tests - Firefox">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="com.your.project.UiTestSuite"/>
        </classes>
    </test>
</suite>
```

### 5.4 Docker Compose Parallel Execution

```yaml
# docker-compose.test.yml
version: "3"
services:
  bdd-smoke:
    build: .
    command: mvn test -Dcucumber.filter.tags="@smoke"
    volumes:
      - ./target/smoke-reports:/app/target/cucumber-reports
    environment:
      - SPRING_PROFILES_ACTIVE=test

  bdd-api:
    build: .
    command: mvn test -Dcucumber.filter.tags="@api"
    volumes:
      - ./target/api-reports:/app/target/cucumber-reports
    depends_on:
      - test-db

  bdd-ui:
    build: .
    command: mvn test -Dcucumber.filter.tags="@ui"
    volumes:
      - ./target/ui-reports:/app/target/cucumber-reports
    environment:
      - BROWSER=chrome
      - HEADLESS=true
    depends_on:
      - selenium-hub
      - chrome-node

  test-db:
    image: postgres:16
    environment:
      POSTGRES_DB: testdb
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test

  selenium-hub:
    image: selenium/hub:latest
    ports:
      - "4444:4444"

  chrome-node:
    image: selenium/node-chrome:latest
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_NODE_MAX_SESSIONS=4
```

---

## 6. Failure Retry Strategies

### 6.1 Maven Surefire Auto Retry

```xml
<!-- pom.xml - Failure retry configuration -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.2.2</version>
    <configuration>
        <!-- Number of reruns on failure -->
        <rerunFailingTestsCount>2</rerunFailingTestsCount>
        <!-- Fail immediately on reruns (flaky test handling) -->
        <failIfNoSpecifiedTests>false</failIfNoSpecifiedTests>
    </configuration>
</plugin>
```

### 6.2 cucumber-js Retry Configuration

```typescript
// cucumber.config.ts
import type { Configuration } from '@cucumber/cucumber';

export default {
    paths: ['features/**/*.feature'],
    require: ['step-definitions/**/*.ts'],
    format: [
        'progress',
        'html:reports/cucumber-report.html',
        'json:reports/cucumber-report.json',
        'rerun:@rerun.txt'  // Generate failed scenario list
    ],
    parallel: 4,
    publishQuiet: true,
} satisfies Configuration;
```

```bash
# Retry logic in CI pipeline
#!/bin/bash
set -e

# First run
npx cucumber-js --config cucumber.config.ts --tags "@regression"
EXIT_CODE=$?

# If there are failures, retry failed scenarios
if [ $EXIT_CODE -ne 0 ] && [ -f @rerun.txt ]; then
    echo "Retrying failed scenarios..."
    npx cucumber-js --config cucumber.config.ts @rerun.txt
fi
```

### 6.3 Jenkins Retry

```groovy
// Jenkinsfile - Retry logic
stage('BDD Tests with Retry') {
    steps {
        script {
            def retryCount = 0
            def maxRetries = 2
            def testSuccess = false

            while (!testSuccess && retryCount <= maxRetries) {
                try {
                    sh 'mvn test -Dcucumber.filter.tags="@regression and not @flaky"'
                    testSuccess = true
                } catch (Exception e) {
                    retryCount++
                    if (retryCount <= maxRetries) {
                        echo "Test failed, retrying... (attempt ${retryCount + 1}/${maxRetries + 1})"
                        sh 'sleep 30'
                    } else {
                        error("All retry attempts exhausted")
                    }
                }
            }
        }
    }
}
```

### 6.4 GitHub Actions Retry

```yaml
# GitHub Actions - Using retry action
- name: Run BDD Tests with Retry
  uses: nick-fields/retry@v2
  with:
    timeout_minutes: 30
    max_attempts: 3
    retry_on: error
    command: mvn test -Dcucumber.filter.tags="@regression and not @flaky"
```

### 6.5 Flaky Test Isolation Strategy

```groovy
// Jenkinsfile - Isolate flaky tests
pipeline {
    agent any

    stages {
        stage('Run Stable Tests') {
            steps {
                sh 'mvn test -Dcucumber.filter.tags="not @flaky and not @wip"'
            }
        }

        stage('Run Flaky Tests (Non-blocking)') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh 'mvn test -Dcucumber.filter.tags="@flaky"'
                }
            }
        }
    }
}
```

---

## 7. Reporting and Artifact Management

### 7.1 Report Types Overview

| Report Type | Format | Purpose | CI Integration |
|----------|------|------|---------|
| **JSON** | `.json` | CI/CD parsing, trend analysis | All platforms |
| **HTML** | `.html` | Human review | All platforms |
| **JUnit XML** | `.xml` | CI test report integration | Jenkins, GitLab, GitHub |
| **Allure** | Multi-format | Advanced reporting & trend analysis | All platforms |
| **Rerun** | `.txt` | Failed scenario re-execution | Command line |

### 7.2 Cucumber Configuration Properties

> Source: Cucumber Official Documentation[^37^]

| Property | Description | Example |
|------|------|------|
| `cucumber.plugin` | Report plugin | `pretty, json:path/to/report.json` |
| `cucumber.filter.tags` | Tag expression filter | `@smoke and not @slow` |
| `cucumber.glue` | Step definitions package path | `com.example.glue` |
| `cucumber.features` | Feature file path | `path/to/example.feature` |
| `cucumber.execution.dry-run` | Dry-run mode | `true` or `false` |
| `cucumber.execution.order` | Execution order | `lexical`, `reverse`, `random` |

### 7.3 Multi-Environment Configuration (cucumber.yml)

```yaml
# cucumber.yml - Multi-environment configuration
default:
  --require-module ts-node/register
  --require features/support/*.ts
  --require features/step_definitions/*.ts
  --format progress
  --publish-quiet

ci:
  --require-module ts-node/register
  --require features/support/*.ts
  --require features/step_definitions/*.ts
  --format json:reports/cucumber.json
  --format html:reports/cucumber-report.html
  --format junit:reports/cucumber.xml
  --publish-quiet

html_report:
  --require-module ts-node/register
  --require features/support/*.ts
  --require features/step_definitions/*.ts
  --format html:reports/cucumber-report.html
  --publish-quiet

rerun:
  --require-module ts-node/register
  --require features/support/*.ts
  --require features/step_definitions/*.ts
  --format rerun:@rerun.txt
  --publish-quiet

debug:
  --require-module ts-node/register
  --require features/support/*.ts
  --require features/step_definitions/*.ts
  --tags "@debug"
  --format pretty
  --publish-quiet
```

```bash
# Using different configurations
npx cucumber-js --profile ci
npx cucumber-js --profile html_report
npx cucumber-js --profile debug
npx cucumber-js --profile rerun @rerun.txt
```

### 7.4 Jenkins Artifact Management

```groovy
// Jenkinsfile - Artifact management
post {
    always {
        // Publish Cucumber report
        cucumber(
            buildStatus: 'UNSTABLE',
            fileIncludePattern: '**/cucumber.json',
            fileExcludePattern: '',
            trendsLimit: 10,
            classifications: [
                [key: 'Browser', value: env.BROWSER ?: 'chrome'],
                [key: 'Environment', value: env.ENVIRONMENT ?: 'test'],
                [key: 'Branch', value: env.BRANCH_NAME ?: 'unknown']
            ]
        )

        // Publish HTML report
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'target/cucumber-html-report',
            reportFiles: 'index.html',
            reportName: 'Cucumber HTML Report',
            reportTitles: 'BDD Test Report'
        ])

        // Publish Allure report
        allure([
            includeProperties: false,
            jdk: '',
            properties: [],
            reportBuildPolicy: 'ALWAYS',
            results: [[path: 'target/allure-results']]
        ])

        // Archive artifacts
        archiveArtifacts artifacts: '''
            target/cucumber-reports/**/*,
            target/screenshots/**/*,
            target/allure-results/**/*
        ''', allowEmptyArchive: true

        // JUnit test results
        junit testResults: 'target/cucumber.xml', allowEmptyResults: true
    }
}
```

### 7.5 GitHub Actions Artifact Management

```yaml
# GitHub Actions - Artifact upload
- name: Upload Test Results
  uses: actions/upload-artifact@v4
  if: always()
  with:
    name: cucumber-reports-${{ matrix.browser }}
    path: |
      target/cucumber.json
      target/cucumber-html-report/
      target/screenshots/
      target/allure-results/
    retention-days: 30

# Download all artifacts for aggregation
- name: Download All Artifacts
  uses: actions/download-artifact@v4
  with:
    path: all-reports
    pattern: cucumber-reports-*

# Publish to GitHub Pages
- name: Deploy Report to GitHub Pages
  uses: peaceiris/actions-gh-pages@v3
  if: always() && github.ref == 'refs/heads/main'
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: target/cucumber-html-report
    destination_dir: reports/${{ github.run_number }}
```

### 7.6 GitLab CI Artifact Management

```yaml
bdd-tests:
  script:
    - mvn test -Dcucumber.filter.tags="@regression"
  artifacts:
    paths:
      - target/cucumber-reports/
      - target/cucumber.json
      - target/cucumber.xml
      - target/screenshots/
    reports:
      junit: target/cucumber.xml
    expire_in: 1 month
    when: always

# Display test reports on merge requests
pages:
  stage: deploy
  dependencies:
    - bdd-tests
  script:
    - mkdir -p public
    - cp -r target/cucumber-html-report/* public/
  artifacts:
    paths:
      - public
  only:
    - main
```

### 7.7 Test Trend Tracking

```groovy
// Jenkinsfile - Trend tracking
cucumber(
    buildStatus: 'UNSTABLE',
    fileIncludePattern: '**/cucumber.json',
    trendsLimit: 20,  // Retain trends for 20 builds
    classifications: [
        [key: 'Project', value: 'MyApplication'],
        [key: 'Version', value: env.APP_VERSION]
    ]
)
```

---

## 8. Quality Gates

### 8.1 Quality Gate Metrics

| Metric | Threshold Recommendation | Description |
|------|----------|------|
| **Test Pass Rate** | >= 95% | Proportion of scenarios that must pass |
| **Smoke Tests** | 100% pass | Smoke tests must all pass |
| **Code Coverage** | >= 80% | Unit test coverage |
| **Flaky Test Ratio** | < 5% | Proportion of tests marked @flaky |
| **Test Execution Time** | < 30 minutes | Full regression test duration |
| **New Test Coverage** | >= 1 per feature | New features must include BDD scenarios |

### 8.2 Jenkins Quality Gate

```groovy
// Jenkinsfile - Quality gate
pipeline {
    agent any

    stages {
        stage('BDD Tests') {
            steps {
                sh 'mvn test -Dcucumber.filter.tags="@regression"'
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    // Read Cucumber JSON report
                    def cucumberReport = readJSON file: 'target/cucumber.json'

                    def total = 0
                    def passed = 0
                    def failed = 0

                    cucumberReport.each { feature ->
                        feature.elements?.each { scenario ->
                            total++
                            def scenarioPassed = true
                            scenario.steps?.each { step ->
                                if (step.result?.status != 'passed') {
                                    scenarioPassed = false
                                }
                            }
                            if (scenarioPassed) {
                                passed++
                            } else {
                                failed++
                            }
                        }
                    }

                    def passRate = total > 0 ? (passed / total) * 100 : 0

                    echo "Test Results: ${passed}/${total} passed (${passRate}%)"
                    echo "Failed: ${failed}"

                    // Quality gate check
                    if (passRate < 95) {
                        error("Quality Gate Failed: Pass rate ${passRate}% is below 95% threshold")
                    }

                    if (failed > 5) {
                        error("Quality Gate Failed: ${failed} tests failed (threshold: 5)")
                    }
                }
            }
        }
    }
}
```

### 8.3 GitHub Actions Quality Gate

```yaml
# .github/workflows/quality-gate.yml
name: Quality Gate

on:
  workflow_run:
    workflows: ["BDD Cucumber Tests"]
    types:
      - completed

jobs:
  quality-gate:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - name: Download Test Results
        uses: actions/download-artifact@v4
        with:
          name: cucumber-reports
          path: reports/

      - name: Check Quality Gate
        run: |
          # Parse JSON report
          TOTAL=$(jq '[.[].elements | length] | add' reports/cucumber.json)
          PASSED=$(jq '[.[].elements[]?.steps[]?.result.status == "passed"] | length' reports/cucumber.json)

          if [ "$TOTAL" -eq 0 ]; then
            echo "No tests found!"
            exit 1
          fi

          PASS_RATE=$((PASSED * 100 / TOTAL))
          echo "Test pass rate: ${PASS_RATE}% (${PASSED}/${TOTAL})"

          if [ "$PASS_RATE" -lt 95 ]; then
            echo "Quality Gate FAILED: Pass rate ${PASS_RATE}% is below 95%"
            exit 1
          fi

          echo "Quality Gate PASSED"

      - name: Update PR Status
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.repos.createCommitStatus({
              owner: context.repo.owner,
              repo: context.repo.repo,
              sha: context.sha,
              state: 'success',
              context: 'Quality Gate',
              description: 'All quality criteria met'
            })
```

### 8.4 GitLab CI Quality Gate

```yaml
# .gitlab-ci.yml - Quality gate
bdd-regression:
  stage: test
  script:
    - mvn test -Dcucumber.filter.tags="@regression"
  artifacts:
    reports:
      junit: target/cucumber.xml
    paths:
      - target/cucumber.json
    expire_in: 1 week

# Use GitLab built-in quality gate
quality-gate:
  stage: report
  needs: [bdd-regression]
  script:
    - |
      # Check test pass rate
      TOTAL=$(jq '[.[].elements | length] | add' target/cucumber.json)
      PASSED=$(jq '[.[].elements[]?.steps[]?.result.status == "passed"] | length' target/cucumber.json)
      PASS_RATE=$((PASSED * 100 / TOTAL))

      echo "Pass rate: ${PASS_RATE}%"

      if [ "$PASS_RATE" -lt 95 ]; then
        echo "Quality gate failed"
        exit 1
      fi
  allow_failure: false  # Block merge on failure

# GitLab merge request test reports
include:
  - template: Jobs/Code-Quality.gitlab-ci.yml
```

### 8.5 PR Pre-Merge Checklist

```yaml
# .github/workflows/pr-checklist.yml
name: PR Quality Check

on:
  pull_request:
    branches: [main]

jobs:
  pr-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Check if new features have BDD scenarios
      - name: Check BDD Coverage
        run: |
          # Check feature file changes
          FEATURE_CHANGES=$(git diff --name-only origin/main...HEAD | grep -c '\.feature$' || true)

          # Check code changes
          CODE_CHANGES=$(git diff --name-only origin/main...HEAD | grep -E '\.(java|ts|js|py)$' | grep -v 'test' | grep -v 'spec' | wc -l || true)

          echo "Feature file changes: $FEATURE_CHANGES"
          echo "Code changes: $CODE_CHANGES"

          if [ "$CODE_CHANGES" -gt 0 ] && [ "$FEATURE_CHANGES" -eq 0 ]; then
            echo "WARNING: Code changes detected without BDD feature file updates"
            echo "Please ensure all new features are covered by BDD scenarios"
            # Don't fail, just warn
          fi

      # Check for @wip scenarios
      - name: Check WIP Scenarios
        run: |
          WIP_COUNT=$(grep -r "@wip" features/ --include="*.feature" | wc -l || true)
          echo "WIP scenarios: $WIP_COUNT"

          if [ "$WIP_COUNT" -gt 5 ]; then
            echo "WARNING: Too many WIP scenarios ($WIP_COUNT). Consider completing them."
          fi
```

### 8.6 Quality Gate Summary

```
+---------------------+      +---------------------+      +---------------------+
|   Smoke Tests       |  --> |   BDD Scenarios     |  --> |   Quality Gate      |
|   (100% pass)       |      |   (>= 95% pass)     |      |   (all criteria)     |
+---------------------+      +---------------------+      +---------------------+
         |                              |                           |
         v                              v                           v
   [BLOCK if fail]               [WARN if < 100%]            [BLOCK if fail]
```

| Gate Level | Trigger Condition | Result |
|----------|----------|------|
| **Smoke Gate** | Every commit | Block subsequent stages on failure |
| **API Gate** | Every PR | Block merge on failure |
| **UI Gate** | Every PR | Block merge on failure |
| **Regression Gate** | Merge to main | Block deployment on failure |
| **Flaky Test Gate** | Daily check | Alert if too many flaky tests |

---

## Source Index

### Tier 1 - Official Documentation

| # | Source | URL | Referenced Content |
|---|------|-----|---------|
| [^85^] | Cucumber CI Guide | https://cucumber.io/docs/guides/continuous-integration/ | Official CI Integration Guide |
| [^58^] | Cucumber Parallel | https://cucumber.io/docs/guides/parallel-execution/ | Parallel Execution Configuration |
| [^91^] | cucumber-js Releases | https://github.com/cucumber/cucumber-js/releases | v12.x Sharding Support |

### Tier 2 - Authoritative Technical Sources

| # | Source | URL | Referenced Content |
|---|------|-----|---------|
| [^7^] | TestMu Jenkins | https://www.testmuai.com/blog/cucumber-with-jenkins-integration/ | Jenkins Integration Steps |
| [^105^] | Daniela Baron | https://danielabaron.me/blog/sustainable-feature-testing-in-rails-with-cucumber/ | GitHub Actions Configuration |
| [^107^] | TestQuality CI/CD | https://testquality.com/how-to-integrate-gherkin-testing-in-your-ci-cd-pipeline/ | CI/CD Configuration Examples |
| [^108^] | Testomat.io | https://testomat.io/blog/bdd-development-workflow/ | cucumber.yml Multi-Environment Configuration |
| [^111^] | Varseno | https://www.varseno.com/playwright-cucumber-ci-cd-github-actions/ | GitHub Actions Full Configuration |
| [^12^] | LambdaTest Jenkins | https://www.lambdatest.com/blog/cucumber-with-jenkins-integration/ | Jenkins Detailed Configuration |

### Tier 3 - Community and Tools

| # | Source | URL | Referenced Content |
|---|------|-----|---------|
| [^87^] | Jenkins Cucumber Plugin | https://plugins.jenkins.io/cucumber-reports/ | Jenkins Plugin Configuration |
| [^76^] | revanab.com | https://revanab.com/blog/ui-testing-with-cucumber-bdd | Cluecumber Report Configuration |
| [^57^] | Volcengine | https://www.volcengine.com/article/212986 | Maven Fork Parallel Execution |

---

> This reference manual is based on comprehensive research from 25+ authoritative sources, covering Cucumber official documentation (Tier 1), authoritative technical blogs (Tier 2), and community best practices (Tier 3). Pipeline configurations can be adjusted according to specific project technology stacks and environments.
