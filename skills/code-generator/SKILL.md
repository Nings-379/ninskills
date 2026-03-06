---
name: code-generator
description: Generate complete React page component code from feature YAML definitions, including UI, forms, tables, validation, API calls, and state management
---

# Code Generator Skill

## Purpose

Automatically generate complete React page component code based on feature definitions in `docs/features/*.yaml`.

## Use Cases

- User provides a feature key or name and requests generation of the corresponding page component
- User requests "implement XX feature" and needs the complete flow from feature to code
- User requests modification or optimization of generated components

## Prerequisites

Before generating code, **must** read the following files as context:

1. **Product Specifications**
   - `docs/product/terminology.yaml` - Terminology standards
   - `docs/product/system-model.yaml` - Entity and state machine definitions

2. **UI Specifications**
   - `docs/ui/component-mapping.yaml` - Feature to UI component mapping
   - `docs/ui/layout-patterns.yaml` - Page layout patterns
   - `docs/ui/interaction-patterns.yaml` - Interaction flow specifications
   - `docs/ui/navigation-structure.yaml` - Route and menu structure

3. **API Specifications**
   - `docs/api/api-conventions.yaml` - API endpoint mapping and conventions

4. **Type Definitions**
   - `packages/fe/src/types/entities.ts` - Entity types
   - `packages/fe/src/types/enums.ts` - Enum types
   - `packages/fe/src/types/api-requests.ts` - Request types
   - `packages/fe/src/types/api-responses.ts` - Response types

5. **Feature Definitions**
   - `docs/features/*.yaml` - Complete definition of target feature

6. **Frontend Coding Standards**
   - `.github/instructions/frontend.instructions.md` - Frontend code standards

## Code Generation Flow

### Step 1: Parse Feature Definition

```yaml
# Example: query-user-list feature
key: query-user-list
name: Query User List
description: Query user list in the system, supports pagination, sorting, and filtering
entities: [User]
actors: [property, system-admin]
acceptance_criteria:
  - Support filtering by role
  - Support filtering by status
  - Support keyword search
  - Paginated display, default 20 items per page
```

**Extract Information**:

- Feature type: `query` (inferred from key)
- Primary entity: `User`
- Filter conditions: role, status, keyword
- Permissions: property, system-admin

### Step 2: Determine UI Component Type

Based on `feature_type_classification` in `component-mapping.yaml`:

```yaml
query:
  ui_pattern: table-with-filters
  components:
    - FilterBar # Filter bar
    - DataTable # Data table
    - Pagination # Pagination
```

### Step 3: Determine Layout Pattern

Look up `table-with-filters` from `layout-patterns.yaml`:

```yaml
table-with-filters:
  structure:
    - PageContainer
      - FilterBar
      - Form (inline)
      - ActionBar
      - DataTable
      - Pagination
```

### Step 4: Generate API Calls

Get endpoint information from `api-conventions.yaml`:

```typescript
// 根据 feature key 映射到端点
const endpoint = "/api/v1/users"; // GET
const requestType = "QueryUserListRequest";
const responseType = "QueryUserListResponse";
```

### Step 5: Generate Entity Form Fields

Get field configuration from `entity_field_mappings.User` in `component-mapping.yaml`:

```typescript
const formFields = [
  { name: "name", label: "Name", type: "text", required: true },
  {
    name: "phone",
    label: "Phone",
    type: "text",
    required: true,
    rules: [{ pattern: /^1[3-9]\d{9}$/, message: "Invalid phone number format" }],
  },
  { name: "role", label: "Role", type: "select", options: USER_ROLE, required: true },
];
```

### Step 6: Generate Table Column Definitions

Get column configuration from `table_column_configs.User` in `component-mapping.yaml`:

```typescript
const columns = [
  { title: 'Name', dataIndex: 'name', key: 'name' },
  { title: 'Phone', dataIndex: 'phone', key: 'phone' },
  { title: 'Role', dataIndex: 'role', key: 'role',
    render: (role) => <Badge color={getRoleColor(role)} text={getRoleText(role)} /> },
  { title: 'Status', dataIndex: 'status', key: 'status',
    render: (status) => <Badge status={status === 'active' ? 'success' : 'default'} /> },
  { title: 'Actions', key: 'actions', render: (_, record) => <ActionButtons record={record} /> },
];
```

### Step 7: Generate Interaction Logic

Get interaction flow from `interaction-patterns.yaml`:

```typescript
// Form submission flow
const handleSubmit = async (values) => {
  setLoading(true);
  try {
    const response = await api.post("/api/v1/users", values);
    message.success("Created successfully");
    form.resetFields();
    onSuccess?.();
  } catch (error) {
    if (error.code === 1005) {
      // Validation error
      error.data.forEach(({ field, reason }) => {
        form.setFields([{ name: field, errors: [reason] }]);
      });
    } else {
      message.error(error.message || "Operation failed");
    }
  } finally {
    setLoading(false);
  }
};
```

### Step 8: Generate Complete Component Code

Combine all the above parts to generate a complete React component.

## Code Templates

### Query Feature (List Query)

```typescript
import { useState, useEffect } from 'react';
import { Card, Table, Form, Input, Select, Button, Space, message } from 'antd';
import { SearchOutlined, ReloadOutlined } from '@ant-design/icons';
import type { {EntityName} } from '@/types/entities';
import type { {RequestType}, {ResponseType} } from '@/types/api-requests';
import { cn } from '@/lib/utils';

export default function {FeatureName}Page() {
  const [form] = Form.useForm();
  const [loading, setLoading] = useState(false);
  const [dataSource, setDataSource] = useState<{EntityName}[]>([]);
  const [pagination, setPagination] = useState({ current: 1, pageSize: 20, total: 0 });

  // Fetch data
  const fetchData = async (params?: Partial<{RequestType}>) => {
    setLoading(true);
    try {
      const response = await fetch('{endpoint}?' + new URLSearchParams({
        page: pagination.current.toString(),
        pageSize: pagination.pageSize.toString(),
        ...params,
      }));
      const result: {ResponseType} = await response.json();

      if (result.code === 0) {
        setDataSource(result.data.items);
        setPagination(prev => ({ ...prev, total: result.data.total }));
      } else {
        message.error(result.message);
      }
    } catch (error) {
      message.error('Query failed');
    } finally {
      setLoading(false);
    }
  };

  // Initial load
  useEffect(() => {
    fetchData();
  }, []);

  // Search
  const handleSearch = (values: any) => {
    setPagination(prev => ({ ...prev, current: 1 }));
    fetchData(values);
  };

  // Reset
  const handleReset = () => {
    form.resetFields();
    setPagination(prev => ({ ...prev, current: 1 }));
    fetchData();
  };

  // Pagination change
  const handleTableChange = (newPagination: any) => {
    setPagination(newPagination);
    fetchData({ ...form.getFieldsValue(), page: newPagination.current });
  };

  // Table column definitions
  const columns = [
    {/* Generate columns based on entity */}
  ];

  return (
    <div className={cn('p-6')}>
      <Card>
        {/* Filter bar */}
        <Form
          form={form}
          layout="inline"
          onFinish={handleSearch}
          className={cn('mb-4')}
        >
          {/* Generate filter fields based on acceptance_criteria */}

          <Form.Item>
            <Space>
              <Button type="primary" htmlType="submit" icon={<SearchOutlined />}>
                Search
              </Button>
              <Button onClick={handleReset} icon={<ReloadOutlined />}>
                Reset
              </Button>
            </Space>
          </Form.Item>
        </Form>

        {/* Data table */}
        <Table
          columns={columns}
          dataSource={dataSource}
          loading={loading}
          rowKey="id"
          pagination={pagination}
          onChange={handleTableChange}
        />
      </Card>
    </div>
  );
}
```

### Create Feature (Create Form)

```typescript
import { useState } from 'react';
import { Modal, Form, Input, Select, Button, message } from 'antd';
import type { {RequestType} } from '@/types/api-requests';
import { cn } from '@/lib/utils';

interface {FeatureName}ModalProps {
  open: boolean;
  onCancel: () => void;
  onSuccess: () => void;
}

export default function {FeatureName}Modal({ open, onCancel, onSuccess }: {FeatureName}ModalProps) {
  const [form] = Form.useForm();
  const [loading, setLoading] = useState(false);

  const handleSubmit = async () => {
    try {
      const values = await form.validateFields();
      setLoading(true);

      const response = await fetch('{endpoint}', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(values),
      });

      const result = await response.json();

      if (result.code === 0) {
        message.success('创建成功');
        form.resetFields();
        onSuccess();
      } else {
        // 处理验证错误
        if (result.code === 1005 && result.data) {
          result.data.forEach(({ field, reason }: any) => {
            form.setFields([{ name: field, errors: [reason] }]);
          });
        } else {
          message.error(result.message);
        }
      }
    } catch (error) {
      message.error('操作失败');
    } finally {
      setLoading(false);
    }
  };

  return (
    <Modal
      title="{feature.name}"
      open={open}
      onCancel={onCancel}
      onOk={handleSubmit}
      confirmLoading={loading}
      width={600}
    >
      <Form form={form} layout="vertical">
        {/* 根据 entity.attributes 生成表单项 */}
      </Form>
    </Modal>
  );
}
```

### Update Feature (更新表单)

类似 Create,但需要:

1. 接收 `initialValues` prop
2. 使用 PATCH 方法
3. 端点为 `{endpoint}/:id`

### Delete Feature (删除确认)

```typescript
import { Modal, message } from 'antd';
import { ExclamationCircleOutlined } from '@ant-design/icons';

export const handleDelete = (id: number, onSuccess: () => void) => {
  Modal.confirm({
    title: '确认删除',
    icon: <ExclamationCircleOutlined />,
    content: '删除后无法恢复，确定要删除吗？',
    okText: '确定',
    okType: 'danger',
    cancelText: '取消',
    async onOk() {
      try {
        const response = await fetch(`{endpoint}/${id}`, {
          method: 'DELETE',
        });
        const result = await response.json();

        if (result.code === 0) {
          message.success('删除成功');
          onSuccess();
        } else {
          message.error(result.message);
        }
      } catch (error) {
        message.error('删除失败');
      }
    },
  });
};
```

### Action Feature (操作按钮)

```typescript
import { Button, message } from 'antd';
import { {IconName} } from '@ant-design/icons';

export const handle{ActionName} = async (id: number, onSuccess: () => void) => {
  try {
    const response = await fetch(`{endpoint}/${id}/{action}`, {
      method: 'POST',
    });
    const result = await response.json();

    if (result.code === 0) {
      message.success('{action} 成功');
      onSuccess();
    } else {
      message.error(result.message);
    }
  } catch (error) {
    message.error('操作失败');
  }
};
```

## 代码生成检查清单

生成代码后,确保:

- [ ] 所有 import 语句正确
- [ ] 使用 `cn()` 处理所有 className
- [ ] 使用 TypeScript 类型(从 `@/types` 导入)
- [ ] 错误处理完整(401/403/404/500)
- [ ] Loading 状态管理
- [ ] 表单验证规则符合 `interaction-patterns.yaml`
- [ ] API 端点符合 `api-conventions.yaml`
- [ ] 组件名称使用 PascalCase
- [ ] 文件名使用 kebab-case
- [ ] 遵循 `frontend.instructions.md` 中的所有规则

## 文件输出位置

根据 feature 所属 domain 确定输出路径:

```
packages/fe/src/pages/
├── user/                    # user-management domain
│   ├── list/
│   │   ├── index.tsx       # query-user-list
│   │   └── hooks/
│   │       └── use-user-list.ts
│   ├── create-modal.tsx    # create-user
│   └── update-modal.tsx    # update-user-info
├── device/                  # device-management domain
│   ├── list/
│   │   └── index.tsx
│   └── control-panel.tsx
├── permission/              # permission-management domain
│   └── list/
│       └── index.tsx
└── access/                  # access-monitoring domain
    ├── log/
    │   └── index.tsx
    └── realtime/
        └── index.tsx
```

## 高级场景

### 实时数据监控

对于 `realtime-access-monitor` 类型的 feature:

```typescript
useEffect(() => {
  // 轮询或 WebSocket
  const interval = setInterval(() => {
    fetchData();
  }, 5000); // 5秒刷新

  return () => clearInterval(interval);
}, []);
```

### 批量操作

```typescript
const [selectedRowKeys, setSelectedRowKeys] = useState<number[]>([]);

const rowSelection = {
  selectedRowKeys,
  onChange: setSelectedRowKeys,
};

const handleBatchAction = async () => {
  if (selectedRowKeys.length === 0) {
    message.warning("请先选择要操作的记录");
    return;
  }

  // 批量操作逻辑
};
```

### 导出功能

```typescript
const handleExport = async () => {
  try {
    const response = await fetch("{endpoint}/export", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(form.getFieldsValue()),
    });

    const blob = await response.blob();
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "export.xlsx";
    a.click();
    message.success("导出成功");
  } catch (error) {
    message.error("导出失败");
  }
};
```

## 示例：生成完整的"用户列表"页面

**输入**: feature key = `query-user-list`

**输出**: `packages/fe/src/pages/user/list/index.tsx`

```typescript
import { useState, useEffect } from 'react';
import { Card, Table, Form, Input, Select, Button, Space, message, Tag } from 'antd';
import { SearchOutlined, ReloadOutlined, PlusOutlined } from '@ant-design/icons';
import type { User } from '@/types/entities';
import type { QueryUserListRequest, QueryUserListResponse } from '@/types/api-requests';
import { USER_ROLE } from '@/types/enums';
import { cn } from '@/lib/utils';
import CreateUserModal from '../create-modal';
import UpdateUserModal from '../update-modal';

export default function UserListPage() {
  const [form] = Form.useForm();
  const [loading, setLoading] = useState(false);
  const [dataSource, setDataSource] = useState<User[]>([]);
  const [pagination, setPagination] = useState({ current: 1, pageSize: 20, total: 0 });
  const [createModalOpen, setCreateModalOpen] = useState(false);
  const [updateModalOpen, setUpdateModalOpen] = useState(false);
  const [selectedUser, setSelectedUser] = useState<User | null>(null);

  const fetchData = async (params?: Partial<QueryUserListRequest>) => {
    setLoading(true);
    try {
      const queryParams = new URLSearchParams({
        page: (params?.page || pagination.current).toString(),
        pageSize: (params?.pageSize || pagination.pageSize).toString(),
        ...params,
      });

      const response = await fetch(`/api/v1/users?${queryParams}`);
      const result: QueryUserListResponse = await response.json();

      if (result.code === 0) {
        setDataSource(result.data.items);
        setPagination(prev => ({
          ...prev,
          total: result.data.total,
          current: result.data.page,
          pageSize: result.data.pageSize,
        }));
      } else {
        message.error(result.message);
      }
    } catch (error) {
      message.error('查询失败');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, []);

  const handleSearch = (values: any) => {
    setPagination(prev => ({ ...prev, current: 1 }));
    fetchData({ ...values, page: 1 });
  };

  const handleReset = () => {
    form.resetFields();
    setPagination(prev => ({ ...prev, current: 1 }));
    fetchData({ page: 1 });
  };

  const handleTableChange = (newPagination: any) => {
    fetchData({
      ...form.getFieldsValue(),
      page: newPagination.current,
      pageSize: newPagination.pageSize,
    });
  };

  const handleEdit = (record: User) => {
    setSelectedUser(record);
    setUpdateModalOpen(true);
  };

  const handleDelete = (id: number) => {
    // 删除逻辑
  };

  const getRoleText = (role: string) => {
    const roleMap: Record<string, string> = {
      'property': '物业',
      'owner': '业主',
      'tenant': '租户',
      'security': '保安',
      'committee': '业委会',
      'visitor': '访客',
      'system-admin': '系统管理员',
    };
    return roleMap[role] || role;
  };

  const columns = [
    { title: '姓名', dataIndex: 'name', key: 'name' },
    { title: '手机号', dataIndex: 'phone', key: 'phone' },
    {
      title: '角色',
      dataIndex: 'role',
      key: 'role',
      render: (role: string) => <Tag color="blue">{getRoleText(role)}</Tag>,
    },
    {
      title: '状态',
      dataIndex: 'status',
      key: 'status',
      render: (status: string) => (
        <Tag color={status === 'active' ? 'success' : 'default'}>
          {status === 'active' ? '正常' : '停用'}
        </Tag>
      ),
    },
    {
      title: '操作',
      key: 'actions',
      render: (_: any, record: User) => (
        <Space>
          <Button type="link" size="small" onClick={() => handleEdit(record)}>
            编辑
          </Button>
          <Button type="link" size="small" danger onClick={() => handleDelete(record.user_id)}>
            删除
          </Button>
        </Space>
      ),
    },
  ];

  return (
    <div className={cn('p-6')}>
      <Card>
        <Form
          form={form}
          layout="inline"
          onFinish={handleSearch}
          className={cn('mb-4')}
        >
          <Form.Item name="keyword" label="关键词">
            <Input placeholder="姓名或手机号" style={{ width: 200 }} />
          </Form.Item>

          <Form.Item name="role" label="角色">
            <Select placeholder="请选择" style={{ width: 150 }} allowClear>
              {Object.entries(USER_ROLE).map(([key, value]) => (
                <Select.Option key={value} value={value}>
                  {getRoleText(value)}
                </Select.Option>
              ))}
            </Select>
          </Form.Item>

          <Form.Item name="status" label="状态">
            <Select placeholder="请选择" style={{ width: 120 }} allowClear>
              <Select.Option value="active">正常</Select.Option>
              <Select.Option value="inactive">停用</Select.Option>
            </Select>
          </Form.Item>

          <Form.Item>
            <Space>
              <Button type="primary" htmlType="submit" icon={<SearchOutlined />}>
                搜索
              </Button>
              <Button onClick={handleReset} icon={<ReloadOutlined />}>
                重置
              </Button>
              <Button
                type="primary"
                icon={<PlusOutlined />}
                onClick={() => setCreateModalOpen(true)}
              >
                新建用户
              </Button>
            </Space>
          </Form.Item>
        </Form>

        <Table
          columns={columns}
          dataSource={dataSource}
          loading={loading}
          rowKey="user_id"
          pagination={pagination}
          onChange={handleTableChange}
        />
      </Card>

      {/* 创建用户弹窗 */}
      <CreateUserModal
        open={createModalOpen}
        onCancel={() => setCreateModalOpen(false)}
        onSuccess={() => {
          setCreateModalOpen(false);
          fetchData();
        }}
      />

      {/* 更新用户弹窗 */}
      {selectedUser && (
        <UpdateUserModal
          open={updateModalOpen}
          user={selectedUser}
          onCancel={() => {
            setUpdateModalOpen(false);
            setSelectedUser(null);
          }}
          onSuccess={() => {
            setUpdateModalOpen(false);
            setSelectedUser(null);
            fetchData();
          }}
        />
      )}
    </div>
  );
}
```

## 注意事项

1. **类型安全**: 所有 API 调用必须使用生成的 TypeScript 类型
2. **错误处理**: 必须处理所有可能的错误场景
3. **Loading 状态**: 异步操作必须显示 loading
4. **用户反馈**: 操作成功/失败必须有明确提示
5. **权限控制**: 根据 `actors` 字段动态显示/隐藏操作按钮
6. **代码风格**: 严格遵循 `frontend.instructions.md`

## 生成后验证

生成代码后,必须运行:

```bash
pnpm exec tsc -b        # TypeScript 类型检查
pnpm exec eslint .      # ESLint 检查
pnpm exec vite build    # 构建验证
```

确保所有检查通过,没有错误。
