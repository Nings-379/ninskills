---
name: unit-test-generator
description: Generate comprehensive unit tests for functions, components, and hooks using Vitest and React Testing Library
---

# Unit Test Generator Skill

## Objective

Generate high-quality unit tests for frontend code, covering utility functions, React components, custom hooks, and data processing logic.

## Test Scope

### Code that requires unit tests

1. **Utility Functions** (`src/lib/**/*.ts`)
   - String processing, data transformation, formatting functions
   - Examples: `cn()`, `formatDate()`, `generateId()`

2. **Pure UI Components** (`src/components/**/*.tsx`)
   - Presentation components without data fetching
   - Examples: Button, Card, Modal, Form components

3. **Custom Hooks** (`src/hooks/**/*.ts`)
   - State management, data processing logic
   - Examples: `useLocalStorage()`, `useDebounce()`

4. **Type Guards and Validation Functions**
   - Runtime type checking
   - Examples: `isValidEmail()`, `validateUserInput()`

### Code that does NOT need unit tests

- ❌ Page components (use E2E tests instead)
- ❌ Route configurations
- ❌ Pure type definitions (TypeScript provides type safety)

## Tech Stack

- **Test Framework**: Vitest
- **React Testing**: @testing-library/react
- **Assertions**: Vitest (built-in expect)
- **Mocking**: Vitest (vi.fn, vi.mock)

## File Naming Convention

**All unit test files should be placed in `tests/unit/` directory**, organized by the source file's category:

```
src/lib/utils.ts                        → tests/unit/lib/utils.test.ts
src/components/common/button.tsx        → tests/unit/components/button.test.tsx
src/hooks/useDebounce.ts                → tests/unit/hooks/useDebounce.test.ts
```

**Directory Structure:**

```
tests/
  unit/
    lib/              # Tests for utility functions
      utils.test.ts
    components/       # Tests for UI components
      button.test.tsx
      modal.test.tsx
    hooks/            # Tests for custom hooks
      useDebounce.test.ts
  e2e/               # End-to-end tests (separate)
    user-list.spec.ts
```

## Test Templates

### 1. Utility Function Test Template

```typescript
// tests/unit/lib/utils.test.ts
import { describe, it, expect } from "vitest";
import { cn } from "../../../src/lib/utils";

describe("cn utility function", () => {
  it("should merge single class name", () => {
    expect(cn("px-4")).toBe("px-4");
  });

  it("should merge multiple class names", () => {
    expect(cn("px-4", "py-2", "bg-white")).toBe("px-4 py-2 bg-white");
  });

  it("should handle conditional classes", () => {
    expect(cn("px-4", false && "hidden", "py-2")).toBe("px-4 py-2");
  });

  it("should merge conflicting Tailwind classes correctly", () => {
    // tailwind-merge should keep the last one
    expect(cn("px-4", "px-8")).toBe("px-8");
  });

  it("should handle empty input", () => {
    expect(cn()).toBe("");
  });

  it("should handle undefined and null values", () => {
    expect(cn("px-4", undefined, "py-2", null)).toBe("px-4 py-2");
  });
});
```

### 2. React Component Test Template

```typescript
// tests/unit/components/button.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '../../../src/components/button';

describe('Button Component', () => {
  it('should render with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('should call onClick handler when clicked', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('should be disabled when disabled prop is true', () => {
    render(<Button disabled>Disabled Button</Button>);
    const button = screen.getByText('Disabled Button');
    expect(button).toBeDisabled();
  });

  it('should apply custom className', () => {
    render(<Button className="custom-class">Styled Button</Button>);
    const button = screen.getByText('Styled Button');
    expect(button).toHaveClass('custom-class');
  });

  it('should render with icon', () => {
    const Icon = () => <span data-testid="icon">🔍</span>;
    render(<Button icon={<Icon />}>Search</Button>);
    expect(screen.getByTestId('icon')).toBeInTheDocument();
  });
});
```

### 3. Component with i18n Test Template

```typescript
// tests/unit/components/language-switcher.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { I18nextProvider } from 'react-i18next';
import i18n from '@/lib/i18n';
import { LanguageSwitcher } from './language-switcher';

// Mock i18n
vi.mock('react-i18next', () => ({
  useTranslation: () => ({
    t: (key: string) => key,
    i18n: {
      language: 'zh',
      changeLanguage: vi.fn(),
    },
  }),
}));

describe('LanguageSwitcher Component', () => {
  it('should render language options', () => {
    render(
      <I18nextProvider i18n={i18n}>
        <LanguageSwitcher />
      </I18nextProvider>
    );
    expect(screen.getByText('中文')).toBeInTheDocument();
  });

  it('should change language when clicked', () => {
    const { i18n } = require('react-i18next').useTranslation();
    render(
      <I18nextProvider i18n={i18n}>
        <LanguageSwitcher />
      </I18nextProvider>
    );

    fireEvent.click(screen.getByText('English'));
    expect(i18n.i18n.changeLanguage).toHaveBeenCalledWith('en');
  });
});
```

### 4. Custom Hook Test Template

```typescript
// tests/unit/hooks/useDebounce.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";
import { renderHook, waitFor } from "@testing-library/react";
import { useDebounce } from "./useDebounce";

describe("useDebounce Hook", () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.restoreAllMocks();
  });

  it("should return initial value immediately", () => {
    const { result } = renderHook(() => useDebounce("initial", 500));
    expect(result.current).toBe("initial");
  });

  it("should debounce value changes", async () => {
    const { result, rerender } = renderHook(({ value }) => useDebounce(value, 500), {
      initialProps: { value: "initial" },
    });

    // Change value
    rerender({ value: "updated" });

    // Value should not change immediately
    expect(result.current).toBe("initial");

    // Fast-forward time
    vi.advanceTimersByTime(500);

    // Now value should be updated
    await waitFor(() => {
      expect(result.current).toBe("updated");
    });
  });

  it("should cancel previous timeout on rapid changes", async () => {
    const { result, rerender } = renderHook(({ value }) => useDebounce(value, 500), {
      initialProps: { value: "v1" },
    });

    rerender({ value: "v2" });
    vi.advanceTimersByTime(300);

    rerender({ value: "v3" });
    vi.advanceTimersByTime(300);

    // Should still be initial
    expect(result.current).toBe("v1");

    // After full delay
    vi.advanceTimersByTime(200);
    await waitFor(() => {
      expect(result.current).toBe("v3");
    });
  });
});
```

## Coverage Requirements

- ✅ Utility functions: **100%**
- ✅ UI components: **80%+**
- ✅ Hooks: **90%+**

## Running Tests

```bash
# Run all unit tests
pnpm test

# Run specific test file
pnpm test tests/unit/lib/utils.test.ts

# Watch mode (for development)
pnpm test --watch

# Generate coverage report
pnpm test --coverage
```

## Best Practices

### ✅ DO

1. **Test behavior, not implementation details**

   ```typescript
   // ✅ Good - test what users see
   expect(screen.getByText("Submit")).toBeInTheDocument();

   // ❌ Bad - test implementation
   expect(component.state.buttonText).toBe("Submit");
   ```

2. **Use descriptive test names**

   ```typescript
   // ✅ Good
   it("should display error message when email is invalid", () => {});

   // ❌ Bad
   it("test email validation", () => {});
   ```

3. **One test validates one thing**

   ```typescript
   // ✅ Good
   it("should disable button when loading", () => {});
   it("should show spinner when loading", () => {});

   // ❌ Bad
   it("should handle loading state", () => {
     // Tests multiple things
   });
   ```

4. **Follow AAA pattern (Arrange-Act-Assert)**

   ```typescript
   it("should calculate total price", () => {
     // Arrange
     const items = [{ price: 10 }, { price: 20 }];

     // Act
     const total = calculateTotal(items);

     // Assert
     expect(total).toBe(30);
   });
   ```

### ❌ DON'T

1. Don't test third-party libraries
2. Don't test framework internals
3. Don't include business logic in tests
4. Don't skip failing tests (using it.skip)
5. Don't write order-dependent tests

## Common Questions

### Q: How to mock TanStack Query?

```typescript
import { vi } from "vitest";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: { retry: false },
  },
});

vi.mock("@tanstack/react-query", async () => {
  const actual = await vi.importActual("@tanstack/react-query");
  return {
    ...actual,
    useQuery: vi.fn(),
  };
});

// In test
import { useQuery } from "@tanstack/react-query";
vi.mocked(useQuery).mockReturnValue({
  data: mockData,
  isLoading: false,
  error: null,
});
```

### Q: How to test async operations?

```typescript
import { waitFor } from '@testing-library/react';

it('should load data asynchronously', async () => {
  render(<AsyncComponent />);

  // Wait for element to appear
  await waitFor(() => {
    expect(screen.getByText('Loaded')).toBeInTheDocument();
  });
});
```

### Q: How to test user interactions?

```typescript
import { fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

it('should handle user input', async () => {
  const user = userEvent.setup();
  render(<Form />);

  // Type into input
  await user.type(screen.getByLabelText('Email'), 'test@example.com');

  // Click button
  await user.click(screen.getByText('Submit'));

  // Or use fireEvent for simpler cases
  fireEvent.change(screen.getByLabelText('Email'), {
    target: { value: 'test@example.com' },
  });
});
```

## References

- [Vitest 文档](https://vitest.dev/)
- [React Testing Library](https://testing-library.com/react)
- [Testing Library 最佳实践](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
