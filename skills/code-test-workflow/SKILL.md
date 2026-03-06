---
name: code-test-workflow
description: 从 YAML 生成代码并自动创建和运行测试的完整工作流
---

# 代码生成 + 测试验证工作流

## 目的

建立一个自动化工作流：**YAML 定义 → 生成代码 → 生成测试 → 运行验证**

确保生成的代码质量可靠，符合规范，并通过测试验证。

---

## 工作流程概览

```
[1] 读取 Feature YAML
      ↓
[2] 生成页面组件代码 (code-generator)
      ↓
[3] 生成单元测试 (unit-test-generator)
      ↓
[4] 生成 E2E 测试 (e2e-test-generator)
      ↓
[5] 运行所有测试并验证
      ↓
[6] 构建验证 (TypeScript + ESLint + Vite)
      ↓
[7] 输出结果报告
```

---

## 详细步骤

### 步骤 1：读取 Feature YAML

**输入**：Feature key 或 YAML 文件路径

**必读文件**：

```
docs/product/terminology.yaml
docs/product/system-model.yaml
docs/ui/component-mapping.yaml
docs/ui/layout-patterns.yaml
docs/ui/interaction-patterns.yaml
docs/api/api-conventions.yaml
docs/features/{domain}-management.yaml
```

**输出**：完整的功能定义上下文

---

### 步骤 2：生成页面组件代码

**Skill**: `code-generator`

**输入**：

- Feature key (如 `query-user-list`)
- 目标路径 (如 `packages/fe/src/pages/user/list/`)

**生成内容**：

```
packages/fe/src/pages/{domain}/{page}/
  ├── index.tsx          # 页面主组件
  ├── types.ts          # 本地类型定义 (如需要)
  └── hooks.ts          # 自定义 hooks (如需要)
```

**代码规范**：

- 必须使用 TypeScript
- 必须使用 i18next 多语言
- 必须使用 @tanstack/react-query
- 必须使用 Ant Design 组件
- 必须使用 mock 服务（`@/services/mock/`）
- 必须遵循 `.github/instructions/frontend.instructions.md`

**验证检查**：

- [ ] 所有类型正确导入
- [ ] i18n key 完整定义
- [ ] mock 服务已存在
- [ ] 组件结构符合 layout-patterns
- [ ] API 调用符合 api-conventions

---

### 步骤 3：生成单元测试

**Skill**: `unit-test-generator`

**输入**：生成的页面组件路径

**生成内容**：

```
tests/unit/{domain}/{page}.test.tsx
```

**测试覆盖**：

- ✅ 组件渲染
- ✅ 筛选器交互
- ✅ 表格数据显示
- ✅ 按钮点击事件
- ✅ API 调用 (mocked)
- ✅ 错误处理
- ✅ 加载状态

**立即运行**：

```bash
pnpm test:run --project=unit tests/unit/{domain}/{page}.test.tsx
```

**预期结果**：所有单元测试通过 ✅

---

### 步骤 4：生成 E2E 测试

**Skill**: `e2e-test-generator`

**输入**：

- Feature key
- 页面路由
- 用户操作流程

**生成内容**：

```
tests/e2e/{domain}-{page}.spec.ts
tests/e2e/pages/{domain}-{page}.page.ts  # Page Object Model
```

**测试场景**：

- ✅ 页面加载和渲染
- ✅ 筛选器功能
- ✅ 表格交互
- ✅ 创建功能 (如有)
- ✅ 编辑功能 (如有)
- ✅ 删除功能 (如有)
- ✅ 多浏览器兼容性

**立即运行**：

```bash
pnpm test:e2e tests/e2e/{domain}-{page}.spec.ts --project=chromium
```

**预期结果**：所有 E2E 测试通过 ✅

---

### 步骤 5：运行完整测试套件

```bash
# 单元测试
pnpm test:run --project=unit

# E2E 测试 (只跑 chromium 加快速度)
pnpm test:e2e --project=chromium

# 如需完整测试所有浏览器
pnpm test:e2e
```

**验证要点**：

- [ ] 所有单元测试通过
- [ ] 所有 E2E 测试通过
- [ ] 无测试超时
- [ ] 无测试失败

---

### 步骤 6：构建验证

```bash
cd packages/fe

# TypeScript 类型检查
pnpm exec tsc -b

# ESLint 代码检查
pnpm exec eslint .

# 生产构建
pnpm run build
```

**验证要点**：

- [ ] 无 TypeScript 类型错误
- [ ] 无 ESLint 错误
- [ ] 构建成功

---

### 步骤 7：输出结果报告

**成功输出**：

```
✅ 代码生成成功
  - 页面组件: packages/fe/src/pages/{domain}/{page}/index.tsx
  - 单元测试: tests/unit/{domain}/{page}.test.tsx
  - E2E 测试: tests/e2e/{domain}-{page}.spec.ts

✅ 测试验证通过
  - 单元测试: 15/15 通过
  - E2E 测试: 8/8 通过

✅ 构建验证通过
  - TypeScript: ✓
  - ESLint: ✓
  - Production Build: ✓

🎉 工作流完成！代码已生成并通过所有验证。
```

**失败输出**：

```
❌ 工作流失败

失败阶段: [测试验证]

错误详情:
  - 单元测试: 13/15 通过，2 个失败
    × tests/unit/user/list.test.tsx:45 - TypeError: Cannot read property 'name'
    × tests/unit/user/list.test.tsx:67 - AssertionError: Expected "用户" but got undefined

建议:
  1. 检查 i18n 配置是否完整
  2. 检查 mock 数据结构是否匹配
  3. 修复后重新运行测试

修复命令:
  pnpm test:run --project=unit tests/unit/user/list.test.tsx
```

---

## 使用示例

### 示例 1：生成用户列表页面

```
用户: 请根据 query-user-list 生成完整的用户列表页面，包括代码和测试

Agent:
1. [读取] docs/features/user-management.yaml
   - 功能: query-user-list
   - 页面类型: table-with-filters
   - 筛选字段: keyword, role, status
   - 表格列: user_id, name, phone, role, status, created_at

2. [生成代码] packages/fe/src/pages/user/list/index.tsx
   ✅ 组件生成成功，使用 getMockUsers mock 服务

3. [生成单元测试] tests/unit/user/list.test.tsx
   ✅ 15 个测试用例已生成

4. [运行单元测试]
   $ pnpm test:run --project=unit tests/unit/user/list.test.tsx
   ✅ 15/15 通过

5. [生成 E2E 测试] tests/e2e/user-list.spec.ts
   ✅ 11 个测试场景已生成

6. [运行 E2E 测试]
   $ pnpm test:e2e tests/e2e/user-list.spec.ts --project=chromium
   ✅ 11/11 通过

7. [构建验证]
   $ pnpm exec tsc -b && pnpm exec eslint . && pnpm run build
   ✅ 所有检查通过

🎉 完成！用户列表页面已生成并通过所有验证。
```

### 示例 2：修改现有页面

```
用户: 在用户列表页面添加"批量删除"功能

Agent:
1. [读取现有代码] packages/fe/src/pages/user/list/index.tsx

2. [更新 YAML] docs/features/user-management.yaml
   - 添加 batch-delete-users 功能定义

3. [更新代码] 添加批量选择和批量删除按钮
   ✅ 代码已更新

4. [更新单元测试] tests/unit/user/list.test.tsx
   - 添加批量选择测试
   - 添加批量删除测试
   ✅ 新增 5 个测试用例

5. [运行单元测试]
   ✅ 20/20 通过

6. [更新 E2E 测试] tests/e2e/user-list.spec.ts
   - 添加批量删除场景
   ✅ 新增 2 个测试场景

7. [运行 E2E 测试]
   ✅ 13/13 通过

8. [构建验证]
   ✅ 所有检查通过

🎉 批量删除功能已添加并验证完成！
```

---

## 最佳实践

### 1. 始终从 YAML 开始

- ❌ 直接修改代码
- ✅ 先更新 YAML，再生成代码

### 2. 保持 YAML 完整性

- YAML 应包含业务定义和详细的 UI 配置
- 业务定义：功能描述、价值、验收标准
- UI 配置：页面类型、筛选器、表格、表单、验证规则
- API 配置：Mock 服务和真实端点

### 3. Mock 数据优先

- 在后端 API 未就绪前，始终使用 mock 服务
- Mock 数据结构应与真实 API 响应一致

### 4. 逐个验证

- 不要一次性生成所有页面
- 每个页面生成后立即运行测试
- 确保当前页面通过后再生成下一个

### 5. 增量开发

- 先生成基础功能（列表、详情）
- 再添加复杂功能（创建、编辑、删除）
- 每次添加功能后都要更新测试

---

## 故障排查

### 问题 1：单元测试失败 - i18n key 未定义

**症状**：

```
× Expected "用户列表" but got undefined
```

**原因**：i18n 翻译文件缺少对应的 key

**解决**：

1. 检查 `packages/fe/src/locales/zh-CN.json`
2. 添加缺失的 translation key
3. 重新运行测试

### 问题 2：E2E 测试失败 - 元素未找到

**症状**：

```
× locator.click: Timeout 5000ms exceeded
  Element not found: getByRole('button', { name: /创建/ })
```

**原因**：

- i18n key 不匹配
- 元素选择器错误
- 页面加载超时

**解决**：

1. 检查页面是否正确渲染
2. 检查 i18n key 是否正确
3. 增加等待时间或使用更精确的选择器

### 问题 3：TypeScript 类型错误

**症状**：

```
error TS2339: Property 'user_id' does not exist on type 'User'
```

**原因**：类型定义与实际数据不匹配

**解决**：

1. 检查 `packages/fe/src/types/entities.ts`
2. 更新类型定义
3. 确保 mock 数据结构与类型一致

### 问题 4：Mock 服务未找到

**症状**：

```
error TS2307: Cannot find module '@/services/mock/user.mock'
```

**原因**：Mock 服务文件不存在或路径错误

**解决**：

1. 检查 `packages/fe/src/services/mock/` 目录
2. 创建缺失的 mock 服务文件
3. 导出所需的 mock 函数

---

## 工作流检查清单

在执行工作流前，确认以下条件：

**YAML 准备**：

- [ ] Feature YAML 已完整定义
- [ ] UI 配置已详细说明
- [ ] API 配置已明确（mock 和 real endpoint）
- [ ] YAML 语法验证通过 (`pnpm run validate:yaml`)

**环境准备**：

- [ ] 项目依赖已安装 (`pnpm install`)
- [ ] Mock 服务已创建
- [ ] i18n 翻译文件已准备
- [ ] 类型定义已完善

**生成后验证**：

- [ ] 代码文件已创建
- [ ] 测试文件已创建
- [ ] 所有导入路径正确
- [ ] 单元测试通过
- [ ] E2E 测试通过
- [ ] 构建验证通过

---

## 相关 Skills

- **code-generator** - 从 YAML 生成 React 组件代码
- **unit-test-generator** - 生成 Vitest 单元测试
- **e2e-test-generator** - 生成 Playwright E2E 测试
- **storybook-generator** - 生成 Storybook stories (可选)
- **yaml-validation** - 验证 YAML 语法

---

## 参考文档

- [Frontend Instructions](.github/instructions/frontend.instructions.md)
- [Test Instructions](.github/instructions/test.instructions.md)
- [Component Mapping](docs/ui/component-mapping.yaml)
- [Layout Patterns](docs/ui/layout-patterns.yaml)
- [API Conventions](docs/api/api-conventions.yaml)
