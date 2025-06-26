# shadcn/ui and Tailwind CSS in React + Vite: Why and How to Use Them

## Introduction

**shadcn/ui** is an emerging collection of prebuilt, highly customizable React components designed to integrate seamlessly with Tailwind CSS. Unlike traditional toolkits, shadcn/ui is not installed as an external library. Instead, it provides the source code for the components so you can include them directly in your project. This means you own the components' codebase.

**Tailwind CSS**, on the other hand, is a utility-first CSS framework that offers predefined classes for nearly any style. It allows you to design interfaces by applying these classes directly to your components without having to write custom CSS in most cases.

## Why Use shadcn/ui and Tailwind Together?

### Development Speed
By combining ready-to-use components with utility classes, you can build complex interfaces quickly. shadcn/ui provides fully functional components (with logic and accessibility already handled) that you just insert into your code, avoiding the need to "reinvent the wheel" for common elements like buttons, menus, dialogs, etc. Tailwind speeds up styling through utility classes, eliminating repetitive manual CSS writing. Together, this combo enables rapid prototyping and feature iteration, reducing time-to-market.

### Visual Consistency
Tailwind establishes a consistent design system (color palette, sizing, spacing, typography) applied uniformly across all components. shadcn/ui leverages this by using Tailwind classes by default, ensuring that every UI element adheres to the same styling guidelines.

### Full Customization and Control
Because shadcn/ui components live inside your project, you can freely modify them to suit your needs. You're not constrained by third-party abstractions — if a component doesn't fit exactly what you want, you can edit its JSX/TSX and Tailwind classes directly. Additionally, shadcn/ui's architecture is designed for flexibility, using utilities like `class-variance-authority` to define component variants elegantly. You have full control over markup and styling, making it easy to build unique designs without straying from a stable foundation. Importantly, since there's no external package dependency, your components won't change unexpectedly due to library updates — they only change if you decide to update them.

### Bundle Optimization
One of the most notable technical benefits is the reduction in bundle size and dependencies. shadcn/ui adds virtually no extra weight to your application since it's not a bundled library — its components compile as part of your own code. In contrast, many traditional UI libraries can add dozens or even hundreds of KB to the final JavaScript/CSS bundle (e.g., Material UI can add ~90 KB). With shadcn/ui + Tailwind, you only include the code and styles for the components you actually use. Tailwind CSS also purges unused classes in production, leaving only the necessary CSS. The result is a potentially lighter and faster-loading application.

### Built-in Accessibility and Best Practices
shadcn/ui is built on top of **Radix UI**, a set of accessible primitives for React. This means that components like dialogs, dropdowns, tooltips, etc., already meet accessibility standards (WAI-ARIA) by default. In other words, by using shadcn/ui, you get components with built-in accessible behavior (keyboard navigability, proper ARIA roles, focus management, etc.) without extra work. Tailwind complements this by making it easy to apply accessible styles.

### Easy Adoption in React + Vite Projects
Both shadcn/ui and Tailwind CSS are designed to integrate smoothly with modern tools. Tailwind works perfectly in Vite environments (fast builds, HMR), and shadcn/ui offers a CLI that automatically configures everything in a React project. This reduces the friction of adding them to your stack. Additionally, the community around React and Tailwind is massive, so there's plenty of support, plugins, and resources available.

## Comparison with Other Popular Alternatives

### shadcn/ui vs. Material UI (MUI)

**Material UI** is one of the most established component libraries in React, implementing Google's Material Design system. It offers dozens of ready-to-use components with a familiar predefined style and a large community. However, that very defined design identity can be limiting. MUI strictly follows Material Design guidelines, which is great if you want a consistent and recognizable aesthetic, but it makes it hard to deviate from the default look. Deeply customizing MUI so it doesn't "look like Material Design" often means overriding many styles, dealing with its theming system, and sometimes fighting CSS specificity.

In contrast, **shadcn/ui** doesn't enforce any specific design language beyond what you define with Tailwind. Its components come with neutral base styles but are designed to be easily adapted. This makes it possible to create a unique design for your app beyond Material Design, without the feeling of "fighting" the defaults. If your project requires heavy visual customization, shadcn/ui is a great fit.

There's also a technical difference in performance and bundle size: MUI is a mature but heavy library, with around **93 KB** of minified base component code (not including icon dependencies, etc.). It uses its own styling system (based on CSS-in-JS with Emotion in recent versions), which adds runtime overhead due to dynamically injected styles.

On the other hand, shadcn/ui adds virtually no extra bundle size — its components are compiled as part of your own React code, using only Tailwind utilities and a few small helper libraries. A project using 5 shadcn/ui components only compiles those 5 components, whereas MUI may include a significant portion of the library even if you use only a few parts. This often makes shadcn/ui lighter and more efficient.

### shadcn/ui vs. Chakra UI

**Chakra UI** is another popular React component library. Unlike MUI, Chakra doesn't follow a specific design aesthetic like Material — instead, it provides a minimal base style that is easy to theme. It stands out for its simplicity and focus on accessibility, offering modular components that are easy to use and customize via props. For example, Chakra lets you change a component's style by passing props like `colorScheme="blue"` or `size="lg"`, and it handles the CSS details internally. It also has built-in support for dark mode and other utilities.

Philosophically, Chakra UI and shadcn/ui share the goal of simplicity and accessibility, but they differ in how customization is achieved. Chakra offers convenience through abstraction — you use its components and predefined props to achieve different styles. This is very convenient for developers who don't want to deal with CSS classes — Chakra's API is very beginner-friendly, and the documentation is clear.

However, this simplicity can come with some rigidity: Chakra's components are designed to fit nicely into its system (based on Styled System), so if you need something outside the norm, you may end up writing extra CSS or using the `sx` prop (custom styles with JS objects), which sacrifices some of that initial elegance.

With **shadcn/ui**, customization is much broader because there are no proprietary abstraction layers — you simply edit JSX and Tailwind classes. You can make any modification you'd do with standard HTML/CSS, fully leveraging Tailwind's flexibility. This means shadcn/ui offers greater control over customization than Chakra UI. If you want a component with entirely unique behavior or design, you can build it using primitives in shadcn/ui, whereas with Chakra you're more constrained by the structure they provide.

Another difference is the styling approach: Chakra UI uses CSS-in-JS (Emotion) internally, applying styles via props at runtime. This means you don't see class names in JSX (many find `<Box bg="red.500">` more readable than a long list of Tailwind classes), but it introduces a runtime performance cost similar to MUI. With shadcn/ui + Tailwind, all styling is static CSS generated at build time — there's no JavaScript running to apply styles. This can make the resulting app more efficient in terms of load time and rendering.

### Tailwind CSS vs. CSS Modules and Styled Components

**CSS Modules** allow you to write regular CSS in separate files while automatically scoping styles to components. This solves the style isolation problem and works with any framework. The main difference with Tailwind is that with CSS Modules, you still write your own CSS classes for each component.

For example, you might have a file `Button.module.css` where you define:

```css
.button {
  background-color: blue;
  padding: 8px;
}
```

And then import and use it in your React component.

Tailwind eliminates the need to write these classes manually by providing predefined utilities. Instead of creating a `.button` class, you'd simply apply:

```jsx
className="bg-blue-500 p-2"
```

This achieves the same visual result without naming a class or writing custom CSS from scratch. The benefit is speed and consistency: Tailwind's design system ensures that `p-2` means the exact same padding across all components, avoiding discrepancies due to human error. With CSS Modules, consistency depends on the developer's discipline.

Regarding bundle size, both approaches generate static CSS, but Tailwind has an advantage: during the build process, it removes all unused classes, leaving only the necessary CSS. With CSS Modules, theoretically, you only load what you import, but if a module defines many classes and you use only a few, the rest are still included as long as the file is imported. In practice, this difference is often small, but Tailwind's consistency benefit is notable.

### Tailwind CSS vs. Styled Components (CSS-in-JS)

**Styled Components** is a popular CSS-in-JS library for React. It lets you write CSS inside JS/TS files using tagged template literals. For example:

```jsx
const Button = styled.button`
  background: blue;
  padding: 8px;
`;
```

This integrates styles at the component level in a clean way and supports logic for dynamic styles. The major difference with Tailwind is that Styled Components inject styles into the DOM at runtime, whereas Tailwind applies static CSS classes generated at build time.

In practice, Styled Components provides full CSS syntax and avoids the need to remember class names or Tailwind utilities — everything is self-contained. However, this convenience comes at a runtime performance cost: each styled component generates CSS in runtime, with overhead for parsing and injecting styles, plus the weight of the runtime library itself. For small apps this may be negligible, but in large apps with hundreds of components, the overhead becomes significant.

With Tailwind, there's no runtime cost for styles: you just apply classes that already exist in the loaded CSS. Additionally, you don't need an extra library like Styled Components, reducing overall weight. Many teams have migrated from CSS-in-JS to Tailwind seeking performance improvements and build-time simplicity.

In terms of readability and maintainability, both approaches have pros and cons. With Styled Components, a component's styles are colocated in the same file (or nearby), promoting cohesion and avoiding clutter in the JSX.

For example, this is easier for some to read:

```jsx
<Button>Hello</Button>
```

And then inspect the CSS in the Button definition, compared to:

```jsx
<button className="bg-blue-500 text-white font-bold py-2 px-4 rounded">
  Hello
</button>
```

Where you need to interpret multiple Tailwind classes. In highly styled components, a long list of Tailwind classes can be hard to read, especially for those unfamiliar with the utility names. In such cases, Styled Components "wins" in clarity, since it uses standard CSS instead of abbreviations and keeps structure (JSX) separate from appearance (CSS).

However, Tailwind promotes maintainability in a different way: through utility composition. If a set of classes is repeated or long, you can abstract it into a smaller component or use the `@apply` directive in your global CSS to create a reusable style. There are also libraries like `clsx`, `twMerge`, or `classnames` to conditionally apply and cleanly combine classes in JSX.

## How to Use shadcn/ui with Tailwind CSS in React + Vite

We'll assume you already have Node.js installed.

### 1. Create a New React Project with Vite

The first step is to generate a React project using Vite. Run the following command:

```bash
npm create vite@latest
```

You'll be prompted to name the project, select the framework (choose **React**), and the language. It's recommended to select **TypeScript** to take full advantage of shadcn/ui's type system, though plain JavaScript is also supported.

Once the setup is complete, enter your new project directory.

### 2. Install and Configure Tailwind CSS

Next, we'll add Tailwind CSS to the project. Tailwind requires some dev dependencies and configuration files. Inside your project, run:

```bash
npm install tailwindcss @tailwindcss/vite
```

Then, open `vite.config.ts` and use the following configuration:

```typescript
// vite.config.ts
import path from "path";
import tailwindcss from "@tailwindcss/vite";
import react from "@vitejs/plugin-react";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"), // Use this if you'll follow the next step
    },
  },
});
```

Tailwind will scan files defined in content for class names (make sure it includes `index.html` and all files under `src/`). This is essential for purging unused classes in production and keeping your final CSS bundle optimized.

Next, import Tailwind's base directives in your global styles. Vite creates a default file at `src/index.css`. Open it, clear its contents, and add:

```css
@import "tailwindcss";
```

This loads Tailwind's resets, base styles, components, and utility classes like `mt-4`, `flex`, `text-xl`, etc.

Tailwind CSS is now integrated into your project.

### 3. Configure Path Aliases (Important for shadcn/ui)

In recent Vite + TypeScript projects, you'll find two config files: `tsconfig.json` and `tsconfig.app.json`. You need to edit both to define the alias. In `tsconfig.json`, under `compilerOptions`, add:

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

And in `tsconfig.app.json`:

```json
// tsconfig.app.json
{
  "compilerOptions": {
    // ...
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
    // ...
  }
}
```

This tells the compiler and your IDE that `@/components/anything` corresponds to `src/components/anything`.

Now edit `vite.config.ts` again (if you didn't already in the Tailwind setup) to ensure Vite resolves the alias at runtime:

```typescript
// vite.config.ts
import path from "path";
import tailwindcss from "@tailwindcss/vite";
import react from "@vitejs/plugin-react";
import { defineConfig } from "vite";

// https://vite.dev/config/
export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

### 4. Install shadcn/ui via CLI

The easiest way to add shadcn/ui is through its CLI tool, distributed via npm. It will guide you through initial setup. Run the following command in your project root:

```bash
npx shadcn@latest init
```

On first run, the CLI will download necessary utilities and prompt you with configuration options:

- **Style**: Select "default" for the base style. You can try other options later.
- **Base color**: It defaults to "Neutral" (a neutral gray palette). You can choose others (e.g., slate, emerald), but Neutral is a safe and easy starting point.
- **CSS variables**: You'll be asked whether to use CSS variables for theming. If you answer **No**, shadcn/ui will use Tailwind classes directly (e.g., `bg-neutral-950`). If you answer **Yes**, it will use semantic classes tied to CSS variables, which makes theme and dark mode switching easier. For simplicity, you can choose **No** for now — this can be enabled later.

Once the CLI finishes, a `components.json` file will be created with your configuration, and necessary dependencies will be installed automatically.

### 5. Add Your First UI Component

With setup complete, you can start importing UI components from shadcn. The CLI provides commands to add specific components. For example, to add a Button, run:

```bash
npx shadcn@latest add button
```

This will generate the necessary files for the Button component (typically in `src/components/ui/button.tsx`, along with styles if needed). You can repeat this for other components like "alert-dialog", "card", "input", etc.

Now, use the component in your React app. Let's test the button in your main App component. Open `src/App.tsx` and add:

```jsx
import { Button } from "@/components/ui/button";

function App() {
  return (
    <div className="p-6">
      <Button>Click me</Button>
    </div>
  );
}

export default App;
```

Start your dev server — you should see your styled button rendered on the page.

## Initial Examples of Component Usage

We've already tried using `<Button>`, but the library offers much more — from simple components (like form inputs) to more complex combinations (modal dialogs, dropdown menus, tables, etc.). They all follow a similar pattern: import them from `"@/components/ui/<name>"` and use them like any standard React component, passing props as needed.

### Button Variants

The Button component, for instance, accepts props to modify its appearance. It includes a `variant` prop with options like "default", "outline", "secondary", etc., and a `size` prop ("sm", "md", "lg"). These variants are defined using Tailwind classes. For example, `variant="outline"` gives the button a transparent background with a border. You can use them like this:

```jsx
<Button variant="outline" size="lg">Large Button</Button>
<Button variant="destructive">Dangerous Action</Button>
```

These will render a large outlined button and a "destructive"-styled button. These variants are implemented using the `cva` (class-variance-authority) utility, which generates the appropriate Tailwind classes based on the selected variant. There's even a helper called `buttonVariants` that allows you to apply the same styles to other elements, like making a `<a>` tag look like a button.

### Dialog Component (AlertDialog)

To illustrate a more complex component, let's try `AlertDialog`, used for displaying confirmation modal dialogs. It consists of several subcomponents (provided by Radix UI under the hood). Example:

```jsx
import {
  AlertDialog,
  AlertDialogTrigger,
  AlertDialogContent,
  AlertDialogHeader,
  AlertDialogFooter,
  AlertDialogTitle,
  AlertDialogDescription,
  AlertDialogAction,
  AlertDialogCancel
} from "@/components/ui/alert-dialog";
import { Button } from "@/components/ui/button";

function DeleteButton() {
  return (
    <AlertDialog>
      <AlertDialogTrigger asChild>
        <Button variant="destructive">Delete Account</Button>
      </AlertDialogTrigger>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>Are you sure?</AlertDialogTitle>
          <AlertDialogDescription>
            This action cannot be undone. All your data will be permanently deleted.
          </AlertDialogDescription>
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel>Cancel</AlertDialogCancel>
          <AlertDialogAction>Continue</AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  );
}
```

In this example:

- `<AlertDialog>` wraps everything and manages the open/close state.
- `<AlertDialogTrigger>` is the element that opens the dialog on click (here we use `asChild` to make the trigger our destructive button).
- `<AlertDialogContent>` defines the modal content, including the title, description, and the cancel/confirm buttons using subcomponents.

By using `<AlertDialogAction>` and `<AlertDialogCancel>`, shadcn/ui handles focus management and dialog closing automatically. Everything is styled by default using Tailwind — for instance, `<AlertDialogContent>` has classes for background, entrance animation, etc., which you can modify in code if needed.

This composition pattern is common in several shadcn/ui components (like Dropdown Menu, Popover, Menubar, etc.): they usually consist of a main container and child components for each section. The shadcn/ui documentation explains the structure and usage of each one.

### Other Basic Components

You also have access to simple components like `<Input>`, `<Checkbox>`, `<Switch>`, `<Card>`, `<Tabs>`. For example, to use an input with a label:

```jsx
import { Label } from "@/components/ui/label";
import { Input } from "@/components/ui/input";

<Label htmlFor="email">Email</Label>
<Input id="email" type="email" placeholder="your@email.com" />
```

These examples show that using shadcn/ui is virtually the same as using your own components — there are no surprises in terms of React behavior, and you can wrap them with your own components if you need additional logic.

## Customizing and Creating Your Own Components

### Customizing Styles of Existing Components

Since shadcn/ui components are built with Tailwind, customizing them can be as simple as changing, removing, or adding utility classes directly in the component's JSX. For example, suppose the outline style doesn't fit your design and you want a gradient button. Open `src/components/ui/button.tsx` and look for the style variant definitions. You'll find something like this:

```typescript
const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-ring disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        outline: "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        // ... other variants
      },
      size: {
        sm: "h-9 px-3 rounded-md",
        md: "h-10 px-4 rounded-md",
        lg: "h-11 px-8 rounded-md"
      }
    },
    defaultVariants: {
      variant: "default",
      size: "md"
    }
  }
);
```

This syntax comes from the `class-variance-authority` (cva) utility, and essentially defines a variants object. You can add your own variant or modify an existing one. For example, let's add a "gradient" variant:

```typescript
variant: {
  default: "bg-primary text-primary-foreground hover:bg-primary/90",
  outline: "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
  secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
  gradient: "bg-gradient-to-r from-blue-500 to-purple-500 text-white hover:opacity-90",
}
```

Now you can use:

```jsx
<Button variant="gradient">Click Me</Button>
```

to get a new gradient-style button. Similarly, you can change colors, fonts, sizes, etc., directly in these classes.

### Using Additional CSS Styles

Tailwind doesn't cover 100% of use cases (e.g., complex animations or very specific scenarios), so shadcn/ui includes a small CSS file for animations (`tw-animate.css`). You can also add your own global CSS classes in `src/index.css` below the Tailwind directives, or use Tailwind's `@apply` to create reusable styles from utility classes.

For example, if you want all modals to have a higher z-index, you could write:

```css
/* in index.css, after @tailwind utilities; */
.modal {
  @apply z-50;
}
```

Then add the `modal` class to the desired element in `alert-dialog.tsx`.

That said, it's often easier to just modify the existing class directly in the component (e.g., edit `AlertDialogContent` to include `z-50`).

In short, you have full freedom — unlike third-party libraries, nothing breaks when you touch the code. It's yours to control.

## Extending the Tailwind CSS Theme for shadcn/ui

Tailwind CSS is highly configurable. You can adapt its theme to match your product's branding. Brand colors, corporate fonts, custom border radius values, shadows, etc., can all be defined inside the `theme.extend` object in your `tailwind.config.js`.

Integration with shadcn/ui happens indirectly but effectively: shadcn/ui components use Tailwind classes that ultimately reference your theme configuration. For example, when you see `bg-primary` in a Button, that class relies on whatever `primary` is defined as in your Tailwind theme.

By default, when initializing shadcn/ui, you might not have a `primary` color explicitly defined in your theme. The base "Neutral" configuration often means components will use generic utilities instead of semantic ones like `bg-primary`.

However, you can take advantage of Tailwind's theme system to assign your brand colors to specific names. One way is to define them like this in your `tailwind.config.js`:

```javascript
theme: {
  extend: {
    colors: {
      primary: {
        DEFAULT: '#1D4ED8', // example blue
        50: '#eff6ff',
        100: '#dbeafe',
        200: '#bfdbfe',
        300: '#93c5fd',
        400: '#60a5fa',
        500: '#3b82f6',
        600: '#2563eb',
        700: '#1d4ed8',
        800: '#1e40af',
        900: '#1e3a8a'
      },
      secondary: {
        DEFAULT: '#f59e0b', // amber
        // ...more variants
      },
      brandGreen: '#10b981' // you can also define single custom colors
    },
    borderRadius: {
      xl: '1rem' // for example, globally larger border radius
    }
  }
}
```

Now, in your components, if you want the default Button to use your corporate blue, just ensure it uses `bg-primary`, and confirm that `primary` in your theme is set to your blue color. Alternatively, you can change it to `bg-brandGreen` if you've defined a specific color by that name.

Tailwind lets you rename or add any color you want — making it simple to align your UI components with your brand identity.

## Best Practices

To get the most out of shadcn/ui and Tailwind CSS in your React + Vite stack, keep the following best practices in mind:

### Clear Component Organization

Keep your UI components well-organized, ideally inside a dedicated folder (e.g., `src/components/ui/`) as suggested by shadcn. This makes it easier to locate and maintain the imported components. If you create custom components, place them in the same structure for consistency.

### Avoid Unnecessary Over-Customization

Having access to the component code can tempt you to modify everything. Do it only when it adds value. If the default style works and follows UX guidelines, there's no need to change it. Every customization you make becomes your responsibility to maintain. Find a good balance between using the components as-is (already tested) and adapting them where it truly improves the experience.

### Use Tailwind Theming Consistently

As mentioned before, centralize colors and sizes in your `tailwind.config.js`. Avoid using "magic values" directly in class names (e.g., don't write `bg-[#123456]` in 10 different components; define it in the theme with a meaningful name instead). This ensures that if your brand identity changes (e.g., a new brand color), updating the theme will update your entire UI accordingly. Tailwind makes this easy, and shadcn/ui is built to work with named theme values.

### Take Advantage of Integration Utilities

shadcn/ui installs `clsx` and `tailwind-merge`. Use them when building conditional class strings or combining variants. For example, if you're composing two components and need to merge classes, `tailwind-merge` will prevent duplicates and conflicts. `clsx` helps you construct class strings conditionally. These tools help keep your code clean when listing static classes isn't enough.

### Atomic vs. Composite Components

Some shadcn components (e.g., the previously mentioned `AlertDialog`) require multiple subcomponents to be used together. It's recommended to encapsulate these into your own wrapper component if you always use them in a specific way. For example, you could create:

```jsx
<ConfirmDialog title={..} description={..} onConfirm={..} />
```

which internally uses `AlertDialog` but exposes a simpler interface. This is optional, but in large projects it can help avoid repeating lots of markup and simplify repeated patterns.

### Stay Updated — On Your Terms

The shadcn/ui repo evolves rapidly with new components and improvements. Since your project won't update automatically, check the changelog or community notes from time to time to see if there are updates you'd like to incorporate. For example, if a new component "X" is released, you can add it via the CLI. Or if a component's accessibility has improved, you might want to adopt those changes manually. It's up to you — the benefit is that you control when to update. If everything works, no rush. But being aware of updates can help you benefit from improvements.

## Scalability and Long-Term Maintenance

Some considerations as your project grows:

### Code Ownership

Unlike relying on an external library from npm, here you own the UI code. This means you're responsible for maintaining it, but you also have full autonomy. In the long term, if shadcn/ui stops being updated, your project won't break — you already have all the code you need. Likewise, if a new version of React or Vite is released, you can adapt immediately without waiting for a library patch. The components are yours to modify.

### Dependency Updates

Although the components are local, you still rely on some underlying libraries (like Radix UI, Lucide React for icons, etc.). Keep these dependencies updated with care. For example, Radix UI might release important fixes — you can bump the version in `package.json` and verify everything still works. Tailwind CSS will also evolve — upgrading Tailwind may require config changes. shadcn/ui typically provides migration guides when necessary.

### CSS Scalability

Thanks to Tailwind JIT, only the classes you use are generated, so even as your app grows in pages and features, the final CSS stays as small as possible. However, in very large projects with thousands of utility classes, your final CSS might still reach several hundred KB — which is often smaller or equivalent to a custom-designed CSS framework. **Important**: avoid unpredictable dynamic class names (e.g., building class names by concatenating strings from props), since Tailwind can't detect and purge them.

### Design Maintenance

Design requirements evolve over time. With this stack, applying a global redesign can be more agile: thanks to Tailwind, many changes (colors, fonts, etc.) are centralized in your theme. Even if you need to adjust components, since you own them, you can plan and implement changes gradually. For example, a brand color update can be done in minutes via the theme. A deeper redesign may require editing multiple components — but at least you have everything under your control.

### Runtime Performance

React and Tailwind are both production-proven at scale. The shadcn/ui components — by nature — don't introduce significant complexity beyond what React and Radix already do. That means your runtime performance remains solid, with minimal added cost.
