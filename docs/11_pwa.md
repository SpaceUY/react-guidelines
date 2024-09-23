---
title: PWA
layout: default
nav_order: 11
---

# Introduction

This document outlines the key principles and technologies for building Progressive Web Apps (PWAs). It covers essential features, caching strategies, and the advantages and disadvantages of two popular frameworks for PWA development: Vite-PWA with CapacitorJS.

## Features to Look for That Make PWAs Great:

A solid foundation is the basic requirement for creating PWAs. To implement this foundation, it is necessary to design and code according to the restrictions of the Web using some principles:

- **Use mobile devices as a focus constraint**: Make sure that each view of your design focuses only on the essential content and interactions.
- **Emphasize the content and main functionality of the design process**.
- **Progressively improve when necessary**: Start by compiling the core content and functionality of a component using the simplest and most widely available tools. Make it accessible. Then, test the advanced functions you would like to use and enhance your component with them.
- **Provide a fast and good user experience**: Focus on user-centric web performance metrics, obtain metrics from real users, and improve performance for all users, regardless of their network connection, input type, CPU, or GPU power.
- **Responsive Design**: Instead of fixing exact dimensions in pixels, use responsive design that dynamically adapts to the screen size of the device. This ensures a consistent user experience on devices of different sizes, such as mobile phones, tablets, and desktop computers.
- **Fast Loading**: PWAs benefit from fast loading times. Consider internet connection speed constraints and optimize your resources to minimize loading time. Use techniques like progressive loading, resource caching, and compression to improve efficiency.
- **Offline-Centric Design**: A key feature of PWAs is their ability to function offline. Design your application considering offline scenarios and ensure that users can access basic content and functionality even when they are not connected to the Internet.
- **Adaptability to Different Browsers**: Take into account the differences between browsers and ensure that your PWA works consistently across a variety of web browsers. This may involve using standard web features and conducting cross-browser testing.
- **Data Consumption Efficiency**: Some users may have limits on the amount of data they can consume. Design your PWA to be efficient in data usage, minimizing the amount of information transferred between the server and the client.

> 💡 When we say that the PWA should provide an offline experience, it does not mean that all services and content should be available offline. For example, a note-taking app may not synchronize its notes when there is no connectivity, but it can allow users to write the note and synchronize new changes when they come back online. At the very least, you should render the app's user interface with an appropriate message if it requires an active connection.

## Introducing Lighthouse:

Lighthouse is an open-source auditing tool by Google that helps developers improve their Progressive Web Apps (PWAs). It analyzes your PWA against various criteria and provides detailed reports with scores and actionable recommendations.

Here's what makes Lighthouse valuable for PWAs:

- **PWA-Specific Audit**: Unlike generic audits, Lighthouse offers a dedicated "Progressive Web App" audit. This audit checks for essential PWA features like service worker implementation, installability, and offline functionality.
- **Multi-Faceted Evaluation**: Lighthouse evaluates your PWA across five key categories:
  - **Performance**: How fast your PWA loads and interacts.
  - **Progressive Web App**: Compliance with PWA core principles.
  - **Best Practices**: Code quality and adherence to web development best practices.
  - **Accessibility**: How usable your PWA is for users with disabilities.
  - **SEO**: How well your PWA is optimized for search engines.
- **Actionable Insights**: Lighthouse doesn't just highlight issues, it provides specific recommendations to fix them.
- **Integration with Developer Tools**: Lighthouse is seamlessly integrated with Chrome DevTools, making it readily accessible for developers during their workflow.

### Getting Started with Lighthouse:

1. **Open Chrome DevTools**: Press F12 on your keyboard.
2. **Navigate to the Lighthouse tab**: Look for the lighthouse icon.
3. **Select the "Progressive Web App" audit**: Choose the specific PWA audit.
4. **Run the audit**: Click "Generate report".
5. **Analyze the results**: Review the scores, recommendations, and detailed breakdowns for each category.

Remember, Lighthouse is just one tool in your PWA development toolbox. Use it alongside other practices and testing methods to ensure you build high-quality, engaging PWAs.

## How Do Offline Mode and Silent Update Work?

PWAs use an API called service workers. These service workers operate on a separate thread rather than the main JS thread, so they have a set of abilities that we cannot perform within the main JS thread. For instance, with the ability to pre-cache, they can download assets from the website listed in the `sw.js` file and store them directly in the browser. This way, we can use the application offline (assuming we don't make any API calls within the application). Additionally, if you configure PWA, you can also pre-cache your API calls.

## Cache Strategies:

When implementing caching strategies for a Progressive Web App (PWA), there are several approaches you can use to optimize content loading and provide a better user experience. Here are some of the common caching strategies:

- **Cache First (Cache Falling Back to Network)**: This strategy involves checking the cache first for requested resources and serving them if they are available. If not, it falls back to the network request.

  - Ideal for: Static assets that don't change often, like CSS, images, or fonts.
  - Pros: Fast loading as it avoids network requests when possible.
  - Cons: May serve outdated content if not updated regularly.

- **Network First (Network Falling Back to Cache)**: The network is always the primary source for fetching resources. If the network request fails (e.g., offline), it falls back to the cache.

  - Ideal for: Dynamic content where freshness is more critical.
  - Pros: Ensures up-to-date content.
  - Cons: Slower load times, especially with poor connectivity.

- **Cache Only**: Forces the application to only serve cached resources and never use the network.

  - Ideal for: Essential app shell components.
  - Pros: Instant loading.
  - Cons: Content might become outdated if not managed properly.

- **Network Only**: Forces the application to always use the network for resources, bypassing the cache.

  - Ideal for: Features that don't need offline support.
  - Pros: Always fresh content.
  - Cons: Not available offline and slow when the network is poor.

- **Stale-While-Revalidate**: Serves cached content while fetching the latest version from the network in the background.

  - Ideal for: Resources where freshness is important but immediate use of the latest version is not critical.
  - Pros: Quick load times with eventual freshness.
  - Cons: Might briefly serve stale content.

- **Cache then Network**: Sends parallel requests to both the cache and the network. The app uses whichever returns first, but updates with the network version when it arrives.
  - Ideal for: User-generated content or semi-dynamic resources.
  - Pros: Fast response, with updates as they become available.
  - Cons: More complex to implement and might cause redundant data usage.

Each strategy has its trade-offs, and often a combination of these strategies is used in a real-world PWA to balance between speed, freshness, and reliability. The choice of strategy depends on the specific use case and user needs of your application. Always consider the user experience and the nature of the content when selecting a caching strategy.

## Vite Plugin:

This plugin simplifies PWA setup, for instance, precaching comes by default, eliminating the need for manual configuration.

### Service Worker Update:

The plugin automatically manages updates. When building your project, it scans your asset files and creates a list of preloaded files within the `sw.js` file. It appends a revision number for new files, allowing for cache invalidation and updates when deployed.

## Vite-PWA with CapacitorJS

### Pros:

- **Web Technology**: Utilizing familiar web technologies can speed up the development process.
- **Access to Native APIs**: CapacitorJS provides access to many native APIs, making the application more feature-rich.
- **One Codebase, Various Platforms**: PWA with CapacitorJS allows you to deploy the same app across the web, desktop, and mobile platforms.

### Cons:

- **Non-Native UI/UX**: A web-based UI can't fully match the native UI/UX of a platform, which may feel out of place for regular users.
- **Performance**: Satisfactory for less intensive tasks, but native apps will perform better for heavy operations or graphics.
- Depends on capacitor functionalities, capacity and intelligence to manage native options.
