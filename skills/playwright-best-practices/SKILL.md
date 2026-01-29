---
name: playwright-best-practices
description: Guide for writing robust, maintainable Playwright tests in TypeScript. Use when writing new Playwright tests, reviewing test code, setting up test infrastructure, debugging flaky tests, implementing Page Object Model, configuring CI/CD pipelines for Playwright, or optimizing test performance. Covers E2E, component, API, and visual regression testing.
---

# Playwright Best Practices

This skill provides guidance for writing high-quality Playwright tests following industry best practices.

## Quick Start

### Project Setup

```bash
npm init playwright@latest
```

### Basic Test Structure

```typescript
import { test, expect } from '@playwright/test';

test.describe('Feature Name', () => {
  test('should perform expected behavior', async ({ page }) => {
    await page.goto('/path');
    await page.getByRole('button', { name: 'Submit' }).click();
    await expect(page.getByText('Success')).toBeVisible();
  });
});
```

## Core Principles

1. **Use user-facing locators** - Prefer `getByRole`, `getByLabel`, `getByText` over CSS/XPath
2. **Leverage auto-waiting** - Avoid explicit waits; use web-first assertions
3. **Isolate tests** - Each test should be independent and repeatable
4. **Use Page Object Model** - For maintainable, reusable test code
5. **Run in parallel** - Design tests for concurrent execution

## Reference Guides

Consult these guides based on your task:

| Topic | When to Use | Reference |
|-------|-------------|-----------|
| **Locators** | Selecting elements, fixing selector issues | [locators.md](references/locators.md) |
| **Page Object Model** | Structuring test code, creating page classes | [page-object-model.md](references/page-object-model.md) |
| **Fixtures & Hooks** | Test setup/teardown, sharing state | [fixtures-hooks.md](references/fixtures-hooks.md) |
| **Assertions & Waiting** | Verifying state, handling async operations | [assertions-waiting.md](references/assertions-waiting.md) |
| **Test Organization** | E2E, component, API, visual tests structure | [test-organization.md](references/test-organization.md) |
| **CI/CD** | GitHub Actions, Docker, reporting | [ci-cd.md](references/ci-cd.md) |
| **Debugging** | Trace viewer, debugging flaky tests | [debugging.md](references/debugging.md) |
| **Performance** | Parallelization, sharding, optimization | [performance.md](references/performance.md) |

## Common Patterns

### Authentication Setup

```typescript
// auth.setup.ts
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.TEST_EMAIL!);
  await page.getByLabel('Password').fill(process.env.TEST_PASSWORD!);
  await page.getByRole('button', { name: 'Sign in' }).click();
  await page.context().storageState({ path: '.auth/user.json' });
});
```

### API Mocking

```typescript
test('displays mocked data', async ({ page }) => {
  await page.route('**/api/users', route =>
    route.fulfill({ json: [{ id: 1, name: 'Test User' }] })
  );
  await page.goto('/users');
  await expect(page.getByText('Test User')).toBeVisible();
});
```

### Network Interception

```typescript
test('waits for API response', async ({ page }) => {
  const responsePromise = page.waitForResponse('**/api/data');
  await page.getByRole('button', { name: 'Load' }).click();
  const response = await responsePromise;
  expect(response.status()).toBe(200);
});
```

## Configuration Essentials

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html'], ['list']],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'setup', testMatch: /.*\.setup\.ts/ },
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
      dependencies: ['setup'],
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
|--------------|---------|----------|
| `page.locator('.btn-primary')` | Brittle, implementation-dependent | `page.getByRole('button', { name: 'Submit' })` |
| `await page.waitForTimeout(5000)` | Slow, flaky | Use auto-waiting or `waitForResponse` |
| Shared mutable state between tests | Race conditions, order dependencies | Use fixtures for isolation |
| Testing implementation details | Breaks on refactoring | Test user-visible behavior |
| Long test files | Hard to maintain | Split by feature, use POM |
