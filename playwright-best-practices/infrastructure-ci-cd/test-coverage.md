# Test Coverage

## Table of Contents

1. [Coverage Setup](#coverage-setup)
2. [Collecting Coverage](#collecting-coverage)
3. [Coverage Reports](#coverage-reports)
4. [Coverage Thresholds](#coverage-thresholds)
5. [Advanced Patterns](#advanced-patterns)
6. [CI Integration](#ci-integration)

## Coverage Setup

### Install Dependencies

Playwright's coverage API is Chromium-only. JavaScript coverage entries contain `functions` with V8 execution counts and `source`; CSS coverage entries contain `ranges` and `text`. They are different formats: do not combine them and read `.ranges` from JavaScript entries.

```bash
npm install -D v8-to-istanbul istanbul-lib-coverage nyc tsx
```

### Basic Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  projects: [
    { name: "coverage", use: { ...devices["Desktop Chrome"] } },
  ],
  use: { baseURL: "http://localhost:3000" },
});
```

### V8 Coverage Fixture

Collect from the same `page` that the test uses, not an unused page in a separate worker-scoped context. Each test writes a unique output file. Import this fixture instead of `@playwright/test` in tests that need automatic coverage.

```typescript
// fixtures/coverage.ts
import { test as base } from "@playwright/test";
import fs from "node:fs/promises";

export const test = base.extend<{ collectCoverage: void }>({
  collectCoverage: [
    async ({ page }, use, testInfo) => {
      await page.coverage.startJSCoverage({ resetOnNavigation: false });
      await page.coverage.startCSSCoverage({ resetOnNavigation: false });
      await use();
      const [js, css] = await Promise.all([
        page.coverage.stopJSCoverage(),
        page.coverage.stopCSSCoverage(),
      ]);
      // outputPath creates the test output directory.
      await fs.writeFile(testInfo.outputPath("js-coverage.json"), JSON.stringify(js));
      await fs.writeFile(testInfo.outputPath("css-coverage.json"), JSON.stringify(css));
    },
    { auto: true },
  ],
});

export { expect } from "@playwright/test";
```

## Collecting Coverage

### Per-Test Coverage

For manual collection, do not also enable the automatic fixture on the same page:

```typescript
import { test, expect } from "@playwright/test";

test("collect JavaScript coverage", async ({ page }) => {
  await page.coverage.startJSCoverage({ resetOnNavigation: false });
  await page.goto("/app");
  await page.getByRole("button", { name: "Submit" }).click();
  await expect(page.getByText("Success")).toBeVisible();
  const coverage = await page.coverage.stopJSCoverage();
  const appCoverage = coverage.filter((entry) => entry.url.includes("/src/"));
  console.log(`Covered ${appCoverage.length} source files`);
});
```

### Coverage for Specific Files

Select the intended module by its served URL. For JavaScript byte/line percentages, convert V8 function ranges to Istanbul rather than adding overlapping V8 ranges or treating them as CSS ranges.

### CSS Coverage

CSS ranges can be used directly for a stylesheet's used-byte estimate:

```typescript
test("collect CSS coverage", async ({ page }) => {
  await page.coverage.startCSSCoverage();
  await page.goto("/app");
  await page.getByRole("button").hover();
  const cssCoverage = await page.coverage.stopCSSCoverage();
  for (const entry of cssCoverage) {
    const usedBytes = entry.ranges.reduce((sum, r) => sum + r.end - r.start, 0);
    const percentage = entry.text.length === 0 ? 0 : usedBytes / entry.text.length * 100;
    console.log(`${entry.url}: ${percentage.toFixed(1)}% CSS used`);
  }
});
```

## Coverage Reports

### Converting to Istanbul Format

This script consumes only JavaScript files from the fixture. Supply the captured source to `v8-to-istanbul`: browser script URLs are usually HTTP URLs, not local `file://` URLs. Select your app's served script paths; the `/src/` filter below must match your build. To map bundled code back to original source files, build with inline source maps (for example, `build.sourcemap: "inline"` in Vite or `devtool: "inline-source-map"` in webpack): `v8-to-istanbul` cannot fetch a separate `.map` file from an HTTP URL. The script uses `fs.readdir` with `recursive: true`, which needs Node.js 20.1 or later.

```typescript
// scripts/convert-coverage.ts
import fs from "node:fs/promises";
import path from "node:path";
import v8ToIstanbul from "v8-to-istanbul";
import istanbulCoverage from "istanbul-lib-coverage";
import type { Page } from "@playwright/test";

type JSCoverage = Awaited<ReturnType<Page["coverage"]["stopJSCoverage"]>>;
const map = istanbulCoverage.createCoverageMap({});

// Optional arguments allow processing downloaded shard output directories.
const roots = process.argv.slice(2);
if (roots.length === 0) roots.push("test-results");
for (const root of roots) {
  const files = await fs.readdir(root, { recursive: true });
  for (const file of files.filter((f) => f.endsWith("js-coverage.json"))) {
    const entries: JSCoverage = JSON.parse(await fs.readFile(path.join(root, file), "utf8"));
    for (const entry of entries) {
      if (!entry.url.includes("/src/") || entry.source === undefined) continue;
      const converter = v8ToIstanbul(entry.url, 0, { source: entry.source });
      await converter.load();
      converter.applyCoverage(entry.functions);
      map.merge(converter.toIstanbul());
    }
  }
}
await fs.mkdir(".nyc_output", { recursive: true });
await fs.writeFile(".nyc_output/coverage.json", JSON.stringify(map.toJSON()));
```

Run the script as an ES module (for example, set `"type": "module"` in the test project's `package.json`):

```bash
npx tsx scripts/convert-coverage.ts
```

### Generating HTML Report

```bash
npx nyc report --temp-dir=.nyc_output --report-dir=coverage --reporter=html --reporter=text --reporter=lcov --reporter=json-summary
```

`coverage/coverage-summary.json` now contains actual Istanbul line/statement/function/branch summaries, not a byte percentage relabeled as line coverage.

### Custom Coverage Reporter

Run conversion/report generation after Playwright exits, so all test-scoped fixture teardowns have written their files. A test's `afterAll` hook is not a global aggregation point across workers. Custom reporters can consume the generated summary after conversion.

## Coverage Thresholds

### Enforcing Minimum Coverage

```bash
npx nyc check-coverage --temp-dir=.nyc_output --lines=80 --statements=80 --functions=80 --branches=70
```

### Per-Directory Thresholds

Use nyc's `--include` and `--per-file` options for a selected source directory after source-map conversion:

```bash
npx nyc check-coverage --temp-dir=.nyc_output --include='**/src/core/**' --per-file --lines=90
```

Ensure your app's expected source files appear in the report. Runtime coverage only includes loaded scripts; an empty or incomplete report is not proof that unvisited source files meet a threshold.

## Advanced Patterns

### Merging Coverage Across Shards

Download each shard's `test-results/` into a separate directory, then pass all roots to the conversion script. `CoverageMap.merge()` combines execution counts for the same source file; concatenating raw ranges and adding lengths would double-count overlapping coverage.

```bash
npx tsx scripts/convert-coverage.ts shard-1/test-results shard-2/test-results
npx nyc report --temp-dir=.nyc_output --report-dir=coverage --reporter=html --reporter=lcov
```

### Incremental Coverage

Compare changed source paths with the source-mapped Istanbul report, not browser bundle URLs. Retain the full report as the baseline; an unloaded changed file must not silently count as covered.

## CI Integration

### GitHub Actions

```yaml
# Steps in an existing job, after checkout, npm ci and browser installation.
- name: Run tests with coverage fixture
  run: npx playwright test --project=coverage
- name: Convert and report coverage
  run: |
    npx tsx scripts/convert-coverage.ts
    npx nyc report --temp-dir=.nyc_output --report-dir=coverage --reporter=lcov --reporter=json-summary
    npx nyc check-coverage --temp-dir=.nyc_output --lines=80
- uses: actions/upload-artifact@v4
  if: ${{ !cancelled() }}
  with:
    name: coverage
    path: coverage/
```

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Solution |
| --- | --- | --- |
| Coverage for coverage's sake | Gaming metrics | Focus on critical paths |
| 100% coverage target | Diminishing returns | Set realistic thresholds |
| Ignoring coverage drops | Technical debt | Enforce thresholds in CI |
| No source map support | Wrong source locations | Enable source maps in build |
| Collecting on an unused page | Misses test activity | Use the test's page fixture |
| Mixing JS and CSS coverage schemas | Invalid calculations/conversion | Process each format separately |

## Related References

- **CI/CD**: See [ci-cd.md](ci-cd.md) for pipeline configuration
- **Performance**: See [performance.md](performance.md) for optimizing coverage collection
