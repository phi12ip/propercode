
[Table of Contents](../)
|
[Next][next]

[next]: ../ch2

# Introducing Express

## The JavaScript Revolution

Many people are just now starting to take JavaScript seriously, even though the language as we know it now has been around since 1996 (although many of its more attractive features were added in 2005).

In 2009, years after people had started to realize the power and expressiveness of JavaScript as a browser scripting language, Ryan Dahl saw JavaScript’s potential as a server-side language, and Node.js was born. This was a fertile time for internet technology. Ruby (and Ruby on Rails) took some great ideas from academic computer science, combined them with some new ideas of its own, and showed the world a quicker way to build websites and web applications. Microsoft, in a valiant effort to become relevant in the internet age, did amazing things with .NET and learned not only from Ruby and JavaScript but also from Java’s mistakes, while borrowing heavily from the halls of academia.

## Introducing Express

Express is:
- minimal
- flexible
- web application framework
- fast
- unopinionated

## Server-Side and Client-Side Applications

A server-side application is one where the pages in the application are rendered on the server (as HTML, CSS, images and other multimedia assets, and JavaScript) and sent to the client. 

A client-side application, by contrast, renders most of its own user interface from an initial application bundle that is sent only once. That is, once the browser receives the initial (generally very minimal) HTML, it uses JavaScript to modify the DOM dynamically and doesn’t need to rely on the server to display new pages (though raw data usually still comes from the server).

Prior to 1999, server-side applications were the standard. As a matter of fact, the term web application was officially introduced that year. I think of the period roughly between 1999 and 2012 as the Web 2.0 era, during which the technologies and techniques that would eventually become client-side applications were being developed. 

By 2012, with smartphones firmly entrenched, it was common practice to send as little information as possible over the network, a practice that favored client-side applications.

Server-side applications are often called server-side rendered (SSR), and client-side applications are usually called single-page applications (SPAs).

In reality, there are many blurred lines between server-side applications and client-side applications. Many client-side applications have two to three HTML bundles that can be sent to that client (for example, the public interface and the logged-in interface, or a regular interface and an admin interface). Furthermore, SPAs are often combined with SSR to increase first-page-load performance and aid in search engine optimization (SEO).

In general, if the server sends a small number of HTML files (generally one to three), and the user experiences a rich, multiview experience based on dynamic DOM manipulation, we consider that client-side rendering. The data (usually in the form of JSON) and multimedia assets for different views generally still come from the network.

While SPAs have definitively “won” as the predominant web application architecture, this book begins with examples consistent with server-side applications. They are still relevant, and the conceptual difference between serving one HTML bundle or many is small. There is an SPA example in Chapter 16.

## Node

In a way, Node has a lot in common with other popular web servers, like Microsoft’s Internet Information Services (IIS) or Apache. What is more interesting, though, is how it differs, so let’s start there.

Much like Express, Node’s approach to web servers is very minimal. Unlike IIS or Apache, which a person can spend many years mastering, Node is easy to set up and configure. That is not to say that tuning Node servers for maximum performance in a production setting is a trivial matter; it’s just that the configuration options are simpler and more straightforward.

Node is single threaded.

In terms of the way apps are written, Node apps have more in common with PHP or Ruby apps than .NET or Java apps.

While the JavaScript engine that Node uses (Google’s V8) does compile JavaScript to native machine code (much like C or C++), it does so transparently, so from the user’s perspective, it behaves like a purely interpreted language

## The Node Ecosystem

NPM. 

## Licensing

MIT License (MIT)

* painlessly permissive, allowing you to do almost anything you want, including use the package in closed source software

GNU General Public License (GPL)

* The GPL is a popular open source license that has been cleverly crafted to keep software free. That means if you use GPL-licensed code in your project, your project must also be GPL licensed. Naturally, this means your project can’t be closed source.

Apache 2.0

* This license, like MIT, allows you to use a different license for your project, including a closed source license. You must, however, include notice of components that use the Apache 2.0 license.

Berkeley Software Distribution (BSD)

* Similar to Apache, this license allows you to use whatever license you wish for your project, as long as you include notice of the BSD-licensed components.