

# SSR — Server Side Rendering in Angular

To understand what SSR means, let’s remember that in a normal single page application, we usually bring the data to the client and then build the HTML that represents that data last second on the client side.

In a **single-page app** (SPA) using client-side rendering, the client application generates HTML within the browser using JavaScript. When the client app sends an initial request, the web server responds with a minimal HTML file to serve as the app container. The browser then proceeds to download and execute the JavaScript bundlers referenced in the HTML file to bootstrap the app.

CSR has a few disadvantages, including:

- **Blank page in initial load time**: There is a delay before the JavaScript bundle is downloaded and the app is fully bootstrapped. During this time, users may see a blank page, which impact the user experience
- **Non-SEO friendly**: Webpages relying on CSR mostly contain minimal HTML with links to the JavaScript bundle, so web crawlers may have difficulty in indexing the page content, which may result in reduced visibility in search engine results.

SSR, however, addresses both the blank page issue and SEO concerns.

Using SSR, Angular render an application on the *server*, generating *static* HTML content, which represents an application state. Once the HTML content is rendered in a browser, Angular bootstraps an application and reuses the information available in the server-generated HTML.

There are a few reasons you may want to use Server-Side Rendering with your Angular application.

- It provides a side better user experience by **decreasing the initial page load time**.
- As the application will load faster, the search engine bots will crawl and index the pages better than before. Thus, gives a **better SEO experience**.
- Provides **an optimum solution for users having a slow internet connection**.
- A pleasant preview will be seen on copying a page link and sending it to others or posting it on social media, consisting of the page title, image, and description. It helps in **Social Media Optimization** (SMO).

# SSR with Angular

Angular supports SSR through its Angular Universal package, a nickname for server-side rendering in Angular that refers to its ability to render at both the client and server sides.

First introduced in Angular 4, Angular Universal provides server-side rendering, including dynamic server rendering as well as static pre-rendering. However, this form of SSR had a few limitations due to its “destructive” nature.

To better understand how Angular Universal works, we need to have a look at the concept of hydration.

# Angular’s destructive hydration

Angular hydration is the process of adding interactivity to a server-side rendered HTML page by adding event listeners and states. Efficient hydration is critical to user experience, but it is also challenging to implement as there are many moving parts to manage.

Before Angular 16, the hydration process in Angular Universal is destructive hydration.When a pre-Angular 16 app starts, the following events take place:

1. The browser sends a request to a web server
2. The web server immediately returns the DOM structure markup of the webpage
3. The browser renders the initial version of the page markup, but it isn’t interactive yet, as the JavaScript hasn’t loaded
4. The JavaScript bundles are downloaded in the browser
5. The Angular client app takes over, loads the bundle, and is bootstrapped
6. The whole page is reloaded

The above process is called destructive hydration because the client app discards the pre-rendered HTML and reloads the entire page.

It is worth noting that in step 3, the page displays some content known as the First Meaningful Paint (FMP). A quicker FMP is one of the primary benefits of SSR, particularly for performance-critical apps.

The problem of destructive hydration is page flickering, which happens when the server-side-rendered markup is replaced by the client-side rendered content. Multiple issues (i.e., [#13446](https://github.com/angular/angular/issues/13446) & #[23427](https://github.com/angular/angular/issues/23427)) have been opened in the Angular GitHub repo to resolve this, and it is considered to be a main limitation of the Angular Universal.

# Angular 16: Non-destructive hydration

In v16, Angular solves this problem by the introduction of non-destructive hydration. In non-destructive hydration, existing server-side-rendered DOM markup is reused. That means the server-side-rendered DOM markup isn’t destroyed; instead, Angular will traverse through the DOM structure, attach the event listeners, and bind the data to complete rendering.

Below is a comparison diagram to illustrate the difference between destructive and non-destructive hydration:

![img](https://miro.medium.com/v2/resize:fit:1280/1*pWFBvqDeCBwG8QN0BoZXew.jpeg)

Another related improvement is that `HttpClient` has been updated to enable server-side request caching. This enhancement prevents redundant data retrieval on the client side by caching previously fetched data, resulting in better performance.

# Applying SSR to an existing Angular app

Assuming you have an existing Angular 16 app, we just need to run the following command to enable server-side rendering:

```
ng add @nguniversal/express-engine
```

This command will update your app to add a server-side application for SSR support.

To enable non-destructive hydration, we need to import the `provideClientHydration `function as the provider of `AppModule`:

```
import {provideClientHydration} from '@angular/platform-browser';
// ...

@NgModule({
 // ...
 providers: [ provideClientHydration() ],  // add this line
 bootstrap: [ AppComponent ]
})
export class AppModule {
 // ...
}
```

If you open `package.json`, you will find a few new commands were added:

```
"dev:ssr": "ng run angularSSR:serve-ssr",
   "serve:ssr": "node dist/angularSSR/server/main.js",
   "build:ssr": "ng build && ng run angularSSR:server",
   "prerender": "ng run angularSSR:prerender"
```

To test server-side rendering in your local, run this command:

```
npm run dev:ssr
```

The app is running in SSR mode now!

> *It* ***is worth noting that the new SSR feature is available in developer preview, which means that although the feature is fully functional and polished, the APIs may change at any time, including possible breaking changes.\***

Lastly, not every application warrants server-side rendering. Rather than explaining it myself, I would recommend reading [this article](https://blog.usejournal.com/when-should-i-server-side-render-c2a383ff2d0f) for a thorough overview of when SSR is right for your project.