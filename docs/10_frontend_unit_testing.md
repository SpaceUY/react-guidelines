---
title: Unit testing
layout: default
nav_order: 10
---

## Frontend Test

Technology Review
Writer: Leonel Mazzan

### Introduction

Testing is a crucial aspect of modern web development, ensuring that applications are reliable and maintainable. React, a popular JavaScript library for building user interfaces, often pairs with testing tools like Jest and React Testing Library for this purpose. This document outlines the benefits of using Jest and React Testing Library for testing React applications and provides an example of their implementation in a Vite project.

#### Why Use Jest and React Testing Library?

**Vitest**

- **Fast Execution**: Vitest, being a Vite-native test runner, leverages Vite's speed and esbuild's efficiency, leading to quicker test execution times.
- **Easy Configuration**: It integrates seamlessly with Vite's existing configuration, reducing the setup complexity.
- **Modern Tooling**: Supports modern JavaScript features and TypeScript out of the box.

**React Testing Library**

- **User-Centric Approach**: Focuses on testing components as users interact with them, rather than testing implementation details.
- **Maintainable Tests**: Encourages writing maintainable tests, leading to more resilient code.
- **Wide Adoption**: Being part of the Testing Library family, it's widely adopted and supported in the React community.

### Why not Jest?

Vitest is preferred over Jest for testing in Vite due to its seamless integration and native support for Vite's ES module system. This results in faster performance and easier setup. While Jest is powerful, it requires extra configuration to work with Vite, leading to potential complexity and slower test execution.

#### Little example of implementation:

npm install --save-dev vitest @testing-library/jest-dom @testing-library/react @testing-library/user-event

Add to your package.json

```
{
  "scripts": {
    "test": "vitest"
  }
}
```

npm install --save-dev jsdom

Create a stup.js in src/tests/setup.js

```
import { afterEach } from "vitest";
import { cleanup } from "@testing-library/react";
import "@testing-library/jest-dom/vitest";


// runs a clean after each test case (e.g. clearing jsdom)
afterEach(() => {
  cleanup();
});
```

In your vite.config.js

```
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import eslintPlugin from "vite-plugin-eslint";


// https://vitejs.dev/config/
export default defineConfig({
  ...other
  test: {
    // 👋 add the line below to add jsdom to vite
    environment: "jsdom",
    // hey! 👋 over here
    globals: true,
    setupFiles: "./src/tests/setup.js",
  },
});
```

Example of a test in src / tests / App.test.jsx

```
import { render, screen } from "@testing-library/react";
import { describe, it, expect } from "vitest";
import App from "./../App";


describe("A truthy statement", () => {
  it("should be equal to 2", () => {
    expect(1 + 1).toEqual(2);
  });
});


describe('App Component', () => {
  it('debe tener el texto específico', () => {
    render(<App />);


    const linkElement = screen.getByText(/Click on the Vite and React logos to learn more/i);


    expect(linkElement).toBeInTheDocument();
  });
});
```

[Source Link](https://victorbruce82.medium.com/vitest-with-react-testing-library-in-react-created-with-vite-3552f0a9a19a)
