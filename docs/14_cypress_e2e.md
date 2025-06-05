---
title: E2E Testing with Cypress
layout: default
nav_order: 14
---

# E2E Testing with Cypress

## Core Packages Versions

- cypress: 13.15.2
- @cypress/vite-dev-server: 5.2.0
- @cypress/react: 8.0.2
- typescript: ~5.7.2

## End-to-End Testing

We use Cypress for end-to-end testing to validate complete user workflows and frontend behavior. E2E tests simulate real user interactions in a browser environment, focusing on the complete user experience from start to finish.

Key principles:
- Test complete user journeys and workflows
- Focus on user interactions and frontend behavior
- Validate UI state changes and visual feedback
- Test responsive design and accessibility
- Ensure seamless navigation flows

## Installation

To get up and running with Cypress E2E Testing in a Vite React TypeScript project:

```bash
# Install Cypress
npm install cypress --save-dev
# or
pnpm add cypress --save-dev

# Install TypeScript types for Cypress
npm install @types/cypress --save-dev
# or
pnpm add @types/cypress --save-dev

# Open Cypress for first-time setup
npx cypress open
# or
pnpm cypress open
```

Choose **E2E Testing** during the setup process.

## Configuration

### Cypress Configuration for Vite + TypeScript

Create `cypress.config.ts` in your project root:

```typescript
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:5173', // Vite dev server default port
    setupNodeEvents(on, config) {
      // implement node event listeners here
    },
    specPattern: 'cypress/e2e/**/*.cy.{ts,tsx}',
    supportFile: 'cypress/support/e2e.ts',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: false, // Set to true for CI/CD
    screenshotOnRunFailure: true,
    defaultCommandTimeout: 10000,
    requestTimeout: 10000,
    responseTimeout: 10000,
    experimentalStudio: true,
  },
})
```

### TypeScript Configuration

Create `cypress/tsconfig.json`:

```json
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM"],
    "types": ["cypress", "node"],
    "isolatedModules": false,
    "skipLibCheck": true
  },
  "include": [
    "**/*.ts",
    "**/*.tsx"
  ],
  "exclude": []
}
```

### Package.json Scripts

Add these scripts to your `package.json`:

```json
{
  "scripts": {
    "cypress:open": "cypress open",
    "cypress:run": "cypress run",
    "cypress:run:headless": "cypress run --headless",
    "test:e2e": "start-server-and-test dev http://localhost:5173 cypress:run",
    "test:e2e:open": "start-server-and-test dev http://localhost:5173 cypress:open"
  },
  "devDependencies": {
    "start-server-and-test": "^2.0.3",
    "@types/cypress": "^1.1.3"
  }
}
```

## Directory Structure

```
cypress/
├── e2e/
│   ├── auth/
│   │   └── login.cy.ts
│   ├── users/
│   │   ├── create-user.cy.ts
│   │   ├── edit-user.cy.ts
│   │   └── delete-user.cy.ts
│   └── forms/
│       └── step-form.cy.ts
├── fixtures/
│   └── users.json
├── support/
│   ├── commands.ts
│   ├── e2e.ts
│   ├── index.d.ts
│   └── pages/
│       ├── LoginPage.ts
│       ├── UserFormPage.ts
│       └── DashboardPage.ts
├── downloads/
└── tsconfig.json
```

## TypeScript Types and Interfaces

Create type definitions in `cypress/support/types.ts`:

```typescript
// cypress/support/types.ts
export interface UserData {
  firstName: string
  lastName: string
  email: string
  password: string
  phoneNumber?: string
}

export interface FormData {
  [key: string]: string | number | boolean
}

export interface NavigationItem {
  label: string
  url: string
  testId: string
}

export interface UIState {
  isLoading: boolean
  isVisible: boolean
  isDisabled: boolean
  hasError: boolean
}
```

## Simple CRUD Flow Tests

### 1. Login Flow

```typescript
// cypress/e2e/auth/login.cy.ts
describe('Login Flow', () => {
  beforeEach(() => {
    cy.visit('/login')
  })

  it('should login and navigate to dashboard', () => {
    // Fill login form
    cy.get('[data-testid="email-input"]').type('admin@example.com')
    cy.get('[data-testid="password-input"]').type('password123')
    cy.get('[data-testid="login-button"]').click()
    
    // Verify redirect to dashboard
    cy.url().should('include', '/dashboard')
    cy.get('[data-testid="welcome-message"]').should('be.visible')
    cy.get('[data-testid="create-user-button"]').should('be.visible')
  })

  it('should show validation errors', () => {
    cy.get('[data-testid="login-button"]').click()
    
    cy.get('[data-testid="email-error"]').should('contain', 'Email is required')
    cy.get('[data-testid="password-error"]').should('contain', 'Password is required')
  })
})
```

### 2. Complete User Creation Flow

```typescript
// cypress/e2e/users/create-user.cy.ts
describe('Create User Flow', () => {
  beforeEach(() => {
    // Login first
    cy.login('admin@example.com', 'password123')
    cy.visit('/dashboard')
  })

  it('should create a new user through step form', () => {
    // Start user creation
    cy.get('[data-testid="create-user-button"]').click()
    cy.url().should('include', '/users/create')
    
    // Step 1: Basic Information
    cy.get('[data-testid="step-indicator"]').should('contain', 'Step 1 of 3')
    cy.get('[data-testid="first-name-input"]').type('John')
    cy.get('[data-testid="last-name-input"]').type('Doe')
    cy.get('[data-testid="email-input"]').type('john.doe@example.com')
    cy.get('[data-testid="next-button"]').click()
    
    // Step 2: Contact Information
    cy.get('[data-testid="step-indicator"]').should('contain', 'Step 2 of 3')
    cy.get('[data-testid="phone-input"]').type('+1234567890')
    cy.get('[data-testid="address-input"]').type('123 Main St')
    cy.get('[data-testid="city-input"]').type('New York')
    cy.get('[data-testid="country-select"]').select('United States')
    cy.get('[data-testid="next-button"]').click()
    
    // Step 3: Role and Permissions
    cy.get('[data-testid="step-indicator"]').should('contain', 'Step 3 of 3')
    cy.get('[data-testid="role-select"]').select('User')
    cy.get('[data-testid="permissions-admin"]').check()
    cy.get('[data-testid="permissions-read"]').check()
    
    // Submit form
    cy.get('[data-testid="submit-button"]').click()
    
    // Verify success
    cy.get('[data-testid="success-message"]')
      .should('be.visible')
      .and('contain', 'User created successfully')
    
    // Verify redirect to users list
    cy.url().should('include', '/users')
    cy.get('[data-testid="users-table"]').should('contain', 'john.doe@example.com')
  })

  it('should validate required fields in each step', () => {
    cy.get('[data-testid="create-user-button"]').click()
    
    // Step 1 validation
    cy.get('[data-testid="next-button"]').click()
    cy.get('[data-testid="first-name-error"]').should('be.visible')
    cy.get('[data-testid="last-name-error"]').should('be.visible')
    cy.get('[data-testid="email-error"]').should('be.visible')
    
    // Fill step 1 and move to step 2
    cy.get('[data-testid="first-name-input"]').type('John')
    cy.get('[data-testid="last-name-input"]').type('Doe')
    cy.get('[data-testid="email-input"]').type('john@example.com')
    cy.get('[data-testid="next-button"]').click()
    
    // Step 2 validation
    cy.get('[data-testid="next-button"]').click()
    cy.get('[data-testid="phone-error"]').should('be.visible')
    cy.get('[data-testid="address-error"]').should('be.visible')
  })

  it('should allow navigation between steps', () => {
    cy.get('[data-testid="create-user-button"]').click()
    
    // Fill step 1
    cy.get('[data-testid="first-name-input"]').type('John')
    cy.get('[data-testid="last-name-input"]').type('Doe')
    cy.get('[data-testid="email-input"]').type('john@example.com')
    cy.get('[data-testid="next-button"]').click()
    
    // Go back to step 1
    cy.get('[data-testid="back-button"]').click()
    cy.get('[data-testid="step-indicator"]').should('contain', 'Step 1 of 3')
    
    // Verify data is preserved
    cy.get('[data-testid="first-name-input"]').should('have.value', 'John')
    cy.get('[data-testid="last-name-input"]').should('have.value', 'Doe')
    cy.get('[data-testid="email-input"]').should('have.value', 'john@example.com')
  })
})
```

### 3. Edit User Flow

```typescript
// cypress/e2e/users/edit-user.cy.ts
describe('Edit User Flow', () => {
  beforeEach(() => {
    cy.login('admin@example.com', 'password123')
    cy.visit('/users')
  })

  it('should edit an existing user', () => {
    // Find and edit first user
    cy.get('[data-testid="user-row"]').first().within(() => {
      cy.get('[data-testid="edit-button"]').click()
    })
    
    cy.url().should('include', '/users/edit/')
    
    // Verify form is pre-filled
    cy.get('[data-testid="first-name-input"]').should('not.have.value', '')
    cy.get('[data-testid="email-input"]').should('not.have.value', '')
    
    // Edit user information
    cy.get('[data-testid="first-name-input"]').clear().type('Jane')
    cy.get('[data-testid="phone-input"]').clear().type('+9876543210')
    
    // Save changes
    cy.get('[data-testid="save-button"]').click()
    
    // Verify success
    cy.get('[data-testid="success-message"]').should('contain', 'User updated successfully')
    cy.url().should('include', '/users')
    cy.get('[data-testid="users-table"]').should('contain', 'Jane')
  })
})
```

### 4. Delete User Flow

```typescript
// cypress/e2e/users/delete-user.cy.ts
describe('Delete User Flow', () => {
  beforeEach(() => {
    cy.login('admin@example.com', 'password123')
    cy.visit('/users')
  })

  it('should delete a user with confirmation', () => {
    // Count initial users
    cy.get('[data-testid="user-row"]').its('length').as('initialCount')
    
    // Delete first user
    cy.get('[data-testid="user-row"]').first().within(() => {
      cy.get('[data-testid="user-name"]').invoke('text').as('userName')
      cy.get('[data-testid="delete-button"]').click()
    })
    
    // Confirm deletion in modal
    cy.get('[data-testid="confirm-modal"]').should('be.visible')
    cy.get('@userName').then((userName) => {
      cy.get('[data-testid="confirm-text"]').should('contain', userName)
    })
    cy.get('[data-testid="confirm-delete-button"]').click()
    
    // Verify user is deleted
    cy.get('[data-testid="success-message"]').should('contain', 'User deleted successfully')
    
    cy.get('@initialCount').then((initialCount) => {
      cy.get('[data-testid="user-row"]').should('have.length', Number(initialCount) - 1)
    })
  })

  it('should cancel deletion', () => {
    cy.get('[data-testid="user-row"]').its('length').as('initialCount')
    
    cy.get('[data-testid="user-row"]').first().within(() => {
      cy.get('[data-testid="delete-button"]').click()
    })
    
    // Cancel deletion
    cy.get('[data-testid="cancel-button"]').click()
    cy.get('[data-testid="confirm-modal"]').should('not.exist')
    
    // Verify count unchanged
    cy.get('@initialCount').then((initialCount) => {
      cy.get('[data-testid="user-row"]').should('have.length', initialCount)
    })
  })
})
```

## Custom Commands

```typescript
// cypress/support/commands.ts
declare global {
  namespace Cypress {
    interface Chainable {
      login(email: string, password: string): Chainable<void>
      fillUserForm(userData: {
        firstName: string
        lastName: string
        email: string
        phone?: string
        address?: string
        city?: string
        role?: string
      }): Chainable<void>
      goToStep(stepNumber: number): Chainable<void>
    }
  }
}

Cypress.Commands.add('login', (email: string, password: string) => {
  cy.session([email, password], () => {
    cy.visit('/login')
    cy.get('[data-testid="email-input"]').type(email)
    cy.get('[data-testid="password-input"]').type(password)
    cy.get('[data-testid="login-button"]').click()
    cy.url().should('include', '/dashboard')
  })
})

Cypress.Commands.add('fillUserForm', (userData) => {
  if (userData.firstName) {
    cy.get('[data-testid="first-name-input"]').type(userData.firstName)
  }
  if (userData.lastName) {
    cy.get('[data-testid="last-name-input"]').type(userData.lastName)
  }
  if (userData.email) {
    cy.get('[data-testid="email-input"]').type(userData.email)
  }
  if (userData.phone) {
    cy.get('[data-testid="phone-input"]').type(userData.phone)
  }
  if (userData.address) {
    cy.get('[data-testid="address-input"]').type(userData.address)
  }
  if (userData.city) {
    cy.get('[data-testid="city-input"]').type(userData.city)
  }
  if (userData.role) {
    cy.get('[data-testid="role-select"]').select(userData.role)
  }
})

Cypress.Commands.add('goToStep', (stepNumber: number) => {
  // Navigate to specific step in multi-step form
  for (let i = 1; i < stepNumber; i++) {
    cy.get('[data-testid="next-button"]').click()
  }
  cy.get('[data-testid="step-indicator"]').should('contain', `Step ${stepNumber}`)
})
```

## Page Objects (Simplified)

```typescript
// cypress/support/pages/UserFormPage.ts
export class UserFormPage {
  fillBasicInfo(firstName: string, lastName: string, email: string) {
    cy.get('[data-testid="first-name-input"]').type(firstName)
    cy.get('[data-testid="last-name-input"]').type(lastName)
    cy.get('[data-testid="email-input"]').type(email)
    return this
  }

  fillContactInfo(phone: string, address: string, city: string) {
    cy.get('[data-testid="phone-input"]').type(phone)
    cy.get('[data-testid="address-input"]').type(address)
    cy.get('[data-testid="city-input"]').type(city)
    return this
  }

  selectRole(role: string) {
    cy.get('[data-testid="role-select"]').select(role)
    return this
  }

  nextStep() {
    cy.get('[data-testid="next-button"]').click()
    return this
  }

  submit() {
    cy.get('[data-testid="submit-button"]').click()
    return this
  }

  verifySuccess() {
    cy.get('[data-testid="success-message"]').should('be.visible')
    return this
  }
}

// Usage example
describe('User Form with Page Object', () => {
  it('should create user using page object', () => {
    const userForm = new UserFormPage()
    
    cy.login('admin@example.com', 'password123')
    cy.visit('/users/create')
    
    userForm
      .fillBasicInfo('John', 'Doe', 'john@example.com')
      .nextStep()
      .fillContactInfo('+1234567890', '123 Main St', 'New York')
      .nextStep()
      .selectRole('User')
      .submit()
      .verifySuccess()
  })
})
```

## Form Validation Examples

```typescript
// cypress/e2e/forms/step-form.cy.ts
describe('Step Form Validation', () => {
  beforeEach(() => {
    cy.login('admin@example.com', 'password123')
    cy.visit('/users/create')
  })

  it('should validate email format', () => {
    cy.get('[data-testid="email-input"]').type('invalid-email')
    cy.get('[data-testid="next-button"]').click()
    
    cy.get('[data-testid="email-error"]')
      .should('be.visible')
      .and('contain', 'Please enter a valid email')
  })

  it('should validate phone format', () => {
    // Complete step 1
    cy.fillUserForm({
      firstName: 'John',
      lastName: 'Doe',
      email: 'john@example.com'
    })
    cy.get('[data-testid="next-button"]').click()
    
    // Test invalid phone in step 2
    cy.get('[data-testid="phone-input"]').type('invalid-phone')
    cy.get('[data-testid="next-button"]').click()
    
    cy.get('[data-testid="phone-error"]')
      .should('be.visible')
      .and('contain', 'Please enter a valid phone number')
  })

  it('should show loading state during submission', () => {
    // Fill all steps
    cy.fillUserForm({
      firstName: 'John',
      lastName: 'Doe',
      email: 'john@example.com'
    })
    cy.get('[data-testid="next-button"]').click()
    
    cy.fillUserForm({
      phone: '+1234567890',
      address: '123 Main St',
      city: 'New York'
    })
    cy.get('[data-testid="next-button"]').click()
    
    cy.get('[data-testid="role-select"]').select('User')
    
    // Submit and verify loading state
    cy.get('[data-testid="submit-button"]').click()
    cy.get('[data-testid="submit-button"]').should('be.disabled')
    cy.get('[data-testid="loading-spinner"]').should('be.visible')
  })
})
```

## Test Data

```json
// cypress/fixtures/users.json
{
  "validUser": {
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@example.com",
    "phone": "+1234567890",
    "address": "123 Main St",
    "city": "New York",
    "role": "User"
  },
  "adminUser": {
    "firstName": "Admin",
    "lastName": "User",
    "email": "admin@example.com",
    "phone": "+9876543210",
    "address": "456 Admin Ave",
    "city": "Admin City",
    "role": "Admin"
  }
}
```

## Running Tests

```bash
# Open Cypress Test Runner
npm run cypress:open

# Run tests headlessly
npm run cypress:run

# Run specific test file
npm run cypress run --spec "cypress/e2e/users/create-user.cy.ts"

# Run all user CRUD tests
npm run cypress run --spec "cypress/e2e/users/**/*.cy.ts"
```

## Best Practices for Simple CRUD Testing

### Test Organization
- One test file per main operation (create, edit, delete)
- Group related tests in describe blocks
- Use clear, descriptive test names

### Data Management
- Use `cy.session()` for login to avoid repeated authentication
- Create test data in fixtures for reusability
- Clean up test data when necessary

### Form Testing
- Test each step of multi-step forms independently
- Verify form validation on each field
- Test navigation between form steps
- Verify data persistence across steps

### Assertions
- Focus on user-visible outcomes
- Verify success/error messages
- Check URL changes after operations
- Validate data appears in lists/tables

### Selectors
- Always use `data-testid` attributes
- Make selectors specific and stable
- Avoid CSS class or text-based selectors

This simplified approach focuses on practical CRUD workflows that are common in business applications, making it easy to understand and implement. 