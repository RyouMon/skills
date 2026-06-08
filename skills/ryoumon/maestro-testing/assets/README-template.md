# Maestro Test Suite

End-to-end UI tests for [App Name] using [Maestro](https://maestro.dev).

## Prerequisites

- Java 17+
- Maestro CLI: `curl -fsSL "https://get.maestro.mobile.dev" | bash`
- [iOS] macOS + Xcode + iOS Simulator
- [Android] Android Studio + Emulator or physical device

## Project Structure

```
maestro/
├── config.yaml          # Test discovery, tags, env vars
├── flows/               # Test flows
│   ├── auth/            # Authentication tests
│   ├── [feature]/       # Feature-specific tests
│   └── common/          # Reusable subflows (setup, login, teardown)
├── scripts/             # JavaScript helpers
│   ├── selectors.js     # Page Object Model definitions
│   └── helpers.js       # Test data, API helpers
└── .env                 # Environment variables (gitignored)
```

## Quick Start

```bash
# Run all tests
maestro test flows/

# Run smoke tests only
maestro test --include-tags smoke flows/

# Run a specific flow
maestro test flows/auth/login.yaml

# Run with custom env vars
maestro test -e APP_ID=com.example.app -e ENV=staging flows/
```

## Environment Variables

Copy `.env.example` to `.env` and configure:

```bash
MAESTRO_APP_ID=com.example.app
MAESTRO_TEST_USER_EMAIL=test@example.com
MAESTRO_TEST_USER_PASSWORD=TestPass123
MAESTRO_BASE_URL=https://api.example.com
```

## CI/CD

See `.github/workflows/maestro.yml` for GitHub Actions integration.

## Tags

| Tag | Description |
|-----|-------------|
| `smoke` | Critical path tests (run on every PR) |
| `regression` | Full test suite (run nightly) |
| `critical` | Must-pass business-critical flows |
| `skip` | Excluded from all runs |
| `wip` | Work in progress |

## Writing Tests

See the [Maestro Documentation](https://docs.maestro.dev) for command reference.

Key patterns:
- Use `id` selectors (testID) for stability
- Reuse subflows via `runFlow`
- Tag utility flows with `skip`
- Never use hard-coded `sleep()`
