---
title: Unit Testing with Vitest
layout: default
nav_order: 12
---

# Unit Testing with Vitest

## Core Packages Versions

- vite: 5.4.10
- vitest: 3.0.5
- @vitest/coverage-v8: 3.0.5
- @vitest/ui: 3.0.5

## Testing Components

We use `@testing-library/react`, `@testing-library/user-event` & `@testing-library/jest-dom/vitest` with Vitest's DOM integration for component testing. Our components follow the React Development Standards where we ensure:

- Core component functionality must be tested
- Focus on unit tests, not integration tests
- Keep tests alongside components (`component.test.tsx`)

### Example Component Test:

```tsx
import { render, screen } from "@testing-library/react";
import Button from "./Button";

describe("Button", () => {
	it("renders with correct text", () => {
		render(<Button>Click me</Button>);
		expect(screen.getByText("Click me")).toBeInTheDocument();
	});

	it("handles disabled state", () => {
		render(<Button disabled>Click me</Button>);
		expect(screen.getByRole("button")).toBeDisabled();
	});

  // Calls a prop function when clicked
  it("calls onButtonClick when clicked", async () => {
    const handleClick = vi.fn();
    render(<ExampleButton onButtonClick={handleClick} />);
    
    const button = screen.getByRole("button");
    await userEvent.click(button);
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it("handles fetching RTK - API data on button click", async () => {
		const mockFetchData = vi.fn().mockResolvedValue({});
		vi.mocked(useFetchDataMutation).mockImplementation(() => [
			mockFetchData,
			{
				isLoading: false,
				reset: vi.fn(),
				status: "idle",
				isError: false,
				isSuccess: false,
			},
		]);

		customRender(<FetchDataButton />);
		const button = screen.getByRole("button");
		expect(button).not.toBeDisabled();

		await userEvent.click(button);
		await waitFor(() => expect(mockFetchData).toHaveBeenCalled(1));
	});
});
```

### Testing Hooks

For testing hooks, we use `@testing-library/react`. This allows us to test custom hooks in isolation:

```tsx
import { expect } from "vitest";
import { renderHook } from "@testing-library/react";
import { useCounter } from "./useCounter";

describe("useCounter hook", () => {
	it("should increment counter", () => {
		const { result } = renderHook(() => useCounter());
		expect(result.current.count).toBe(0);

		result.current.increment();
		expect(result.current.count).toBe(1);
	});
});
```

### Testing RTK Slices

We test Redux Toolkit slices independently:

```tsx
import { counterSlice, increment, decrement } from "./counterSlice";

describe("Counter Slice", () => {
	it("should handle increment", () => {
		const initialState = { value: 0 };
		const nextState = counterSlice.reducer(initialState, increment());
		expect(nextState.value).toBe(1);
	});

	it("should handle decrement", () => {
		const initialState = { value: 1 };
		const nextState = counterSlice.reducer(initialState, decrement());
		expect(nextState.value).toBe(0);
	});
});
```

### Testing Views/Pages

We test view components focusing on their core rendering logic:

```tsx
import { render, screen } from "@testing-library/react";
import { Provider } from "react-redux";
import { store } from "../store";
import HomeView from "./HomeView";

describe("HomeView", () => {
	it("renders main sections", () => {
		render(
			<Provider store={store}>
				<HomeView />
			</Provider>
		);

		expect(
			screen.getByRole("heading", { name: /welcome/i })
		).toBeInTheDocument();
		expect(screen.getByTestId("main-content")).toBeInTheDocument();
	});
});
```

### Testing Store Actions

For testing store actions and reducers:

```tsx
import { configureStore } from "@reduxjs/toolkit";
import authReducer, { setUser, logout } from "./authSlice";

describe("Auth Slice", () => {
	let store;

	beforeEach(() => {
		store = configureStore({
			reducer: { auth: authReducer },
		});
	});

	it("should handle user login", () => {
		const user = { id: 1, name: "Test User" };
		store.dispatch(setUser(user));
		const state = store.getState().auth;
		expect(state.user).toEqual(user);
		expect(state.isAuthenticated).toBe(true);
	});

	it("should handle logout", () => {
		store.dispatch(logout());
		const state = store.getState().auth;
		expect(state.user).toBeNull();
		expect(state.isAuthenticated).toBe(false);
	});
});
```

### Testing Zustand Store

For testing Zustand stores:

```tsx
import { renderHook, act } from "@testing-library/react";
import useStore from "./store";

describe("Zustand Store", () => {
	beforeEach(() => {
		useStore.getState().reset();
	});

	it("should update theme", () => {
		const { result } = renderHook(() => useStore());
		act(() => {
			result.current.setTheme("dark");
		});
		expect(result.current.theme).toBe("dark");
	});
});
```

### Testing Form Components

Example of testing form components with validation:

```tsx
import { render, screen, fireEvent } from "@testing-library/react";
import LoginForm from "./LoginForm";

describe("LoginForm", () => {
	it("shows validation errors for empty fields", async () => {
		render(<LoginForm onSubmit={vi.fn()} />);

		const submitButton = screen.getByRole("button", { name: /submit/i });
		fireEvent.click(submitButton);

		expect(await screen.findByText(/email is required/i)).toBeInTheDocument();
		expect(
			await screen.findByText(/password is required/i)
		).toBeInTheDocument();
	});

	it("calls onSubmit with form data when valid", async () => {
		const mockSubmit = vi.fn();
		render(<LoginForm onSubmit={mockSubmit} />);

		fireEvent.change(screen.getByLabelText(/email/i), {
			target: { value: "test@example.com" },
		});
		fireEvent.change(screen.getByLabelText(/password/i), {
			target: { value: "password123" },
		});
		fireEvent.click(screen.getByRole("button", { name: /submit/i }));

		expect(mockSubmit).toHaveBeenCalledWith({
			email: "test@example.com",
			password: "password123",
		});
	});
});
```

### Mocking Strategies

We use Vitest's built-in mocking capabilities:

```tsx
// Mocking RTK Query endpoints
vi.mock("@/common/services/custom.api", () => ({
	useGetPointsQuery: vi.fn(() => ({
		data: {
			data: [],
		},
		isLoading: false,
		isError: false,
		isFetching: false,
		refetch: vi.fn(),
	})),
	useGetPointsQuery: vi.fn(() => ({
		data: {
			data: {
				records: [{ points: 100 }],
			},
		},
		isLoading: false,
		isError: false,
	})),
	useClaimPointsMutation: vi.fn(() => [
		vi.fn().mockResolvedValue({}),
		{
			isLoading: false,
			isError: false,
			reset: vi.fn(),
			status: "idle",
			isSuccess: false,
		},
	]),
	customApi: {
		reducerPath: "custom",
		reducer: () => ({}),
		middleware: () => () => {},
	},
}));

// Mocking modules
vi.mock("react-router-dom", () => ({
	useNavigate: () => vi.fn(),
}));
```

Configure Store

```tsx
import { MemoryRouter } from "react-router-dom";
import { Provider } from "react-redux";
import { configureStore } from "@reduxjs/toolkit";
import { customApi } from "@/common/services/custom.api";

const customRender = (ui: React.ReactElement) => {
	const store = configureStore({
		reducer: {
			[customApi.reducerPath]: customApi.reducer,
		},
		middleware: (getDefaultMiddleware) =>
			getDefaultMiddleware().concat(customApi.middleware),
	});
	return render(
		<MemoryRouter>
			<Provider store={store}>{ui}</Provider>
		</MemoryRouter>
	);
};
```

This example shows how to:

- Mock RTK Query hooks with their full response structure
- Mock query states (loading, error, etc.)
- Mock mutation hooks with their return tuple
- Mock API configuration objects

### Test Utilities

We've created helper files in `src/test-utils/`:

- `mockStore.ts` - Configures store for testing
- `testUtils.ts` - Common testing utilities

## Running Tests

```bash
# Run all tests
pnpm run test
# or: vitest

# Generate coverage report
npm run coverage
# or: vitest run --coverage

# Vitest UI
pnpm run vitest:ui
# or: vitest --ui
```

## Vitest UI

Vitest includes a powerful UI interface that helps visualize and debug tests. To use it:

```bash
pnpm run vitest:ui
# or: vitest --ui
```

### Features

#### Test Explorer

![Vitest UI Test Explorer](/assets/img/12/vitest_ui_example.png)

- Tree view of all test files
- Real-time test execution status
- Filter and search capabilities
- Click to run individual tests

#### Test Details

- Detailed test results
- Stack traces for failures
- Test execution time
- Code coverage visualization

#### Console Output

- View console.log outputs
- Error messages and stack traces
- Test runtime information

### Benefits

- Interactive debugging
- Real-time test feedback
- Visual code coverage
- Easy test filtering and search
- Better developer experience for test maintenance

## Coverage Configuration

In `vitest.config.js`:

```ts
export default defineConfig({
	test: {
		globals: true,
		environment: "jsdom",
		includeTaskLocation: true,
		coverage: {
			provider: "v8",
			reporter: ["text", "json", "html"],
			include: ["src/**/*.{ts,tsx}"],
			exclude: ["**/*.stories.tsx", "**/*.d.ts", "src/main.tsx", "**/index.ts"],
			thresholds: {
				lines: 80,
				functions: 80,
				branches: 80,
				statements: 80,
			},
		},
});
```

## Testing Standards

- Write unit tests for individual components and functions
- Test state management logic in isolation (RTK, Zustand, etc)
- Maintain high coverage thresholds
- Keep tests simple and focused
