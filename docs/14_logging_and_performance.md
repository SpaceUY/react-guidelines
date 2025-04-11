# Logging & Performance with Sentry

This guide outlines how to implement Sentry in React SPAs for error tracking, performance monitoring, and issue identification.

## Setup

### 1. Installation

```bash
pnpm add @sentry/react
```

### 2. Basic Configuration

```jsx
// main.jsx or index.jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import * as Sentry from "@sentry/react";

Sentry.init({
	dsn: "YOUR_DSN_HERE",
	integrations: [
		new Sentry.BrowserTracing({
			// Set sampling rate for performance monitoring
			// This sets 10% of transactions to be captured
			tracesSampleRate: 0.1,
		}),
		new Sentry.Replay({
			// Capture 10% of all sessions for replay
			sessionSampleRate: 0.1,
			// Increase sampling to 100% of error sessions
			errorSampleRate: 1.0,
		}),
	],
	// Control which release version the errors are associated with
	release: "my-app@1.0.0",
	// Environment helps separate issues by deployment
	environment: "production", // or "development", "staging", etc.
});

ReactDOM.createRoot(document.getElementById("root")).render(
	<React.StrictMode>
		<App />
	</React.StrictMode>
);
```

### 3. Error Boundary

While Sentry will capture unhandled exceptions without error boundaries, using them provides several benefits:

- Prevents the entire application from crashing when a component error occurs
- Provides a graceful fallback UI for users
- Helps Sentry capture React-specific rendering errors with proper component context
- Limits the error impact to only the affected component tree

Wrap your components with Sentry's Error Boundary:

```jsx
// App.jsx
import React from "react";
import * as Sentry from "@sentry/react";
import MainContent from "./components/MainContent";

const FallbackComponent = () => (
	<div className="error-boundary">
		<h2>Something went wrong</h2>
		<p>We've been notified and are working on a fix</p>
		<button onClick={() => window.location.reload()}>Refresh the page</button>
	</div>
);

function App() {
	return (
		<Sentry.ErrorBoundary fallback={FallbackComponent}>
			<MainContent />
		</Sentry.ErrorBoundary>
	);
}

export default App;
```

You can also use error boundaries strategically around specific error-prone components rather than wrapping the entire application.

## Error Tracking & Logging

### 1. Automatic Error Tracking

Sentry automatically captures:

- Unhandled exceptions
- Promise rejections
- Network errors
- React rendering errors (via Error Boundary)

### 2. Manual Error Capturing

For try/catch blocks and custom error handling:

```jsx
try {
	// Some risky operation
	const data = await riskyOperation();
	return data;
} catch (error) {
	Sentry.captureException(error);
	// Show user-friendly error or fallback
}
```

### 3. Custom Messages and Context

Add custom information to errors:

```jsx
// Add extra context that will be available for all future events
Sentry.configureScope((scope) => {
	scope.setTag("feature", "checkout");
	scope.setUser({
		id: "user-123",
		email: "user@example.com",
	});
});

// Add context to a specific event
try {
	processOrder(orderId);
} catch (error) {
	Sentry.captureException(error, {
		tags: { orderId: "1234" },
		extra: { productDetails: productData },
	});
}
```

### 4. Standardized Error Reporting Utility

Creating a centralized error reporting utility can help standardize error handling throughout your application while combining Sentry reporting with local logging:

```ts
// src/utils/errorReporting.ts
import * as Sentry from "@sentry/react";
import logger from "./logger";

interface ErrorContext {
	component: string;
	endpoint?: string;
	extra?: Record<string, unknown>;
}

export const errorReporting = {
	/**
	 * Report an error with both Sentry and local logger
	 */
	captureError: (error: unknown, context: ErrorContext) => {
		// Log to Sentry
		Sentry.captureException(error, {
			tags: {
				component: context.component,
				...(context.endpoint && { endpoint: context.endpoint }),
			},
			...(context.extra && { extra: context.extra }),
		});

		// Log locally in development
		logger.error(error, context.component);
	},

	/**
	 * Report a message with custom level
	 */
	captureMessage: (
		message: string,
		level: Sentry.SeverityLevel,
		context: ErrorContext
	) => {
		// Log to Sentry
		Sentry.captureMessage(message, {
			level,
			tags: {
				component: context.component,
				...(context.endpoint && { endpoint: context.endpoint }),
			},
			...(context.extra && { extra: context.extra }),
		});

		// Log locally in development
		switch (level) {
			case "error":
				logger.error(message, context.component);
				break;
			case "warning":
				logger.warn(message, context.component);
				break;
			case "info":
				logger.info(message, context.component);
				break;
			case "debug":
				logger.debug(message, context.component);
				break;
			default:
				logger.info(message, context.component);
		}
	},
};
```

Example usage:

```tsx
// In a component
import { errorReporting } from "../utils/errorReporting";

const UserProfile = () => {
	const fetchUserData = async (userId: string) => {
		try {
			const response = await fetch(`/api/users/${userId}`);
			if (!response.ok) {
				throw new Error(`Failed to fetch user: ${response.statusText}`);
			}
			return await response.json();
		} catch (error) {
			errorReporting.captureError(error, {
				component: "UserProfile",
				endpoint: `/api/users/${userId}`,
				extra: { userId }
			});
			return null;
		}
	};

	// Log a non-error message
	const handleUserAction = () => {
		errorReporting.captureMessage(
			"User updated their profile picture",
			"info",
			{ component: "UserProfile" }
		);
	};

	return (
		// Component JSX
	);
};
```

This utility ensures consistent error reporting across your application while maintaining detailed context about where and when errors occur.

## Using the Sentry Dashboard

### 1. Identifying Issues

The Sentry dashboard organizes errors by:

- Frequency and impact (how many users affected)
- Environment (dev, staging, production)
- Release version
- Browser/device information

Key features:

- Stack traces showing where errors occurred
- React component tree at the time of error
- User session information
- Source maps integration for readable code

### 2. Issue Replication

Sentry Replay provides session replay capabilities to see exactly what users experienced:

- Record user interactions leading up to errors
- Capture DOM changes
- View console logs
- Track network requests

To enable detailed breadcrumbs:

```jsx
Sentry.addBreadcrumb({
	category: "ui.click",
	message: "User clicked checkout button",
	level: "info",
});
```

## Performance Monitoring

### 1. Transaction Tracking

Monitor specific operations:

```jsx
// Manual performance transaction
const transaction = Sentry.startTransaction({
	name: "checkout-process",
	op: "purchase",
});

Sentry.configureScope((scope) => {
	scope.setSpan(transaction);
});

try {
	// Process checkout
	await validateCart();
	await processPayment();
	await generateOrder();

	transaction.finish();
} catch (error) {
	transaction.finish();
	Sentry.captureException(error);
}
```

### 2. React Component Profiling

Monitor component render performance:

```jsx
import { withProfiler } from "@sentry/react";

function ExpensiveComponent() {
	// Component that might cause performance issues
}

export default withProfiler(ExpensiveComponent);
```

### 3. Uptime Monitoring

Sentry provides uptime monitoring via:

- Error rates
- Transaction success rates
- Response time thresholds
- Alerts on performance degradation

Configure alerts in the Sentry dashboard for:

- Spike in error rates
- Performance regression
- Failed transactions
- Custom metric thresholds

## Best Practices

1. **Use source maps** in production (securely) for accurate debugging
2. **Set appropriate sampling rates** to balance cost with coverage
3. **Tag and categorize errors** for easier filtering
4. **Add user context when possible** for better debugging
5. **Set up issue ownership** to route issues to the right team
6. **Configure alert thresholds** based on application needs
7. **Use release tracking** to identify which deployments introduced issues
8. **Clean up or silence noisy errors** that don't need attention

## Security Considerations

1. Never log personally identifiable information (PII) without proper policies
2. Use allowUrls and denyUrls in Sentry config to control what errors are reported
3. Sanitize any custom data sent with error reports
4. Consider lower sampling rates for sensitive areas of the application
