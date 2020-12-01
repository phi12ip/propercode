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

So go to your application root and create a directory called server; this is where your Express server will live. Don’t create a directory for your client app; CRA will do that for us.

Before we run CRA, we should install Yarn. Yarn is a package manager like npm…actually, yarn is mostly a drop-in replacement for npm. It’s not mandatory for React development, but it is the de facto standard, and not using it would be swimming upstream. There are some minor differences in usage between Yarn and npm, but the only one you’ll probably notice is that you run yarn add instead of npm install. To install Yarn, simply follow the Yarn installation instructions.

Once you’ve installed Yarn, run the following from your application root:

``` sh
yarn create react-app client
```

Now go into your client directory and type yarn start. After a few seconds, you’ll see a new browser window pop up, with your React app running in it!


## React Basics

React has excellent documentation, which I won’t re-create here. So if you’re new to React, start with the Intro to React tutorial, and then the Main Concepts guide.

You’ll find that React is organized around components, which are the main building blocks of React.

Let’s take a look at client/src/App.js (the contents of yours may differ slightly—CRA does change over time):

``` js
import React from 'react';
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
}

export default App;
```

One of the core concepts in React is that the UI is generated by functions.

You may be looking at this and thinking that it isn’t valid JavaScript; it looks like HTML is mixed in! The reality is a little more complicated. React, by default, enables a superset of JavaScript called JSX. JSX allows you to write what looks like HTML. It’s not actually HTML; it creates React elements, and the purpose of a React element is to (eventually) correspond to a DOM element.

At the end of the day, however, you can think of it as HTML. Here, App is a function that will render the HTML corresponding to the JSX it returns.

A couple of things to note: since JSX is close to—but not exactly—HTML, there are some subtle differences. You may have already noticed we use className instead of class, which is because class is a reserved word in JavaScript.

All you have to do to specify HTML is to start an HTML element anywhere an expression is expected. You can also “go back to” JavaScript with curly braces within the HTML. For example:

``` js
const value = Math.floor(Math.random()*6) + 1
const html = <div>You rolled a {value}!</div>
```

Any valid JavaScript expression can be contained within curly brackets within JSX—including other HTML elements! A common use case of this is rendering lists:

``` js
const colors = ['red', 'green', 'blue']
const html = (
  <ul>
    {colors.map(color =>
      <li key={color}>{color}</li>
    )}
  </ul>
)
```

First, note that we mapped over our colors to return the `<li>` elements. This is critical: JSX works entirely by evaluating expressions. So the `<ul>` has to contain either an expression or an array of expressions. If you changed the map to a forEach, you would find that the `<li>` elements would not get rendered. 

Second, note that the `<li>` elements receive a property key: this is a performance concession. For React to know when to re-render the elements in an array, it needs a unique key for each element.


## The Home Page

Let’s start by focusing on what’s in the `<body>` tag (except the scripts):

``` hbs
<div class="container">
  <header>
    <h1>Meadowlark Travel</h1>
    <a href="/"><img src="/img/logo.png" alt="Meadowlark Travel Logo"></a>
  </header>
  {{{body}}}
</div>
```

The only other tricky bit is what to do about {{{body}}}? In our views, this is where another view would be rendered—the content for the specific page you’re on. We can replicate the same basic idea in React. Since all content is rendered in the form of components, we’re just going to render another component here. We’ll start with an empty Home component and build that out in a moment:

``` js
import React from 'react'
import logo from './img/logo.png'
import './App.css'

function Home() {
  return (<i>coming soon</i>)
}

function App() {
  return (
    <div className="container">
      <header>
        <h1>Meadowlark Travel</h1>
        <img src={logo} alt="Meadowlark Travel Logo" />
      </header>
      <Home />
    </div>
  )
}

export default App
```


## Routing

The core concept of routing we learned about in Chapter 14 hasn’t changed: we’re still using the URL path to determine what part of the interface the user is seeing. The difference is that it’s up to the client application to handle that.

Changing the UI based on the route is the client app’s responsibility: if the navigation requires new or updated data from the server, that’s fine, and it’s up to the client app to request that from the server.

We’ll get started by installing the DOM version of React Router (there’s also a version for React Native, for mobile development):

``` sh
yarn add react-router-dom
```

Now we’ll hook up the router, and add an About and a Not Found page. We’ll also link the site logo back to the home page:

``` js
import React from 'react'
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link
} from 'react-router-dom'
import logo from './img/logo.png'
import './App.css'

function Home() {
  return (
    <div>
      <h2>Welcome to Meadowlark Travel</h2>
      <p>Check out our "<Link to="/about">About</Link>" page!</p>
    </div>
  )
}

function About() {
  return (<i>coming soon</i>)
}

function NotFound() {
  return (<i>Not Found</i>)
}

function App() {
  return (
    <Router>
      <div className="container">
        <header>
          <h1>Meadowlark Travel</h1>
          <Link to="/"><img src={logo} alt="Meadowlark Travel Logo" /></Link>
        </header>
        <Switch>
          <Route path="/" exact component={Home} />
          <Route path="/about" exact component={About} />
          <Route component={NotFound} />
        </Switch>
      </div>
    </Router>
  )
}

export default App
```

The first thing to notice is that we’re wrapping our entire application in a `<Router>` component.

Inside `<Router>`, we can use `<Route>` to conditionally render a component based on the URL path. We’ve placed our content routes inside a `<Switch>` component: this ensures that only one of the components contained therein gets rendered.

The second thing to notice is the use of `<Link>`. You might be wondering why we don’t just use `<a>` tags. The problem with `<a>` tags is that—without some extra work—the browser will dutifully treat them as “going elsewhere” even if it’s on the same site, and it will result in a new HTTP request to the server…and the HTML and the CSS will be downloaded again, defeating the SPA agenda. It will work in the sense that when the page loads, React Router will do the right thing, but it won’t be as fast or efficient, invoking unnecessary network requests. Seeing the difference is actually an instructive exercise that should drive home the nature of SPAs. As an experiment, create two navigation elements, one using `<Link>` and another using `<a>`:

``` jsx
<Link to="/">Home (SPA)</Link>
<a href="/">Home (reload)</Link>
```


## Vacations Page—Visual Design

So far we’ve just been building a pure frontend application…so where does Express come in? Our server is still the single source of truth. In particular, it maintains the database of vacations that we want to display on our site. Fortunately, we’ve already done most of the work in Chapter 15: we exposed an API that will return our vacations in JSON format, ready for use in a React application.

In the previous section, we included all of the content pages in client/src/App.js, which is generally considered poor practice: its more conventional for each component to live in its own file. So we’ll take the time to break our Vacations component out into its own component. Create the file client/src/Vacations.js:

``` js
import React, { useState, useEffect } from 'react'
import { Link } from 'react-router-dom'

function Vacations() {
  const [vacations, setVacations] = useState([])
  return (
    <>
      <h2>Vacations</h2>
      <div className="vacations">
        {vacations.map(vacation =>
          <div key={vacation.sku}>
            <h3>{vacation.name}</h3>
            <p>{vacation.description}</p>
            <span className="price">{vacation.price}</span>
          </div>
        )}
      </div>
    </>
  )
}

export default Vacations
```

What we have so far is pretty simple: we’re just returning a `<div>` that contains additional `<div>` elements, each of which represents a vacation. So where is this vacations variable coming from? In this example, we’re using a newer feature of React, called React hooks. Prior to hooks, if a component wanted to have its own state (in this case, a list of vacations), you had to use a class implementation. 

Hooks enable us to have function-based components that have their own state. In our Vacations function, we call useState to set up our state. Note we pass an empty array to useState: that will be the initial value of vacations in state (we’ll discuss how we populate that shortly). What setState returns is an array containing the state value itself (vacations) and a way to update the state (setVacations).

You may wonder why we can’t modify vacations directly: it’s just an array, so couldn’t we call push to add vacations to it? We could, but this would be defeating the very purpose of React’s state management system, which ensures consistency, performance, and communication between components.

You may also be wondering about what looks like an empty component (<>…</>) surrounding our vacations. This is called a fragment. The fragment is necessary because every component must render a single element. In our case, we have two elements, the <h2> and the <div>. The fragment simply provides a “transparent” root element in which to contain these two elements so we can render a single element.

Let’s add our Vacations component to our application, even though there aren’t yet any vacations to show. In client/src/App.js, first import your vacations page:

``` js
import Vcations from './Vacations'
```

Then all we have to do is create a route for it in our router’s <Switch> component:

``` jsx
<Switch>
  <Route path="/" exact component={Home} />
  <Route path="/about" exact component={About} />
  <Route path="/vacations" exact component={Vacations} />
  <Route component={NotFound} />
</Switch>
```


## Vacations Page—Server Integration

We’ve already done most of the work necessary for the Vacations page; we have an API endpoint that gets vacations from the database and returns them in JSON format. Now we have to figure out how to get the server and the client communicating.

We can start with our work from Chapter 15; we don’t need to add anything to it, but we can take some things away that we no longer need. We can remove the following:

* Handlebars and views support (we’ll leave the static middleware, though, for reasons we’ll see later).

* Cookies and sessions (our SPA may still use cookies, but it no longer needs the server’s help here…and we think about sessions in a completely different way).

*  All routes that render a view (we obviously keep the API routes, however).


The configuration for CRA comes with proxy support, allowing you to pass web requests on to your API. Edit your client/package.json file, and add the following:

``` json
"proxy": "http://localhost:3033"
```

Now that that configuration is in place, let’s use an effect (another React hook) to fetch and update vacation data. Here’s the entire Vacations component with the useEffect hook:

``` js
function Vacations() {
  // set up state
  const [vacations, setVacations] = useState([])

  // fetch initial data
  useEffect(() => {
    fetch('/api/vacations')
      .then(res => res.json())
      .then(setVacations)
  }, [])

  return (
    <>
      <h2>Vacations</h2>
      <div className="vacations">
        {vacations.map(vacation =>
          <div key={vacation.sku}>
            <h3>{vacation.name}</h3>
            <p>{vacation.description}</p>
            <span className="price">{vacation.price}</span>
          </div>
        )}
      </div>
    </>
  )
}
```

As before, useState is configuring our component state to have a vacations array, with a companion setter. Now we’ve added useEffect, which calls our API to retrieve vacations, and then calls that setter asynchronously. Note that we pass in an empty array as the second argument to useEffect; this is a signal to React that this effect should be run only once, when the component is mounted. On the surface, that may seem like an odd way to signal that, but once you learn more about hooks, you’ll see that it’s actually quite consistent.

Hooks are relatively new—they were added in version 16.8 in February 2019—so even if you have some experience with React, you may not be familiar with hooks. I firmly believe that hooks are an excellent innovation in the React architecture, and, while they may seem alien at first, you’ll find that they actually simplify your components and reduce some of the trickier state-related mistakes that people commonly make.


## Sending Information to the Server

We already have an API endpoint to make changes on the server; we have an endpoint to be emailed when is back in season. Let’s go ahead and modify our Vacations component to show a sign-up form for vacations that are out of season. In true React fashion, we’ll create two new components: we’ll break out the individual vacation view into Vacation and a NotifyWhenInSeason component. We could do it all in one, but the recommended approach to React development is to have many specific-purpose components instead of gigantic multipurpose components (for the sake of brevity, however, we are going to stop short of putting these components in their own files: I’ll leave that as a reader’s exercise):

``` js 
import React, { useState, useEffect } from 'react'

function NotifyWhenInSeason({ sku }) {
  return (
    <>
      <i>Notify me when this vacation is in season:</i>
      <input type="email" placeholder="(your email)" />
      <button>OK</button>
    </>
  )
}

function Vacation({ vacation }) {
  return (
    <div key={vacation.sku}>
      <h3>{vacation.name}</h3>
      <p>{vacation.description}</p>
      <span className="price">{vacation.price}</span>
      {!vacation.inSeason &&
        <div>
          <p><i>This vacation is not currently in season.</i></p>
          <NotifyWhenInSeason sky={vacation.sku} />
        </div>
      }
    </div>
  )
}

function Vacations() {
  const [vacations, setVacations] = useState([])
  useEffect(() => {
    fetch('/api/vacations')
      .then(res => res.json())
      .then(setVacations)
  }, [])
  return (
    <>
      <h2>Vacations</h2>
      <div className="vacations">
        {vacations.map(vacation =>
          <Vacation key={vacation.sku} vacation={vacation} />
        )}
      </div>
    </>
  )
}

export default Vacations
```

Now, if you have any vacations that have inSeason as false (and you will, unless you changed your database or initialization scripts), you will update the form. Now let’s hook up our button to make the API call. Modify NotifyWhenInSeason:

``` js
function NotifyWhenInSeason({ sku }) {
  const [registeredEmail, setRegisteredEmail] = useState(null)
  const [email, setEmail] = useState('')
  function onSubmit(event) {
    fetch(`/api/vacation/${sku}/notify-when-in-season`, {
        method: 'POST',
        body: JSON.stringify({ email }),
        headers: { 'Content-Type': 'application/json' },
      })
      .then(res => {
        if(res.status < 200 || res.status > 299)
          return alert('We had a problem processing this...please try again.')
        setRegisteredEmail(email)
      })
    event.preventDefault()
  }
  if(registeredEmail) return (
    <i>You will be notified at {registeredEmail} when
    this vacation is back in season!</i>
  )
  return (
    <form onSubmit={onSubmit}>
      <i>Notify me when this vacation is in season:</i>
      <input
        type="email"
        placeholder="(your email)"
        value={email}
        onChange={({ target: { value } }) => setEmail(value)}
        />
      <button type="submit">OK</button>
    </form>
  )
}
```

We’re choosing here to have the component track two different values: the email address as the user types it, and the final value after they press OK. The former is a technique known as controlled components, and you can read more about it on the React forms documentation.

The latter we’re keeping track of so we can know when the user took the action of pressing OK so we can change the UI accordingly. We could have also had a simple boolean “registered,” but this allows our UI to remind the user what email they registered with.

We also had to do a little more work with our API communication: we had to specify the method (POST), encode the body as JSON, and specify the content type.


## State Management

Most of the architectural work that goes into planning and designing a React application is focused around state management—and not usually the state management of single components, but how they share and coordinate state. Our sample application does share some state: the Vacations component passes down a vacation object to the Vacation component, and the Vacation component in turn passes down the vacation’s SKU to the NotifyWhenInSeason listener. But so far, our information is only flowing down the tree; what happens when information needs to go back up?

The most common approach is to pass functions around that are responsible for updating state. For example, the Vacations component might have a function for modifying a vacation, which it could pass to Vacation, which could in turn be passed down to NotifyWhenInSeason. When NotifyWhenInSeason calls it to modify the vacation, Vacations, at the top of the tree, would recognize that things had changed, which would cause it to re-render, which in turns causes all of its descendants to re-render.

It sounds exhausting and complicated, and sometimes it can be, but there are techniques that can help. They are so varied and sometimes complex that we can’t completely cover them here (nor is this a book about React), but I can point you to some further reading:

Redux

* Redux is usually the first thing that comes to people’s minds when they think about comprehensive state management for React applications. It was one of the first formalized state management architectures, and it is still incredibly popular. In concept, it is extremely simple, and it is still the state management framework that I prefer. Even if you don’t end up choosing Redux, I recommend you watch the free tutorial videos by its creator, Dan Abramov.

MobX

* MobX came along after Redux. It has gained an impressive following in a short amount of time and is probably the second most popular state container, behind Redux. MobX can certainly result in code that seems easier to write, but I still feel that Redux has an edge in providing a good framework as your application scales, even with its increased boilerplate.

Apollo

* Apollo isn’t a state management library per se, but the way its used often takes the place of one. It’s essentially a frontend interface for GraphQL--an alternative to REST APIs—that offers a lot of integration with React. If you’re using GraphQL (or interested in it), it’s definitely worth looking into.

React Context

* React itself has gotten into the game by providing the Context API, now built into React. It accomplishes some of the same things that Redux does with less boilerplate. However, I feel that React Context is less robust and that Redux is a better choice for applications as they grow.

When you start out with React, you can essentially ignore the complexities of state management across your application, but pretty quickly you’ll realize the need for a more organized way to manage state. When you reach that point, you’ll want to look into some of these options and pick one that resonates with you.


## Deployment Options

So far, we’ve been using CRA’s built-in development server—which really is the best choice for development, and I recommend sticking with it. However, when it comes time for deployment, it’s not a suitable choice. Fortunately, CRA comes loaded with a build script that creates a bundle optimized for production, and then you have many options. When you’re ready to create a deployment bundle, simply run yarn build, and a build directory will be created. All of the assets in the build directory are static and can be deployed anywhere.

My current deployment of choice is to put the CRA build in an AWS S3 bucket with Static Website Hosting turned on. This is far from the only option: every major cloud provider and CDN offers something similar.

In this configuration, we have to create routing so that the API calls are routed to your Express server and your static bundle is served from a CDN. For my AWS deployments, I use AWS CloudFront to perform this routing; the static assets are served from the aforementioned S3 bucket, and the API requests are routed to either an Express server on an EC2 instance, or on a Lambda.

Another option is to let Express do the whole thing. This has the advantage of being able to consolidate your entire application onto a single server, which makes for a pretty simple deployment, and makes management easy. It may not be ideal for scalability or performance, but it’s a valid choice for small applications.

To serve your application entirely from Express, simply take contents of the build directory that was created when you ran yarn build, and copy it into the public directory in your Express application. As long as you have your static middleware linked in, it will automatically serve the index.html file, which is all you need.

