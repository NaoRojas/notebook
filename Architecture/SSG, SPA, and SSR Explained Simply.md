# SSG, SPA, and SSR Explained Simply 

# SSG - Static Site Generation

With this approach, the server serves the client with the page's pre-rendered HTML file that was completely rendered at build time. The page doesn't use Javascript and, therefore, is not interactive (static).

## Advantages:

- The client served with a fully rendered page very quickly.

## Disadvantages:

- The page is not interactive.

## Diagram:

![SSG Diagram](https://miro.medium.com/v2/resize:fit:1400/1*c27RDnu-sww0L7JLrWND9w.png)

## When should I use this rendering method?

SSG is mostly used for sites where content doesn't change very often like blog or documentation sites.

# SPA - Single Page App

With this approach, the server serves the client with a very 'shallow' HTML, as seen in Create-React-App's default HTML file.:

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico" />
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

Well, that looks like a very empty HTML file, so what's the catch?

With this approach, the content of the page is generated using Javascript, **which runs on the client**.

## Disadvantages:

- JavaScript is one of the slowest assets that you can load per-byte, which means that our clients may see a blank page for an extended period, especially if their internet connection is slow.
- Web crawlers, which are used by search engines to determine which sites to recommend, do not execute JavaScript on your site. As a result, they will see a blank page and, therefore, will not recommend your site.

## Diagram:

![SPA Diagram](https://miro.medium.com/v2/resize:fit:1400/1*-8JbQSgnTQvfujr5sGSw8g.png)

## When should I use this rendering method?

SPA is mostly used for highly interactive sites that don't have a lot of content, much like [ExcaliDraw](https://excalidraw.com/), the site I used for creating the diagrams and the video above.

# SSR - Server Side Rendering

With this approach, the server loads and then runs the JavaScript before serving it to the client, which means our clients will see content much earlier than they would with SPA. The tradeoff made here is that after being served with the site's content, the client has to load and then run the JavaScript as well so that the page will become interactive.

## Advantages:

- Compared to SPA, the client sees content relatively faster using this rendering method.
- Compared to SPA, web crawlers will be served with the site's content, increasing the likelihood that your site will be recommended by the search engine.

## Disadvantages:

- It takes a bit longer to fully render the page (since JavaScript is being loaded twice), but usually, it's a reasonable tradeoff to make.

## Diagram:

![SSR Diagram](https://miro.medium.com/v2/resize:fit:1400/1*4A06BoqW-KjZj3dm2HPunw.png)

## When should I use this rendering method?

SSR is mostly used by sites that aren't highly interactive and do have static content.





Before the rise of Single Page Applications (SPAs), Server-Side Rendering (SSR), and Static Site Generation (SSG), websites were primarily built using a traditional server-rendered approach.

This approach involves the server generating HTML pages and sending them to the client when a user requests a new page.

HTML files rest on a server, awaiting client requests. When a client seeks a file, it traverses network protocols, obtaining DNS details and the server’s IP address. Once communication is established via protocols like TCP, the server promptly sends the requested HTML file to the client. This exchange concludes with the client receiving the essential content, ready to shape the visual presentation of the web page.

As the internet and the web grew in popularity, engineers and developers indeed sought ways to enhance the speed and efficiency of web processes.

This exploration led to the development of various technologies, frameworks, and architectural patterns, such as Single Page Applications (SPAs), Server-Side Rendering (SSR), and Static Site Generation (SSG), among others.

These innovations aimed to improve user experiences, optimize performance, and address the evolving needs of web applications in a dynamic online landscape.

# Single Page Applications (SPAs):

SPAs are web applications that load a single HTML page and dynamically update the content as the user interacts with the app.
This means that when a user visits a website that uses JavaScript, the browser will download the JavaScript files and store them locally.

This can improve the performance of the website, as the browser will not need to download the JavaScript files again each time the user visits the website.

The browser will also update the HTML with the JavaScript code on user request. This means that when a user interacts with a website that uses JavaScript, the JavaScript code will be executed and the HTML will be updated accordingly. This allows for dynamic and interactive websites.
Examples include Gmail, Facebook, and Twitter.

# Sever-side Rendering (SSG):

Server-side rendering (SSR) in web development involves the server creating the HTML for a web page prior to transmitting it to the browser. In this approach, the browser’s responsibility is solely to display the HTML, not downloading and storing as the case of SPAs, eliminating the need to parse and execute JavaScript for rendering.

This not only enhances the initial page load speed but also contributes to an overall improved user experience.

This will lead to an overall increase in app performance.

# Static Site Generation (SSG):

SSG is a web development method that pre-renders the HTML, CSS, and JavaScript for a website upfront during the build process instead of generating them dynamically for each user request.

This approach offers several advantages, including quicker page loading times and improved search engine optimization (SEO) due to the ease with which search engines can index and crawl the static content.

**How SSG works:**

1. **Content creation:** Developers create content using a markup language like Markdown or a content management system (CMS), e.g. WordPress.
2. **Data fetching:** SSG frameworks extract data from APIs or databases and integrate it into the content.
   In NextJs this data can be called using [**getStaticPaths**](https://nextjs.org/docs/pages/building-your-application/data-fetching/get-static-paths) data fetching method, which will only run during build in production, it will not be called during runtime.
3. **Templating:** The content is combined with templates to generate static HTML files.

**🤔 How to Choose?**

🚀 SPA for dynamic, real-time applications.
🌐 SSR for content-rich sites requiring SEO optimization.
🏗️ SSG for blazing-fast static content with excellent security.

**💡 Considerations:**
🚄 Evaluate performance needs.
🔍 SEO requirements.
🧩 Application complexity.

I hope you’ve gained something from this article, if yes ?
Let’s keep the conversation going!
What’s your preference and experience with these approaches? Share your insights! 🚀