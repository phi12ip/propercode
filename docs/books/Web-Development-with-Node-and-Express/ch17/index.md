[Prev][prev]
|
[Table of Contents](../)
|
[Next][next]

[prev]: ../ch16
[next]: ../ch18

# Static Content

Static content refers to the resources your app will be serving that don’t change on a per-request basis. Here are the usual suspects:


Multimedia

* Images, videos, and audio files. It’s quite possible to generate image files on the fly, of course (and video and audio, though that’s far less common), but most multimedia resources are static.

HTML

* If our web application is using views to render dynamic HTML, it wouldn’t generally qualify as static HTML (though for performance reasons, you may dynamically generate HTML, cache it, and serve it as a static resource). SPA applications, as we’ve seen, commonly send a single, static HTML file to the client, which is the most common reason to treat HTML as a static resource. Note that requiring the client to use an .html extension is not very modern, so most servers now allow static HTML resources to be served without the extension (so /foo and /foo.html would return the same content).

CSS

* Even if you use an abstracted CSS language like LESS, Sass, or Stylus, at the end of the day, your browser needs plain CSS, which is a static resource.1

JavaScript

* Just because the server is running JavaScript doesn’t mean there won’t be client-side JavaScript. Client-side JavaScript is considered a static resource. Of course, now the line is starting to get a bit hazy: what if there was common code that we wanted to use on the backend and client side? There are ways to solve this problem, but at the end of the day, the JavaScript that gets sent to the client is generally static.

Binary downloads

* This is the catchall category: any PDFs, ZIP files, Word documents, installers, and the like.


## Performance Considerations

 The two primary performance considerations are reducing the number of requests and reducing content size.

Of the two, reducing the number of (HTTP) requests is more critical, especially for mobile (the overhead of making an HTTP request is significantly higher over a cellular network). Reducing the number of requests can be accomplished in two ways: combining resources and browser caching.

> The importance of reducing HTTP requests will diminish over time as HTTP/2 becomes more commonplace. One of the primary improvements in HTTP/2 is request and response multiplexing, which reduces the overhead of fetching multiple resources in parallel. See “Introduction to HTTP/2” by Ilya Grigorikfor more information.


## Content Delivery Networks

A CDN is a server that’s optimized for delivering static resources. It leverages special headers (that we’ll learn about soon) that enable browser caching.

CDNs also can enable geographic optimization (often called edge caching); that is, they can deliver your static content from a server that is geographically closer to your client. While the internet is very fast indeed (not operating at the speed of light, exactly, but close enough), it is still faster to deliver data over a hundred miles than a thousand. Individual time savings may be small, but if you multiply across all of your users, requests, and resources, it adds up fast.

Most of your static resources will be referenced in HTML views (<link> elements to CSS files, <script> references to JavaScript files, <img> tags referencing images, and multimedia embedding tags). It is also common to have static references in CSS, usually the background-image property. Lastly, static resources are sometimes referenced in JavaScript, such as JavaScript code that dynamically changes or inserts <img> tags or the background-image property.

> You generally don’t have to worry about cross-domain resource sharing (CORS) when using a CDN. External resources loaded in HTML aren’t subject to CORS policy: you have to enable CORS only for resources that are loaded via Ajax (see Chapter 15).


## Designing for CDNs

Most CDNs let you configure routing rules to determine where to send incoming requests. 

### Server-Rendered Website

So far in this book, we’ve been using the Express static middleware as if it were hosting all of the static assets at the root. That is, if we put a static asset foo.png in the public directory, we reference it with the URL path /foo.png, not /static/foo.png. We could, of course, create a subdirectory static inside our existing public directory, so /public/static/foo.png would have the URL /static/foo.png but that seems a little silly. Fortunately, the static middleware saves us from that silliness. All we have to do is specify a different path when we call app.use:

``` js
app.use('/static', express.static('public'))
```

### Single-Page Applications

Single-page applications will typically be the opposite of a server-rendered website: only the API will be routed to your server (for example, any request prefixed with /api), and everything else will be rerouted to your static file store.

---

incomplete