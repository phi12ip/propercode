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