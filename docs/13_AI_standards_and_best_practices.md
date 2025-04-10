---
title: AI Rules | Development Standards & Best Practices
layout: default
nav_order: 13
---

# React Development Standards & Best Practices

## Code Quality & Maintainability

- **Type Safety**

  - Use TypeScript for all new components
  - Avoid `any` type (enable `tsconfig.json` `noImplicitAny`)
  - Define types/interfaces in the same file as components when possible

- **Security**

  - Never hardcode sensitive keys/credentials
  - Use environment variables (`.env` files) with proper `.gitignore`
  - Validate all external inputs and API responses

- **Component Design**
  - Single Responsibility Principle (one component = one purpose)
  - Keep component size < 300 lines (split larger components)
  - Prefer functional components with hooks
  - Memoize expensive computations with `useMemo`

## Component Implementation

- **Props Management**

  ```typescript
  // Good
  interface ButtonProps {
    variant: 'primary' | 'secondary';
    onClick: () => void;
  }

  const Button = ({ variant, onClick }: ButtonProps) => (...)

  // Bad (types separate)
  type ButtonProps = any;
  ```

- **State Management**

  - Prefer Zustand for global state management
  - Use React Context or Zustand only for cross-component state within a feature
  - Avoid prop drilling through component composition
  - Keep form state and UI state localized when possible

  ```tsx
  // Zustand store example
  import { create } from "zustand";

  interface AuthStore {
  	user: User | null;
  	setUser: (user: User) => void;
  	logout: () => void;
  }

  const useAuthStore = create<AuthStore>((set) => ({
  	user: null,
  	setUser: (user) => set({ user }),
  	logout: () => set({ user: null }),
  }));

  // Context usage example
  const ThemeContext = create<"light" | "dark">("light");
  export const useTheme = () => useContext(ThemeContext);
  ```

- **API Handling**

  - Use React Query as primary solution
  - Fallback to RTK Query if using Redux
  - Axios for simple cases with interceptors
  - Cache management through query client only

  ```typescript
  // Preferred stack
  const queryClient = new QueryClient({
  	defaultOptions: {
  		queries: {
  			staleTime: 5 * 60 * 1000, // 5 minutes
  		},
  	},
  });

  // Axios instance with interceptors
  const api = axios.create({
  	baseURL: import.meta.env.VITE_API_URL,
  });

  api.interceptors.request.use((config) => {
  	const token = localStorage.getItem("access_token");
  	if (token) config.headers.Authorization = `Bearer ${token}`;
  	return config;
  });
  ```

  ```tsx
  // React Query example
  import { useQuery } from "@tanstack/react-query";

  const fetchUser = async (id: string) => {
  	const { data } = await api.get(`/users/${id}`);
  	return data;
  };

  export const useUser = (id: string) => {
  	return useQuery({
  		queryKey: ["user", id],
  		queryFn: () => fetchUser(id),
  		staleTime: 5 * 60 * 1000,
  		retry: 2,
  	});
  };

  // Axios interceptor example
  api.interceptors.response.use(
  	(response) => response,
  	(error) => {
  		if (error.response?.status === 401) {
  			window.location.href = "/login";
  		}
  		return Promise.reject(error);
  	}
  );
  ```

- **Query Invalidation**

  ```tsx
  // React Query cache invalidation
  const mutation = useMutation({
  	mutationFn: updateUser,
  	onSuccess: () => {
  		queryClient.invalidateQueries({ queryKey: ["users"] });
  	},
  });

  // Optimistic updates
  useMutation({
  	mutationFn: updateUser,
  	onMutate: async (newUser) => {
  		await queryClient.cancelQueries({ queryKey: ["user", newUser.id] });
  		const previousUser = queryClient.getQueryData(["user", newUser.id]);
  		queryClient.setQueryData(["user", newUser.id], newUser);
  		return { previousUser };
  	},
  	onError: (err, newUser, context) => {
  		queryClient.setQueryData(["user", newUser.id], context.previousUser);
  	},
  });
  ```

## Styling

- **CSS Practices**

  - Use TailwindCSS as primary styling solution
  - Material UI only for existing implementations
  - Mobile-first responsive design approach
  - Use CSS variables for dynamic theming

  ```tsx
  // Good tailwind usage
  <div className="flex flex-col md:flex-row gap-4 p-4">{children}</div>
  ```

- **Tailwind Best Practices**

  ```tsx
  // Responsive card component
  const Card = ({ children }: { children: ReactNode }) => (
  	<div className="bg-white dark:bg-gray-800 rounded-lg shadow-sm p-4 md:p-6 lg:p-8 transition-colors duration-300">
  		{children}
  	</div>
  );

  // Loading skeleton
  const SkeletonLoader = () => (
  	<div className="animate-pulse space-y-4">
  		<div className="h-4 bg-gray-200 rounded w-3/4"></div>
  		<div className="h-4 bg-gray-200 rounded"></div>
  	</div>
  );
  ```

## Testing & Quality

- **Unit Testing**

  - 100% test coverage for utility/helper functions
  - Core component functionality must be tested
  - Use React Testing Library with Vitest
  - Test user flows, not implementation details

## Performance

- **Memoization & Optimization**

  - Profile with React DevTools
  - Avoid unnecessary re-renders
  - Virtualize long lists

  ```tsx
  // Memoized component
  const ExpensiveList = React.memo(({ items }: { items: Item[] }) => (
  	<ul>
  		{items.map((item) => (
  			<ListItem key={item.id} item={item} />
  		))}
  	</ul>
  ));

  // useCallback example
  const SearchForm = () => {
  	const [query, setQuery] = useState("");

  	const handleSearch = useCallback(
  		debounce((searchTerm: string) => {
  			searchAPI(searchTerm);
  		}, 300),
  		[]
  	);

  	return <input onChange={(e) => handleSearch(e.target.value)} />;
  };
  ```

  - Implement image CDN with auto-format/compression
  - Lazy load non-critical assets
  - Cache static assets with service workers

## Code Structure

- **File Organization**
  - Follow atomic design pattern
  - Group by feature, not type
  - Keep tests colocated (`Component.test.tsx`)

## Dependency Management

- **Package Rules**
  - Audit dependencies monthly
  - Prefer React ecosystem libraries
  - Limit global state management to one solution

## Documentation

- **Project Documentation**

  - Architecture decision records (ADR)
  - Tech stack justification in `/docs/tech-stack.md`
  - Setup guide with PNPM commands
  - State management flow diagrams

- [Optional] **Component Documentation**
  ```tsx
  /**
   * @component AsyncButton
   * @description Button with loading state for async operations
   * @prop {ReactNode} children - Button content
   * @prop {() => Promise<void>} onClick - Async click handler
   * @prop {boolean} isLoading - Loading state
   * @example
   * <AsyncButton
   *   onClick={handleSubmit}
   *   isLoading={isSubmitting}
   * >
   *   Submit
   * </AsyncButton>
   */
  export const AsyncButton = ({ children, onClick, isLoading }) => (
  	<button
  		onClick={onClick}
  		disabled={isLoading}
  		className={`px-4 py-2 ${isLoading ? "bg-gray-400" : "bg-blue-600"}`}
  	>
  		{isLoading ? <Spinner /> : children}
  	</button>
  );
  ```

## Pipeline Enforcement

- Required checks:

  ```yaml
  - Vitest (min 80% branch coverage)
  - ESLint (typescript-react-preset)
  - TypeScript strict mode
  - Bundle size limit (150kb gzip)
  - Lighthouse CI (performance >= 90)
  - Dependency audit (pnpm audit)
  ```

- **Vitest Configuration**
  ```tsx
  // vitest.config.ts
  export default defineConfig({
  	test: {
  		environment: "jsdom",
  		coverage: {
  			provider: "v8",
  			reporter: ["text", "json", "html"],
  			thresholds: {
  				lines: 85,
  				functions: 85,
  				branches: 85,
  				statements: 85,
  			},
  		},
  	},
  });
  ```

## Development Workflow

- **Package Manager**

  - Prefer PNPM (commit `pnpm-lock.yaml`)
  - Yarn acceptable for legacy projects

- **Responsive Requirements**
  - Loading skeletons for async content
  - Optimistic UI updates
  - Graceful error boundaries

## Image Optimization

- **Image Optimization**
  ```tsx
  // Image component with optimizations
  <img
  	src="/image.jpg"
  	alt="Description"
  	loading="lazy"
  	className="w-full h-auto"
  	srcSet="/image-320w.jpg 320w,
            /image-640w.jpg 640w,
            /image-1024w.jpg 1024w"
  	sizes="(max-width: 640px) 100vw, 50vw"
  />
  ```

## Responsive Requirements

- **Breakpoints**

  ```scss
  // Tailwind default breakpoints
  sm: 640px  // Small
  md: 768px  // Medium
  lg: 1024px // Large
  xl: 1280px // Extra large
  ```

## Component Testing Strategy

```tsx
// Component test example
import { render, screen } from "@testing-library/react";
import { describe, expect, it } from "vitest";

describe("Button component", () => {
	it("renders with correct variant", () => {
		render(<Button variant="primary">Click</Button>);
		expect(screen.getByRole("button")).toHaveClass("bg-primary");
	});
});

// Hook test example
import { renderHook, waitFor } from "@testing-library/react";
import { useUser } from "./userHooks";

describe("useUser hook", () => {
	it("fetches user data", async () => {
		const { result } = renderHook(() => useUser("123"));
		await waitFor(() => expect(result.current.isSuccess).toBe(true));
	});
});
```

## Component Composition

```tsx
// Avoid prop drilling
const UserProfile = ({ user }) => (
	<Card>
		<Avatar user={user} />
	</Card>
);

// Better composition
const UserProfile = ({ children }) => <Card>{children}</Card>;

// Usage
<UserProfile>
	<Avatar user={user} />
</UserProfile>;
```

## Accessibility

- **WCAG Compliance**

  - All interactive elements must have keyboard navigation
  - ARIA labels for icon buttons
  - Color contrast ratio ≥ 4.5:1
  - Semantic HTML structure

  ```tsx
  // Good accessibility practice
  <button
    aria-label="Close modal"
    className="p-2 hover:bg-gray-100"
    onClick={onClose}
  >
    <XIcon className="w-5 h-5" />
  </button>

  // Loading state accessibility
  <div
    role="alert"
    aria-live="polite"
    className={isLoading ? 'block' : 'hidden'}
  >
    Loading results...
  </div>
  ```

## Lighthouse Requirements

```yaml
Performance: 90
Accessibility: 95
Best Practices: 95
SEO: 90
PWA: 70
```

## Additional Best Practices

### SOLID Principles

- **Single Responsibility Principle (SRP):** Each function should have a single responsibility. Do not mix business logic, UI manipulation, and event handling in the same function.
- **Open/Closed Principle (OCP):** Components should be extendable without modifying their base code. Example: A `Form` should not have the `Button` embedded; instead, the `Button` should be an independent component.

### Avoid Repetition (DRY)

- If you detect repeated code (components, functions, or hooks), extract it into a **custom hook**, **reusable component**, or **independent utility**.
- **Example**:
```tsx
  // Bad: Repeated logic in multiple components
  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(setData);
  }, []);

  // Better: Extracting to a reusable hook
  const useFetchData = (url: string) => {
    const [data, setData] = useState(null);
    useEffect(() => {
      fetch(url)
        .then(res => res.json())
        .then(setData);
    }, [url]);
    return data;
  };

  const Component = () => {
    const data = useFetchData('/api/data');
  };
```

# Descriptive Variable Names

- Variable names should be clear and descriptive.

## Correct Examples:

- Use `isLoading` instead of `loading`
- Use `userList` instead of `users`
- Use `modalRef` if using `useRef`
- Use `handleSubmit` for a submit function instead of `submit`

# State Updates Should Use `setState(prevState => {...})`

- Avoid modifying state directly without considering the previous state.

### Incorrect Example:

```tsx
setCounter(counter + 1);
```

### Correct  Example:

```tsx
setCounter(prevCounter => prevCounter + 1);
```