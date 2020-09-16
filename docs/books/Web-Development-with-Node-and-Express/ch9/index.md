[Prev][prev]
|
[Table of Contents](../)
|
[Next][next]

[prev]: ../ch8
[next]: ../ch10

# Cookies and Sessions

In this chapter, you’ll learn how to use cookies and sessions to provide a better experience to your users by remembering their preferences from page to page, and even between browser sessions.

HTTP is a stateless protocol.

That means that when you load a page in your browser and then you navigate to another page on the same website, neither the server nor the browser has any intrinsic way of knowing that it’s the same browser visiting the same site.

Cookies, unfortunately, have gotten a bad name thanks to the nefarious things that people have done with them. This is unfortunate because cookies are really quite essential to the functioning of the “modern web” (although HTML5 has introduced some new features, like local storage, that could be used for the same purpose).

The idea of a cookie is simple: the server sends a bit of information, and the browser stores it for some configurable period of time.

Often it’s just a unique ID number that identifies a specific browser so that the illusion of state can be maintained.

There are some important things you need to know about cookies:

Cookies are not secret from the user

The user can delete or disallow cookies

Regular cookies can be tampered with

Cookies can be used for attacks

Users will notice if you abuse cookies

Prefer sessions over cookies

* For the most part, you can use sessions to maintain state, and it’s generally wise to do so. It’s easier, you don’t have to worry about abusing your users’ storage, and it can be more secure. Sessions rely on cookies, of course, but with sessions, Express will be doing the heavy lifting for you.

Cookies are not magic: when the server wants the client to store a cookie, it sends a header called Set-Cookie containing name/value pairs, and when a client sends a request to a server for which it has cookies, it sends multiple Cookie request headers containing the value of the cookies.

## Externalizing Credentials

To make cookies secure, a cookie secret is necessary. The cookie secret is a string that’s known to the server and used to encrypt secure cookies

To that end, we’re going to externalize our credentials in a JSON file. Create a file called .credentials.development.json:

``` json
{
  "cookieSecret": "...your cookie secret goes here"
}
```

We’re going to add a layer of abstraction on top of this credentials file to make it easier to manage our dependencies as our application grows. Our version will be very simple. Create a file called config.js:

``` js 
const env = process.env.NODE_ENV || 'development'
const credentials = require(`./.credentials.${env}`)
module.exports = { credentials }
```

Now, to make sure we don’t accidentally add credentials to our repository, add .credentials.* to your .gitignore file. To import your credentials into your application, all you need to do is this:

``` js
const { credentials } = require('./config')
```

## Cookies in Express

Before you start setting and accessing cookies in your app, you need to include the cookie-parser middleware. First, use `npm install cookie-parser`,

Then add the following to the server file:

``` js
const cookieParser = require('cookie-parser')
app.use(cookieParser(credentials.cookieSecret))
```

Once you’ve done this, you can set a cookie or a signed cookie anywhere you have access to a response object:

``` js
res.cookie('monster', 'nom nom')
res.cookie('signed_monster', 'nom nom', { signed: true })
```

To retrieve the value of a cookie (if any) sent from the client, just access the cookie or signedCookie properties of the request object:

``` js
const monster = req.cookies.monster
const signedMonster = req.signedCookies.signed_monster
```

> You can use any string you want for a cookie name. For example, we could have used \'signed monster' instead of \'signed_monster', but then we would have to use the bracket notation to retrieve the cookie: req.signedCookies[\'signed monster']. For this reason, I recommend using cookie names without special characters.

To delete a cookie, use req.clearCookie:

``` js
res.clearCookie('monster')
```

When you set a cookie, you can specify the following options:

---

domain

* Controls the domains the cookie is associated with; this allows you to assign cookies to specific subdomains. Note that you cannot set a cookie for a different domain than the server is running on; it will simply do nothing.

path

* Controls the path this cookie applies to. Note that paths have an implicit wildcard after them; if you use a path of / (the default), it will apply to all pages on your site. If you use a path of /foo, it will apply to the paths /foo, /foo/bar, etc.

maxAge

* Specifies how long the client should keep the cookie before deleting it, in milliseconds. If you omit this, the cookie will be deleted when you close your browser. (You can also specify a date for expiration with the expires option, but the syntax is frustrating. I recommend using maxAge.)

secure

* Specifies that this cookie will be sent only over a secure (HTTPS) connection.

httpOnly

* Setting this to true specifies the cookie will be modified only by the server. That is, client-side JavaScript cannot modify it. This helps prevent XSS attacks.

signed

* Setting this to true signs this cookie, making it available in res.signedCookies instead of res.cookies. Signed cookies that have been tampered with will be rejected by the server, and the cookie value will be reset to its original value.


## Examining Cookies

In Chrome, open the developer tools, and select the Application tab. In the tree on the left, you’ll see Cookies. Expand that, and you’ll see the site you’re currently visiting listed. Click that, and you will see all the cookies associated with this site. You can also right-click the domain to clear all cookies or right-click an individual cookie to remove it specifically.

## Sessions

Sessions are really just a more convenient way to maintain state. To implement sessions, something has to be stored on the client; otherwise, the server wouldn’t be able to identify the client from one request to the next. The usual method of doing this is a cookie that contains a unique identifier.

The server then uses that identifier to retrieve the appropriate session information.

Cookies aren’t the only way to accomplish this: during the height of the “cookie scare” (when cookie abuse was rampant), many users were simply turning off cookies, and other ways to maintain state were devised, such as decorating URLs with session information. 

These techniques were messy, difficult, and inefficient, and they are best left in the past. HTML5 provides another option for sessions called local storage, which offers an advantage over cookies if you need to store larger amounts of data. See the MDN documentation for Window.localStorage for more information about this option.

Broadly speaking, there are two ways to implement sessions: store everything in the cookie or store only a unique identifier in the cookie and everything else on the server.

The former are called cookie-based sessions and merely represent a convenience over using cookies. However, it still means that everything you add to the session will be stored on the client’s browser, which is an approach I don’t recommend. I recommend this approach only if you know that you will be storing just a small amount of information, that you don’t mind the user having access to the information, and that it won’t be growing out of control over time. If you want to take this approach, see the cookie-session middleware.

## Memory Stores

If you would rather store session information on the server, which I recommend, you have to have somewhere to store it. The entry-level option is memory sessions. They are easy to set up, but they have a huge downside: when you restart the server (which you will be doing a lot of over the course of this book!), your session information disappears. 

Even worse, if you scale out by having multiple servers (see Chapter 12), a different server could service a request every time; session data would sometimes be there, and sometimes not. This is clearly an unacceptable user experience. 

However, for our development and testing needs, it will suffice. We’ll see how to permanently store session information in Chapter 13.

First, install express-session (npm install express-session); then, after linking in the cookie parser, link in express-session (ch09/meadowalrk.js in the companion repo):

``` js
const expressSession = require('express-session')
// make sure you've linked in cookie middleware before
// session middleware!
app.use(expressSession({
    resave: false,
    saveUninitialized: false,
    secret: credentials.cookieSecret,
}))
```

The express-session middleware accepts a configuration object with the following options:

---

resave

* Forces the session to be saved back to the store even if the request wasn’t modified. Setting this to false is generally preferable; see the express-session documentation for more information.

saveUninitialized

* Setting this to true causes new (uninitialized) sessions to be saved to the store, even if they haven’t been modified. Setting this to false is generally preferable and is required when you need to get the user’s permission before setting a cookie. See the express-session documentation for more information.

secret

*  The key (or keys) used to sign the session ID cookie. This can be the same key used for cookie-parser.

key

* The name of the cookie that will store the unique session identifier. Defaults to connect.sid.

store

* An instance of a session store. Defaults to an instance of MemoryStore, which is fine for our current purposes. We’ll see how to use a database store in Chapter 13.

cookie

* Cookie settings for the session cookie (path, domain, secure, etc.). Regular cookie defaults apply.


## Using Sessions

Once you’ve set up sessions, using them couldn’t be simpler; just use properties of the request object’s session variable:

``` js
req.session.userName = 'Anonymous'
const colorScheme = req.session.colorScheme || 'dark'
```

Note that with sessions, we don’t have to use the request object for retrieving the value and the response object for setting the value; it’s all performed on the request object. (The response object does not have a session property.) To delete a session, you can use JavaScript’s delete operator:

``` js
req.session.userName = null       // this sets 'userName' to null,
                                  // but doesn't remove it

delete req.session.colorScheme    // this removes 'colorScheme'
```

## Using Sessions to Implement Flash Messages

Flash messages (not to be confused with Adobe Flash) are simply a way to provide feedback to users in a way that’s not disruptive to their navigation.

The easiest way to implement flash messages is to use sessions

We’ll be using Bootstrap’s alert messages to display our flash messages, so make sure you have Bootstrap linked in (see Bootstrap’s “getting started” documentation; you can link in the Bootstrap CSS and JavaScript files in your main template—there is an example in the companion repo).

In your template file, somewhere prominent (usually directly below your site’s header), add the following:

``` hbs
{{#if flash}}
  <div class="alert alert-dismissible alert-{{flash.type}}">
    <button type="button" class="close"
      data-dismiss="alert" aria-hidden="true">&times;</button>
    <strong>{{flash.intro}}</strong> {{{flash.message}}}
  </div>
{{/if}}
```

Note that we use three curly brackets for flash.message; this will allow us to provide some simple HTML in our messages (we might want to emphasize words or include hyperlinks). 

Now let’s add some middleware to add the flash object to the context if there’s one in the session. After we’ve displayed a flash message once, we want to remove it from the session so it isn’t displayed on the next request. 

We’ll create some middleware to check the session to see whether there’s a flash message and, if there is, transfer it to the res.locals object, making it available to the views. We’ll put our middleware in a file called lib/middleware/flash.js:

``` js
module.exports = (req, res, next) => {
  // if there's a flash message, transfer
  // it to the context, then clear it
  res.locals.flash = req.session.flash
  delete req.session.flash
  next()
})
```

And in our meadowalrk.js file, we’ll link in the flash message middleware, before any of our view routes:

``` js
const flashMiddleware = require('./lib/middleware/flash')
app.use(flashMiddleware)
```

Now let’s see how to actually use the flash message. Imagine we’re signing up users for a newsletter and we want to redirect them to the newsletter archive after they sign up. This is what our form handler might look like:

``` js
// slightly modified version of the official W3C HTML5 email regex:
// https://html.spec.whatwg.org/multipage/forms.html#valid-e-mail-address
const VALID_EMAIL_REGEX = new RegExp('^[a-zA-Z0-9.!#$%&\'*+\/=?^_`{|}~-]+@' +
  '[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?' +
  '(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)+$')

app.post('/newsletter', function(req, res){
    const name = req.body.name || '', email = req.body.email || ''
    // input validation
    if(VALID_EMAIL_REGEX.test(email)) {
      req.session.flash = {
        type: 'danger',
        intro: 'Validation error!',
        message: 'The email address you entered was not valid.',
      }
      return res.redirect(303, '/newsletter')
    }
    // NewsletterSignup is an example of an object you might create; since
    // every implementation will vary, it is up to you to write these
    // project-specific interfaces. This simply shows how a typical
    // Express implementation might look in your project.
    new NewsletterSignup({ name, email }).save((err) => {
        if(err) {
          req.session.flash = {
            type: 'danger',
            intro: 'Database error!',
            message: 'There was a database error; please try again later.',
          }
          return res.redirect(303, '/newsletter/archive')
        }
        req.session.flash = {
          type: 'success',
          intro: 'Thank you!',
          message: 'You have now been signed up for the newsletter.',
        };
        return res.redirect(303, '/newsletter/archive')
    })
})
```

Note that we’re careful to distinguish between input validation and database errors. Remember that even if we do input validation on the frontend (and you should), you should also perform it on the backend, because malicious users can circumvent frontend validation.

Flash messages are a great mechanism to have available in your website, even if other methods are more appropriate in certain areas (for example, flash messages aren’t always appropriate for multiform “wizards” or shopping cart checkout flows). 

Flash messages are also great during development, because they are an easy way to provide feedback, even if you replace them with a different technique later. Adding support for flash messages is one of the first things I do when setting up a website, and we’ll be using this technique throughout the rest of the book.

> Because the flash message is being transferred from the session to res.locals.flash in middleware, you have to perform a redirect for the flash message to be displayed. If you want to display a flash message without redirecting, set res.locals.flash instead of req.session.flash.


## What to Use Sessions For

Sessions are useful whenever you want to save a user preference that applies across pages. Most commonly, sessions are used to provide user authentication information: you log in, and a session is created. After that, you don’t have to log in again every time you reload the page.

Sessions can be useful even without user accounts, though. It’s quite common for sites to remember how you like things sorted or what date format you prefer—all without your having to log in.

While I encourage you to prefer sessions over cookies, it’s important to understand how cookies work (especially because they enable sessions to work). It will help you with diagnosing issues and understanding the security and privacy considerations of your application.

