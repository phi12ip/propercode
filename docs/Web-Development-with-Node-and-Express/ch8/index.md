[Prev][prev]
|
[Table of Contents](../)
|
[Next][next]

[prev]: ../ch7
[next]: ../ch9

# Form Handling

The usual way you collect information from your users is to use HTML forms. Whether you let the browser submit the form normally, use Ajax, or employ fancy frontend controls, the underlying mechanism is generally still an HTML form. In this chapter, we’ll discuss the different methods for handling forms, form validation, and file uploads.

## Sending Client Data to the Server

The two options for sending client data to the server are the querystring and the request body.

If you're using a querystring, you're making a `GET` request. 

It is a common misperception that `POST` is secure and `GET` is not: in reality, both are secure if you use HTTPS, and neither is secure if you don’t. If you’re not using HTTPS, an intruder can look at the body data for a `POST` just as easily as the querystring of a `GET` request. However, if you’re using `GET` requests, your users will see all of their input (including hidden fields) in the querystring, which is ugly and messy. Also, browsers often place limits on querystring length (there is no such restriction for body length). For these reasons, I generally recommend using `POST` for form submission.

This book is focusing on the server side, but it’s important to understand some basics about constructing HTML forms. Here’s a simple example:

``` html
<form action="/process" method="POST">
    <input type="hidden" name="hush" val="hidden, but not secret!">
    <div>
        <label for="fieldColor">Your favorite color: </label>
        <input type="text" id="fieldColor" name="color">
    </div>
    <div>
        <button type="submit">Submit</button>
    </div>
</form>
```

I recommend keeping your forms logically consistent; a form should contain all the fields you would like submitted at once (optional/empty fields are OK) and none that you don’t.

An example of this would be to have a form for a site search and a separate form for signing up for an email newsletter. It is possible to use one large form and figure out what action to take based on what button a person clicked, but it is a headache and often not friendly for people with disabilities (because of the way accessibility browsers render forms).

## Encoding

When the form is submitted (either by the browser or via Ajax), it must be encoded somehow. If you don’t explicitly specify an encoding, it defaults to `application/x-www-form-urlencoded` (this is just a lengthy media type for “URL encoded”). This is a basic, easy-to-use encoding that’s supported by Express out of the box.

If you need to upload files, things get more complicated. There’s no easy way to send files using URL encoding, so you’re forced to use the `multipart/form-data` encoding type, which is not handled directly by Express.

## Different Approaches to Form Handling

If you’re not using Ajax, your only option is to submit the form through the browser, which will reload the page. However, how the page is reloaded is up to you. 

There are two things to consider when processing forms: what path handles the form (the action) and what response is sent to the browser.

If your form uses `method="POST"` (which is recommended), it is quite common to use the same path for displaying the form and processing the form: these can be distinguished because the former is a `GET` request, and the latter is a `POST` request. If you take this approach, you can omit the action attribute on the form.

The other option is to use a separate path to process the form. For example, if your contact page uses the path /contact, you might use the path `/process-contact` to process the form (by specifying `action="/process-contact"`). If you use this approach, you have the option of submitting the form via GET (which I do not recommend; it needlessly exposes your form fields on the URL). 

Using a separate endpoint for form submission might be preferred if you have multiple URLs that use the same submission mechanism (for example, you might have an email sign-up box on multiple pages on the site).


Direct HTML response

    After processing the form, you can send HTML directly back to the browser (a view, for example). This approach will produce a warning if the user attempts to reload the page and can interfere with bookmarking and the Back button, and for these reasons, it is not recommended.
302 redirect

    While this is a common approach, it is a misuse of the original meaning of the 302 (Found) response code. HTTP 1.1 added the 303 (See Other) response code, which is preferable. Unless you have reason to target browsers made before 1996, you should use 303 instead.
303 redirect

    The 303 (See Other) response code was added in HTTP 1.1 to address the misuse of the 302 redirect. The HTTP specification specifically indicates that the browser should use a GET request when following a 303 redirect, regardless of the original method. This is the recommended method for responding to a form submission request.

Since the recommendation is that you respond to a form submission with a 303 redirect, the next question is, “Where does the redirection point to?” The answer to that is up to you. Here are the most common approaches:

---

Redirect to dedicated success/failure pages

    This method requires that you dedicate URLs for appropriate success or failure messages. For example, if the user signs up for promotional emails but there was a database error, you might want to redirect to /error/database. If a user’s email address were invalid, you could redirect to /error/invalid-email, and if everything was successful, you could redirect to /promo-email/thank-you. One of the advantages of this method is that it’s analytics friendly: the number of visits to your /promo-email/thank-you page should roughly correlate to the number of people signing up for your promotional email. It is also straightforward to implement. It has some downsides, however. It does mean you have to allocate URLs to every possibility, which means pages to design, write copy for, and maintain. Another disadvantage is that the user experience can be suboptimal: users like to be thanked, but then they have to navigate back to where they were or where they want to go next. This is the approach we’ll be using for now: we’ll switch to using flash messages (not to be confused with Adobe Flash) in Chapter 9.

Redirect to the original location with a flash message

    For small forms that are scattered throughout your site (like an email sign-up, for example), the best user experience is not to interrupt the user’s navigation flow. That is, provide a way to submit an email address without leaving the page. One way to do this, of course, is Ajax, but if you don’t want to use Ajax (or you want your fallback mechanism to provide a good user experience), you can redirect back to the page the user was originally on. The easiest way to do this is to use a hidden field in the form that’s populated with the current URL. Since you want there to be some feedback that the user’s submission was received, you can use flash messages.

Redirect to a new location with a flash message

    Large forms generally have their own page, and it doesn’t make sense to stay on that page once you’ve submitted the form. In this situation, you have to make an intelligent guess about where the user might want to go next and redirect accordingly. For example, if you’re building an admin interface, and you have a form to create a new vacation package, you might reasonably expect your user to want to go to the admin page that lists all vacation packages after submitting the form. However, you should still employ a flash message to give the user feedback about the result of the submission.

---

If you are using Ajax, I recommend a dedicated URL. It’s tempting to start Ajax handlers with a prefix (for example, /ajax/enter), but I discourage this approach: it’s attaching implementation details to a URL. Also, as we’ll see shortly, your Ajax handler should handle regular browser submissions as a fail-safe.

## Form Handling with Express

You will need to have the body-parser enabled

Let’s go ahead and add a form to Meadowlark Travel that lets the user sign up for a mailing list.

For demonstration’s sake, we’ll use the querystring, a hidden field, and visible fields in `/views/newsletter-signup.handlebars`:

``` hbs
<h2>Sign up for our newsletter to receive news and specials!</h2>
<form class="form-horizontal" role="form"
    action="/newsletter-signup/process?form=newsletter" method="POST">
  <input type="hidden" name="_csrf" value="{{csrf}}">
  <div class="form-group">
    <label for="fieldName" class="col-sm-2 control-label">Name</label>
    <div class="col-sm-4">
      <input type="text" class="form-control"
      id="fieldName" name="name">
    </div>
  </div>
  <div class="form-group">
    <label for="fieldEmail" class="col-sm-2 control-label">Email</label>
    <div class="col-sm-4">
      <input type="email" class="form-control" required
          id="fieldEmail" name="email">
    </div>
  </div>
  <div class="form-group">
    <div class="col-sm-offset-2 col-sm-4">
      <button type="submit" class="btn btn-primary">Register</button>
    </div>
  </div>
</form>
```

We’ve already linked in our body parser, so now we need to add handlers for our newsletter sign-up page, processing function, and thank-you page (ch08/lib/handlers.js in the companion repo):

``` js 
exports.newsletterSignup = (req, res) => {
  // we will learn about CSRF later...for now, we just
  // provide a dummy value
  res.render('newsletter-signup', { csrf: 'CSRF token goes here' })
}
exports.newsletterSignupProcess = (req, res) => {
  console.log('Form (from querystring): ' + req.query.form)
  console.log('CSRF token (from hidden form field): ' + req.body._csrf)
  console.log('Name (from visible form field): ' + req.body.name)
  console.log('Email (from visible form field): ' + req.body.email)
  res.redirect(303, '/newsletter-signup/thank-you')
}
exports.newsletterSignupThankYou = (req, res) =>
  res.render('newsletter-signup-thank-you')
```

(If you haven’t already, create a views/newsletter-signup-thank-you.handlebars file.)

Lastly, we’ll link our handlers into our application (ch08/meadowlark.js in the companion repo):

``` js
app.get('/newsletter-signup', handlers.newsletterSignup)
app.post('/newsletter-signup/process', handlers.newsletterSignupProcess)
app.get('/newsletter-signup/thank-you', handlers.newsletterSignupThankYou)
```

That’s all there is to it. Note that in our handler, we’re redirecting to a “thank you” view. We could render a view here, but if we did, the URL field in the visitor’s browser would remain /process, which could be confusing. Issuing a redirect solves that problem.

> It’s important that you use a 303 (or 302) redirect, not a 301 redirect in this instance. 301 redirects are “permanent,” meaning your browser may cache the redirection destination. If you use a 301 redirect and try to submit the form a second time, your browser may bypass the /process handler altogether and go directly to /thank-you since it correctly believes the redirect to be permanent. The 303 redirect, on the other hand, tells your browser, “Yes, your request is valid, and you can find your response here,” and does not cache the redirect destination.

Lastly, we’ll link our handlers into our application (ch08/meadowlark.js in the companion repo):

``` js
app.get('/newsletter-signup', handlers.newsletterSignup)
app.post('/newsletter-signup/process', handlers.newsletterSignupProcess)
app.get('/newsletter-signup/thank-you', handlers.newsletterSignupThankYou)
```

## Using Fetch to Send Form Data

Using the `fetch` API to send JSON-encoded form data is a much more modern approach that gives you more control over the client/server communication and allows you to have fewer page refreshes.

Since we are not making round-trip requests to the server, we no longer have to worry about redirects and multiple user URLs (we’ll still have a separate URL for the form processing itself), and for that reason, we’ll just consolidate our entire “newsletter signup experience” under a single URL called /newsletter.

``` html 
<div id="newsletterSignupFormContainer">
  <form class="form-horizontal role="form" id="newsletterSignupForm">
    <!-- the rest of the form contents are the same... -->
  </form>
</div>
```

Then we have a script that intercepts the form submit event and cancels it so we can handle the form processing outselves:

``` html
<script>
  document.getElementById('newsletterSignupForm')
    .addEventListener('submit', evt => {
      evt.preventDefault()
      const form = evt.target
      const body = JSON.stringify({
        _csrf: form.elements._csrf.value,
        name: form.elements.name.value,
        email: form.elements.email.value,
      })
      const headers = { 'Content-Type': 'application/json' }
      const container =
        document.getElementById('newsletterSignupFormContainer')
      fetch('/api/newsletter-signup', { method: 'post', body, headers })
        .then(resp => {
          if(resp.status < 200 || resp.status >= 300)
            throw new Error(`Request failed with status ${resp.status}`)
          return resp.json()
        })
        .then(json => {
          container.innerHTML = '<b>Thank you for signing up!</b>'
        })
        .catch(err => {
          container.innerHTML = `<b>We're sorry, we had a problem ` +
            `signing you up. Please <a href="/newsletter">try again</a>`
        })
  })
</script>
```

and in the server file:

``` js
app.use(bodyParser.json())

//...

app.get('/newsletter', handlers.newsletter)
app.post('/api/newsletter-signup', handlers.api.newsletterSignup)
```

Now we’ll add those endpoints to our `lib/handlers.js` file:

``` js
exports.newsletter = (req, res) => {
  // we will learn about CSRF later...for now, we just
  // provide a dummy value
  res.render('newsletter', { csrf: 'CSRF token goes here' })
}
exports.api = {
  newsletterSignup: (req, res) => {
    console.log('CSRF token (from hidden form field): ' + req.body._csrf)
    console.log('Name (from visible form field): ' + req.body.name)
    console.log('Email (from visible form field): ' + req.body.email)
    res.send({ result: 'success' })
  },
}
```

We can do whatever processing we need in the form processing handler; usually we would be saving the data to the database. If there are problems, we send back a JSON object with an err property (instead of result: success).

## File Uploads

File uploads are complicated.

There are four popular and robust options for multipart form processing: busboy, multiparty, formidable, and multer. I have used all four, and they’re all good, but I feel multiparty is the best maintained, and so we’ll use it here.

Let’s create a file upload form for a Meadowlark Travel vacation photo contest (`views/contest/vacation-photo.handlebars`):

``` hbs
<h2>Vacation Photo Contest</h2>

<form class="form-horizontal" role="form"
    enctype="multipart/form-data" method="POST"
    action="/contest/vacation-photo/{{year}}/{{month}}">
  <input type="hidden" name="_csrf" value="{{csrf}}">
  <div class="form-group">
    <label for="fieldName" class="col-sm-2 control-label">Name</label>
    <div class="col-sm-4">
      <input type="text" class="form-control"
      id="fieldName" name="name">
    </div>
  </div>
  <div class="form-group">
    <label for="fieldEmail" class="col-sm-2 control-label">Email</label>
    <div class="col-sm-4">
      <input type="email" class="form-control" required
          id="fieldEmail" name="email">
    </div>
  </div>
  <div class="form-group">
    <label for="fieldPhoto" class="col-sm-2 control-label">Vacation photo</label>
    <div class="col-sm-4">
      <input type="file" class="form-control" required  accept="image/*"
          id="fieldPhoto" name="photo">
    </div>
  </div>
  <div class="form-group">
    <div class="col-sm-offset-2 col-sm-4">
      <button type="submit" class="btn btn-primary">Register</button>
    </div>
  </div>
</form>
```

Note that we must specify `enctype="multipart/form-data"` to enable file uploads. We’re also restricting the type of files that can be uploaded by using the accept attribute (which is optional).

``` js
const multiparty = require('multiparty')

app.post('/contest/vacation-photo/:year/:month', (req, res) => {
  const form = new multiparty.Form()
  form.parse(req, (err, fields, files) => {
    if(err) return res.status(500).send({ error: err.message })
    handlers.vacationPhotoContestProcess(req, res, fields, files)
  })
})
```

So now we have extra information to pass to our (testable) route handler: the fields (which won’t be in req.body as in previous examples since we’re using a different body parser) and information about the file(s) that were collected. Now that we know what that looks like, we can write our route handler:

``` js
exports.vacationPhotoContestProcess = (req, res, fields, files) => {
  console.log('field data: ', fields)
  console.log('files: ', files)
  res.redirect(303, '/contest/vacation-photo-thank-you')
}
```

You’ll see that your form fields come across as you would expect: as an object with properties corresponding to your field names. The files object contains more data, but it’s relatively straightforward. For each file uploaded, you’ll see there are properties for size, the path it was uploaded to (usually a random name in a temporary directory), and the original name of the file that the user uploaded (just the filename, not the whole path, for security and privacy reasons).

What you do with this file is now up to you: you can store it in a database, copy it to a more permanent location, or upload it to a cloud-based file storage system. Remember that if you’re relying on local storage for saving files, your application won’t scale well, making this a poor choice for cloud-based hosting.

## File Uploads with Fetch

Happily, using fetch for file uploads is nearly identical to letting the browser handle it.

The hard work of file uploads is really in the encoding, which is being handled for us with middleware.

Consider this JavaScript to send our form contents using fetch:

``` html
<script>
  document.getElementById('vacationPhotoContestForm')
    .addEventListener('submit', evt => {
      evt.preventDefault()
      const body = new FormData(evt.target)
      const container =
        document.getElementById('vacationPhotoContestFormContainer')
      const url = '/api/vacation-photo-contest/{{year}}/{{month}}'
      fetch(url, { method: 'post', body })
        .then(resp => {
          if(resp.status < 200 || resp.status >= 300)
            throw new Error(`Request failed with status ${resp.status}`)
          return resp.json()
        })
        .then(json => {
          container.innerHTML = '<b>Thank you for submitting your photo!</b>'
        })
        .catch(err => {
          container.innerHTML = `<b>We're sorry, we had a problem processing ` +
            `your submission.  Please <a href="/newsletter">try again</a>`
        })
    })
</script>
```

The important detail to note here is that we convert the form element to a FormData object, which fetch can accept directly as the request body. That’s all there is to it! Because the encoding is exactly the same as it was when we let the browser handle it, our handler is almost exactly the same. We just want to return a JSON response instead of a redirect:

``` js
exports.api.vacationPhotoContest = (req, res, fields, files) => {
  console.log('field data: ', fields)
  console.log('files: ', files)
  res.send({ result: 'success' })
}
```
