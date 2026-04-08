---
name: product-design
description: Design and model product features from natural language requirements into structured YAML definitions
metadata:
  version: 1.0.0
  author: GitHub Copilot
  tags: [product-design, requirements, yaml, modeling, domain-driven]
---

# Product Design Skill

## Purpose

Transform natural language feature requirements into standardized YAML definitions, ensuring semantic quality and consistency with project terminology.

## When to Use

- User provides unstructured feature ideas or requirements
- Need to formalize user stories into feature definitions
- Creating new feature YAML files
- Reviewing and improving existing feature definitions

## Prerequisites

Before generation, **must read**:

1. `docs/product/product-definition.yaml` - Business scope and system boundaries
2. `docs/product/terminology.yaml` - Standard terminology and naming conventions
3. `docs/product/system-model.yaml` - Entity definitions and state machines

## Output Location

Save generated YAML to: `docs/features/{domain}-management.yaml`

**Supported domains**:

- `user-management` - User account and role management
- `device-management` - Access control device management
- `permission-management` - Access permission configuration
- `access-monitoring` - Access log and real-time monitoring
- `spatial-management` - Building/unit/room hierarchy
- `data-analytics` - Statistical reports and dashboards
- `operation-audit` - Operation log and audit trail

## YAML Structure

### Required Sections

```yaml
meta:
  domain: { domain-name }
  description: Brief domain description
  version: v2.0.0
  last_updated: 2026-03-05
  changelog:
    - v2.0.0: Major change description
    - v1.0.0: Initial version

pages:
  - feature_key: { kebab-case-key }
    name: Display name
    route: /path/to/page
    page_type: table-with-filters | detail-page

    business:
      description: What this feature does
      value: Business value
      priority: high | medium | low
      entities: [Entity1, Entity2]
      actors: [role1, role2]
      acceptance_criteria:
        - Criterion 1
        - Criterion 2

    ui:
      i18n_prefix: domain
      breadcrumb: [...]
      filters: [...] # For list pages
      table: [...] # For list pages
      info_card: [...] # For detail pages
      tabs: [...] # For detail pages
      page_actions: [...]

    api:
      mock: getMockFunction
      endpoint: GET /api/v1/resource
      params:
        query: [param1, param2]

actions:
  - feature_key: { kebab-case-key }
    name: Action name
    action_type: modal-form | confirm-modal

    business:
      description: What this action does
      priority: high | medium | low
      entities: [Entity]
      actors: [role1, role2]
      accept_criteria: [...]

    ui:
      modal_width: 600
      i18n_prefix: domain
      form: [...] # For modal-form
      confirm: [...] # For confirm-modal

    api:
      mock: actionMockFunction
      endpoint: POST /api/v1/resource
      success_message: i18nKey
      error_message: i18nKey
```

## Modeling Principles

### 1. Feature Granularity

**✅ Good - Atomic features**:

- `query-user-list` - Query and display user list
- `create-user` - Create new user
- `update-user-info` - Update user basic information

**❌ Bad - Too coarse**:

- `user-management` - Too broad, combine multiple features

### 2. Business Language

**✅ Good - Business capability**:

- `assign-device-permission` - Assign device access permission
- `suspend-user` - Suspend user account

**❌ Bad - Implementation detail**:

- `update-user-status-to-suspended` - Exposes state machine
- `insert-permission-record` - Database operation

### 3. Terminology Consistency

Always use terms from `terminology.yaml`:

**✅ Correct**:

```yaml
entities: [User, Device, AccessPermission]
actors: [property, system-admin]
```

**❌ Wrong**:

```yaml
entities: [person, machine, auth] # Not in terminology
actors: [admin, manager] # Not defined roles
```

### 4. Entity Alignment

Reference entities from `system-model.yaml`:

```yaml
# ✅ Valid entities from system-model.yaml
entities: [User, Device, AccessPermission, Room, Building]

# ❌ Invalid - not defined
entities: [Customer, Hardware, Access]
```

### 5. State Machine Integration

When features involve state changes, reference state machines:

```yaml
business:
  description: Activate device and make it available for use
  entities: [Device]
  state_transition: inactive → active # References DeviceState machine
```

## UI Configuration Guidelines

### Page Types

**1. table-with-filters** - List page with search/filter

- Use for: User list, device list, log list
- Required UI: `filters`, `table`, `page_actions`

**2. detail-page** - Detail view with tabs

- Use for: User detail, device detail
- Required UI: `info_card`, `tabs`

**3. modal-form** (action) - Form in modal

- Use for: Create, edit operations
- Required UI: `modal_width`, `form`

**4. confirm-modal** (action) - Confirmation dialog

- Use for: Delete, suspend, dangerous operations
- Required UI: `confirm`

### Filter Types

```yaml
filters:
  - field: keyword
    type: input # Text input

  - field: status
    type: select # Dropdown select
    options:
      type: enum
      enum_values: [active, inactive]

  - field: created_at
    type: date-range-picker # Date range
```

### Table Configuration

```yaml
table:
  row_key: id_field
  pagination: true
  columns:
    - field: user_id
      label: id
      width: 80

    - field: status
      label: status
      render: tag # Render as tag
      tag_map:
        active: { color: success }
        inactive: { color: default }

  actions: # Row actions
    - key: view
      type: link
      navigate: /path/{id}

    - key: delete
      type: link
      danger: true
      trigger_feature: delete-user
```

### Form Configuration

```yaml
form:
  layout: vertical
  fields:
    - field: name
      label: name
      type: input
      required: true
      placeholder: namePlaceholder
      rules:
        - { type: required, message: nameRequired }
        - { type: max, value: 50, message: nameTooLong }

    - field: role
      label: role
      type: select
      required: true
      options:
        type: enum
        enum_values: [owner, tenant, property]
        label_prefix: role_
      rules:
        - { type: required, message: roleRequired }
```

## Validation Rules

### Required Fields

- `meta.domain` - Must match file name prefix
- `meta.version` - Semantic version
- `pages[].feature_key` - Unique identifier
- `pages[].route` - Valid route path
- `pages[].page_type` - One of predefined types
- `business.description` - Clear feature description
- `business.priority` - high/medium/low
- `api.mock` - Mock service function name
- `api.endpoint` - RESTful endpoint pattern

### Naming Conventions

- **Feature keys**: kebab-case (`query-user-list`, `create-device`)
- **Field names**: snake_case (`user_id`, `created_at`)
- **i18n keys**: camelCase (`userList`, `createSuccess`)
- **Routes**: lowercase with hyphens (`/user/list`, `/device/detail/:id`)

## Examples

### Example 1: List Page

```yaml
pages:
  - feature_key: query-device-list
    name: 设备列表
    route: /device/list
    page_type: table-with-filters

    business:
      description: 查询和管理门禁设备列表
      value: 快速检索设备信息，实时监控设备状态
      priority: high
      entities: [Device, Building, Unit]
      actors: [property, system-admin]
      acceptance_criteria:
        - 支持按设备名称、位置搜索
        - 支持按设备状态筛选
        - 显示设备实时在线状态
        - 支持批量操作

    ui:
      i18n_prefix: device
      breadcrumb:
        - nav.deviceManagement
        - nav.deviceList

      filters:
        - field: keyword
          type: input
          label: keyword
          placeholder: keywordPlaceholder
          width: 200

        - field: state
          type: select
          label: state
          options:
            type: enum
            enum_values: [active, inactive, maintenance]
            label_prefix: state_

      table:
        row_key: device_id
        pagination: true
        columns:
          - {field: device_id, label: id, width: 80}
          - {field: device_name, label: name, width: 150}
          - {field: location, label: location, width: 200}
          - field: state
            label: state
            width: 100
            render: tag
            tag_map:
              active: {color: success}
              inactive: {color: default}
              maintenance: {color: warning}

        actions:
          - {key: view, label: view, type: link, navigate: /device/detail/{device_id}, i18n_prefix: common}
          - {key: remote-open, label: remoteOpen, type: link}

      page_actions:
        - {key: create, label: create, type: primary, icon: PlusOutlined, trigger_feature: add-device, i18n_prefix: common}

    api:
      mock: getMockDevices
      endpoint: GET /api/v1/devices
      params:
        query: [keyword, state, page, pageSize]
```

### Example 2: Create Action

```yaml
actions:
  - feature_key: create-device
    name: 添加设备
    action_type: modal-form

    business:
      description: 添加新的门禁设备到系统
      priority: high
      entities: [Device, Building, Unit]
      actors: [property, system-admin]
      accept_criteria:
        - 必填：设备名称、设备类型、安装位置
        - 设备序列号唯一性校验
        - 创建后设备状态为 inactive

    ui:
      modal_width: 600
      i18n_prefix: device
      form:
        layout: vertical
        fields:
          - field: device_name
            label: name
            type: input
            required: true
            rules:
              - { type: required, message: nameRequired }

          - field: device_type
            label: type
            type: select
            required: true
            options:
              type: enum
              enum_values: [face_recognition, card_reader, qr_code]
              label_prefix: type_
            rules:
              - { type: required, message: typeRequired }

          - field: location
            label: location
            type: cascader
            required: true
            placeholder: locationPlaceholder
            api:
              mock: getMockLocations
              endpoint: GET /api/v1/locations/tree

    api:
      mock: createMockDevice
      endpoint: POST /api/v1/devices
      success_message: createSuccess
      error_message: createFailed
```

## Workflow

### Step 1: Understand Requirements

Read user input and identify:

- What domain does this belong to?
- What entities are involved?
- Who are the actors?
- What's the business value?

### Step 2: Check Context

Read reference files:

- Verify domain exists in `product-definition.yaml`
- Check terminology in `terminology.yaml`
- Validate entities in `system-model.yaml`

### Step 3: Structure Features

Break down into:

- **Pages**: Full-screen views (list, detail)
- **Actions**: Operations (create, edit, delete)

### Step 4: Define UI

For each feature, specify:

- Page type and layout
- Filters (if list page)
- Table columns (if list page)
- Form fields (if action)
- Validation rules

### Step 5: Map APIs

For each feature, define:

- Mock service function name
- RESTful endpoint
- Request parameters
- Success/error messages

### Step 6: Validate

- Run `pnpm run validate:yaml`
- Check terminology consistency
- Verify entity alignment
- Review acceptance criteria

## Common Patterns

### CRUD Operations

**List + Detail + Actions**:

```yaml
pages:
  - feature_key: query-{domain}-list
  - feature_key: view-{domain}-detail

actions:
  - feature_key: create-{domain}
  - feature_key: update-{domain}-info
  - feature_key: delete-{domain}
```

### State Management

**Activate/Suspend Pattern**:

```yaml
actions:
  - feature_key: activate-{entity}
    action_type: confirm-modal
    state_transition: inactive → active

  - feature_key: suspend-{entity}
    action_type: modal-form # Need reason
    state_transition: active → suspended
```

### Relationship Management

**Bind/Unbind Pattern**:

```yaml
actions:
  - feature_key: bind-{entity1}-to-{entity2}
    entities: [Entity1, Entity2]

  - feature_key: unbind-{entity1}-from-{entity2}
    entities: [Entity1, Entity2]
    action_type: confirm-modal
```

## Quality Checklist

After generation, verify:

- [ ] Feature keys follow kebab-case convention
- [ ] All entities exist in `system-model.yaml`
- [ ] All actors are defined roles
- [ ] Terminology matches `terminology.yaml`
- [ ] UI configuration is complete and valid
- [ ] Mock service names follow naming pattern
- [ ] API endpoints follow RESTful conventions
- [ ] i18n keys are consistent with prefix
- [ ] Validation rules are appropriate
- [ ] Acceptance criteria are testable
- [ ] YAML syntax is valid (`pnpm run validate:yaml`)

## Notes

1. **Semantic over syntax**: Focus on business meaning, not just structure
2. **Consistency**: Reuse patterns from existing features
3. **Completeness**: Define both business and technical specifications
4. **Maintainability**: Keep features atomic and independent
5. **Evolution**: Use changelog to track versions

When in doubt, reference existing feature files in `docs/features/` for patterns and examples.
