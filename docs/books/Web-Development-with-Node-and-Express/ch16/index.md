[Prev][prev]
|
[Table of Contents](../)
|
[Next][next]

[prev]: ../ch15
[next]: ../ch17

# Single Page Applications

The term single-page application (SPA) is something of a misnomer, or it is at least confusing two meanings of the word “page.” SPAs, from the user’s perspective, can (and usually do) still appear to have different pages: the home page, the Vacations page, the About page, and so on. As a matter of fact, you could create a traditional server-side rendered application and an SPA that were indistinguishable to the user.

The “single page” has more to do with where and how the HTML is constructed than the user’s experience. In an SPA, the server delivers a single HTML bundle when the user first loads the application,1 and any changes in the UI (which may appear as different pages to the user) are the result of JavaScript manipulating the DOM in response to user activity or network events.

SPAs still need to communicate frequently with the server, but HTML is usually only sent as part of that first request. After that, only JSON data and static assets are transferred between the client and server.


## A Short History of Web Application Development

The way we approach web development has undergone a massive shift in the last 10 years, but one thing has remained relatively consistent: the components involved in a website or web application. Namely:

* HTML and the Document Object Model (DOM)

* JavaScript

* CSS

* Static assets (generally multimedia: images and videos, etc.)

Put together by a browser, these components are what provide the user experience.

How that experience is constructed, however, started shifting drastically around 2012. Today, the dominant paradigm for web development is single-page applications, or SPAs.

To understand SPAs, we need to understand what to contrast them with, so we’re going to go even further back in time, to 1998, the year before the term “Web 2.0” was first whispered, and eight years before jQuery was introduced.

In 1998, the dominant method for delivering web applications was for web servers to send HTML, CSS, JavaScript, and multimedia assets in response to every request. Imagine you’re watching TV, and you want to change the channel. The metaphorical equivalent here is that you would have to throw away your TV, go buy another one, schlep it into your house, and set it up—just to change the channel (navigate to a different page, even on the same site).

The problem with this approach is that there’s a lot of overhead involved. Sometimes the HTML—or large chunks of it—wouldn’t change at all. The CSS changed even less.

In 1999, the term “Web 2.0” was coined to try to describe the richness of experience that people were beginning to expect from websites. The years between 1999 and 2012 saw technological advancements that were laying the groundwork for SPAs.

In this period from 1999 to 2012, pages were still generally pages: when you first went to a website, you got the HTML, the CSS, and the static assets. When you navigated to a different page, you would get different HTML, different static assets, and sometimes different CSS. However, on each page, the page itself might change in response to user interaction, and instead of asking the server for a whole new application, JavaScript would change the DOM directly. If information needed to be fetched from the server, that information was sent in XML or JSON, without all the attendant HTML. It was, once again, up to the JavaScript to interpret the data and change the user interface accordingly. In 2006, jQuery was introduced, which significantly eased the burden of DOM manipulation and dealing with network requests.

Many of these changes were being driven by the increasing power of computers and—by extension—browsers, Web developers were finding that more and more of the work to make a website or web application look pretty could be done directly on the user’s computer instead of being done on the server and then sent to the user.

This shift in approach went into overdrive in the late 2000s, when smartphones were introduced. Now, not only were browsers capable of doing more, but people wanted to access web applications over wireless networks. Suddenly, the overhead cost of sending data went up, making it even more attractive to ship as little as possible over the network, and let the browser do as much work as possible.

By 2012, it was common practice to try to send as little information as possible over the network, and do as much as possible in the browser. Like the primordial soup giving rise to the first life, this rich environment provided the conditions for the natural evolution of the this technique: the single-page application.

The idea is simple enough: for any given web application, the HTML, JavaScript, and CSS (if any) are shipped exactly once.

Once the browser has the HTML, it is up to the JavaScript to make all changes to the DOM to make the user feel that they are navigating to a different page. No more does the server need to send different HTML when you navigate from the home page to the Vacations page, for example.

Of course the server is still involved: it’s still responsible for providing up-to-date data, and being the “single source of truth” in a multiuser application. But in an SPA architecture, the way the application appears to the user is no longer the concern of the server: it’s the concern of JavaScript and the frameworks that enable this clever illusion.

Interestingly, there are still valid reasons for a web application to be able to serve a specific page (instead of the “generic” page, which will be reformatted by the browser). While this may seem like we are coming full-circle, or throwing away the gains of SPAs, the technique to do this better mirrors the architecture of SPAs. Called server-side rendering (SSR), this technique allows the servers to use the same code that the browser uses to create individual pages to increase first-page load. The key here is that the server doesn’t have to do much thinking: it simply uses the same techniques as the browser to generate a specific page. This kind of SSR is usually done to enhance first-page loading experience, and to support search engine optimization. It’s a more advanced topic that we won’t be covering here, but you should be aware of the practice.


## SPA Technologies

There are many choices for SPA technologies now:

React

* For the moment, React seems to be the king of the SPA hill, though there are former greats (Angular) on one side of it, and ambitious usurpers (Vue) on the other side. Sometime in 2018, React surpassed Angular in usage statics. React is an open source library, but it started its life as a Facebook project, and Facebook is still an active contributor. We’ll be using React for our Meadowlark Travel refactor.

Angular

* By most accounts, the “original” SPA, Google’s Angular became massively popular but was eventually dethroned by React. In late 2014, Angular announced version 2, which was a massive change from the first version, and alienated many existing users and scared off new ones. I believe this shift (while probably necessary) contributed to React eventually outpacing Angular. Another reason is that Angular is a much larger framework than React. This has advantages and disadvantages: Angular provides a much more complete architecture for building full applications, and there’s always a clear “Angular way” to do things, whereas frameworks like React and Vue leave a lot more up to personal choice and creativity. Regardless of which approach is better, bigger frameworks are more ponderous and slow to evolve, which gave React an innovation edge.

Vue.js

* An upstart challenger to React, and the brainchild of a single developer, Evan You. In a remarkably short time, it has gained an impressive following, and it is extremely well-liked by its adherents, but it is still far behind React’s runaway popularity. I have had some experience with Vue, and I appreciate its clear documentation and lightweight approach, but I have come to prefer React’s architecture and philosophy.

Ember

* Like Angular, Ember offers a comprehensive application framework. There’s a large and active development community and, while not as innovative as React or Vue, it offers a lot of functionality and clarity. I have found I far prefer lighter frameworks, and have stuck with React for this reason.

Polymer

* I have no experience with Polymer, but it is backed by Google, which lends it credibility. People seem to be curious about what Polymer is bringing to the table, but I haven’t seen a lot of people rushing to adopt it.


## Creating a React App

The best way to get started with a React app is to use the create-react-app (CRA) utility, which creates all of the boilerplate, developer tooling, and provides a minimal starter application that you can build on. Furthermore, create-react-app will keep its configuration up-to-date so you can focus on building your application instead of on framework tooling. That said, if you ever reach the point where you need to configure your tooling, you can “eject” your application: you’ll lose the ability to keep up-to-date with the latest CRA tooling, but you’ll have full control over all of the application configuration.

Unlike what we’ve been doing so far, where all of our application artifacts lived alongside our Express application, SPAs are best thought of as a completely separate, independent application. 

To that end, we’ll have two application roots instead of one. For clarity, when I’m referring to the directory where your Express application lives, I’ll say the server root, and for the directory where your React application lives, I’ll say the client root. The application root is where both of those directories now live.

