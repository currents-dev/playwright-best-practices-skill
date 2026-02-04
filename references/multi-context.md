# Multi-Tab, Window & Popup Testing

## Table of Contents

1. [Popup Handling](#popup-handling)
2. [New Tab Navigation](#new-tab-navigation)
3. [OAuth Flows](#oauth-flows)
4. [Multiple Windows](#multiple-windows)
5. [Tab Coordination](#tab-coordination)

## Popup Handling

### Basic Popup

```typescript
test("handle popup window", async ({ page }) => {
  await page.goto("/");

  // Start waiting for popup before triggering it
  const popupPromise = page.waitForEvent("popup");
  await page.getByRole("button", { name: "Open Support Chat" }).click();
  const popup = await popupPromise;

  // Wait for popup to load
  await popup.waitForLoadState();

  // Interact with popup
  await popup.getByLabel("Message").fill("Need help");
  await popup.getByRole("button", { name: "Send" }).click();

  await expect(popup.getByText("Message sent")).toBeVisible();

  // Close popup
  await popup.close();
});
```

### Popup with Authentication

```typescript
test("popup login flow", async ({ page }) => {
  await page.goto("/dashboard");

  const popupPromise = page.waitForEvent("popup");
  await page.getByRole("button", { name: "Connect Account" }).click();
  const popup = await popupPromise;

  await popup.waitForLoadState();

  // Complete login in popup
  await popup.getByLabel("Email").fill("user@example.com");
  await popup.getByLabel("Password").fill("password123");
  await popup.getByRole("button", { name: "Log In" }).click();

  // Popup should close automatically after auth
  await popup.waitForEvent("close");

  // Main page should reflect connected state
  await expect(page.getByText("Account connected")).toBeVisible();
});
```

### Handle Blocked Popups

```typescript
test("handle popup blocker", async ({ page }) => {
  await page.goto("/share");

  // Listen for console messages about blocked popup
  page.on("console", (msg) => {
    if (msg.text().includes("popup blocked")) {
      console.log("Popup was blocked");
    }
  });

  const popupPromise = page.waitForEvent("popup").catch(() => null);
  await page.getByRole("button", { name: "Share to Twitter" }).click();
  const popup = await popupPromise;

  if (!popup) {
    // Popup blocked - app should show fallback
    await expect(page.getByText("Copy share link instead")).toBeVisible();
  }
});
```

## New Tab Navigation

### Link Opens in New Tab

```typescript
test("external link opens in new tab", async ({ page, context }) => {
  await page.goto("/resources");

  // Wait for new page in context
  const pagePromise = context.waitForEvent("page");
  await page.getByRole("link", { name: "Documentation" }).click();
  const newPage = await pagePromise;

  await newPage.waitForLoadState();

  expect(newPage.url()).toContain("docs.example.com");
  await expect(newPage.getByRole("heading", { level: 1 })).toBeVisible();

  // Original page still there
  expect(page.url()).toContain("/resources");

  await newPage.close();
});
```

### Intercept New Tab

```typescript
test("prevent new tab for testing", async ({ page }) => {
  await page.goto("/links");

  // Remove target="_blank" to keep navigation in same tab
  await page.evaluate(() => {
    document.querySelectorAll('a[target="_blank"]').forEach((a) => {
      a.removeAttribute("target");
    });
  });

  // Now link opens in same tab
  await page.getByRole("link", { name: "External Site" }).click();

  // Can test the destination page
  await expect(page).toHaveURL(/external-site\.com/);
});
```

## OAuth Flows

### Google OAuth Popup

```typescript
test("Google OAuth login", async ({ page }) => {
  await page.goto("/login");

  const popupPromise = page.waitForEvent("popup");
  await page.getByRole("button", { name: "Sign in with Google" }).click();
  const popup = await popupPromise;

  await popup.waitForLoadState();

  // Handle Google's OAuth flow
  await popup.getByLabel("Email or phone").fill("test@gmail.com");
  await popup.getByRole("button", { name: "Next" }).click();

  await popup.getByLabel("Enter your password").fill("password");
  await popup.getByRole("button", { name: "Next" }).click();

  // Wait for redirect back and popup close
  await popup.waitForEvent("close");

  // Verify logged in on main page
  await expect(page.getByText("Welcome, Test User")).toBeVisible();
});
```

### Mock OAuth (Recommended)

```typescript
test("mock OAuth flow", async ({ page, context }) => {
  // Mock the OAuth callback instead of real flow
  await page.route("**/auth/callback**", async (route) => {
    // Simulate successful OAuth
    const url = new URL(route.request().url());
    url.searchParams.set("code", "mock-auth-code");
    await route.fulfill({
      status: 302,
      headers: { Location: "/dashboard" },
    });
  });

  // Mock token exchange
  await page.route("**/api/auth/token", (route) =>
    route.fulfill({
      json: {
        access_token: "mock-token",
        user: { name: "Test User", email: "test@example.com" },
      },
    }),
  );

  await page.goto("/login");
  await page.getByRole("button", { name: "Sign in with Google" }).click();

  // Should redirect to dashboard without actual OAuth
  await expect(page).toHaveURL("/dashboard");
  await expect(page.getByText("Welcome, Test User")).toBeVisible();
});
```

### OAuth Fixture

```typescript
// fixtures/oauth.fixture.ts
import { test as base } from "@playwright/test";

type OAuthFixtures = {
  mockOAuth: (
    provider: "google" | "github",
    user: { name: string; email: string },
  ) => Promise<void>;
};

export const test = base.extend<OAuthFixtures>({
  mockOAuth: async ({ page }, use) => {
    await use(async (provider, user) => {
      await page.route(`**/auth/${provider}/callback**`, (route) =>
        route.fulfill({
          status: 302,
          headers: {
            Location: `/dashboard?user=${encodeURIComponent(user.email)}`,
          },
        }),
      );

      await page.route("**/api/auth/session", (route) =>
        route.fulfill({
          json: { user, authenticated: true },
        }),
      );
    });
  },
});

// Usage
test("login with mocked Google", async ({ page, mockOAuth }) => {
  await mockOAuth("google", { name: "Test User", email: "test@example.com" });

  await page.goto("/login");
  await page.getByRole("button", { name: "Sign in with Google" }).click();

  await expect(page.getByText("Welcome, Test User")).toBeVisible();
});
```

## Multiple Windows

### Test Across Multiple Windows

```typescript
test("sync between windows", async ({ context }) => {
  // Open two pages
  const page1 = await context.newPage();
  const page2 = await context.newPage();

  await page1.goto("/dashboard");
  await page2.goto("/dashboard");

  // Make change in first window
  await page1.getByRole("button", { name: "Add Item" }).click();
  await page1.getByLabel("Name").fill("New Item");
  await page1.getByRole("button", { name: "Save" }).click();

  // Should sync to second window (if app supports real-time sync)
  await expect(page2.getByText("New Item")).toBeVisible({ timeout: 10000 });
});
```

### Different Users in Different Windows

```typescript
test("multi-user interaction", async ({ browser }) => {
  // Create separate contexts for different users
  const adminContext = await browser.newContext({
    storageState: ".auth/admin.json",
  });
  const userContext = await browser.newContext({
    storageState: ".auth/user.json",
  });

  const adminPage = await adminContext.newPage();
  const userPage = await userContext.newPage();

  // Admin approves user's request
  await userPage.goto("/requests");
  await userPage.getByRole("button", { name: "Submit Request" }).click();

  await adminPage.goto("/admin/requests");
  await expect(adminPage.getByText("Pending Request")).toBeVisible();
  await adminPage.getByRole("button", { name: "Approve" }).click();

  // User sees approval
  await expect(userPage.getByText("Request Approved")).toBeVisible();

  await adminContext.close();
  await userContext.close();
});
```

## Tab Coordination

### Switch Between Tabs

```typescript
test("manage multiple tabs", async ({ context }) => {
  const page1 = await context.newPage();
  await page1.goto("/editor");

  const page2 = await context.newPage();
  await page2.goto("/preview");

  // Edit in first tab
  await page1.bringToFront();
  await page1.getByLabel("Content").fill("Hello World");

  // Check preview in second tab
  await page2.bringToFront();
  await page2.reload(); // If preview needs refresh
  await expect(page2.getByText("Hello World")).toBeVisible();
});
```

### Close All Tabs Except One

```typescript
test("cleanup tabs after test", async ({ context }) => {
  const mainPage = await context.newPage();
  await mainPage.goto("/");

  // Open several popups during test
  for (let i = 0; i < 3; i++) {
    const popup = await context.newPage();
    await popup.goto(`/popup/${i}`);
  }

  // Close all except main page
  for (const page of context.pages()) {
    if (page !== mainPage) {
      await page.close();
    }
  }

  expect(context.pages()).toHaveLength(1);
});
```

## Anti-Patterns to Avoid

| Anti-Pattern            | Problem                        | Solution                                   |
| ----------------------- | ------------------------------ | ------------------------------------------ |
| Not waiting for popup   | Race condition                 | Use `waitForEvent("popup")` before trigger |
| Testing real OAuth      | Slow, flaky, needs credentials | Mock OAuth endpoints                       |
| Assuming popup opens    | May be blocked                 | Handle both open and blocked cases         |
| Not closing extra pages | Resource leak                  | Close pages in cleanup                     |

## Related References

- **Authentication**: See [fixtures-hooks.md](fixtures-hooks.md) for auth patterns
- **Network**: See [network-advanced.md](network-advanced.md) for mocking OAuth
