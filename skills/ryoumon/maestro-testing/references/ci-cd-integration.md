# Maestro CI/CD Integration & Cloud Testing Guide

Complete guide for integrating Maestro into CI/CD pipelines and using Maestro Cloud.

## Table of Contents

1. [Maestro Cloud](#maestro-cloud)
2. [GitHub Actions](#github-actions)
3. [GitLab CI](#gitlab-ci)
4. [CircleCI](#circleci)
5. [Bitrise](#bitrise)
6. [Jenkins](#jenkins)
7. [Other CI Systems](#other-ci-systems)
8. [Environment Management](#environment-management)
9. [Reporting & Notifications](#reporting--notifications)
10. [Performance Optimization](#performance-optimization)

---

## Maestro Cloud

Maestro Cloud is a managed device cloud for running tests at scale.

### Pricing & Features

| Tier | Price | Features |
|------|-------|----------|
| Starter | Free tier | Limited runs |
| Mobile | $250/device/month | Android + iOS virtual devices |
| Web | $125/browser/month | Browser testing |
| Enterprise | Custom | SSO, premium support, custom contracts |

### Cloud CLI Usage

```bash
# Upload and run tests
maestro cloud \
  --api-key $MAESTRO_API_KEY \
  --project-id $MAESTRO_PROJECT_ID \
  --device-model "Pixel 7" \
  --device-os "Android 14" \
  app.apk \
  flows/

# Async execution (don't wait for results)
maestro cloud \
  --api-key $MAESTRO_API_KEY \
  --project-id $MAESTRO_PROJECT_ID \
  --async \
  app.apk \
  flows/

# Specify tags
maestro cloud \
  --api-key $MAESTRO_API_KEY \
  --project-id $MAESTRO_PROJECT_ID \
  --include-tags smoke \
  --exclude-tags skip \
  app.apk \
  flows/

# Environment variables
maestro cloud \
  --api-key $MAESTRO_API_KEY \
  --project-id $MAESTRO_PROJECT_ID \
  -e ENV=staging \
  -e API_URL=https://staging.example.com \
  app.apk \
  flows/
```

### Cloud Parameters

| Parameter | Description |
|-----------|-------------|
| `--api-key` | API key for authentication |
| `--project-id` | Project identifier |
| `--device-model` | Device model (e.g., `Pixel 7`, `iPhone 15`) |
| `--device-os` | OS version (e.g., `Android 14`, `iOS 17`) |
| `--async` | Upload and exit without waiting for results |
| `--include-tags` | Tags to include |
| `--exclude-tags` | Tags to exclude |
| `--repo-name` | Repository name (for PR comments) |
| `--commit-sha` | Commit SHA (for PR comments) |
| `--pull-request-id` | PR ID (for PR comments) |
| `--app-binary-id` | Reuse previously uploaded app binary |

### Cloud Limitations

- **7-minute soft timeout** per test execution
- **Virtual devices only** (not physical devices)
- **Device isolation**: Devices wiped and recreated between tests
- **SOC 2 compliant** with AES-256 at rest, TLS in transit
- **IP allowlist**: 4 static IPs available for firewall configuration

---

## GitHub Actions

### Official Action (Cloud)

```yaml
# .github/workflows/maestro-cloud.yml
name: Maestro Cloud Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  maestro-cloud:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Build your app (Android example)
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build APK
        run: ./gradlew assembleDebug

      # Run Maestro Cloud tests
      - name: Run Maestro Cloud
        uses: mobile-dev-inc/action-maestro-cloud@v2.0.2
        with:
          api-key: ${{ secrets.MAESTRO_API_KEY }}
          project-id: ${{ secrets.MAESTRO_PROJECT_ID }}
          app-file: app/build/outputs/apk/debug/app-debug.apk
          # Optional parameters:
          include-tags: smoke
          exclude-tags: skip
          device-model: Pixel 7
          device-os: Android 14
          env: |
            ENV=staging
            API_URL=https://staging.example.com
```

### Local Emulator Testing (Community Action)

```yaml
# .github/workflows/maestro-local.yml
name: Maestro Local Tests

on:
  pull_request:
    branches: [main]

jobs:
  maestro-local:
    # Use macOS for hardware acceleration
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Enable KVM
        run: |
          echo 'KERNEL=="kvm", GROUP="kvm", MODE="0666", OPTIONS+="static_node=kvm"' | sudo tee /etc/udev/rules.d/99-kvm4all.rules
          sudo udevadm control --reload-rules
          sudo udevadm trigger --name-match=kvm

      - name: AVD Cache
        uses: actions/cache@v4
        id: avd-cache
        with:
          path: |
            ~/.android/avd/*
            ~/.android/adb*
          key: avd-${{ matrix.api-level }}

      - name: Create AVD and generate snapshot
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: 33
          target: google_apis
          arch: x86_64
          force-avd-creation: false
          emulator-options: -no-window -gpu swiftshader_indirect -noaudio -no-boot-anim -camera-back none
          disable-animations: true
          script: echo "Generated AVD snapshot."

      - name: Install Maestro
        run: |
          curl -fsSL "https://get.maestro.mobile.dev" | bash
          echo "$HOME/.maestro/bin" >> $GITHUB_PATH

      - name: Run Maestro Tests
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: 33
          target: google_apis
          arch: x86_64
          force-avd-creation: false
          emulator-options: -no-snapshot-save -no-window -gpu swiftshader_indirect -noaudio -no-boot-anim
          disable-animations: true
          script: maestro test --format junit --output report.xml flows/

      - name: Upload Test Report
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: report.xml
```

> **Warning**: Avoid `ubuntu-latest` for Android emulator testing. Startup takes 10+ minutes and is flaky. Use `macos-latest` for hardware acceleration.

---

## GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test

variables:
  MAESTRO_VERSION: "latest"

build_android:
  stage: build
  image: openjdk:17-jdk
  script:
    - ./gradlew assembleDebug
  artifacts:
    paths:
      - app/build/outputs/apk/debug/app-debug.apk
    expire_in: 1 hour

maestro_tests:
  stage: test
  image: ubuntu:latest
  needs:
    - build_android
  before_script:
    - apt-get update && apt-get install -y curl unzip
    - curl -fsSL "https://get.maestro.mobile.dev" | bash
    - export PATH="$HOME/.maestro/bin:$PATH"
  script:
    - maestro test
        --app-file app/build/outputs/apk/debug/app-debug.apk
        --format junit
        --output maestro-report.xml
        flows/
  artifacts:
    when: always
    reports:
      junit: maestro-report.xml
    paths:
      - maestro-report.xml
      - ~/.maestro/tests/**
```

---

## CircleCI

```yaml
# .circleci/config.yml
version: 2.1

jobs:
  maestro-tests:
    macos:
      xcode: 15.0.0
    steps:
      - checkout

      - run:
          name: Build iOS App
          command: xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Debug -sdk iphonesimulator -derivedDataPath build

      - run:
          name: Install Maestro
          command: |
            curl -fsSL "https://get.maestro.mobile.dev" | bash
            echo 'export PATH="$HOME/.maestro/bin:$PATH"' >> $BASH_ENV

      - run:
          name: Start Simulator
          command: xcrun simctl boot "iPhone 15"

      - run:
          name: Install App
          command: xcrun simctl install booted build/Build/Products/Debug-iphonesimulator/MyApp.app

      - run:
          name: Run Maestro Tests
          command: maestro test --format junit --output report.xml flows/

      - store_test_results:
          path: report.xml

      - store_artifacts:
          path: ~/.maestro/tests/

workflows:
  test:
    jobs:
      - maestro-tests
```

---

## Bitrise

Bitrise has a native Maestro step:

```yaml
# bitrise.yml
workflows:
  maestro-tests:
    steps:
      - activate-ssh-key@4: {}
      - git-clone@8: {}
      - install-missing-android-tools@3: {}
      - gradle-runner@2:
          inputs:
            - gradle_task: assembleDebug
      - maestro-cloud-upload@1:
          inputs:
            - api_key: $MAESTRO_API_KEY
            - project_id: $MAESTRO_PROJECT_ID
            - app_file: $BITRISE_APK_PATH
            - include_tags: smoke
            - exclude_tags: skip
```

---

## Jenkins

```groovy
// Jenkinsfile
pipeline {
    agent { label 'macos' }

    environment {
        MAESTRO_API_KEY = credentials('maestro-api-key')
    }

    stages {
        stage('Build') {
            steps {
                sh './gradlew assembleDebug'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    curl -fsSL "https://get.maestro.mobile.dev" | bash
                    export PATH="$HOME/.maestro/bin:$PATH"
                    maestro test \
                        --app-file app/build/outputs/apk/debug/app-debug.apk \
                        --format junit \
                        --output report.xml \
                        flows/
                '''
            }
        }
    }

    post {
        always {
            junit 'report.xml'
            archiveArtifacts artifacts: 'report.xml,~/.maestro/tests/**/*', allowEmptyArchive: true
        }
    }
}
```

---

## Other CI Systems

### Azure DevOps

```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'macos-latest'

steps:
  - task: JavaToolInstaller@0
    inputs:
      versionSpec: '17'
      jdkArchitectureOption: 'x64'

  - script: ./gradlew assembleDebug
    displayName: 'Build APK'

  - script: |
      curl -fsSL "https://get.maestro.mobile.dev" | bash
      export PATH="$HOME/.maestro/bin:$PATH"
      maestro test --app-file app/build/outputs/apk/debug/app-debug.apk --format junit --output report.xml flows/
    displayName: 'Run Maestro Tests'

  - task: PublishTestResults@2
    inputs:
      testResultsFormat: 'JUnit'
      testResultsFiles: 'report.xml'
    condition: always()
```

### Bitbucket Pipelines

Bitbucket has a native Pipe:

```yaml
# bitbucket-pipelines.yml
image: openjdk:17

pipelines:
  default:
    - step:
        name: Build
        script:
          - ./gradlew assembleDebug
        artifacts:
          - app/build/outputs/apk/debug/app-debug.apk
    - pipe:
        mobiledevinc/maestro-cloud-upload:1.2.3
        variables:
          API_KEY: $MAESTRO_API_KEY
          PROJECT_ID: $MAESTRO_PROJECT_ID
          APP_FILE: 'app/build/outputs/apk/debug/app-debug.apk'
```

---

## Environment Management

### Per-Environment Configuration

```yaml
# config.yaml
env:
  # Default (development)
  BASE_URL: https://dev.example.com
  TIMEOUT: 10000

# Override via CLI for staging:
# maestro test -e BASE_URL=https://staging.example.com flows/
```

### Secrets Management

**Never commit secrets to your repository.** Use CI-native secrets:

| CI System | Secret Storage |
|-----------|---------------|
| GitHub Actions | Repository Secrets (`Settings > Secrets and variables`) |
| GitLab CI | CI/CD Variables (`Settings > CI/CD > Variables`) |
| CircleCI | Contexts + Environment Variables |
| Bitrise | Secrets (`Workflow > Secrets`) |
| Jenkins | Credentials Plugin |
| Azure DevOps | Library Variable Groups |

```yaml
# In GitHub Actions — reference secrets
- name: Run Maestro
  env:
    MAESTRO_API_KEY: ${{ secrets.MAESTRO_API_KEY }}
    TEST_PASSWORD: ${{ secrets.TEST_PASSWORD }}
  run: maestro test -e PASSWORD=$TEST_PASSWORD flows/
```

### App Binary Reuse

For faster CI builds, reuse the same app binary across test runs:

```bash
# Upload binary once
maestro cloud --api-key $KEY --project-id $ID --async app.apk

# Reference by binary ID in subsequent runs
maestro cloud --api-key $KEY --project-id $ID --app-binary-id $BINARY_ID flows/
```

---

## Reporting & Notifications

### Report Formats

```bash
# JUnit XML (for CI integration)
maestro test --format junit --output report.xml flows/

# HTML report
maestro test --format html --output report.html flows/

# Detailed HTML report
maestro test --format html-detailed --output report.html flows/
```

### Slack Notifications (Maestro Cloud)

1. Connect Slack workspace in Maestro Cloud dashboard
2. Choose notification channels
3. Configure failed-only mode (optional)

### Webhook Notifications

```bash
maestro cloud \
  --api-key $KEY \
  --project-id $ID \
  --webhook-url https://hooks.slack.com/services/... \
  --webhook-bearer-token $TOKEN \
  app.apk \
  flows/
```

### Email Notifications

Configure in Maestro Cloud dashboard per project.

---

## Performance Optimization

### Local Execution Speed

```bash
# 1. Disable animations in config.yaml
# 2. Use interpreted YAML (no compilation needed)
# 3. Use watch mode for development
maestro test --continuous Flow.yaml

# 4. Sharded parallel execution
maestro test --shards 4 --shard-split flows/

# 5. Tag-based selective execution
maestro test --include-tags smoke flows/   # PR checks (fast)
maestro test flows/                        # Full suite (nightly)
```

### CI Performance Tips

| Tip | Implementation |
|-----|----------------|
| Cache Maestro CLI | Cache `~/.maestro/` between runs |
| Pin Maestro version | Avoid unexpected changes |
| Use macOS runners | Hardware acceleration for emulators |
| Pre-start emulators | Save AVD snapshots |
| Parallel execution | `--shards N --shard-split` |
| Selective testing | Tag-based test selection per PR |
| Binary reuse | `--app-binary-id` for Cloud |

### Maestro Cloud Parallel Execution

- Up to 100 parallel runners on Enterprise tier
- Up to 90% execution time reduction claimed
- Each flow runs on a fresh device

```bash
# All flows run in parallel by default on Cloud
maestro cloud --api-key $KEY --project-id $ID app.apk flows/
```
