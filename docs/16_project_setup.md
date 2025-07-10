---
title: Project Setup with Vite
layout: default
nav_order: 16
---

# Setting Up a React Project with Vite and TypeScript

Hey team! Let's set up a modern React project using Vite and TypeScript. This is currently the fastest and most efficient way to get started with React development.

## Why Vite?

Vite (French for "fast") is a build tool that provides an extremely fast development server and optimized builds. It's significantly faster than Create React App and has become the go-to choice for modern React projects.

## Quick Setup

### 1. Create the Project

```bash
npm create vite@latest my-react-app -- --template react-ts
cd my-react-app
```

**Alternative method:**
```bash
yarn create vite my-react-app --template react-ts
cd my-react-app
```

### 2. Install Dependencies

```bash
npm install
# or
yarn install
```

### 3. Start Development Server

```bash
npm run dev
# or
yarn dev
```

Your app will be running at `http://localhost:5173` (Vite's default port).

## Project Structure

After creation, your project will look like this:

```
my-react-app/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   │   └── react.svg
│   ├── App.tsx
│   ├── App.css
│   ├── index.css
│   ├── main.tsx
│   └── vite-env.d.ts
├── index.html
├── package.json
├── tsconfig.json
├── tsconfig.node.json
└── vite.config.ts
```

## Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build locally
- `npm run lint` - Run ESLint

## Environment Variables

Create a `.env` file in your project root:

```env
VITE_API_URL=https://api.yourapp.com
VITE_APP_TITLE=My React App
```

Access them in your code:

```typescript
const apiUrl = import.meta.env.VITE_API_URL;
const appTitle = import.meta.env.VITE_APP_TITLE;
```


## Useful Links

- [Vite Documentation](https://vitejs.dev/)
- [React Documentation](https://react.dev/)
- [TypeScript Documentation](https://www.typescriptlang.org/)
- [Let Me Google That - React + Vite + TypeScript](https://letmegooglethat.com/?q=create+a+react+app+with+typescript+and+vite)

---

**Last updated: July 10, 2025**

This setup gives you a modern, fast, and type-safe React development environment. Vite's hot module replacement (HMR) is incredibly fast, making your development experience much more enjoyable! 