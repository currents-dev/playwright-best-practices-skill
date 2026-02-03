# Playwright Best Practices Skill

A skill that gives the AI specialized guidance for writing, debugging, and maintaining **Playwright** tests in **TypeScript**. Use it in any repo where you work with Playwright so the assistant follows best practices for E2E, component, API, and visual regression testing.

## Installation

```bash
npx skills add https://github.com/currents-dev/playwright-best-practices-skill
```

After adding, the AI will use it when your questions or tasks involve writing Playwright tests, debugging failures, fixing flaky tests, setting up CI/CD, implementing POM, or similar Playwright work.

## What This Skill Covers

- **Writing tests** — Structure, locators, assertions, waiting strategies, Page Object Model, fixtures
- **Debugging & troubleshooting** — Trace viewer, flaky tests, selectors, timeouts, race conditions
- **Refactoring & maintenance** — POM migration, test organization, shared setup/teardown
- **Infrastructure** — Project config, CI/CD, parallel execution, performance, reporting

The skill is activity-based: the AI is directed to the right reference (e.g. `locators.md`, `debugging.md`) depending on what you’re doing, so you get focused advice without loading everything at once.

## When the Skill Is Used

The skill triggers when the AI infers you need help with things like:

- Writing new E2E, component, API, or visual regression tests
- Reviewing or refactoring Playwright test code
- Fixing flaky tests or debugging failures
- Setting up or changing test infrastructure and config
- Implementing Page Object Model or improving test organization
- Configuring CI/CD, parallel runs, or performance
- Selectors, timeouts, assertions, fixtures, or authentication

You don’t have to mention “skill” or “Playwright best practices”; describe your task (e.g. “fix this flaky login test” or “add a test for the checkout flow”) and the AI will use the skill when it’s relevant.

## What’s Inside

| Topic                | Reference               | Use for                                            |
| -------------------- | ----------------------- | -------------------------------------------------- |
| Test organization    | `test-organization.md`  | Structure, config, E2E/component/API/visual tests  |
| Locators             | `locators.md`           | Selectors, robustness, avoiding brittle locators   |
| Assertions & waiting | `assertions-waiting.md` | Expect APIs, auto-waiting, avoiding explicit waits |
| Page Object Model    | `page-object-model.md`  | POM structure and patterns                         |
| Fixtures & hooks     | `fixtures-hooks.md`     | Setup, teardown, auth, custom fixtures             |
| Debugging            | `debugging.md`          | Trace viewer, flakiness, common issues             |
| CI/CD                | `ci-cd.md`              | Pipelines, reporting, integration                  |
| Performance          | `performance.md`        | Parallel runs, speed, resource usage               |

The skill’s `SKILL.md` maps your current activity (e.g. “debugging”, “writing E2E tests”) to these references so the right content is used in context.

## License

MIT
