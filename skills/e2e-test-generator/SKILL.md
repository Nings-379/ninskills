---
name: e2e-test-generator
description: Generate comprehensive end-to-end tests using Playwright for user workflows, page interactions, and multi-page scenarios
---

# E2E Test Generator Skill

## 目标

使用 Playwright 生成端到端测试，覆盖完整的用户流程和页面交互场景，确保系统从用户视角正常工作。

## 适用范围

### 必须编写 E2E 测试的场景

1. **用户认证流程**
   - 登录、登出
   - 权限验证（访问受保护页面）
   - Token 过期处理

2. **数据列表页面**（`/user/list`, `/device/list`, `/permission/list`）
   - 页面加载和数据展示
   - 搜索和过滤功能
   - 分页导航
   - 排序功能

3. **详情页面**（`/user/detail/:id`, `/device/detail/:id`）
   - 从列表导航到详情
   - 数据加载展示
   - Tab 切换（如用户详情的基本信息、设备列表、访问记录）

4. **表单提交流程**
   - 创建新记录（创建用户、设备）
   - 编辑现有记录
   - 删除记录（含确认弹窗）
   - 表单验证（必填项、格式验证）

5. **多页面交互场景**
   - 仪表盘 → 用户列表 → 用户详情 → 编辑用户
   - 导航栏切换
   - 面包屑导航

6. **全局功能**
   - 语言切换（中文/英文）
   - 主题切换（如有）
   - 响应式布局（桌面/移动端）

### 不需要 E2E 测试的代码

- ❌ 工具函数（用单元测试）
- ❌ 纯展示组件（用单元测试）
- ❌ API 层逻辑（用集成测试）

## Tech Stack

- **Test Framework**: Playwright
- **Browsers**: Chromium, Firefox, WebKit
- **Assertions**: Playwright built-in expect
- **Test Isolation**: Each test runs independently

## File Structure

```
packages/fe/
  tests/
    e2e/
      fixtures/
        auth.ts              # Auth fixtures
      pages/
        login.page.ts        # Page Object Model (POM)
        user-list.page.ts
      user-list.spec.ts      # E2E test files
      user-crud.spec.ts
  playwright.config.ts       # Playwright config (root level)
```

## Test Templates

### 1. Basic Test Structure

```typescript
// tests/e2e/user-list.spec.ts
import { test, expect } from "@playwright/test";

test.describe("User List Page", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/user/list");
    await page.waitForLoadState("networkidle");
  });

  test("should display user list table", async ({ page }) => {
    const table = page.locator("table");
    await expect(table).toBeVisible();

    await expect(page.locator('th:has-text("Name")')).toBeVisible();
    await expect(page.locator('th:has-text("Email")')).toBeVisible();

    const rows = page.locator("tbody tr");
    await expect(rows).not.toHaveCount(0);
  });

  test("should search users by name", async ({ page }) => {
    await page.fill('input[placeholder*="Name"]', "John");
    await page.click('button:has-text("Search")');

    await page.waitForResponse(
      (resp) => resp.url().includes("/api/users") && resp.status() === 200,
    );

    const firstRow = page.locator("tbody tr").first();
    await expect(firstRow).toContainText("John");
  });

  test("should navigate to user detail", async ({ page }) => {
    await page.click('tbody tr:first-child button:has-text("View")');
    await expect(page).toHaveURL(/\/user\/detail\/\d+/);
  });
});
```

### 2. Using Page Object Model (POM)

```typescript
// e2e/pages/user-list.page.ts
import { Page, Locator } from "@playwright/test";

export class UserListPage {
  readonly page: Page;
  readonly searchInput: Locator;
  readonly searchButton: Locator;
  readonly table: Locator;
  readonly addUserButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.searchInput = page.locator('input[placeholder*="Name"]');
    this.searchButton = page.locator('button:has-text("Search")');
    this.table = page.locator("table");
    this.addUserButton = page.locator('button:has-text("Add User")');
  }

  async goto() {
    await this.page.goto("/user/list");
    await this.page.waitForLoadState("networkidle");
  }

  async searchByName(name: string) {
    await this.searchInput.fill(name);
    await this.searchButton.click();
    await this.page.waitForResponse(
      (resp) => resp.url().includes("/api/users") && resp.status() === 200,
    );
  }

  async clickViewButton(rowIndex: number = 0) {
    const row = this.page.locator(`tbody tr:nth-child(${rowIndex + 1})`);
    await row.locator('button:has-text("View")').click();
  }
}
```

```typescript
// tests/e2e/user-list-with-pom.spec.ts
import { test, expect } from "@playwright/test";
import { UserListPage } from "./pages/user-list.page";

test.describe("User List Page (with POM)", () => {
  let userListPage: UserListPage;

  test.beforeEach(async ({ page }) => {
    userListPage = new UserListPage(page);
    await userListPage.goto();
  });

  test("should search users", async () => {
    await userListPage.searchByName("张三");
    const userName = await userListPage.getFirstUserName();
    expect(userName).toContain("张三");
  });

  test("should navigate to detail", async ({ page }) => {
    await userListPage.clickViewButton(0);
    await expect(page).toHaveURL(/\/user\/detail\/\d+/);
  });
});
```

### 3. 完整的 CRUD 测试流程

```typescript
// e2e/tests/user/user-crud.spec.ts
import { test, expect } from "@playwright/test";
import { UserListPage } from "../../pages/user-list.page";

test.describe("User CRUD Operations", () => {
  test("should create, view, edit, and delete user", async ({ page }) => {
    // === 1. Navigate to user list ===
    await page.goto("/user/list");
    await page.waitForLoadState("networkidle");

    // === 2. Create new user ===
    await page.click('button:has-text("新增用户")');

    // Verify modal opens
    const modal = page.locator(".ant-modal:visible");
    await expect(modal).toBeVisible();
    await expect(modal.locator(".ant-modal-title")).toHaveText("新增用户");

    // Fill form
    const testUserName = `测试用户_${Date.now()}`;
    await page.fill('input[name="name"]', testUserName);
    await page.fill('input[name="email"]', `test${Date.now()}@example.com`);
    await page.fill('input[name="phone"]', "13800138000");

    // Select role
    await page.click('.ant-select:has-text("选择角色")');
    await page.click('.ant-select-item:has-text("普通用户")');

    // Submit form
    await page.click('.ant-modal-footer button:has-text("确定")');

    // Wait for success message
    await expect(page.locator('.ant-message:has-text("创建成功")')).toBeVisible();

    // Verify modal closes
    await expect(modal).not.toBeVisible();

    // === 3. Search for created user ===
    await page.fill('input[placeholder*="姓名"]', testUserName);
    await page.click('button:has-text("搜索")');

    // Verify user appears in list
    const firstRow = page.locator("tbody tr:first-child");
    await expect(firstRow).toContainText(testUserName);

    // === 4. View user detail ===
    await firstRow.locator('button:has-text("查看")').click();
    await expect(page).toHaveURL(/\/user\/detail\/\d+/);
    await expect(page.locator("h1")).toContainText("用户详情");
    await expect(page.locator("text=" + testUserName)).toBeVisible();

    // Navigate back to list
    await page.click('button:has-text("返回")');
    await expect(page).toHaveURL("/user/list");

    // === 5. Edit user ===
    await firstRow.locator('button:has-text("编辑")').click();

    // Verify edit modal
    await expect(modal).toBeVisible();
    await expect(modal.locator(".ant-modal-title")).toHaveText("编辑用户");

    // Update name
    const updatedName = testUserName + "_updated";
    await page.fill('input[name="name"]', updatedName);

    // Submit
    await page.click('.ant-modal-footer button:has-text("确定")');
    await expect(page.locator('.ant-message:has-text("更新成功")')).toBeVisible();

    // Verify updated name
    await expect(firstRow).toContainText(updatedName);

    // === 6. Delete user ===
    await firstRow.locator('button:has-text("删除")').click();

    // Confirm deletion
    const confirmModal = page.locator(".ant-modal-confirm");
    await expect(confirmModal).toBeVisible();
    await expect(confirmModal).toContainText("确认删除");
    await confirmModal.locator('button:has-text("确定")').click();

    // Verify success message
    await expect(page.locator('.ant-message:has-text("删除成功")')).toBeVisible();

    // Verify user no longer in list
    await expect(page.locator(`text=${updatedName}`)).not.toBeVisible();
  });
});
```

### 4. 带认证的测试

```typescript
// e2e/fixtures/auth.ts
import { test as base } from "@playwright/test";

type AuthFixture = {
  authenticatedPage: Page;
};

export const test = base.extend<AuthFixture>({
  authenticatedPage: async ({ page }, use) => {
    // Login before each test
    await page.goto("/login");
    await page.fill('input[name="username"]', "admin");
    await page.fill('input[name="password"]', "admin123");
    await page.click('button:has-text("登录")');

    // Wait for redirect to dashboard
    await page.waitForURL("/dashboard");

    // Verify token is stored
    const token = await page.evaluate(() => localStorage.getItem("token"));
    if (!token) {
      throw new Error("Authentication failed - no token found");
    }

    await use(page);
  },
});
```

```typescript
// e2e/tests/auth/protected-routes.spec.ts
import { test, expect } from "../../fixtures/auth";

test.describe("Protected Routes", () => {
  test("should access user list when authenticated", async ({ authenticatedPage }) => {
    await authenticatedPage.goto("/user/list");
    await expect(authenticatedPage.locator("h1")).toContainText("用户列表");
  });

  test("should redirect to login when not authenticated", async ({ page }) => {
    // Clear storage
    await page.context().clearCookies();
    await page.evaluate(() => localStorage.clear());

    // Try to access protected route
    await page.goto("/user/list");

    // Should redirect to login
    await expect(page).toHaveURL("/login");
  });
});
```

### 5. 多页面导航测试

```typescript
// tests/e2e/multi-page-flow.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Complete User Management Flow", () => {
  test("should navigate through dashboard → user list → user detail", async ({ page }) => {
    // === Step 1: Login ===
    await page.goto("/login");
    await page.fill('input[name="username"]', "admin");
    await page.fill('input[name="password"]', "admin123");
    await page.click('button:has-text("登录")');
    await page.waitForURL("/dashboard");

    // === Step 2: Verify Dashboard ===
    await expect(page.locator("h1")).toContainText("仪表盘");

    // Verify stats cards
    const statsCards = page.locator(".stats-card");
    await expect(statsCards).toHaveCount(4);

    // === Step 3: Navigate to User Management ===
    await page.click('aside a[href="/user/list"]');
    await expect(page).toHaveURL("/user/list");
    await expect(page.locator("h1")).toContainText("用户列表");

    // === Step 4: Click first user ===
    const firstUser = page.locator("tbody tr:first-child");
    const userName = await firstUser.locator("td:nth-child(2)").textContent();
    await firstUser.locator('button:has-text("查看")').click();

    // === Step 5: Verify User Detail ===
    await expect(page).toHaveURL(/\/user\/detail\/\d+/);
    await expect(page.locator("h1")).toContainText("用户详情");
    await expect(page.locator(".user-name")).toContainText(userName!);

    // === Step 6: Navigate back via breadcrumb ===
    await page.click('.ant-breadcrumb a:has-text("用户列表")');
    await expect(page).toHaveURL("/user/list");

    // === Step 7: Return to dashboard via sidebar ===
    await page.click('aside a[href="/dashboard"]');
    await expect(page).toHaveURL("/dashboard");
  });
});
```

### 6. 表单验证测试

```typescript
// tests/e2e/user-form-validation.spec.ts
import { test, expect } from "@playwright/test";

test.describe("User Form Validation", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/user/list");
    await page.click('button:has-text("新增用户")');
  });

  test("should show error for empty required fields", async ({ page }) => {
    // Submit without filling anything
    await page.click('.ant-modal-footer button:has-text("确定")');

    // Verify error messages
    await expect(page.locator('.ant-form-item-explain-error:has-text("请输入姓名")')).toBeVisible();
    await expect(page.locator('.ant-form-item-explain-error:has-text("请输入邮箱")')).toBeVisible();
  });

  test("should validate email format", async ({ page }) => {
    // Fill with invalid email
    await page.fill('input[name="email"]', "invalid-email");
    await page.click('input[name="name"]'); // Trigger blur

    // Verify error message
    await expect(
      page.locator('.ant-form-item-explain-error:has-text("邮箱格式不正确")'),
    ).toBeVisible();
  });

  test("should validate phone number format", async ({ page }) => {
    // Fill with invalid phone
    await page.fill('input[name="phone"]', "123");
    await page.click('input[name="name"]');

    await expect(
      page.locator('.ant-form-item-explain-error:has-text("手机号格式不正确")'),
    ).toBeVisible();
  });

  test("should submit when all fields are valid", async ({ page }) => {
    // Fill valid data
    await page.fill('input[name="name"]', "测试用户");
    await page.fill('input[name="email"]', "test@example.com");
    await page.fill('input[name="phone"]', "13800138000");

    // No error messages should be visible
    await expect(page.locator(".ant-form-item-explain-error")).not.toBeVisible();

    // Submit button should be enabled
    const submitButton = page.locator('.ant-modal-footer button:has-text("确定")');
    await expect(submitButton).toBeEnabled();
  });
});
```

### 7. 数据加载状态测试

```typescript
// e2e/tests/user/loading-states.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Loading States", () => {
  test("should show loading spinner while fetching data", async ({ page }) => {
    // Slow down network to observe loading state
    await page.route("**/api/users", (route) => {
      setTimeout(() => route.continue(), 2000);
    });

    await page.goto("/user/list");

    // Verify loading spinner appears
    await expect(page.locator(".ant-spin")).toBeVisible();

    // Wait for data to load
    await page.waitForLoadState("networkidle");

    // Verify spinner disappears
    await expect(page.locator(".ant-spin")).not.toBeVisible();

    // Verify table is visible
    await expect(page.locator("table")).toBeVisible();
  });

  test("should show empty state when no data", async ({ page }) => {
    // Mock empty response
    await page.route("**/api/users", (route) => {
      route.fulfill({
        status: 200,
        contentType: "application/json",
        body: JSON.stringify({
          code: 0,
          data: { list: [], total: 0 },
        }),
      });
    });

    await page.goto("/user/list");

    // Verify empty state
    await expect(page.locator(".ant-empty")).toBeVisible();
    await expect(page.locator("text=暂无数据")).toBeVisible();
  });

  test("should show error message on API failure", async ({ page }) => {
    // Mock error response
    await page.route("**/api/users", (route) => {
      route.fulfill({
        status: 500,
        contentType: "application/json",
        body: JSON.stringify({
          code: 1,
          message: "服务器错误",
        }),
      });
    });

    await page.goto("/user/list");

    // Verify error message
    await expect(page.locator(".ant-message-error")).toBeVisible();
    await expect(page.locator("text=服务器错误")).toBeVisible();
  });
});
```

## 运行测试

```bash
# 安装 Playwright 浏览器（首次运行）
pnpm exec playwright install

# 运行所有 E2E 测试
pnpm exec playwright test

# 运行特定文件
pnpm exec playwright test tests/e2e/user-list.spec.ts

# 运行并显示浏览器（调试模式）
pnpm exec playwright test --headed

# 运行并打开 UI 模式
pnpm exec playwright test --ui

# 运行特定浏览器
pnpm exec playwright test --project=chromium
pnpm exec playwright test --project=firefox
pnpm exec playwright test --project=webkit

# 生成测试报告
pnpm exec playwright show-report
```

## Playwright 配置

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests/e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: "html",

  use: {
    baseURL: "http://localhost:5173",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
    video: "retain-on-failure",
  },

  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "firefox",
      use: { ...devices["Desktop Firefox"] },
    },
    {
      name: "webkit",
      use: { ...devices["Desktop Safari"] },
    },
    // Mobile viewports
    {
      name: "Mobile Chrome",
      use: { ...devices["Pixel 5"] },
    },
  ],

  webServer: {
    command: "pnpm run dev",
    url: "http://localhost:5173",
    reuseExistingServer: !process.env.CI,
  },
});
```

## 最佳实践

### ✅ DO

1. **使用 Page Object Model 组织代码**
   - 提高代码复用性
   - 便于维护

2. **使用有意义的选择器**

   ```typescript
   // ✅ Good - semantic selectors
   page.locator('button:has-text("提交")');
   page.getByRole("button", { name: "提交" });
   page.getByLabel("用户名");

   // ❌ Bad - fragile selectors
   page.locator("div.css-123abc > button");
   page.locator("button:nth-child(3)");
   ```

3. **正确等待元素**

   ```typescript
   // ✅ Good - auto-waiting
   await expect(page.locator("text=加载完成")).toBeVisible();

   // ❌ Bad - fixed timeout
   await page.waitForTimeout(5000);
   ```

4. **测试用户行为，不是实现**

   ```typescript
   // ✅ Good - 测试用户看到的
   await expect(page.locator("text=登录成功")).toBeVisible();

   // ❌ Bad - 测试实现细节
   expect(await page.evaluate(() => window.localStorage.token)).toBeTruthy();
   ```

5. **独立的测试数据**

   ```typescript
   // ✅ Good - unique test data
   const testEmail = `test_${Date.now()}@example.com`;

   // ❌ Bad - reusing same data
   const testEmail = "test@example.com"; // 可能冲突
   ```

### ❌ DON'T

1. **不要依赖测试顺序**
2. **不要在测试间共享状态**
3. **不要使用固定的 sleep/wait**
4. **不要测试第三方服务**
5. **不要忽略失败的测试**

## 调试技巧

### 1. 使用 Playwright Inspector

```bash
pnpm exec playwright test --debug
```

### 2. 慢动作模式

```typescript
test("slow test", async ({ page }) => {
  await page.goto("/user/list", { slowMo: 1000 }); // 每个操作延迟 1 秒
});
```

### 3. 截图调试

```typescript
await page.screenshot({ path: "debug.png", fullPage: true });
```

### 4. 查看 Trace

```bash
pnpm exec playwright show-trace trace.zip
```

## 常见问题

### Q: 测试不稳定（flaky），时而成功时而失败？

**A**: 使用自动等待机制，避免固定 timeout：

```typescript
// ✅ Good
await expect(page.locator("text=已加载")).toBeVisible();

// ❌ Bad
await page.waitForTimeout(3000);
```

### Q: 如何测试需要登录的页面？

**A**: 使用 fixture 复用认证状态，参考 "带认证的测试" 部分。

### Q: 如何加速测试执行？

**A**:

1. 使用 `test.describe.configure({ mode: 'parallel' })`
2. Mock 不必要的 API 调用
3. 复用认证状态

### Q: 如何测试文件上传？

```typescript
await page.setInputFiles('input[type="file"]', "path/to/file.png");
```

## CI/CD 集成

```yaml
# .github/workflows/e2e-tests.yml
name: E2E Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: pnpm install
      - run: pnpm exec playwright install --with-deps
      - run: pnpm exec playwright test
      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
```

## 参考资源

- [Playwright 官方文档](https://playwright.dev/)
- [Best Practices](https://playwright.dev/docs/best-practices)
- [Selectors Guide](https://playwright.dev/docs/selectors)
- [Auto-waiting](https://playwright.dev/docs/actionability)
