# Request and Response Object

## Parts of a URL

Protocol:

* determines how the request will be transmitted

Host:

* the host is what identifies the server

Port:

* each server has a collection of many ports
* this port is connected to the application we are trying to reach

Path:

* the path is the first part of the url that your app cares about
* the path uniquely identifies pages or other resources

Querystring:

* optional collection of key value pairs
* starts with a `?`
* pairs are separated by `&`
* both names and values should be URL encoded

Fragment:

* start with a `#`
* used to link to a particular part of the page
* is not sent to the server

## HTTP Request Methods

The combination of method, path, and querystring is what your app uses to determine how to respond.

Mostly, you will send GET requests. 

POST is used when sending information to the server

## Request Headers

When you navigate to a page your browser sends a lot of invisible information in the form of HTML headers. The browser will tell the server what language it prefers to receive the page in (for example, if you download Chrome in Spain, it will request the Spanish version of pages you visit, if they exist). 

It will also send information about the user agent (the browser, operating system, and hardware) and other bits of information. All this information is sent as a request header, which is made available to you through the request object’s headers property.

To see the headers in action, create a route that returns them as a response like the following:

``` js
app.get('/headers', (req, res) => {
  res.type('text/plain')
  const headers = Object.entries(req.headers)
    .map(([key, value]) => `${key}: ${value}`)
  res.send(headers.join('\n'))
})
```

## Response Headers

Just as the browser sends hidden information on behalf of the client, when the server sesponds it sends hidden information as well.

Content-Type is an example of one if these headers to tell the browser what type of file it is trying to display. 

The browser will respect the Content-Type reguardless of what the path ends in.

Can also contain headers containing infomation on caching and whether the response is compressed.

It is common for Response Headers to contain information about the server.

If you want to see the response headers, they can be found in your browser’s developer tools. To see the response headers in Chrome, for example:

1. Open the JavaScript console.

2. Click the Network tab.

3. Reload the page.

4. Pick the HTML from the list of requests (it will be the first one).

5. Click the Headers tab; you will see all response headers.

It's recomended to not disclose what server software you are running, so make sure to add the following to your app to disable that header:

``` js
app.disable('x-powered-by')
```

## Internet Media Types

The Content-Type is is an _internet media type_ which consists of a type, subtype, and optional parameters.

These are also called MIME types.

## Request Body

Certain HTTP verbs like POST can have a request body.

The most common media type for POST bodies is `application/x-www-form-urlencoded`, which is simply encoded name/value pairs separated by `&`.

If the POST needs to support file uploads, the media type is `multipart/form-data`, which is a more complicated format. 

Lastly, Ajax requests can use `application/json` for the body.

## The Request Object

The request object (which is passed as the first parameter of a request handler, meaning you can name it whatever you want; it is common to name it req or request) starts its life as an instance of http.IncomingMessage, a core Node object. Express adds further functionality. Let’s look at the most useful properties and methods of the request object (all of these methods are added by Express, except for req.headers and req.url, which originate in Node):

---

req.params

* An array containing the named route parameters. We’ll learn more about this in Chapter 14.

req.query

* An object containing querystring parameters (sometimes called GET parameters) as name/value pairs.

req.body

* An object containing POST parameters. It is so named because POST parameters are passed in the body of the request, not in the URL as querystring parameters are. To make req.body available, you’ll need middleware that can parse the body content type, which we will learn about in Chapter 10.

req.route

* Information about the currently matched route. This is primarily useful for route debugging.

req.cookies/req.signedCookies

* Objects containing cookie values passed from the client. See Chapter 9.

req.headers

* The request headers received from the client. This is an object whose keys are the header names and whose values are the header values. Note that this comes from the underlying http.IncomingMessage object, so you won’t find it listed in the Express documentation.

req.accepts(types)

* A convenience method to determine whether the client accepts a given type or types (optional types can be a single MIME type, such as application/json, a comma-delimited list, or an array). This method is of primary interest to those writing public APIs; it is assumed that browsers will always accept HTML by default.

req.ip

* The IP address of the client.

req.path

* The request path (without protocol, host, port, or querystring).

req.hostname

* A convenience method that returns the hostname reported by the client. This information can be spoofed and should not be used for security purposes.

req.xhr

* A convenience property that returns true if the request originated from an Ajax call.

req.protocol

* The protocol used in making this request (for our purposes, it will be either http or https).

req.secure

* A convenience property that returns true if the connection is secure. This is equivalent to req.protocol === 'https'.

req.url/req.originalUrl

* A bit of a misnomer, these properties return the path and querystring (they do not include protocol, host, or port). req.url can be rewritten for internal routing purposes, but req.originalUrl is designed to remain the original request and querystring.

---

## The Response Object

The response object (which is passed as the second parameter of a request handler, meaning you can name it whatever you want; it is common to name it res, resp, or response) starts its life as an instance of http.ServerResponse, a core Node object. Express adds further functionality. Let’s look at the most useful properties and methods of the response object (all of these are added by Express):

---

res.status(code)

* Sets the HTTP status code. Express defaults to 200 (OK), so you will have to use this method to return a status of 404 (Not Found) or 500 (Server Error), or any other status code you want to use. For redirects (status codes 301, 302, 303, and 307), there is a method redirect, which is preferable. Note that res.status returns the response object, meaning you can chain calls: res.status(404).send('Not found').

res.set(name, value)

* Sets a response header. This is not something you will normally be doing manually. You can also set multiple headers at once by passing a single object argument whose keys are the header names and whose values are the header values.

res.cookie(name, value, [options]), res.clearCookie(name, [options])

* Sets or clears cookies that will be stored on the client. This requires some middleware support; see Chapter 9.

res.redirect([status], url)

* Redirects the browser. The default redirect code is 302 (Found). In general, you should minimize redirection unless you are permanently moving a page, in which case you should use the code 301 (Moved Permanently).

res.send(body)

* Sends a response to the client. Express defaults to a content type of text/html, so if you want to change it to text/plain (for example), you’ll have to call res.type('text/plain’) before calling res.send. If body is an object or an array, the response is sent as JSON (with the content type being set appropriately), though if you want to send JSON, I recommend doing so explicitly by calling res.json instead.

res.json(json)

* Sends JSON to the client.

res.jsonp(json)

* Sends JSONP to the client.

res.end()

* Ends the connection without sending a response. To learn more about the differences between res.send, res.json, and res.end, see this article by Tamas Piros.

res.type(type)

* A convenience method to set the Content-Type header. This is essentially equivalent to res.set(\'Content-Type ', type), except that it will also attempt to map file extensions to an internet media type if you provide a string without a slash in it. For example, res.type(\'txt ') will result in a Content-Type of text/plain. There are areas where this functionality could be useful (for example, automatically serving disparate multimedia files), but in general, you should avoid it in favor of explicitly setting the correct internet media type.

res.format(object)

* This method allows you to send different content depending on the Accept request header. This is of primary use in APIs, and we will discuss this more in Chapter 15. Here’s a simple example: res.format({'text/plain': 'hi there', 'text/html': '<b>hi there</b>'}).

res.attachment([filename]), res.download(path, [filename], [callback])

* Both of these methods set a response header called Content-Disposition to attachment; this will prompt the browser to download the content instead of displaying it in a browser. You may specify filename as a hint to the browser. With res.download, you can specify the file to download, whereas res.attachment just sets the header; you still have to send content to the client.

res.sendFile(path, [options], [callback])

* This method will read a file specified by path and send its contents to the client. There should be little need for this method; it’s easier to use the static middleware and put files you want available to the client in the public directory. However, if you want to have a different resource served from the same URL depending on some condition, this method could come in handy.

res.links(links)

* Sets the Links response header. This is a specialized header that has little use in most applications.

res.locals, res.render(view, [locals], callback)

* res.locals is an object containing default context for rendering views. res.render will render a view using the configured templating engine (the locals parameter to res.render shouldn’t be confused with res.locals: it will override the context in res.locals, but context not overridden will still be available). Note that res.render will default to a response code of 200; use res.status to specify a different response code. Rendering views will be covered in depth in Chapter 7.

## Getting More Information

