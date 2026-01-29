# Fixtures & Hooks

## Table of Contents

1. [Built-in Fixtures](#built-in-fixtures)
2. [Custom Fixtures](#custom-fixtures)
3. [Fixture Scopes](#fixture-scopes)
4. [Hooks](#hooks)
5. [Authentication Patterns](#authentication-patterns)
6. [Database Fixtures](#database-fixtures)

## Built-in Fixtures

### Core Fixtures

```typescript
test('example', async ({ 
  page,           // Isolated page instance
  context,        // Browser context (cookies, localStorage)
  browser,        // Browser instance
  browserName,    // 'chromium', 'firefox', or 'webkit'
  request,        // API request context
}) => {
  // Each test gets fresh instances
});
```

### Request Fixture

```typescript
test('API call', async ({ request }) => {
  const response = await request.get('/api/users');
  expect(response.ok()).toBeTruthy();
  
  const users = await response.json();
  expect(users).toHaveLength(5);
});
```

## Custom Fixtures

### Basic Custom Fixture

```typescript
// fixtures.ts
import { test as base } from '@playwright/test';

// Declare fixture types
type MyFixtures = {
  todoPage: TodoPage;
  apiClient: ApiClient;
};

export const test = base.extend<MyFixtures>({
  // Fixture with setup and teardown
  todoPage: async ({ page }, use) => {
    const todoPage = new TodoPage(page);
    await todoPage.goto();
    
    await use(todoPage); // Test runs here
    
    // Teardown (optional)
    await todoPage.clearTodos();
  },

  // Simple fixture
  apiClient: async ({ request }, use) => {
    await use(new ApiClient(request));
  },
});

export { expect } from '@playwright/test';
```

### Fixture with Options

```typescript
type Options = {
  defaultUser: { email: string; password: string };
};

type Fixtures = {
  authenticatedPage: Page;
};

export const test = base.extend<Options & Fixtures>({
  // Define option with default
  defaultUser: [{ email: 'test@example.com', password: 'pass123' }, { option: true }],

  // Use option in fixture
  authenticatedPage: async ({ page, defaultUser }, use) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill(defaultUser.email);
    await page.getByLabel('Password').fill(defaultUser.password);
    await page.getByRole('button', { name: 'Sign in' }).click();
    await use(page);
  },
});

// Override in config
export default defineConfig({
  use: {
    defaultUser: { email: 'admin@example.com', password: 'admin123' },
  },
});
```

### Automatic Fixtures

```typescript
export const test = base.extend<{}, { setupDb: void }>({
  // Auto-fixture runs for every test without explicit usage
  setupDb: [async ({}, use) => {
    await seedDatabase();
    await use();
    await cleanDatabase();
  }, { auto: true }],
});
```

## Fixture Scopes

### Test Scope (Default)

Created fresh for each test:

```typescript
test.extend({
  page: async ({ browser }, use) => {
    const page = await browser.newPage();
    await use(page);
    await page.close();
  },
});
```

### Worker Scope

Shared across tests in same worker:

```typescript
type WorkerFixtures = {
  sharedAccount: Account;
};

export const test = base.extend<{}, WorkerFixtures>({
  sharedAccount: [async ({ browser }, use) => {
    // Expensive setup - runs once per worker
    const account = await createTestAccount();
    await use(account);
    await deleteTestAccount(account);
  }, { scope: 'worker' }],
});
```

## Hooks

### beforeEach / afterEach

```typescript
test.beforeEach(async ({ page }) => {
  // Runs before each test in file
  await page.goto('/');
});

test.afterEach(async ({ page }, testInfo) => {
  // Runs after each test
  if (testInfo.status !== 'passed') {
    await page.screenshot({ path: `failed-${testInfo.title}.png` });
  }
});
```

### beforeAll / afterAll

```typescript
test.beforeAll(async ({ browser }) => {
  // Runs once before all tests in file
  // Note: Cannot use page fixture here
});

test.afterAll(async () => {
  // Runs once after all tests in file
});
```

### Describe-Level Hooks

```typescript
test.describe('User Management', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/users');
  });

  test('can list users', async ({ page }) => {
    // Starts at /users
  });

  test('can add user', async ({ page }) => {
    // Starts at /users
  });
});
```

## Authentication Patterns

### Global Setup with Storage State

```typescript
// auth.setup.ts
import { test as setup, expect } from '@playwright/test';

const authFile = '.auth/user.json';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.TEST_EMAIL!);
  await page.getByLabel('Password').fill(process.env.TEST_PASSWORD!);
  await page.getByRole('button', { name: 'Sign in' }).click();
  
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
  await page.context().storageState({ path: authFile });
});
```

```typescript
// playwright.config.ts
export default defineConfig({
  projects: [
    { name: 'setup', testMatch: /.*\.setup\.ts/ },
    {
      name: 'chromium',
      use: { 
        ...devices['Desktop Chrome'],
        storageState: '.auth/user.json',
      },
      dependencies: ['setup'],
    },
  ],
});
```

### Multiple Auth States

```typescript
// auth.setup.ts
setup('admin auth', async ({ page }) => {
  await login(page, 'admin@example.com', 'adminpass');
  await page.context().storageState({ path: '.auth/admin.json' });
});

setup('user auth', async ({ page }) => {
  await login(page, 'user@example.com', 'userpass');
  await page.context().storageState({ path: '.auth/user.json' });
});
```

```typescript
// playwright.config.ts
projects: [
  {
    name: 'admin tests',
    testMatch: /.*admin.*\.spec\.ts/,
    use: { storageState: '.auth/admin.json' },
    dependencies: ['setup'],
  },
  {
    name: 'user tests',
    testMatch: /.*user.*\.spec\.ts/,
    use: { storageState: '.auth/user.json' },
    dependencies: ['setup'],
  },
]
```

### Auth Fixture

```typescript
// fixtures/auth.fixture.ts
export const test = base.extend<{ adminPage: Page; userPage: Page }>({
  adminPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: '.auth/admin.json',
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  },

  userPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: '.auth/user.json',
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  },
});
```

## Database Fixtures

### Transaction Rollback Pattern

```typescript
import { test as base } from '@playwright/test';
import { db } from '../db';

export const test = base.extend<{ dbTransaction: Transaction }>({
  dbTransaction: async ({}, use) => {
    const transaction = await db.beginTransaction();
    
    await use(transaction);
    
    await transaction.rollback(); // Clean slate for next test
  },
});
```

### Seed Data Fixture

```typescript
type TestData = {
  testUser: User;
  testProducts: Product[];
};

export const test = base.extend<TestData>({
  testUser: async ({}, use) => {
    const user = await db.users.create({
      email: `test-${Date.now()}@example.com`,
      name: 'Test User',
    });
    
    await use(user);
    
    await db.users.delete(user.id);
  },

  testProducts: async ({ testUser }, use) => {
    const products = await db.products.createMany([
      { name: 'Product A', ownerId: testUser.id },
      { name: 'Product B', ownerId: testUser.id },
    ]);
    
    await use(products);
    
    await db.products.deleteMany(products.map(p => p.id));
  },
});
```

## Fixture Tips

| Tip | Explanation |
|-----|-------------|
| Fixtures are lazy | Only created when used |
| Compose fixtures | Use other fixtures as dependencies |
| Keep setup minimal | Do heavy lifting in worker-scoped fixtures |
| Clean up resources | Use teardown in fixtures, not afterEach |
| Avoid shared state | Each fixture instance should be independent |
