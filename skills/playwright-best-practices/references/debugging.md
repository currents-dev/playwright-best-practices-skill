# Debugging & Troubleshooting

## Table of Contents

1. [Debug Tools](#debug-tools)
2. [Trace Viewer](#trace-viewer)
3. [Debugging Flaky Tests](#debugging-flaky-tests)
4. [Common Issues](#common-issues)
5. [Logging](#logging)
6. [VS Code Integration](#vs-code-integration)

## Debug Tools

### Playwright Inspector

```bash
# Run with inspector
PWDEBUG=1 npx playwright test

# Or specific test
PWDEBUG=1 npx playwright test login.spec.ts
```

Features:
- Step through test actions
- Pick locators visually
- Inspect DOM state
- Edit and re-run

### Headed Mode

```bash
# Run with visible browser
npx playwright test --headed

# Slow down execution
npx playwright test --headed --slow-motion=500
```

### UI Mode

```bash
# Interactive test runner
npx playwright test --ui
```

Features:
- Watch mode
- Test timeline
- DOM snapshots
- Network logs
- Console logs

### Debug in Code

```typescript
test('debug example', async ({ page }) => {
  await page.goto('/');
  
  // Pause and open inspector
  await page.pause();
  
  // Continue test...
  await page.click('button');
});
```

## Trace Viewer

### Enable Traces

```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    trace: 'on-first-retry',        // Record on retry
    // trace: 'on',                 // Always record
    // trace: 'retain-on-failure',  // Keep only failures
  },
});
```

### View Traces

```bash
# Open trace file
npx playwright show-trace trace.zip

# From test-results
npx playwright show-trace test-results/test-name/trace.zip
```

### Trace Contents

- Screenshots at each action
- DOM snapshots
- Network requests/responses
- Console logs
- Action timeline
- Source code

### Programmatic Traces

```typescript
test('manual trace', async ({ page, context }) => {
  await context.tracing.start({ screenshots: true, snapshots: true });
  
  await page.goto('/');
  await page.click('button');
  
  await context.tracing.stop({ path: 'trace.zip' });
});
```

## Debugging Flaky Tests

### Identify Flaky Tests

```bash
# Run test multiple times
npx playwright test --repeat-each=10

# Run until failure
npx playwright test --repeat-each=100 --max-failures=1
```

### Common Causes & Fixes

#### Race Conditions

```typescript
// Bad: Element may not exist yet
await page.click('.dynamic-button');

// Good: Wait for element
await page.getByRole('button', { name: 'Submit' }).click();
```

#### Animation Timing

```typescript
// Bad: Animation may still be running
await page.click('.animated-element');

// Good: Wait for animation
await page.getByRole('dialog').waitFor({ state: 'visible' });
await expect(page.getByRole('dialog')).toBeVisible();
```

#### Network Timing

```typescript
// Bad: Data may not be loaded
await page.goto('/dashboard');
expect(await page.textContent('.user-count')).toBe('100');

// Good: Wait for API response
await page.goto('/dashboard');
await page.waitForResponse('**/api/users');
await expect(page.getByTestId('user-count')).toHaveText('100');
```

#### Test Isolation

```typescript
// Bad: Tests share state
test('add item', async ({ page }) => {
  await page.goto('/items');
  await page.click('text=Add Item');
  expect(await page.locator('.item').count()).toBe(1);
});

test('check items', async ({ page }) => {
  await page.goto('/items');
  // Fails if previous test ran first!
  expect(await page.locator('.item').count()).toBe(0);
});

// Good: Reset state in each test
test.beforeEach(async ({ page }) => {
  await page.request.delete('/api/items/reset');
});
```

### Retry Configuration

```typescript
// playwright.config.ts
export default defineConfig({
  retries: process.env.CI ? 2 : 0,
  
  // Per-project retries
  projects: [
    {
      name: 'stable',
      retries: 0,
    },
    {
      name: 'flaky',
      retries: 3,
    },
  ],
});
```

```typescript
// Per-test retry
test('flaky test', async ({ page }) => {
  test.info().annotations.push({ type: 'fixme', description: 'Investigate flakiness' });
  // ...
});
```

## Common Issues

### Element Not Found

```typescript
// Debug: Check if element exists
console.log(await page.getByRole('button').count());

// Debug: Log all buttons
const buttons = await page.getByRole('button').all();
for (const button of buttons) {
  console.log(await button.textContent());
}

// Debug: Screenshot before action
await page.screenshot({ path: 'debug.png' });
await page.getByRole('button').click();
```

### Timeout Issues

```typescript
// Increase timeout for slow operations
await expect(page.getByText('Loaded')).toBeVisible({ timeout: 30000 });

// Global timeout increase
test.setTimeout(60000);

// Check what's blocking
test('debug timeout', async ({ page }) => {
  await page.goto('/slow-page');
  
  // Log network activity
  page.on('request', request => console.log('>>', request.url()));
  page.on('response', response => console.log('<<', response.url(), response.status()));
});
```

### Selector Issues

```typescript
// Debug: Highlight element
await page.getByRole('button').highlight();

// Debug: Evaluate selector in browser console
// Run in Inspector console:
// playwright.locator('button').first().highlight()

// Debug: Get element info
const element = page.getByRole('button');
console.log('Count:', await element.count());
console.log('Visible:', await element.isVisible());
console.log('Enabled:', await element.isEnabled());
```

### Frame Issues

```typescript
// Debug: List all frames
for (const frame of page.frames()) {
  console.log('Frame:', frame.url());
}

// Debug: Check if element is in iframe
const frame = page.frameLocator('iframe').first();
console.log(await frame.getByRole('button').count());
```

## Logging

### Console Logging

```typescript
test('with logging', async ({ page }) => {
  // Capture browser console
  page.on('console', msg => console.log('Browser:', msg.text()));
  page.on('pageerror', error => console.log('Page error:', error.message));
  
  await page.goto('/');
});
```

### Custom Test Attachments

```typescript
test('with attachments', async ({ page }, testInfo) => {
  await page.goto('/');
  
  // Attach screenshot
  const screenshot = await page.screenshot();
  await testInfo.attach('screenshot', { body: screenshot, contentType: 'image/png' });
  
  // Attach text
  await testInfo.attach('logs', { body: 'Custom log data', contentType: 'text/plain' });
  
  // Attach file
  await testInfo.attach('data', { path: './test-data.json', contentType: 'application/json' });
});
```

### Test Info

```typescript
test('info example', async ({ page }, testInfo) => {
  console.log('Test title:', testInfo.title);
  console.log('Test file:', testInfo.file);
  console.log('Retry:', testInfo.retry);
  console.log('Project:', testInfo.project.name);
  
  // Custom output directory
  const outputPath = testInfo.outputPath('custom-file.txt');
});
```

## VS Code Integration

### Playwright Extension

Install: `ms-playwright.playwright`

Features:
- Run/debug tests from editor
- Pick locators
- Record tests
- View traces

### Debug Configuration

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Playwright Tests",
      "program": "${workspaceFolder}/node_modules/.bin/playwright",
      "args": ["test", "--debug"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal"
    },
    {
      "type": "node",
      "request": "launch", 
      "name": "Debug Current Test File",
      "program": "${workspaceFolder}/node_modules/.bin/playwright",
      "args": ["test", "${relativeFile}", "--debug"],
      "cwd": "${workspaceFolder}",
      "console": "integratedTerminal"
    }
  ]
}
```

### Settings

```json
// .vscode/settings.json
{
  "playwright.showTrace": true,
  "playwright.reuseBrowser": true
}
```

## Troubleshooting Checklist

| Issue | Check |
|-------|-------|
| Element not found | Locator specificity, visibility, iframe |
| Timeout | Network, animations, loading states |
| Flaky test | Race conditions, isolation, state |
| CI failures | Environment, secrets, dependencies |
| Slow tests | Parallelization, network mocking |
