---
title: Client server side
layout: default
nav_order: 2
---

# When to use client-side or server-side?

(Depending on the correct analysis of the business, consult the React referents and React group)

## Trigger questions

What is the area of the business?
What will be given priority?
What would be the future needs and what are they for MVP?
How important is the SEO of the page?
Do we need to sell something?
Do we use external services, what do we have to take into consideration?

### Client-Side Rendering - Vite

Client-side rendering, often associated with Single Page Application (SPA) frameworks like Vite, means most of your application logic, including routing and rendering, runs in the browser.

#### Pros:

- **Interactivity**: Since everything is already loaded and ready to run on the client's device, user interactions can feel faster and more fluid.
- **Decoupled Backend**: The API is usually a separate project, which means frontend and backend can be developed and deployed independently.
- **PWA**: Is easy to create a pwa because you have all data loaded in the user device

#### Cons:

- **SEO**: While search engines have improved at indexing client-rendered websites, server-rendered sites are generally preferable for SEO.
- **Initial Load Time**: Large applications can result in a larger initial bundle size, leading to slower initial load times.
- **Calls to back**: More calls to backend in case the calls have a cost, this would be a problem

### Use Cases:

Interactive web applications like dashboards, admin panels, other internal tools, or even some consumer-facing apps where SEO is not a priority.

### Server-Side Rendering - NextJS

Server-side rendering (SSR) basically means your server generates the HTML for each page on the server and sends that HTML to the client's browser. Common frameworks for this are Next.js and Remix.

#### Pros:

- **SEO**: SSR is great for SEO because search engine crawlers can directly parse the fully rendered HTML sent from the server.
- **Initial Page Load**: The client receives pre-rendered HTML from the server leading to faster initial page load times. This is great when we have different languages or different components/pages by country
- **Call to back cost**: In the event that calls to the back have to be limited due to cost issues, we can run crons that update the properties at a set time

#### Cons:

- **Server Load**: Every request may necessitate a new HTML page rendered on the server, leading to higher server load.
- **Developer Experience**: Dealing with SSR usually requires more consideration and care, especially when integrating with client-side libraries (interacting or including libraries that interact with doom is more difficult).
- **Use Cases**: Blogs, ecommerce sites, public-facing websites with content that needs to be crawled by search engines, or any site where initial load time is crucial.
