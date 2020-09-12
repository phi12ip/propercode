[Table of Contents](README.md)

# Quality Assurance

In web dev, quality can be broken down into four dimensions:

* Reach
* Functionality
* Usability
* Aesthetics

Whenever possible it is important to keep a clear distinction between presentation and logic.

Types of tests:
* integration tests 
* unit tests

Unit testing is more useful for logic testing whereas integration testing is useful for testing both logic and presentaion.

Linting - linting isn't about catching errors, it's about catching potential errors. 

Jest is a very popular testing framework for javascript.

To install Jest, run the following command in a terminal from your project root:

``` sh
npm install --save-dev jest
```

Also add a test script in the project's `package.json` file.

## Unit Testing

One of the chanllenges is writing code that is testable.

Code that tries to do to much is gernerally harder to test than focsed code.

Refactoring parts of the app, like the routing, is a good way to make our app more testable.


## Integration Testing

To export the app when it is not run directory from node:

``` js
if(require.main === module) {
  app.listen(port, () => {
    console.log( `Express started on http://localhost:${port}` +
      '; press Ctrl-C to terminate.' )
  })
} else {
  module.exports = app
}
```


Puppeteer is essentially a controllable, headless version of Chrome. (Headless simply means that the browser is capable of running without actually rendering a UI on-screen.) To install Puppeteer:

`npm install --save-dev puppeteer`


Now we can write an integration that does the following:

1. Starts our application server on an unoccupied port

2. Launches a headless Chrome browser and opens a page

3. Navigates to our application’s home page

4. Finds a link with data-test-id="about" and clicks it

5. Waits for the navigation to happen

6. Verifies that we are on the /about page



``` js
const portfinder = require('portfinder')
const puppeteer = require('puppeteer')

const app = require('../meadowlark.js')

let server = null
let port = null

beforeEach(async () => {
  port = await portfinder.getPortPromise()
  server = app.listen(port)
})

afterEach(() => {
  server.close()
})

test('home page links to about page', async () => {
  const browser = await puppeteer.launch()
  const page = await browser.newPage()
  await page.goto(`http://localhost:${port}`)
  await Promise.all([
    page.waitForNavigation(),
    page.click('[data-test-id="about"]'),
  ])
  expect(page.url()).toBe(`http://localhost:${port}/about`)
  await browser.close()
})
```


## Linting

`npm install --save-dev eslint`
