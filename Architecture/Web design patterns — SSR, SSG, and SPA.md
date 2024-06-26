## **Web design patterns — SSR, SSG, and SPA**

Modern web development sometimes looks complex. The terms SSR, SSG and SPA usually emerge in the vast majority of content out there. When I first heard about these names, I sure was confused. What are those? Which one to use? When to choose one over the other? Can multiple patterns be used in the same app? I’m going to break down these questions for you. Hopefully, at the end of this article, you will have a better understanding of them, and a finer capability to make better architectural decisions for your next project.

### SSR (Server-Side Rendering)

In **SSR**, when a web page is requested, it is rendered on the server, served to the client, and finally rendered by the client.

![Server Side Rendering steps diagram](https://miro.medium.com/v2/resize:fit:700/1*cfxPb1S-oXM49yIS8HLHuQ.png)

Why is this useful? Imagine a templating system, like the Django templating system, where you have dynamic data like below.

```
Hello {{user.name}}!
```

Using SSR, the HTML's dynamic content is evaluated at **runtime**, based on the server state, and returned to the client. So the HTML sent to the user is actually:

```
Hello John!
```

The browser (client-side) just has to render the page into the DOM and voilá, there's the page.

## Why use SSR?

The main advantages of SSR as compared to client-side rendering (CSR) are:

- **Improved performance**: SSR can improve the performance of web applications by delivering fully rendered HTML to the client, which the browser can parse and display even before it downloads the application JavaScript. This can be especially beneficial for users on low-bandwidth connections or mobile devices.
- **Improved Core Web Vitals**: SSR results in performance improvements that can be measured using [Core Web Vitals (CWV)](https://web.dev/learn-core-web-vitals/) statistics, such as reduced First Contentful Paint ([FCP](https://developer.chrome.com/en/docs/lighthouse/performance/first-contentful-paint/)) and Largest Contentful Paint ([LCP](https://web.dev/lcp/)), as well as Cumulative Layout Shift ([CLS](https://web.dev/cls/)).
- **Better SEO**: SSR can improve the search engine optimization (SEO) of web applications by making it easier for search engines to crawl and index the content of the application.

### SSG (Static-Site Generating)

![img](https://miro.medium.com/v2/resize:fit:700/1*JnrcYDCjuFw2hIx_iPY06g.png)

**SSG** has similarities with SSR. The page is also generated in the server, *however,* the page is rendered at **build time**. So, instead of rendering the page on the server upon the receival of a request, the page is already rendered in the server, waiting to be served to the client. I’ll discuss the advantages and disadvantages of this approach, and compare it with SSR further below.

Prerendering, commonly referred to as Static Site Generation (SSG), represents the method by which pages are rendered to static HTML files during the build process.

Prerendering maintains the same performance benefits of [server-side rendering (SSR)](https://angular.dev/guide/ssr#why-use-server-side-rendering) but achieves a reduced Time to First Byte (TTFB), ultimately enhancing user experience. The key distinction lies in its approach that pages are served as static content, and there is no request-based rendering.

When the data necessary for server-side rendering remains consistent across all users, the strategy of prerendering emerges as a valuable alternative. Rather than dynamically rendering pages for each user request, prerendering takes a proactive approach by rendering them in advance.

### SPA (Single-Page Application)

This approach is relatively new compared with the previous two. This is a result of the rapid growth of the internet, software, and hardware industries over the last two decades. With SPA, the server provides the user with an **empty HTML** page and **Javascript**. The latter is where the magic happens. When the browser receives the HTML + Javascript, it loads the Javascript. Once loaded, the JS takes place and, through a set of operations in the DOM, renders the necessary components to the page. The routing is then handled by the browser itself, not hitting the server. This is usually done through a frontend framework (or library) like React, Vue, or Angular.

# Server-side rendering (SSR and SSG) vs Client-side rendering (SPA)

*Note: in this section, when I mention SSR I mean both SSR and SSG, and CSR I mean SPA.*

- **SSR allows better SEO** (Search Engine Optimization) because the content doesn't have to be loaded by Javascript like in CSR, so the search engine's web crawlers can directly parse the information.
- **SSR is better for slow connections** because the HTML is immediately provided, whereas in CSR the user sees a blank page until the Javascript is loaded and renders the page's content.
- **SSR allows seeing content with Javascript disabled**. I know it sounds bizarre, however, users can have it disabled, intentionally or not. See this [link](https://kryogenix.org/code/browser/everyonehasjs.html) where some of these situations are exposed.
- **SSR first load is usually faster** because it doesn't need to fetch the whole website in a Javascript bundle as CSR does (some performance optimizations can be made in CSR to reduce this payload, like dynamic imports).
- **CSR is faster after the first load** since there are no server requests to change pages, which makes it insanely fast.
- **CSR provides a better UX (user experience)** because it gives a native-app feel to the page.

# SSR vs SSG

The difference between these two is that in SSR the server needs to render the page before sending it to the user (render at **runtime**), and with SSG this is not necessary, because it's done at **build time**. You may feel like "why on earth should I use SSR then?". It's simple: most websites usually have dynamic, stateful data (e.g.: user email/logo on the navbar when a user is logged in). This means we can't pre-render at build time templates with this data, so the only way to have this data with SSG would be to import it via Javascript (which would defeat SSG's core purpose).

# When to use what?

All three approaches have their upsides and downsides. There's no better option overall. You can potentially use all three in the same project. In this section, I'm going to give you some examples of practical cases.

## The documentation website

Imagine you're building React's documentation. What would you want in it? Surely **accessibility** to a broad audience, including users with slow internet and event javascript disabled. **SEO** would also be a concern (you would want to have the users hitting the docs when they search for something React-related). With this, we can narrow it down to either SSR or SSG. Which one to choose? **I would go for SSG**. The documentation doesn't have dynamic content, hence it would be both redundant and slower to render them on the server per request. With SSG, we would deliver the content to the client fast.

*Side note: React does actually use SSG for their docs (they use Gastby).*

## The blog

A blog has data being inserted on a regular basis. Both **SEO** and **accessibility** are requirements for a successful blog. So, we would be deciding between SSR and SSG. This time, **I would go for SSR**. This is due to the fact that we have dynamic data being inserted often, and we want to provide the user with the most up-to-date data.

## The CRM (Customer Relationship Manager)

A CRM has different requirements. **SEO isn't a concern**, since most of the pages are behind a login, hence the search engines can't reach there. Users often access the CRM in a **stable environment** (usually in the office, with good internet speed and a capable computer). Therefore, using **SPA would be my choice**. It would provide a native app feel, which the users would appreciate. Note that we can use other rendering methods in the same project. For example, we could have SSG on the login and FAQ pages.



# What is Angular Hydration?

Angular hydration is the process of converting the server-rendered HTML into a fully functional client-side Angular application. When Angular applications are server-side rendered, the initial HTML is pre-rendered on the server to improve the time-to-first-paint and enhance search engine optimization (SEO).

During hydration, Angular takes control of the pre-rendered HTML, attaches event listeners, initializes components, and establishes the client-side application state. This seamless transition from server-rendered HTML to a fully interactive client-side application is what makes hydration a crucial aspect of Angular’s SSR.

# Key Aspects of Angular Hydration:

## 1. Bootstrapping the Angular Application:

During hydration, Angular bootstraps the application on the client side. This involves initializing the Angular framework, creating the root component, and establishing the component tree based on the pre-rendered HTML received from the server.

## 2. Attaching Event Listeners:

Angular hydration involves attaching event listeners to the server-rendered HTML elements. This ensures that the client-side application responds to user interactions, providing a seamless and interactive experience.

## 3. Data Rehydration:

As part of hydration, Angular ensures that the client-side application retains the data initially rendered on the server. This ensures consistency between the server-rendered content and the client-side application state.

## 4. Optimizing Performance:

Hydration contributes to improved performance by minimizing the time needed for the client-side application to become interactive. Users experience faster page loads and smoother transitions, especially in scenarios where the server-rendered HTML provides a meaningful initial rendering.

# Why Hydration Matters:

## 1. Faster Initial Page Load:

By pre-rendering content on the server and efficiently hydrating the client-side application, Angular significantly reduces the time-to-interactive. Users experience a faster transition from loading to interacting with the application.

## 2. SEO Benefits:

Server-side rendering and hydration enhance search engine optimization by providing search engines with fully rendered HTML content. This ensures that search engines can index the content accurately, leading to improved discoverability in search results.

## 3. Improved User Experience:

Hydration contributes to a more seamless and consistent user experience. Users see a meaningful initial rendering on the server, and as they interact with the application, Angular takes over to provide a fully dynamic and responsive experience.

## 4. Progressive Enhancement:

Angular’s approach to hydration aligns with the principles of progressive enhancement. Users with slower network connections or devices benefit from the pre-rendered content, while users with more capable devices and faster connections enjoy the full client-side application experience.

# Conclusion:

Angular hydration is a fundamental aspect of server-side rendering, contributing to enhanced performance, improved SEO, and a better user experience. By understanding the role of hydration in Angular SSR, developers can unlock the full potential of server-side rendering in their Angular applications. Whether you’re building a content-heavy website or aiming for better SEO, Angular hydration stands as a powerful technique to optimize your web applications. Embrace the benefits of server-side rendering and let Angular’s hydration bring your applications to life!