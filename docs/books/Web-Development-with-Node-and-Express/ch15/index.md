[Prev][prev]
|
[Table of Contents](../)
|
[Next][next]

[prev]: ../ch13
[next]: ../ch15

# REST APIs and JSON

While we saw some REST API examples in Chapter 8, our paradigm so far has mostly been “process the data on the server side and send formatted HTML to the client.” Increasingly, this is not the default mode of operation for web applications. Instead, most modern web applications are single-page applications (SPAs) that receive all of their HTML and CSS in one static bundle and then rely on receiving unstructured data as JSON and manipulating HTML directly. Similarly, the importance of posting forms to communicate changes to the server is giving way to communicating directly using HTTP requests to an API.

Web service is a general term that means any application programming interface (API) that’s accessible over HTTP. The idea of web services has been around for quite some time, but until recently, the technologies that enabled them were stuffy, Byzantine, and overcomplicated. There are still systems that use those technologies (such as SOAP and WSDL), and there are Node packages that will help you interface with these systems. We won’t be covering those, though. Instead, we will be focused on providing so-called RESTful services, which are much more straightforward to interface with.

From a practical standpoint, the constraints of HTTP actually make it difficult to create an API that’s not RESTful; you’d have to go out of your way to establish state, for example. So our work is mostly cut out for us.


## JSON and XML

Vital to providing an API is having a common language to speak in. Part of the communication is dictated for us: we must use HTTP methods to communicate with the server. But past that, we are free to use whatever data language we choose. Traditionally, XML has been a popular choice, and it remains an important markup language. While XML is not particularly complicated, Douglas Crockford saw that there was room for something more lightweight, and JavaScript Object Notation (JSON) was born. In addition to being JavaScript-friendly (though it is by no means proprietary; it is an easy format for any language to parse), it also has the advantage of being generally easier to write by hand than XML.


## Our API

Here are our API endpoints:

GET /api/vacations

* Retrieves vacations

GET /api/vacation/:sku

* Returns a vacation by its SKU

POST /api/vacation/:sku/notify-when-in-season

* Takes email as a querystring parameter and adds a notification listener for the specified vacation

DELETE /api/vacation/:sku

* Requests the deletion of a vacation; takes email (the person requesting the deletion) and notes as querystring parameters

> There are many HTTP verbs available. GET and POST are the most common, followed by DELETE and PUT. It has become a standard to use POST for creating something, and PUT for updating (or modifying) something. The English meaning of these words doesn’t support this distinction in any way, so you may want to consider using the path to distinguish between these two operations to avoid confusion. If you want more information about HTTP verbs, I recommend starting with this Tamas Piros article.

There are many ways we could have described our API. Here, we’ve chosen to use combinations of HTTP methods and paths to distinguish our API calls, and a mix of querystring and body parameters for passing data. As an alternative, we could have had different paths (such as /api/vacations/delete) with the same method.1 We could also have passed data in a consistent way. For example, we might have chosen to pass all the necessary information for retrieving parameters in the URL instead of using a querystring: DEL /api/vacation/:id/:email/:notes. To avoid excessively long URLs, I recommend using the request body to pass large blocks of data (for example, the deletion request notes).

> There is a popular and well-respected convention for JSON APIs, creatively named JSON:API. It’s a bit verbose and repetitive for my taste, but I also believe that an imperfect standard is better than no standard at all. While we’re not using JSON:API for this book, you will learn everything you need to adopt the conventions laid down by JSON:API. See the JSON:API home page for more information.


## API Error Reporting

Error reporting in HTTP APIs is usually achieved through HTTP status codes. If the request returns 200 (OK), the client knows the request was successful. If the request returns 500 (Internal Server Error), the request failed. In most applications, however, not everything can (or should be) categorized coarsely into “success” or “failure.” For example, what if you request something by an ID but that ID doesn’t exist? This does not represent a server error. The client has asked for something that doesn’t exist. In general, errors can be grouped into the following categories:


Catastrophic errors

* Errors that result in an unstable or unknown state for the server. Usually, this is the result of an unhandled exception. The only safe way to recover from a catastrophic error is to restart the server. Ideally, any pending requests would receive a 500 response code, but if the failure is severe enough, the server may not be able to respond at all, and the request will time out.

Recoverable server errors

* Recoverable errors do not require a server restart, or any other heroic action. The error is a result of an unexpected error condition on the server (for example, a database connection being unavailable). The problem may be transient or permanent. A 500 response code is appropriate in this situation.

Client errors

* Client errors are a result of the client making the mistake—usually missing or invalid parameters. It isn’t appropriate to use a 500 response code. After all, the server has not failed. Everything is working normally; the client just isn’t using the API correctly. You have a couple of options here: you could respond with a status code of 200 and describe the error in the response body, or you could additionally try to describe the error with an appropriate HTTP status code. I recommend the latter approach. The most useful response codes in this case are 404 (Not Found), 400 (Bad Request), and 401 (Unauthorized). Additionally, the response body should contain an explanation of the specifics of the error. If you want to go above and beyond, the error message would even contain a link to documentation. Note that if the user requests a list of things and there’s nothing to return, this is not an error condition. It’s appropriate to simply return an empty list.


## Cross-Origin Resource Sharing

If you’re publishing an API, you’ll likely want to make the API available to others. This will result in a cross-site HTTP request.

Cross-site HTTP requests have been the subject of many attacks and have therefore been restricted by the same-origin policy, which restricts where scripts can be loaded from. Specifically, the protocol, domain, and port must match.

This makes it impossible for your API to be used by another site, which is where cross-origin resource sharing (CORS) comes in. CORS allows you to lift this restriction on a case-by-case basis, even allowing you to list which domains specifically are allowed to access the script.

CORS is implemented through the Access-Control-Allow-Origin header. 

The easiest way to implement it in an Express application is to use the cors package (npm install cors). To enable CORS for your application, use this:

``` js
const cors = require('cors')

app.use(cors())
```

Because the same-origin API is there for a reason (to prevent attacks), I recommend applying CORS only where necessary. In our case, we want to expose our entire API (but only the API), so we’re going to restrict CORS to paths starting with /api:

``` js
const cors = require('cors')

app.use('/api', cors())
```


## Our Tests

Before we write tests for our API, we need a way to actually call a REST API. For that, we’ll be using a Node package called node-fetch, which replicates the browser’s fetch API:

``` sh
npm install --save-dev node-fetch@2.6.0
```

We’ll put the tests for the API calls we’re going to implement in tests/api/api.test.js (ch15/test/api/api.test.js in the companion repo):

``` js
const fetch = require('node-fetch')

const baseUrl = 'http://localhost:3000'

const _fetch = async (method, path, body) => {
  body = typeof body === 'string' ? body : JSON.stringify(body)
  const headers = { 'Content-Type': 'application/json' }
  const res = await fetch(baseUrl + path, { method, body, headers })
  if(res.status < 200 || res.status > 299)
    throw new Error(`API returned status ${res.status}`)
  return res.json()
}

describe('API tests', () => {

  test('GET /api/vacations', async () => {
    const vacations = await _fetch('get', '/api/vacations')
    expect(vacations.length).not.toBe(0)
    const vacation0 = vacations[0]
    expect(vacation0.name).toMatch(/\w/)
    expect(typeof vacation0.price).toBe('number')
  })

  test('GET /api/vacation/:sku', async() => {
    const vacations = await _fetch('get', '/api/vacations')
    expect(vacations.length).not.toBe(0)
    const vacation0 = vacations[0]
    const vacation = await _fetch('get', '/api/vacation/' + vacation0.sku)
    expect(vacation.name).toBe(vacation0.name)
  })

  test('POST /api/vacation/:sku/notify-when-in-season', async() => {
    const vacations = await _fetch('get', '/api/vacations')
    expect(vacations.length).not.toBe(0)
    const vacation0 = vacations[0]
    // at this moment, all we can do is make sure the HTTP request is successful
    await _fetch('post', `/api/vacation/${vacation0.sku}/notify-when-in-season`,
      { email: 'test@meadowlarktravel.com' })
  })

  test('DELETE /api/vacation/:id', async() => {
    const vacations = await _fetch('get', '/api/vacations')
    expect(vacations.length).not.toBe(0)
    const vacation0 = vacations[0]
    // at this moment, all we can do is make sure the HTTP request is successful
    await _fetch('delete', `/api/vacation/${vacation0.sku}`)
  })

})
```

Our test suite starts off with a helper function _fetch, which handles some common housekeeping. It will JSON encode the body if it isn’t already, add the appropriate headers, and throw an appropriate error if the response status code isn’t in the 200s.

We have a single test for each of our API endpoints. I’m not suggesting that these tests are robust or complete; even with this simple API, we could (and should) have several tests for each endpoint. What we have here is more of a starting point that illustrates techniques for testing an API.

There are a couple of important characteristics of these tests that deserve mention. One is that we are relying on the API being already started and running on port 3000. A more robust test suite would find an open port, start the API on that port as part of its setup, and stop it when all the tests have run. Second, this test relies on data already being present in our API. For example, the first test expects there to be at least one vacation, and for that vacation to have a name and a price. In a real application, you may not be able to make these assumptions (for example, you may start with no data, and you may want to test for allowable missing data). Again, a more robust testing framework would have a way of setting and resetting the initial data in the API so you could start from a known state every time. For example, you might have scripts that set up and seed a test database, attach the API to it, and tear it down for every test run. As we saw in Chapter 5, testing is a large and complicated topic, and we can only scratch the surface here.

The first test covers our GET /api/vacations endpoint. It fetches all of the vacations, validates that there is at least one, and checks the first one to see if it has a name and a price. We could also conceivably test other data properties. I’ll leave it as a reader’s exercise to think about which properties are most important to test.

The second test covers our GET /api/vacation/:sku endpoint. Since we don’t have consistent test data, we start by fetching all of the vacations and getting the SKU from the first one so we can test this endpoint.

Our last two tests cover our POST /api/vacation/:sku/notify-when-in-season and DELETE /api/vacation/:sku endpoints. Unfortunately, with our current API and testing framework, we can do very little to verify that these endpoints are doing what they are supposed to, so we default to invoking them and trusting the API is doing the right thing when it doesn’t return an error. If we wanted to make these tests more robust, we would have to either add endpoints that allow us to verify the actions (for example, an endpoint that determined if a given email was registered for a specific vacation) or somehow give the tests “backdoor” access to our database.

If you run the tests now, they will time out and fail…because we haven’t implemented our API or even started our server. So let’s get started!


## Using Express to Provide an API

We’ll start by creating the handlers in lib/handlers.js (we could create a separate file, such as lib/api.js, but let’s keep things simple for now):

``` js
exports.getVacationsApi = async (req, res) => {
  const vacations = await db.getVacations({ available: true })
  res.json(vacations)
}

exports.getVacationBySkuApi = async (req, res) => {
  const vacation = await db.getVacationBySku(req.params.sku)
  res.json(vacation)
}

exports.addVacationInSeasonListenerApi = async (req, res) => {
  await db.addVacationInSeasonListener(req.params.sku, req.body.email)
  res.json({ message: 'success' })
}

exports.requestDeleteVacationApi = async (req, res) => {
  const { email, notes } = req.body
  res.status(500).json({ message: 'not yet implemented' })
}
```

Then we hook up the API in meadowlark.js:

```js
app.get('/api/vacations', handlers.getVacationsApi)
app.get('/api/vacation/:sku', handlers.getVacationBySkuApi)
app.post('/api/vacation/:sku/notify-when-in-season',
  handlers.addVacationInSeasonListenerApi)
app.delete('/api/vacation/:sku', handlers.requestDeleteVacationApi)
```

I am leaving requestDeleteVacationsApi as a reader’s exercise, mainly because this functionality could be implemented so many different ways. The simplest approach would be to just modify our vacation schema to have “delete requested” fields that just get updated with the email and notes when the API is called. A more sophisticated approach would be to have a separate table, like a moderation queue, that records the deletion requests separately, referencing the vacation in question, which would better lend itself to administrator use.

Assuming you set up Jest correctly in Chapter 5, you should just be able to run npm test, and the API tests will be picked up (Jest will look for any file that ends in .test.js). You’ll see we have three passing tests and one failing one: the incomplete DELETE /api/vacation/:sku.

