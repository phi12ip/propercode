# Quality Assurance

In web dev, quality can be broken down into four dimensions:

* Reach
* Functionality
* Usability
* Aesthetics

Whenever possible it is important to keep a clear distinction between presentation and logic

Types of tests:
* integration tests 
* unit tests

Unit testing is more useful for logic testing whereas integration testing is useful for testing both logic and presentaion.

Linting - linting isn't about catching errors, it's about catching potential errors. 

To install Jest, a testing framework for Javascript, run the following command in a terminal running inside your project folder:

``` sh
npm install --save-dev jest
```

Also add a test script in the project's `package.json` file.