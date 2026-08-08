---
title: React 19 Features
layout: default
nav_order: 15
---

# TBD: New folder with news ? features ? extras ?

# New Features in React 19

React 19 introduces several exciting features and improvements aimed at enhancing developer experience and application performance. This document outlines the key additions.

## Server Actions

Functions that are executed on the server and can be called from the client just by using "use server" directive.

### [Actions](https://react.dev/blog/2024/12/05/react-19#actions)

> By convention, functions that use async transitions are called “Actions”. 

Adding support for using async functions in transitions to handle pending states, errors, forms, and optimistic updates automatically.

For example, you can use useTransition to handle the pending state for you:

```tsx
// Using pending state from Actions
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      } 
      redirect("/path");
    })
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

The async transition will immediately set the isPending state to true, make the async request(s), and switch isPending to false after any transitions. This allows you to keep the current UI responsive and interactive while the data is changing.

### useActionState

useActionState accepts a function (the “Action”), and returns a wrapped Action to call. This works because Actions compose. When the wrapped Action is called, useActionState will return the last result of the Action as data, and the pending state of the Action as pending.

To make the common cases easier for Actions, we’ve added a new hook called useActionState:


```tsx
const [error, submitAction, isPending] = useActionState(
  async (previousState, newName) => {
    const error = await updateName(newName);
    if (error) {
      // You can return any result of the action.
      // Here, we return only the error.
      return error;
    }

    // handle success
    return null;
  },
  null,
);
```

## [Forms](https://react.dev/blog/2024/12/05/react-19#form-actions)

Actions are also integrated with React 19’s new `<form>` features for react-dom. We’ve added support for passing functions as the action and formAction props of `<form>`, `<input>`, and `<button>` elements to automatically submit forms with Actions:

```tsx
<form action={actionFunction}>
```

### useFormStatus

useFormStatus reads the status of the parent <form> as if the form was a Context provider.

```tsx
import {useFormStatus} from 'react-dom';

function DesignButton() {
  const {pending} = useFormStatus();
  return <button type="submit" disabled={pending} />
}
```

### [useOptimistic](https://react.dev/blog/2024/12/05/react-19#new-hook-optimistic-updates)

Another common UI pattern when performing a data mutation is to show the final state optimistically while the async request is underway. In React 19, we’re adding a new hook called useOptimistic to make this easier:

```tsx
function ChangeName({currentName, onUpdateName}) {
  const [optimisticName, setOptimisticName] = useOptimistic(currentName);

  const submitAction = async formData => {
    const newName = formData.get("name");
    setOptimisticName(newName);
    const updatedName = await updateName(newName);
    onUpdateName(updatedName);
  };

  return (
    <form action={submitAction}>
      <p>Your name is: {optimisticName}</p>
      <p>
        <label>Change Name:</label>
        <input
          type="text"
          name="name"
          disabled={currentName !== optimisticName}
        />
      </p>
    </form>
  );
}
```

The useOptimistic hook will immediately render the optimisticName while the updateName request is in progress. When the update finishes or errors, React will automatically switch back to the currentName value.

## [use](https://react.dev/blog/2024/12/05/react-19#new-feature-use)

New API to read resources in render: `use`. For example, you can read a promise with use, and React will Suspend until the promise resolves:

```tsx
import {use} from 'react';

function Comments({commentsPromise}) {
  // `use` will suspend until the promise resolves.
  const comments = use(commentsPromise);
  return comments.map(comment => <p key={comment.id}>{comment}</p>);
}

function Page({commentsPromise}) {
  // When `use` suspends in Comments,
  // this Suspense boundary will be shown.
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Comments commentsPromise={commentsPromise} />
    </Suspense>
  )
}
```

> `use` does not support promises created in render. 

You can also read context with `use`, allowing you to read Context conditionally such as after early returns:

```tsx
import {use} from 'react';
import ThemeContext from './ThemeContext'

function Heading({children}) {
  if (children == null) {
    return null;
  }
  
  // This would not work with useContext
  // because of the early return.
  const theme = use(ThemeContext);
  return (
    <h1 style={{color: theme.color}}>
      {children}
    </h1>
  );
}
```

The use API can only be called in render, similar to hooks. Unlike hooks, use can be called conditionally.

## [New React DOM Static APIs](https://react.dev/blog/2024/12/05/react-19#new-react-dom-static-apis)

## React Server Components 

### [Server Components](https://react.dev/blog/2024/12/05/react-19#server-components)

Server Components are a new option that allows rendering components ahead of time, before bundling, in an environment separate from your client application or SSR server. This separate environment is the “server” in React Server Components. Server Components can run once at build time on your CI server, or they can be run for each request using a web server.

For more, see the docs for [React Server Components](https://react.dev/reference/rsc/server-components).

### [Server Actions](https://react.dev/blog/2024/12/05/react-19#server-actions)

Server Actions allow Client Components to call async functions executed on the server.

When a Server Action is defined with the `"use server"` directive, your framework will automatically create a reference to the server function, and pass that reference to the Client Component. When that function is called on the client, React will send a request to the server to execute the function, and return the result.

> **There is no directive for Server Components.**
> A common misunderstanding is that Server Components are denoted by "use server", but there is no directive for Server Components. The "use server" directive is used for Server Actions.

## Improvements in React 19 

### [`ref` as a prop](https://react.dev/blog/2024/12/05/react-19#ref-as-a-prop)

Starting in React 19, you can now access ref as a prop for function components:

```tsx
function MyInput({placeholder, ref}) {
  return <input placeholder={placeholder} ref={ref} />
}

//...
<MyInput ref={ref} />
```

New function components will no longer need forwardRef, and we will be publishing a codemod to automatically update your components to use the new ref prop. In future versions we will deprecate and remove forwardRef.

### [`useDeferredValue` initial value](https://react.dev/blog/2024/12/05/react-19#use-deferred-value-initial-value)

We’ve added an `initialValue` option to `useDeferredValue`:

```tsx
function Search({deferredValue}) {
  // On initial render the value is ''.
  // Then a re-render is scheduled with the deferredValue.
  const value = useDeferredValue(deferredValue, '');
  
  return (
    <Results query={value} />
  );
}
```

When `initialValue` is provided, `useDeferredValue` will return it as value for the initial render of the component, and schedules a re-render in the background with the deferredValue returned.

### [Support for Document Metadata](https://react.dev/blog/2024/12/05/react-19#support-for-metadata-tags)

In HTML, document metadata tags like `<title>`, `<link>`, and `<meta>` are reserved for placement in the `<head>` section of the document. In React, the component that decides what metadata is appropriate for the app may be very far from the place where you render the `<head>` or React does not render the `<head>` at all. In the past, these elements would need to be inserted manually in an effect, or by libraries like `react-helmet`, and required careful handling when server rendering a React application.

In React 19, we’re adding support for rendering document metadata tags in components natively:

```tsx
function BlogPost({post}) {
  return (
    <article>
      <h1>{post.title}</h1>
      <title>{post.title}</title>
      <meta name="author" content="Josh" />
      <link rel="author" href="https://twitter.com/joshcstory/" />
      <meta name="keywords" content={post.keywords} />
      <p>
        Eee equals em-see-squared...
      </p>
    </article>
  );
}
```

When React renders this component, it will see the `<title>` `<link>` and `<meta>` tags, and automatically hoist them to the `<head>` section of document. By supporting these metadata tags natively, we’re able to ensure they work with client-only apps, streaming SSR, and Server Components.

> While this helps on small scenarios and apps, these features make it easier for frameworks and libraries to support metadata tags, rather than replace them.

### [Support for stylesheets](https://react.dev/blog/2024/12/05/react-19#support-for-stylesheets)

### [Support for async scripts](https://react.dev/blog/2024/12/05/react-19#support-for-stylesheets)

Basically, improved imports of these tags, prevent duplication and helps declare priorities.

### [Support for preloading resources](https://react.dev/blog/2024/12/05/react-19#support-for-preloading-resources)

During initial document load and on client side updates, telling the Browser about resources that it will likely need to load as early as possible can have a dramatic effect on page performance.

React 19 includes a number of new APIs for loading and preloading Browser resources to make it as easy as possible to build great experiences that aren’t held back by inefficient resource loading.

```tsx
import { prefetchDNS, preconnect, preload, preinit } from 'react-dom'
function MyComponent() {
  preinit('https://.../path/to/some/script.js', {as: 'script' }) // loads and executes this script eagerly
  preload('https://.../path/to/font.woff', { as: 'font' }) // preloads this font
  preload('https://.../path/to/stylesheet.css', { as: 'style' }) // preloads this stylesheet
  prefetchDNS('https://...') // when you may not actually request anything from this host
  preconnect('https://...') // when you will request something but aren't sure what
}
```

```html
<!-- the above would result in the following DOM/HTML -->
<html>
  <head>
    <!-- links/scripts are prioritized by their utility to early loading, not call order -->
    <link rel="prefetch-dns" href="https://...">
    <link rel="preconnect" href="https://...">
    <link rel="preload" as="font" href="https://.../path/to/font.woff">
    <link rel="preload" as="style" href="https://.../path/to/stylesheet.css">
    <script async="" src="https://.../path/to/some/script.js"></script>
  </head>
  <body>
    ...
  </body>
</html>
```

### [Compatibility with third-party scripts and extensions](https://react.dev/blog/2024/12/05/react-19#compatibility-with-third-party-scripts-and-extensions)

We’ve improved hydration to account for third-party scripts and browser extensions.

When hydrating, if an element that renders on the client doesn’t match the element found in the HTML from the server, React will force a client re-render to fix up the content. Previously, if an element was inserted by third-party scripts or browser extensions, it would trigger a mismatch error and client render.

### [Better error reporting](https://react.dev/blog/2024/12/05/react-19#error-handling)

### [Support for Custom Elements](https://react.dev/blog/2024/12/05/react-19#how-to-upgrade)

## [How to upgrade](https://react.dev/blog/2024/12/05/react-19#how-to-upgrade)
