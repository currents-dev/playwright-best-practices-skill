---
name: playwright-best-practices
description: Provides comprehensive guidance for writing, debugging, and maintaining Playwright tests in TypeScript. Use when writing Playwright tests, fixing flaky tests, debugging failures, implementing Page Object Model, configuring CI/CD, optimizing performance, mocking APIs, handling authentication or OAuth, testing accessibility (axe-core), file uploads/downloads, date/time mocking, WebSockets, geolocation, permissions, multi-tab/popup flows, mobile/responsive layouts, touch gestures, GraphQL, error handling, offline mode, multi-user collaboration, custom reporters, third-party services (payments, email verification), console error monitoring, global setup/teardown, test annotations (skip, fixme, slow), project dependencies, security testing (XSS, CSRF, auth), or performance budgets (Web Vitals, Lighthouse). Covers E2E, component, API, visual, accessibility, and security testing.
---

# Playwright Best Practices

This skill provides comprehensive guidance for all aspects of Playwright test development, from writing new tests to debugging and maintaining existing test suites.

## Activity-Based Reference Guide

Consult these references based on what you're doing:

### Writing New Tests

**When to use**: Creating new test files, writing test cases, implementing test scenarios

| Activity                            | Reference Files                                                                                                                                           |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Writing E2E tests**               | [test-organization.md](references/test-organization.md), [locators.md](references/locators.md), [assertions-waiting.md](references/assertions-waiting.md) |
| **Writing component tests**         | [test-organization.md](references/test-organization.md), [locators.md](references/locators.md)                                                            |
| **Writing API tests**               | [test-organization.md](references/test-organization.md), [assertions-waiting.md](references/assertions-waiting.md)                                        |
| **Writing visual regression tests** | [test-organization.md](references/test-organization.md)                                                                                                   |
| **Structuring test code with POM**  | [page-object-model.md](references/page-object-model.md), [test-organization.md](references/test-organization.md)                                          |
| **Setting up test data/fixtures**   | [fixtures-hooks.md](references/fixtures-hooks.md), [test-data.md](references/test-data.md)                                                                |
| **Handling authentication**         | [fixtures-hooks.md](references/fixtures-hooks.md), [third-party.md](references/third-party.md)                                                            |
| **Testing date/time features**      | [clock-mocking.md](references/clock-mocking.md)                                                                                                           |
| **Testing file upload/download**    | [file-operations.md](references/file-operations.md)                                                                                                       |
| **Testing accessibility**           | [accessibility.md](references/accessibility.md)                                                                                                           |
| **Testing security (XSS, CSRF)**    | [security-testing.md](references/security-testing.md)                                                                                                     |
| **Using test annotations**          | [annotations.md](references/annotations.md)                                                                                                               |

### Mobile & Responsive Testing

**When to use**: Testing mobile devices, touch interactions, responsive layouts

| Activity                        | Reference Files                                                                          |
| ------------------------------- | ---------------------------------------------------------------------------------------- |
| **Device emulation**            | [mobile-testing.md](references/mobile-testing.md)                                        |
| **Touch gestures (swipe, tap)** | [mobile-testing.md](references/mobile-testing.md)                                        |
| **Viewport/breakpoint testing** | [mobile-testing.md](references/mobile-testing.md)                                        |
| **Mobile-specific UI**          | [mobile-testing.md](references/mobile-testing.md), [locators.md](references/locators.md) |

### Real-Time & Browser APIs

**When to use**: Testing WebSockets, geolocation, permissions, multi-tab flows

| Activity                        | Reference Files                                                                              |
| ------------------------------- | -------------------------------------------------------------------------------------------- |
| **WebSocket/real-time testing** | [websockets.md](references/websockets.md)                                                    |
| **Geolocation mocking**         | [browser-apis.md](references/browser-apis.md)                                                |
| **Permission handling**         | [browser-apis.md](references/browser-apis.md)                                                |
| **Clipboard testing**           | [browser-apis.md](references/browser-apis.md)                                                |
| **Camera/microphone mocking**   | [browser-apis.md](references/browser-apis.md)                                                |
| **Multi-tab/popup flows**       | [multi-context.md](references/multi-context.md)                                              |
| **OAuth popup handling**        | [multi-context.md](references/multi-context.md), [third-party.md](references/third-party.md) |

### Debugging & Troubleshooting

**When to use**: Test failures, element not found, timeouts, unexpected behavior

| Activity                                          | Reference Files                                                                                                                                 |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Debugging test failures**                       | [debugging.md](references/debugging.md), [assertions-waiting.md](references/assertions-waiting.md)                                              |
| **Fixing flaky tests**                            | [flaky-tests.md](references/flaky-tests.md), [debugging.md](references/debugging.md), [assertions-waiting.md](references/assertions-waiting.md) |
| **Debugging flaky parallel runs**                 | [flaky-tests.md](references/flaky-tests.md), [performance.md](references/performance.md), [fixtures-hooks.md](references/fixtures-hooks.md)     |
| **Ensuring test isolation / avoiding state leak** | [flaky-tests.md](references/flaky-tests.md), [fixtures-hooks.md](references/fixtures-hooks.md), [performance.md](references/performance.md)     |
| **Fixing selector issues**                        | [locators.md](references/locators.md), [debugging.md](references/debugging.md)                                                                  |
| **Investigating timeout issues**                  | [assertions-waiting.md](references/assertions-waiting.md), [debugging.md](references/debugging.md)                                              |
| **Using trace viewer**                            | [debugging.md](references/debugging.md)                                                                                                         |
| **Debugging race conditions**                     | [flaky-tests.md](references/flaky-tests.md), [debugging.md](references/debugging.md), [assertions-waiting.md](references/assertions-waiting.md) |
| **Debugging console/JS errors**                   | [console-errors.md](references/console-errors.md), [debugging.md](references/debugging.md)                                                      |

### Error & Edge Case Testing

**When to use**: Testing error states, offline mode, network failures, validation

| Activity                       | Reference Files                                                                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------------------ |
| **Error boundary testing**     | [error-testing.md](references/error-testing.md)                                                        |
| **Network failure simulation** | [error-testing.md](references/error-testing.md), [network-advanced.md](references/network-advanced.md) |
| **Offline mode testing**       | [error-testing.md](references/error-testing.md)                                                        |
| **Loading state testing**      | [error-testing.md](references/error-testing.md)                                                        |
| **Form validation testing**    | [error-testing.md](references/error-testing.md)                                                        |

### Multi-User & Collaboration Testing

**When to use**: Testing features involving multiple users, roles, or real-time collaboration

| Activity                       | Reference Files                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------ |
| **Multiple users in one test** | [multi-user.md](references/multi-user.md)                                            |
| **Real-time collaboration**    | [multi-user.md](references/multi-user.md), [websockets.md](references/websockets.md) |
| **Role-based access testing**  | [multi-user.md](references/multi-user.md)                                            |
| **Concurrent action testing**  | [multi-user.md](references/multi-user.md)                                            |

### Refactoring & Maintenance

**When to use**: Improving existing tests, code review, reducing duplication

| Activity                             | Reference Files                                                                                                  |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| **Refactoring to Page Object Model** | [page-object-model.md](references/page-object-model.md), [test-organization.md](references/test-organization.md) |
| **Improving test organization**      | [test-organization.md](references/test-organization.md), [page-object-model.md](references/page-object-model.md) |
| **Extracting common setup/teardown** | [fixtures-hooks.md](references/fixtures-hooks.md)                                                                |
| **Replacing brittle selectors**      | [locators.md](references/locators.md)                                                                            |
| **Removing explicit waits**          | [assertions-waiting.md](references/assertions-waiting.md)                                                        |
| **Creating test data factories**     | [test-data.md](references/test-data.md)                                                                          |

### Infrastructure & Configuration

**When to use**: Setting up projects, configuring CI/CD, optimizing performance

| Activity                                | Reference Files                                                                                                          |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Configuring Playwright project**      | [test-organization.md](references/test-organization.md), [projects-dependencies.md](references/projects-dependencies.md) |
| **Setting up CI/CD pipelines**          | [ci-cd.md](references/ci-cd.md), [performance.md](references/performance.md)                                             |
| **Global setup & teardown**             | [global-setup.md](references/global-setup.md)                                                                            |
| **Project dependencies**                | [projects-dependencies.md](references/projects-dependencies.md)                                                          |
| **Optimizing test performance**         | [performance.md](references/performance.md), [test-organization.md](references/test-organization.md)                     |
| **Configuring parallel execution**      | [performance.md](references/performance.md)                                                                              |
| **Isolating test data between workers** | [fixtures-hooks.md](references/fixtures-hooks.md), [performance.md](references/performance.md)                           |
| **Setting up test reporting**           | [ci-cd.md](references/ci-cd.md), [reporters.md](references/reporters.md)                                                 |
| **Custom reporters & analytics**        | [reporters.md](references/reporters.md)                                                                                  |
| **Slack/Teams notifications**           | [reporters.md](references/reporters.md)                                                                                  |

### Advanced Patterns

**When to use**: Complex scenarios, API mocking, network interception

| Activity                             | Reference Files                                                                                                  |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| **Mocking API responses**            | [test-organization.md](references/test-organization.md), [network-advanced.md](references/network-advanced.md)   |
| **Network interception**             | [network-advanced.md](references/network-advanced.md), [assertions-waiting.md](references/assertions-waiting.md) |
| **GraphQL mocking**                  | [network-advanced.md](references/network-advanced.md)                                                            |
| **HAR recording/playback**           | [network-advanced.md](references/network-advanced.md)                                                            |
| **Custom fixtures**                  | [fixtures-hooks.md](references/fixtures-hooks.md)                                                                |
| **Advanced waiting strategies**      | [assertions-waiting.md](references/assertions-waiting.md)                                                        |
| **OAuth/SSO mocking**                | [third-party.md](references/third-party.md), [multi-context.md](references/multi-context.md)                     |
| **Payment gateway mocking**          | [third-party.md](references/third-party.md)                                                                      |
| **Email/SMS verification mocking**   | [third-party.md](references/third-party.md)                                                                      |
| **Failing on console errors**        | [console-errors.md](references/console-errors.md)                                                                |
| **Security testing (XSS, CSRF)**     | [security-testing.md](references/security-testing.md)                                                            |
| **Performance budgets & Web Vitals** | [performance-testing.md](references/performance-testing.md)                                                      |
| **Lighthouse integration**           | [performance-testing.md](references/performance-testing.md)                                                      |
| **Test annotations (skip, fixme)**   | [annotations.md](references/annotations.md)                                                                      |
| **Test steps for reporting**         | [annotations.md](references/annotations.md)                                                                      |

## Quick Decision Tree

```
What are you doing?
│
├─ Writing a new test?
│  ├─ E2E test → test-organization.md, locators.md, assertions-waiting.md
│  ├─ Component test → test-organization.md, locators.md
│  ├─ API test → test-organization.md, assertions-waiting.md
│  ├─ Visual test → test-organization.md
│  ├─ Accessibility test → accessibility.md
│  ├─ Mobile/responsive test → mobile-testing.md
│  └─ Multi-user test → multi-user.md
│
├─ Testing specific features?
│  ├─ File upload/download → file-operations.md
│  ├─ Date/time dependent → clock-mocking.md
│  ├─ WebSocket/real-time → websockets.md
│  ├─ Geolocation/permissions → browser-apis.md
│  ├─ OAuth/popups → multi-context.md, third-party.md
│  ├─ Payments/email/SMS → third-party.md
│  ├─ Security (XSS, CSRF) → security-testing.md
│  └─ Performance/Web Vitals → performance-testing.md
│
├─ Test is failing/flaky?
│  ├─ Flaky test investigation → flaky-tests.md
│  ├─ Element not found → locators.md, debugging.md
│  ├─ Timeout issues → assertions-waiting.md, debugging.md
│  ├─ Race conditions → flaky-tests.md, debugging.md
│  ├─ Flaky only with multiple workers → flaky-tests.md, performance.md
│  ├─ State leak / isolation → flaky-tests.md, fixtures-hooks.md
│  ├─ Console/JS errors → console-errors.md, debugging.md
│  └─ General debugging → debugging.md
│
├─ Testing error scenarios?
│  ├─ Network failures → error-testing.md, network-advanced.md
│  ├─ Offline mode → error-testing.md
│  ├─ Error boundaries → error-testing.md
│  └─ Form validation → error-testing.md
│
├─ Refactoring existing code?
│  ├─ Implementing POM → page-object-model.md
│  ├─ Improving selectors → locators.md
│  ├─ Extracting fixtures → fixtures-hooks.md
│  └─ Creating data factories → test-data.md
│
├─ Setting up infrastructure?
│  ├─ CI/CD → ci-cd.md
│  ├─ Global setup/teardown → global-setup.md
│  ├─ Project dependencies → projects-dependencies.md
│  ├─ Test performance → performance.md
│  ├─ Custom reporters → reporters.md
│  ├─ Slack/Teams alerts → reporters.md
│  └─ Project config → test-organization.md, projects-dependencies.md
│
└─ Organizing tests?
   ├─ Skip/fixme/slow tests → annotations.md
   ├─ Test steps → annotations.md
   └─ Conditional execution → annotations.md
```

## Test Validation Loop

After writing or modifying tests:

1. **Run tests**: `npx playwright test --reporter=list`
2. **If tests fail**:
   - Review error output and trace (`npx playwright show-trace`)
   - Fix locators, waits, or assertions
   - Re-run tests
3. **Only proceed when all tests pass**
4. **Run multiple times** for critical tests: `npx playwright test --repeat-each=5`
