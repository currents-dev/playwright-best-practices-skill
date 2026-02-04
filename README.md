```

░█▀█░█░░░█▀█░█░█░█░█░█▀▄░▀█▀░█▀▀░█░█░▀█▀░░░█▀▄░█▀▀░█▀▀░▀█▀░░░█▀█░█▀▄░█▀█░█▀▀░▀█▀░▀█▀░█▀▀░█▀▀░█▀▀░
░█▀▀░█░░░█▀█░░█░░█▄█░█▀▄░░█░░█░█░█▀█░░█░░░░█▀▄░█▀▀░▀▀█░░█░░░░█▀▀░█▀▄░█▀█░█░░░░█░░░█░░█░░░█▀▀░▀▀█░
░▀░░░▀▀▀░▀░▀░░▀░░▀░▀░▀░▀░▀▀▀░▀▀▀░▀░▀░░▀░░░░▀▀░░▀▀▀░▀▀▀░░▀░░░░▀░░░▀░▀░▀░▀░▀▀▀░░▀░░▀▀▀░▀▀▀░▀▀▀░▀▀▀░
```

# Playwright Best Practices Skill

A skill that gives the AI specialized guidance for writing, debugging, and maintaining **Playwright** tests in **TypeScript**. Use it in any repo where you work with Playwright so the assistant follows best practices for E2E, component, API, visual regression, accessibility, security, and i18n testing.

## Installation

```bash
npx skills add https://github.com/currents-dev/playwright-best-practices-skill
```

The skill is activity-based: the AI is directed to the right reference depending on what you're doing, so you get focused advice without loading everything at once.

## When the Skill Is Used

The skill triggers when the AI infers you need help with things like:

- Writing new E2E, component, API, visual regression, or accessibility tests
- Testing mobile/responsive layouts, touch gestures, or device emulation
- Implementing file uploads/downloads, date/time mocking, or WebSocket testing
- Handling OAuth popups, geolocation, permissions, or multi-tab flows
- Testing iframes, canvas/WebGL, service workers, or PWA features
- Internationalization (i18n), locales, RTL layouts, or date/number formats
- Testing error states, offline mode, or network failure scenarios
- Security testing (XSS, CSRF, authentication, authorization)
- Performance testing with Web Vitals or Lighthouse
- Reviewing or refactoring Playwright test code
- Fixing flaky tests or debugging failures
- Setting up CI/CD, test coverage, or global setup/teardown
- Configuring projects, dependencies, parallel runs, or sharding

You don't have to mention "skill" or "Playwright best practices"; describe your task (e.g. "fix this flaky login test" or "add accessibility tests") and the AI will use the skill when it's relevant.

## What's Inside

### Core Testing

| Topic                | Reference               | Use for                                           |
| -------------------- | ----------------------- | ------------------------------------------------- |
| Test organization    | `test-organization.md`  | Structure, config, E2E/component/API/visual tests |
| Locators             | `locators.md`           | Selectors, robustness, avoiding brittle locators  |
| Assertions & waiting | `assertions-waiting.md` | Expect APIs, auto-waiting, polling                |
| Page Object Model    | `page-object-model.md`  | POM structure and patterns                        |
| Fixtures & hooks     | `fixtures-hooks.md`     | Setup, teardown, auth, custom fixtures            |
| Test data            | `test-data.md`          | Factories, Faker, data-driven testing             |
| Annotations          | `annotations.md`        | skip, fixme, slow, test steps                     |

### Specialized Testing

| Topic               | Reference                | Use for                                        |
| ------------------- | ------------------------ | ---------------------------------------------- |
| Accessibility       | `accessibility.md`       | Axe-core, keyboard nav, ARIA, focus management |
| Mobile testing      | `mobile-testing.md`      | Device emulation, touch gestures, viewports    |
| Component testing   | `component-testing.md`   | CT setup, mounting, props, mocking             |
| File operations     | `file-operations.md`     | Upload, download, drag-and-drop                |
| Clock mocking       | `clock-mocking.md`       | Date/time mocking, timezones, timers           |
| WebSockets          | `websockets.md`          | Real-time testing, SSE, reconnection           |
| Browser APIs        | `browser-apis.md`        | Geolocation, permissions, clipboard, camera    |
| Multi-context       | `multi-context.md`       | Popups, new tabs, OAuth flows                  |
| Multi-user          | `multi-user.md`          | Collaboration, RBAC, concurrent actions        |
| iFrames             | `iframes.md`             | Cross-origin, nested, dynamic iframes          |
| Canvas/WebGL        | `canvas-webgl.md`        | Canvas testing, charts, WebGL, games           |
| Service workers     | `service-workers.md`     | PWA, caching, offline, push notifications      |
| i18n                | `i18n.md`                | Locales, RTL, date/number formats              |
| Error testing       | `error-testing.md`       | Error boundaries, offline, network failures    |
| Security testing    | `security-testing.md`    | XSS, CSRF, auth security, authorization        |
| Performance testing | `performance-testing.md` | Web Vitals, budgets, Lighthouse                |

### Infrastructure & Advanced

| Topic            | Reference                  | Use for                                  |
| ---------------- | -------------------------- | ---------------------------------------- |
| Flaky tests      | `flaky-tests.md`           | Detection, diagnosis, fixing, quarantine |
| Debugging        | `debugging.md`             | Trace viewer, inspector, common issues   |
| CI/CD            | `ci-cd.md`                 | Pipelines, sharding, Docker              |
| Performance      | `performance.md`           | Parallel runs, optimization              |
| Global setup     | `global-setup.md`          | globalSetup/Teardown, DB migrations      |
| Projects         | `projects-dependencies.md` | Project config, dependencies, filtering  |
| Reporters        | `reporters.md`             | Custom reporters, Slack/Teams, analytics |
| Test coverage    | `test-coverage.md`         | V8 coverage, reports, thresholds, CI     |
| Network advanced | `network-advanced.md`      | GraphQL, HAR, request modification       |
| Third-party      | `third-party.md`           | OAuth, payments, email/SMS mocking       |
| Console errors   | `console-errors.md`        | Capturing and failing on JS errors       |

The skill's `SKILL.md` maps your current activity to these references so the right content is used in context.

## License

MIT
