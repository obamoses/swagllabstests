# Sauce Demo – Enterprise Test Automation Framework

End-to-end UI test automation for [Sauce Demo](https://www.saucedemo.com/), built with **Playwright** and **TypeScript**, with **Cucumber (BDD)** and **GitHub Actions CI** as the target architecture.

The framework is designed as an enterprise-style reference: layered, maintainable, environment-driven, and CI-first.

---

## Status

| Area | Status |
|---|---|
| Playwright + TypeScript setup | In place |
| Page Object Model | In progress |
| Cucumber (BDD) integration | Planned |
| GitHub Actions CI | Planned |
| Parallel execution and sharding | Planned |
| Reporting (HTML, Allure) | Planned |

Update this table as each item lands so the README never overstates what exists.

---

## Application Under Test

- **URL:** https://www.saucedemo.com/
- **Type:** Public demo e-commerce site (login, inventory, cart, checkout)
- **Test users:** Sauce Demo publishes its demo accounts on the login page. All share one password.

| Username | Purpose |
|---|---|
| `standard_user` | Happy-path flows |
| `locked_out_user` | Negative login scenario |
| `problem_user` | UI and behavior defects |
| `performance_glitch_user` | Slow responses |
| `error_user` | Error handling |
| `visual_user` | Visual regression |

Credentials are read from environment variables, not hardcoded in tests (see [Configuration](#configuration)).

---

## Tech Stack

- [Playwright](https://playwright.dev/) – browser automation and test runner
- TypeScript – typed test code
- Cucumber / Gherkin – business-readable scenarios (planned)
- GitHub Actions – CI/CD (planned)
- ESLint + Prettier – code quality

---

## Prerequisites

- Node.js 20 LTS or later
- npm
- Git

---

## Getting Started

```bash
# 1. Clone
git clone https://github.com/obamoses/swagllabstests.git
cd swagllabstests

# 2. Install dependencies
npm ci

# 3. Install Playwright browsers
npx playwright install --with-deps

# 4. Create your local environment file
cp .env.example .env
```

---

## Running Tests

```bash
# Run all tests (headless)
npx playwright test

# Run in UI mode (watch, debug, time-travel)
npx playwright test --ui

# Run in a visible browser
npx playwright test --headed

# Run with the Inspector
npx playwright test --debug

# Run one file
npx playwright test tests/login.spec.ts

# Filter by test title
npx playwright test -g "login"

# Run one browser project
npx playwright test --project=chromium

# Open the last HTML report
npx playwright show-report
```

### Slow-motion playback

Set `slowMo` under `use.launchOptions` in `playwright.config.ts`, then run with `--headed`:

```ts
use: {
  headless: false,
  launchOptions: { slowMo: 1000 },
}
```

---

## Configuration

Environment is controlled through variables, never hardcoded.

`.env.example`:

```
BASE_URL=https://www.saucedemo.com
STANDARD_USER=standard_user
LOCKED_USER=locked_out_user
USER_PASSWORD=secret_sauce
```

- Commit `.env.example`. Never commit `.env`.
- In CI, supply values through GitHub Actions secrets and variables.

---

## Target Project Structure

```
.
├── .github/
│   ├── workflows/
│   │   └── playwright.yml        # CI pipeline
│   ├── CODEOWNERS
│   └── pull_request_template.md
├── features/                     # Gherkin feature files (planned)
│   ├── login.feature
│   ├── inventory.feature
│   ├── cart.feature
│   └── checkout.feature
├── src/
│   ├── pages/                    # Page objects
│   ├── components/               # Reusable UI components (header, cart badge)
│   ├── steps/                    # Cucumber step definitions (planned)
│   ├── fixtures/                 # Playwright fixtures
│   ├── data/                     # Test data and factories
│   └── utils/                    # Helpers
├── tests/                        # Playwright spec tests
│   ├── smoke/
│   └── regression/
├── playwright.config.ts
├── tsconfig.json
├── package.json
├── .env.example
├── .gitignore
└── README.md
```

---

## Design Principles

- **Page Object Model.** Locators and page actions live in `src/pages`. Tests contain intent and assertions only.
- **Resilient locators.** Prefer `getByRole`, `getByLabel`, and `data-test` attributes. Avoid brittle CSS and XPath chains.
- **No hard waits.** Rely on Playwright auto-waiting and web-first assertions.
- **Independent tests.** Each test sets up its own state and can run in any order, in parallel.
- **Config over code.** URLs, credentials, and timeouts come from environment configuration.
- **Tagging.** Tag tests `@smoke`, `@regression`, `@negative` so CI can run focused subsets with `--grep`.

---

## Roadmap

- [ ] Finish page objects: Login, Inventory, Cart, Checkout, Order Complete
- [ ] Integrate Cucumber with Playwright; write feature files and step definitions
- [ ] Authenticated session reuse (`storageState`) to skip repeated logins
- [ ] ESLint, Prettier, and pre-commit hooks
- [ ] GitHub Actions workflow: smoke on pull requests, full regression on merge and on schedule
- [ ] Sharded parallel runs in CI
- [ ] Upload HTML report, traces, and videos as artifacts on failure
- [ ] Allure or equivalent reporting
- [ ] Branch protection on `main` with required passing checks

---

## Continuous Integration (planned)

The GitHub Actions workflow will:

1. Check out the repo and install dependencies with `npm ci`
2. Install Playwright browsers
3. Run the suite (smoke subset on pull requests, full regression on `main`)
4. Upload the Playwright report and traces as build artifacts

Minimal starting workflow, `.github/workflows/playwright.yml`:

```yaml
name: Playwright Tests
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: lts/*
          cache: npm
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 14
```

---

## Contributing

1. Branch from `main`: `git checkout -b feature/<short-description>`
2. Add or update tests following the conventions above
3. Run the suite locally before pushing
4. Open a pull request; CI must pass before merge

---

## License

Add a license before sharing this repository publicly.