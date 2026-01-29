# Test Organization

## Table of Contents

1. [Test Types Overview](#test-types-overview)
2. [E2E Tests](#e2e-tests)
3. [Component Tests](#component-tests)
4. [API Tests](#api-tests)
5. [Visual Regression Tests](#visual-regression-tests)
6. [Directory Structure](#directory-structure)
7. [Tagging & Filtering](#tagging--filtering)

## Test Types Overview

| Type | Purpose | Speed | Scope |
|------|---------|-------|-------|
| E2E | Full user flows | Slow | Entire app |
| Component | UI component behavior | Fast | Single component |
| API | Backend endpoints | Fast | API layer |
| Visual | UI appearance | Medium | Screenshots |

## E2E Tests

Full user journey tests through the browser.

### Structure

```typescript
// tests/e2e/checkout.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Checkout Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/products');
  });

  test('complete purchase as guest', async ({ page }) => {
    // Add to cart
    await page.getByRole('button', { name: 'Add to Cart' }).first().click();
    await expect(page.getByTestId('cart-count')).toHaveText('1');

    // Go to checkout
    await page.getByRole('link', { name: 'Cart' }).click();
    await page.getByRole('button', { name: 'Checkout' }).click();

    // Fill shipping
    await page.getByLabel('Email').fill('guest@example.com');
    await page.getByLabel('Address').fill('123 Test St');
    await page.getByRole('button', { name: 'Continue' }).click();

    // Payment
    await page.getByLabel('Card Number').fill('4242424242424242');
    await page.getByRole('button', { name: 'Pay Now' }).click();

    // Confirmation
    await expect(page.getByRole('heading')).toHaveText('Order Confirmed');
  });

  test('apply discount code', async ({ page }) => {
    await page.getByRole('button', { name: 'Add to Cart' }).first().click();
    await page.getByRole('link', { name: 'Cart' }).click();
    
    await page.getByLabel('Discount Code').fill('SAVE10');
    await page.getByRole('button', { name: 'Apply' }).click();
    
    await expect(page.getByText('10% discount applied')).toBeVisible();
  });
});
```

### Best Practices

- Test critical user journeys
- Keep tests independent
- Use realistic data
- Clean up test data in teardown

## Component Tests

Test individual components in isolation.

### Setup

```bash
npm init playwright@latest -- --ct
```

### Configuration

```typescript
// playwright-ct.config.ts
import { defineConfig, devices } from '@playwright/experimental-ct-react';

export default defineConfig({
  testDir: './tests/component',
  use: {
    ctPort: 3100,
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
});
```

### Component Test Example

```typescript
// tests/component/Button.spec.tsx
import { test, expect } from '@playwright/experimental-ct-react';
import { Button } from '../../src/components/Button';

test.describe('Button', () => {
  test('renders with text', async ({ mount }) => {
    const component = await mount(<Button>Click me</Button>);
    await expect(component).toContainText('Click me');
  });

  test('handles click', async ({ mount }) => {
    let clicked = false;
    const component = await mount(
      <Button onClick={() => { clicked = true; }}>Click me</Button>
    );
    
    await component.click();
    expect(clicked).toBe(true);
  });

  test('disabled state', async ({ mount }) => {
    const component = await mount(<Button disabled>Click me</Button>);
    
    await expect(component).toBeDisabled();
    await expect(component).toHaveCSS('opacity', '0.5');
  });

  test('loading state', async ({ mount }) => {
    const component = await mount(<Button loading>Submit</Button>);
    
    await expect(component.getByRole('progressbar')).toBeVisible();
    await expect(component).toBeDisabled();
  });
});
```

### Testing with Props Updates

```typescript
test('updates on prop change', async ({ mount }) => {
  const component = await mount(<Counter value={0} />);
  await expect(component).toContainText('0');
  
  await component.update(<Counter value={5} />);
  await expect(component).toContainText('5');
});
```

## API Tests

Test backend APIs without browser.

### Using Request Fixture

```typescript
// tests/api/users.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Users API', () => {
  const baseURL = process.env.API_URL || 'http://localhost:3000/api';

  test('GET /users returns list', async ({ request }) => {
    const response = await request.get(`${baseURL}/users`);
    
    expect(response.ok()).toBeTruthy();
    const users = await response.json();
    expect(users).toBeInstanceOf(Array);
    expect(users.length).toBeGreaterThan(0);
  });

  test('POST /users creates user', async ({ request }) => {
    const response = await request.post(`${baseURL}/users`, {
      data: {
        name: 'Test User',
        email: 'test@example.com',
      },
    });

    expect(response.status()).toBe(201);
    const user = await response.json();
    expect(user.name).toBe('Test User');
    expect(user.id).toBeDefined();
  });

  test('PUT /users/:id updates user', async ({ request }) => {
    // Create user first
    const createResponse = await request.post(`${baseURL}/users`, {
      data: { name: 'Original', email: 'original@example.com' },
    });
    const { id } = await createResponse.json();

    // Update
    const updateResponse = await request.put(`${baseURL}/users/${id}`, {
      data: { name: 'Updated' },
    });

    expect(updateResponse.ok()).toBeTruthy();
    const updated = await updateResponse.json();
    expect(updated.name).toBe('Updated');
  });

  test('DELETE /users/:id removes user', async ({ request }) => {
    // Create user first
    const createResponse = await request.post(`${baseURL}/users`, {
      data: { name: 'ToDelete', email: 'delete@example.com' },
    });
    const { id } = await createResponse.json();

    // Delete
    const deleteResponse = await request.delete(`${baseURL}/users/${id}`);
    expect(deleteResponse.status()).toBe(204);

    // Verify deleted
    const getResponse = await request.get(`${baseURL}/users/${id}`);
    expect(getResponse.status()).toBe(404);
  });
});
```

### API Client Fixture

```typescript
// fixtures/api.fixture.ts
import { test as base, APIRequestContext } from '@playwright/test';

class ApiClient {
  constructor(private request: APIRequestContext, private baseURL: string) {}

  async getUsers() {
    const response = await this.request.get(`${this.baseURL}/users`);
    return response.json();
  }

  async createUser(data: { name: string; email: string }) {
    const response = await this.request.post(`${this.baseURL}/users`, { data });
    return response.json();
  }
}

export const test = base.extend<{ api: ApiClient }>({
  api: async ({ request }, use) => {
    await use(new ApiClient(request, process.env.API_URL || 'http://localhost:3000/api'));
  },
});
```

## Visual Regression Tests

Compare screenshots to detect visual changes.

### Basic Visual Test

```typescript
// tests/visual/homepage.spec.ts
import { test, expect } from '@playwright/test';

test('homepage visual', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('homepage.png');
});

test('component visual', async ({ page }) => {
  await page.goto('/components');
  
  const button = page.getByRole('button', { name: 'Primary' });
  await expect(button).toHaveScreenshot('primary-button.png');
});
```

### Visual Test Options

```typescript
test('dashboard visual', async ({ page }) => {
  await page.goto('/dashboard');
  
  await expect(page).toHaveScreenshot('dashboard.png', {
    fullPage: true,                    // Capture entire scrollable page
    maxDiffPixels: 100,               // Allow up to 100 different pixels
    maxDiffPixelRatio: 0.01,          // Or 1% difference
    threshold: 0.2,                    // Pixel comparison threshold
    animations: 'disabled',            // Disable animations
    mask: [page.getByTestId('date')], // Mask dynamic content
  });
});
```

### Handling Dynamic Content

```typescript
test('page with dynamic content', async ({ page }) => {
  await page.goto('/profile');
  
  // Mask elements that change
  await expect(page).toHaveScreenshot('profile.png', {
    mask: [
      page.getByTestId('timestamp'),
      page.getByTestId('avatar'),
      page.getByRole('img'),
    ],
  });
});

// Or hide elements via CSS
test('page hiding dynamic elements', async ({ page }) => {
  await page.goto('/profile');
  
  await page.addStyleTag({
    content: `
      .dynamic-content { visibility: hidden !important; }
      [data-testid="ad-banner"] { display: none !important; }
    `,
  });
  
  await expect(page).toHaveScreenshot('profile-stable.png');
});
```

### Visual Test Configuration

```typescript
// playwright.config.ts
export default defineConfig({
  expect: {
    toHaveScreenshot: {
      maxDiffPixels: 50,
      animations: 'disabled',
    },
  },
  projects: [
    {
      name: 'visual-chrome',
      use: { 
        ...devices['Desktop Chrome'],
        viewport: { width: 1280, height: 720 },
      },
      testMatch: /.*visual.*\.spec\.ts/,
    },
  ],
});
```

### Update Snapshots

```bash
# Update all snapshots
npx playwright test --update-snapshots

# Update specific test
npx playwright test homepage.spec.ts --update-snapshots
```

## Directory Structure

```
tests/
├── e2e/                    # End-to-end tests
│   ├── auth.spec.ts
│   ├── checkout.spec.ts
│   └── dashboard.spec.ts
├── component/              # Component tests
│   ├── Button.spec.tsx
│   └── Modal.spec.tsx
├── api/                    # API tests
│   ├── users.spec.ts
│   └── products.spec.ts
├── visual/                 # Visual regression tests
│   └── homepage.spec.ts
├── fixtures/               # Custom fixtures
│   ├── auth.fixture.ts
│   └── api.fixture.ts
└── pages/                  # Page objects
    ├── login.page.ts
    └── dashboard.page.ts
```

## Tagging & Filtering

### Using Tags

```typescript
test('user login @smoke @auth', async ({ page }) => {
  // ...
});

test('checkout flow @e2e @critical', async ({ page }) => {
  // ...
});

test.describe('API tests @api', () => {
  test('create user', async ({ request }) => {
    // ...
  });
});
```

### Running Tagged Tests

```bash
# Run smoke tests
npx playwright test --grep @smoke

# Run all except slow tests
npx playwright test --grep-invert @slow

# Combine tags
npx playwright test --grep "@smoke|@critical"
```

### Project-Based Filtering

```typescript
// playwright.config.ts
export default defineConfig({
  projects: [
    {
      name: 'smoke',
      testMatch: /.*\.spec\.ts/,
      grep: /@smoke/,
    },
    {
      name: 'e2e',
      testDir: './tests/e2e',
    },
    {
      name: 'api',
      testDir: './tests/api',
    },
  ],
});
```

```bash
# Run specific project
npx playwright test --project=smoke
npx playwright test --project=api
```
