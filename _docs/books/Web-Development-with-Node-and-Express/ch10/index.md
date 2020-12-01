[Prev][prev]
|
[Table of Contents](../)
|
[Next][next]

[prev]: ../ch9
[next]: ../ch11

# Middleware

Conceptually, middleware is a way to encapsulate functionality—specifically, functionality that operates on an HTTP request to your application.

Practically, middleware is simply a function that takes three arguments: a request object, a response object, and a next() function

> There is also a form that takes four arguments, for error handling, covered later

Middleware is executed in what’s known as a pipeline.

You can imagine a physical pipe, carrying water. The water gets pumped in at one end, and then there are gauges and valves before the water gets where it’s going. The important part about this analogy is that order matters; if you put a pressure gauge before a valve, it has a different effect than if you put the pressure gauge after the valve. Similarly, if you have a valve that injects something into the water, everything “downstream” from that valve will contain the added ingredient. 

In an Express app, you insert middleware into the pipeline by calling app.use.

Prior to Express 4.0, the pipeline was complicated by your having to link in the router explicitly. Depending on where you linked in the router, routes could be linked in out of order, making the pipeline sequence less clear when you mix middleware and route handlers. 

In Express 4.0, middleware and route handlers are invoked in the order in which they were linked in, making the sequence much clearer.

It’s common practice to have the last middleware in your pipeline be a catchall handler for any request that doesn’t match any other routes. This middleware usually returns a status code of 404 (Not Found).

If you don’t call `next()`, the request terminates with that middleware and you should send a response to the client (res.send, res.json, res.render, etc.); if you don’t, the client will hang and eventually time out.


If you do call `next()`, it’s generally inadvisable to send a response to the client. If you do, middleware or route handlers further down the pipeline will be executed, but any client responses they send will be ignored.

## Middleware Examples

If you want to see this in action, let’s try some really simple middleware:

``` js
app.use((req, res, next) => {
  console.log(`processing request for ${req.url}....`)
  next()
})

app.use((req, res, next) => {
  console.log('terminating request')
  res.send('thanks for playing!')
  // note that we do NOT call next() here...this terminates the request
})

app.use((req, res, next) => {
  console.log(`whoops, i'll never get called!`)
})
```

Now let’s consider a more complicated, complete example (ch10/01-routing-example.js in the companion repo):

``` js
const express = require('express')
const app = express()

app.use((req, res, next) => {
  console.log('\n\nALLWAYS')
  next()
})

app.get('/a', (req, res) => {
  console.log('/a: route terminated')
  res.send('a')
})
app.get('/a', (req, res) => {
  console.log('/a: never called');
})
app.get('/b', (req, res, next) => {
  console.log('/b: route not terminated')
  next()
})
app.use((req, res, next) => {
  console.log('SOMETIMES')
  next()
})
app.get('/b', (req, res, next) => {
  console.log('/b (part 2): error thrown' )
  throw new Error('b failed')
})
app.use('/b', (err, req, res, next) => {
  console.log('/b error detected and passed on')
  next(err)
})
app.get('/c', (err, req) => {
  console.log('/c: error thrown')
  throw new Error('c failed')
})
app.use('/c', (err, req, res, next) => {
  console.log('/c: error detected but not passed on')
  next()
})

app.use((err, req, res, next) => {
  console.log('unhandled error detected: ' + err.message)
  res.send('500 - server error')
})

app.use((req, res) => {
  console.log('route not handled')
  res.send('404 - not found')
})

const port = process.env.PORT || 3000
app.listen(port, () => console.log( `Express started on http://localhost:${port}` +
  '; press Ctrl-C to terminate.'))
```

Note also that a module can export a function, which can in turn be used directly as middleware. For example, here’s a module called lib/tourRequiresWaiver.js (Meadowlark Travel’s rock-climbing packages require a liability waiver):

``` js
module.exports = (req,res,next) => {
  const { cart } = req.session
  if(!cart) return next()
  if(cart.items.some(item => item.product.requiresWaiver)) {
    cart.warnings.push('One or more of your selected ' +
      'tours requires a waiver.')
  }
  next()
}
```

We could link this middleware in like so (ch10/02-item-waiver.example.js in the companion repo):

``` js
const requiresWaiver = require('./lib/tourRequiresWaiver')
app.use(requiresWaiver)
```

More commonly, though, you would export an object that contains properties that are middleware. For example, let’s put all of our shopping cart validation code in lib/cartValidation.js:

``` js
module.exports = {

  resetValidation(req, res, next) {
    const { cart } = req.session
    if(cart) cart.warnings = cart.errors = []
    next()
  },

  checkWaivers(req, res, next) {
    const { cart } = req.session
    if(!cart) return next()
    if(cart.items.some(item => item.product.requiresWaiver)) {
      cart.warnings.push('One or more of your selected ' +
        'tours requires a waiver.')
    }
    next()
  },

  checkGuestCounts(req, res, next) {
    const { cart } = req.session
    if(!cart) return next()
    if(cart.items.some(item => item.guests > item.product.maxGuests )) {
      cart.errors.push('One or more of your selected tours ' +
        'cannot accommodate the number of guests you ' +
        'have selected.')
    }
    next()
  },

}
```

Then you could link the middleware in like this (ch10/03-more-cart-validation.js in the companion repo):

``` js
const cartValidation = require('./lib/cartValidation')

app.use(cartValidation.resetValidation)
app.use(cartValidation.checkWaivers)
app.use(cartValidation.checkGuestCounts)
```


## Common Middleware

While there are thousands of middleware projects on npm, there are a handful that are common and fundamental, and at least some of these will be found in every non-trivial Express project. Some of this middleware was so common that it was actually bundled with Express, but it has long since been moved into individual packages. The only middleware still bundled with Express itself is static.

---

This list attempts to cover the most common middleware:

basicauth-middleware

    Provides basic access authorization. Keep in mind that basic auth offers only the most basic security, and you should use basic auth only over HTTPS (otherwise, usernames and passwords are transmitted in the clear). You should use basic auth only when you need something quick and easy and you’re using HTTPS.
body-parser

    Provides parsing for HTTP request bodies. Provides middleware for parsing both URL-encoded and JSON-encoded bodies, as well as others.
busboy, multiparty, formidable, multer

    All of these middleware options parse request bodies encoded with multipart/form-data.
compression

    Compresses response data with gzip or deflate. This is a good thing, and your users will thank you, especially those on slow or mobile connections. It should be linked in early, before any middleware that might send a response. The only thing that I recommend linking in before compress is debugging or logging middleware (which do not send responses). Note that in most production environments, compression is handled by a proxy like NGINX, making this middleware unnecessary.
cookie-parser

    Provides cookie support. See Chapter 9.
cookie-session

    Provides cookie-storage session support. I do not generally recommend this approach to sessions. It must be linked in after cookie-parser. See Chapter 9.
express-session

    Provides session ID (stored in a cookie) session support. Defaults to a memory store, which is not suitable for production and can be configured to use a database store. See Chapter 9 and Chapter 13.
csurf

    Provides protection against cross-site request forgery (CSRF) attacks. This uses sessions, so it must be linked in after express-session middleware. Unfortunately, simply linking in this middleware does not magically protect against CSRF attacks; see Chapter 18 for more information.
serve-index

    Provides directory listing support for static files. There is no need to include this middleware unless you specifically need directory listing.
errorhandler

    Provides stack traces and error messages to the client. I do not recommend linking this in on a production server, as it exposes implementation details, which can have security or privacy consequences. See Chapter 20 for more information.

serve-favicon

* Serves the favicon (the icon that appears in the title bar of your browser). This is not strictly necessary; you can simply put a favicon.ico in the root of your static directory, but this middleware can improve performance. If you use it, it should be linked in high in the middleware stack. It also allows you to designate a filename other than favicon.ico.

morgan

* Provides automated logging support; all requests will be logged. See Chapter 20 for more information.

method-override

* Provides support for the x-http-method-override request header, which allows browsers to “fake” using HTTP methods other than GET and POST. This can be useful for debugging. This is needed only if you’re writing APIs.

response-time

* Adds the X-Response-Time header to the response, providing the response time in milliseconds. You usually don’t need this middleware unless you are doing performance tuning.

static

    Provides support for serving static (public) files. You can link in this middleware multiple times, specifying different directories. See Chapter 17 for more details.

vhost

* Virtual hosts (vhosts), a term borrowed from Apache, makes subdomains easier to manage in Express. See Chapter 14 for more information.


## Third-Party Middleware

Currently, there is no comprehensive “store” or index for third-party middleware. Almost all Express middleware, however, will be available on npm, so if you search npm for “Express” and “middleware,” you’ll get a pretty good list. The official Express documentation also contains a useful list of middleware.

