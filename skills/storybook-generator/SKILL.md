---
name: storybook-generator
description: Automatically generate Storybook stories files for React components, providing interactive documentation and demonstration
---

# Storybook Generator Skill

## Purpose

Automatically generate corresponding `.stories.ts` or `.stories.tsx` files for React components in the `packages/fe/src` directory, for Storybook interactive documentation.

## Use Cases

- A new component has been created and needs stories generated
- User requests "create Storybook documentation for XX component"
- Batch generate stories for all components in a directory

## Prerequisites

Before generating stories, **must**:

1. Confirm the target component exists
2. Read the component's props definition (TypeScript interface)
3. Understand the component's use cases and state variants

## Stories Generation Rules

### Component Classification and Stories Strategy

Generate different stories based on component type:

#### 1. Pure UI Components (components/common)

Create stories for each visual state and size:

- **Default**: Default state
- **Variants**: Different visual variants
- **Sizes**: Different sizes (small, medium, large)
- **States**: Different states (disabled, loading, error)
- **Interactive**: Interaction examples

#### 2. Layout Components (components/layout)

Demonstrate layout structure and responsive behavior:

- **Empty**: Empty content state
- **WithContent**: With content state
- **Responsive**: Display on different screen sizes

#### 3. Business Components (components/business)

Demonstrate business scenarios and data states:

- **Default**: Default data
- **Loading**: Loading state
- **Empty**: Empty data state
- **Error**: Error state
- **WithMockData**: Complete mock data

## Stories File Template

### Basic Template

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { fn } from 'storybook/test';
import { {ComponentName} } from './{component-file}';

const meta = {
  title: '{Category}/{ComponentName}',
  component: {ComponentName},
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
  argTypes: {
    // Automatically generate argTypes based on props
  },
  args: {
    // Default args
  },
} satisfies Meta<typeof {ComponentName}>;

export default meta;
type Story = StoryObj<typeof meta>;

// Stories
export const Default: Story = {
  args: {
    // Default props
  },
};
```

### Props to ArgTypes Mapping

Automatically generate argTypes based on TypeScript props types:

```typescript
// Props definition
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  onClick?: () => void;
}

// Generated argTypes
argTypes: {
  label: {
    control: 'text',
    description: 'Button text',
  },
  variant: {
    control: 'select',
    options: ['primary', 'secondary', 'outline'],
    description: 'Button variant',
  },
  size: {
    control: 'radio',
    options: ['small', 'medium', 'large'],
    description: 'Button size',
  },
  disabled: {
    control: 'boolean',
    description: 'Whether disabled',
  },
  loading: {
    control: 'boolean',
    description: 'Whether loading',
  },
  onClick: {
    action: 'clicked',
    description: 'Click event',
  },
}
```

## Specific Examples

### Example 1: Pure UI Component - Button

**File**: `packages/fe/src/components/common/button/button.stories.ts`

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { fn } from 'storybook/test';
import { Button } from './button';

const meta = {
  title: 'Common/Button',
  component: Button,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'outline', 'ghost'],
      description: 'Button variant',
    },
    size: {
      control: 'radio',
      options: ['small', 'medium', 'large'],
      description: 'Button size',
    },
    disabled: {
      control: 'boolean',
      description: 'Whether disabled',
    },
    loading: {
      control: 'boolean',
      description: 'Whether loading',
    },
  },
  args: {
    onClick: fn(),
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Button',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Button',
  },
};

export const Outline: Story = {
  args: {
    variant: 'outline',
    children: 'Button',
  },
};

export const Small: Story = {
  args: {
    size: 'small',
    children: 'Small Button',
  },
};

export const Large: Story = {
  args: {
    size: 'large',
    children: 'Large Button',
  },
};

export const Disabled: Story = {
  args: {
    disabled: true,
    children: 'Disabled Button',
  },
};

export const Loading: Story = {
  args: {
    loading: true,
    children: 'Loading Button',
  },
};

export const WithIcon: Story = {
  args: {
    icon: <SearchOutlined />,
    children: 'Search',
  },
};
```

### Example 2: Layout Component - FilterBar

**File**: `packages/fe/src/components/layout/filter-bar/filter-bar.stories.tsx`

```typescript
import type { Meta, StoryObj } from '@storybook/react';
import { Form, Input, Select, Button } from 'antd';
import { FilterBar } from './filter-bar';

const meta = {
  title: 'Layout/FilterBar',
  component: FilterBar,
  parameters: {
    layout: 'fullscreen',
  },
  tags: ['autodocs'],
} satisfies Meta<typeof FilterBar>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    children: (
      <>
        <Form.Item name="keyword" label="Keyword">
          <Input placeholder="Please enter" />
        </Form.Item>
        <Form.Item name="status" label="Status">
          <Select placeholder="Please select">
            <Select.Option value="active">Active</Select.Option>
            <Select.Option value="inactive">Inactive</Select.Option>
          </Select>
        </Form.Item>
        <Form.Item>
          <Button type="primary">Search</Button>
          <Button>Reset</Button>
        </Form.Item>
      </>
    ),
  },
};

export const WithManyFilters: Story = {
  args: {
    children: (
      <>
        <Form.Item name="keyword" label="Keyword">
          <Input placeholder="Please enter" />
        </Form.Item>
        <Form.Item name="role" label="Role">
          <Select placeholder="Please select" style={{ width: 150 }}>
            <Select.Option value="owner">Owner</Select.Option>
            <Select.Option value="tenant">Tenant</Select.Option>
          </Select>
        </Form.Item>
        <Form.Item name="status" label="Status">
          <Select placeholder="Please select" style={{ width: 120 }}>
            <Select.Option value="active">Active</Select.Option>
            <Select.Option value="inactive">Inactive</Select.Option>
          </Select>
        </Form.Item>
        <Form.Item name="community" label="Community">
          <Select placeholder="Please select" style={{ width: 200 }}>
            <Select.Option value="1">Sunshine Garden</Select.Option>
            <Select.Option value="2">Green Lake Estate</Select.Option>
          </Select>
        </Form.Item>
        <Form.Item>
          <Button type="primary">Search</Button>
          <Button>Reset</Button>
        </Form.Item>
      </>
    ),
  },
};
```

### Example 3: Business Component - UserSelector

**File**: `packages/fe/src/components/business/user-selector/user-selector.stories.tsx`

```typescript
import type { Meta, StoryObj } from "@storybook/react";
import { UserSelector } from "./user-selector";

const meta = {
  title: "Business/UserSelector",
  component: UserSelector,
  parameters: {
    layout: "centered",
  },
  tags: ["autodocs"],
  argTypes: {
    mode: {
      control: "radio",
      options: ["single", "multiple"],
      description: "Selection mode",
    },
    filterRole: {
      control: "select",
      options: ["owner", "tenant", "property", "security"],
      description: "Filter by role",
    },
  },
} satisfies Meta<typeof UserSelector>;

export default meta;
type Story = StoryObj<typeof meta>;

const mockUsers = [
  { id: 1, name: "Zhang San", role: "owner", phone: "13800138000" },
  { id: 2, name: "Li Si", role: "tenant", phone: "13900139000" },
  { id: 3, name: "Wang Wu", role: "property", phone: "13700137000" },
];

export const Default: Story = {
  args: {
    placeholder: "Please select user",
  },
};

export const WithMockData: Story = {
  args: {
    placeholder: "Please select user",
    // Inject mock data
    __mockData: mockUsers,
  },
};

export const Multiple: Story = {
  args: {
    mode: "multiple",
    placeholder: "Please select users (multiple selection)",
    __mockData: mockUsers,
  },
};

export const FilterByRole: Story = {
  args: {
    filterRole: "owner",
    placeholder: "Select owner",
    __mockData: mockUsers.filter((u) => u.role === "owner"),
  },
};

export const Loading: Story = {
  args: {
    loading: true,
    placeholder: "Loading...",
  },
};

export const Disabled: Story = {
  args: {
    disabled: true,
    placeholder: "Disabled",
  },
};
```

### Example 4: Modal Component

**File**: `packages/fe/src/pages/user/create-modal.stories.tsx`

```typescript
import type { Meta, StoryObj } from "@storybook/react";
import { fn } from "storybook/test";
import CreateUserModal from "./create-modal";

const meta = {
  title: "Pages/User/CreateModal",
  component: CreateUserModal,
  parameters: {
    layout: "fullscreen",
  },
  tags: ["autodocs"],
  argTypes: {
    open: {
      control: "boolean",
      description: "Whether to open the modal",
    },
  },
  args: {
    onCancel: fn(),
    onSuccess: fn(),
  },
} satisfies Meta<typeof CreateUserModal>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Open: Story = {
  args: {
    open: true,
  },
};

export const Closed: Story = {
  args: {
    open: false,
  },
};
```

## Stories Directory Structure

Stories files are placed in the same directory as component files:

```
packages/fe/src/
├── components/
│   ├── common/
│   │   ├── button/
│   │   │   ├── button.tsx
│   │   │   └── button.stories.ts
│   │   └── input/
│   │       ├── input.tsx
│   │       └── input.stories.ts
│   ├── layout/
│   │   ├── filter-bar/
│   │   │   ├── filter-bar.tsx
│   │   │   └── filter-bar.stories.tsx
│   │   └── page-container/
│   │       ├── page-container.tsx
│   │       └── page-container.stories.tsx
│   └── business/
│       ├── user-selector/
│       │   ├── user-selector.tsx
│       │   └── user-selector.stories.tsx
│       └── device-selector/
│           ├── device-selector.tsx
│           └── device-selector.stories.tsx
└── pages/
    └── user/
        ├── create-modal.tsx
        └── create-modal.stories.tsx
```

## Storybook Title Naming Convention

Use forward-slash separated hierarchical structure:

- `Common/ComponentName` - Generic components
- `Layout/ComponentName` - Layout components
- `Business/ComponentName` - Business components
- `Pages/Domain/ComponentName` - Page components

## Mock Data Handling

### Method 1: Using MSW (Mock Service Worker)

Configure MSW in `.storybook/preview.ts`:

```typescript
import { initialize, mswLoader } from "msw-storybook-addon";

initialize();

export const loaders = [mswLoader];
```

Define mock handlers in story:

```typescript
import { http, HttpResponse } from "msw";

export const WithMockData: Story = {
  parameters: {
    msw: {
      handlers: [
        http.get("/api/v1/users", () => {
          return HttpResponse.json({
            code: 0,
            message: "Success",
            data: {
              items: [
                /* mock data */
              ],
              total: 100,
              page: 1,
              pageSize: 20,
            },
          });
        }),
      ],
    },
  },
};
```

### Method 2: Inject Mock Data via Props

Add optional `__mockData` prop to component (only for Storybook):

```typescript
interface UserSelectorProps {
  // Normal props
  value?: number;
  onChange?: (value: number) => void;

  // Storybook mock data (not used in production)
  __mockData?: User[];
}
```

## Generation Checklist

After generating stories, ensure:

- [ ] File name uses kebab-case: `component-name.stories.ts`
- [ ] Title uses correct hierarchy: `Category/ComponentName`
- [ ] Contains at least Default story
- [ ] Contains main state variants (Loading, Error, Empty)
- [ ] ArgTypes correctly map all props
- [ ] Interactive props use `fn()` for listening
- [ ] Mock data is complete and realistic

## Running Storybook

```bash
cd packages/fe
pnpm run storybook
```

Visit `http://localhost:6006` to view the generated stories.

## Notes

1. **Don't depend on real APIs**: Stories should use mock data and not depend on backend
2. **Cover all states**: Include loading, error, empty, disabled, etc.
3. **Interactivity**: Use args and actions to make stories interactive
4. **Documentation**: Use `autodocs` tag to automatically generate documentation
5. **Consistency**: Use similar stories structure for similar components

## Batch Generation

Users can request batch generation of stories for all components in a directory:

```
Generate stories for all components in the components/business directory
```

In this case:

1. List all components in that directory
2. Read component code one by one
3. Analyze props definitions
4. Generate corresponding stories file for each component
