[Prev][prev]
|
[Table of Contents](../)
|
[Next][next]

[prev]: ../ch10
[next]: ../ch12

# Sending Email

## SMTP, MSAs, and MTAs

The lingua franca for sending email is the Simple Mail Transfer Protocol (SMTP). While it is possible to use SMTP to send an email directly to the recipient’s mail server, this is generally a bad idea: unless you are a “trusted sender” like Google or Yahoo!, chances are your email will be tossed directly into the spam bin. It’s better to use a mail submission agent (MSA), which will deliver the email through trusted channels, reducing the chance that your email will be marked as spam.

In addition to ensuring that your email arrives, MSAs handle nuisances like temporary outages and bounced emails.

The final piece of the equation is the mail transfer agent (MTA), which is the service that actually sends the email to its final destination.

For the purposes of this book, MSA, MTA, and SMTP server are essentially equivalent.

So you’ll need access to an MSA. While it is possible to get started using a free consumer email service such as Gmail, Outlook, or Yahoo!, these services are no longer as friendly to automated emails as they once were (in an effort to cut down on abuse).

Fortunately, there are a couple of excellent email services to choose from that have a free option for low-volume use: Sendgrid and Mailgun. I’ve used both services, and I like them both. The examples in this book will be using SendGrid.

If you’re working for an organization, the organization itself may have an MSA; you can contact your IT department and ask them if there’s an SMTP relay available for sending automated emails.

If you’re using SendGrid or Mailgun, go ahead and set up your account now. For SendGrid, you’ll need to create an API key (which will be your SMTP password).

## Recieving Email

Most websites only need the ability to send email, like password reset instructions and promotional emails. However, some applications need to receive email as well. A good example is an issue-tracking system that sends out an email when someone updates an issue, and if you reply to that email, the issue is automatically updated with your response.

Unfortunately, receiving email is much more involved and will not be covered in this book. If this is functionality you need, you should allow your mail provider to maintain the mailbox and have a periodic process to access it with an IMAP agent such as imap-simple.


## Email Headers

An email message consists of two parts: the header and the body (very much like an HTTP request). The header contains information about the email: who it’s from, who it’s addressed to, the date it was received, the subject, and more. Those are the headers that are normally displayed to the user in an email application, but there are many more headers. 

Most email clients allow you to look at the headers; if you’ve never done so, I recommend you take a look. 

The headers give you all the information about how the email got to you; every server and MTA that the email passed through will be listed in the header.

It often comes as a surprise to people that some headers, like the “from” address, can be set arbitrarily by the sender.


## Email Formats

When the internet was new, all email was simply ASCII text. The world has changed a lot since then, and people want to send email in different languages and do more sophisticated things like include formatted text, images, and attachments. This is where things start to get ugly: email formats and encoding are a horrible jumble of techniques and standards.

Fortunately, we won’t really have to address these complexities. Nodemailer will handle that for us. What’s important for you to know is that your email can be either plain text (Unicode) or HTML.

Almost all modern email applications support HTML email, so it’s generally pretty safe to format your emails in HTML. Still, there are “text purists” out there who eschew HTML email, so I recommend always including both text and HTML email. If you don’t want to have to write text and HTML email, Nodemailer supports a shortcut that will automatically generate the plain text version from the HTML.


## HTML Email

HTML email is a topic that could fill an entire book. Unfortunately, it’s not as simple as just writing HTML as you would for your site: most mail clients support only a small subset of HTML. 

Mostly, you have to write HTML as if it were still 1996; it’s not much fun. In particular, you have to go back to using tables for layout (cue sad music).

First, I encourage you to read [MailChimp’s excellent article](https://mailchimp.com/help/about-html-email/) about writing HTML email. It does a good job covering the basics and explaining the things you need to keep in mind when writing HTML email.

The next is a real time-saver: [HTML Email Boilerplate](http://htmlemailboilerplate.com/). It’s essentially a very well-written, rigorously tested template for HTML email.

On the other hand, if your formatting is modest, there’s no need for an expensive testing service like [Litmus](https://www.litmus.com/email-testing/). If you’re sticking to things like headers, bold/italic text, horizontal rules, and some image links, you’re pretty safe.


## Nodemailer

First, we need to install nodemailer:

``` sh
npm install nodemailer
```

Then, require the nodemailer package and create a Nodemailer instance (a transport in Nodemailer parlance):

``` js
const nodemailer = require('nodemailer')

const mailTransport = nodemailer.createTransport({

  auth: {
    user: credentials.sendgrid.user,
    pass: credentials.sendgrid.password,
  }
})
```

Update your .credentials.development.json file accordingly:

``` json
{
  "cookieSecret": "your cookie secret goes here",
  "sendgrid": {
    "user": "your sendgrid username",
    "password": "your sendgrid password"
  }
}
```

Common configuration options for SMTP are the port, authentication type, and TLS options. However, most of the major mail services use the default options.


## Sending Mail

Now that we have our mail transport instance, we can send mail. 

We’ll start with a simple example that sends text mail to only one recipient (ch11/00-smtp.js in the companion repo):

``` js
try {
  const result = await mailTransport.sendMail({
    from: '"Meadowlark Travel" <info@meadowlarktravel.com>',
    to: 'joecustomer@gmail.com',
    subject: 'Your Meadowlark Travel Tour',
    text: 'Thank you for booking your trip with Meadowlark Travel.  ' +
      'We look forward to your visit!',
  })
  console.log('mail sent successfully: ', result)
} catch(err) {
  console.log('could not send mail: ' + err.message)
}
```

You’ll notice that we’re handling errors here, but it’s important to understand that no errors doesn’t necessarily mean your email was delivered successfully to the recipient. The callback’s error parameter will be set only if there was a problem communicating with the MSA (such as a network or authentication error). 

If the MSA was unable to deliver the email (for example, because of an invalid email address or an unknown user), you will have to check your account activity in your mail service, which you can do either from the admin interface or through an API.

Nodemail supports sending mail to multiple recipients by using commas (ch11/01-multiple-recipients.js in the companion repo):

``` js
try {
  const result = await mailTransport.sendMail({
    from: '"Meadowlark Travel" <info@meadowlarktravel.com>',
    to: 'joe@gmail.com, "Jane Customer" <jane@yahoo.com>, ' +
      'fred@hotmail.com',
    subject: 'Your Meadowlark Travel Tour',
    text: 'Thank you for booking your trip with Meadowlark Travel.  ' +
      'We look forward to your visit!',
  })
  console.log('mail sent successfully: ', result)
} catch(err) {
  console.log('could not send mail: ' + err.message)
}
```

Note that, in this example, we mixed plain email addresses (joe@gmail.com) with email addresses specifying the recipient’s name (“Jane Customer” <jane@yahoo.com>). This is allowed syntax.


## Sending HTML Email

``` js
const result = await mailTransport.sendMail({
  from: '"Meadowlark Travel" <info@meadowlarktravel.com>',
  to: 'joe@gmail.com, "Jane Customer" <jane@yahoo.com>, ' +
    'fred@hotmail.com',
  subject: 'Your Meadowlark Travel Tour',
  html: '<h1>Meadowlark Travel</h1>\n<p>Thanks for book your trip with ' +
    'Meadowlark Travel.  <b>We look forward to your visit!</b>',
  text: 'Thank you for booking your trip with Meadowlark Travel.  ' +
    'We look forward to your visit!',
})
```

## Images in HTML Email

While it is possible to embed images in HTML email, I strongly discourage it. They bloat your email messages, and it isn’t generally considered good practice. 

Instead, you should make images you want to use in email available on your web server and link appropriately from the email.

It is best to have a dedicated location in your static assets folder for email images.

You should even keep assets that you use both on your site and in emails separate.

It reduces the chance of negatively affecting the layout of your emails.

Let’s consider our “Thank you for booking your trip with Meadowlark Travel” email example, which we’ll expand a little bit. Let’s imagine that we have a shopping cart object that contains our order information. That shopping cart object will be stored in the session. Let’s say the last step in our ordering process is a form that’s processed by /cart/checkout, which sends a confirmation email. Let’s start by creating a view for the thank-you page, views/cart-thank-you.handlebars:

``` hbs
<p>Thank you for booking your trip with Meadowlark Travel,
  {{cart.billing.name}}!</p>
<p>Your reservation number is {{cart.number}}, and an email has been
sent to {{cart.billing.email}} for your records.</p>
```

Then we’ll create an email template for the email. Download HTML Email Boilerplate, and put in views/email/cart-thank-you.handlebars. Edit the file, and modify the body:

``` js
<table cellpadding="0" cellspacing="0" border="0" id="backgroundTable">
  <tr>
    <td valign="top">
      <table cellpadding="0" cellspacing="0" border="0" align="center">
        <tr>
          <td width="200" valign="top"><img class="image_fix"
            src="//placehold.it/100x100"
            alt="Meadowlark Travel" title="Meadowlark Travel"
            width="180" height="220" /></td>
        </tr>
        <tr>
          <td width="200" valign="top"><p>
            Thank you for booking your trip with Meadowlark Travel,
            {{cart.billing.name}}.</p><p>Your reservation number
            is {{cart.number}}.</p></td>
        </tr>
        <tr>
          <td width="200" valign="top">Problems with your reservation?
          Contact Meadowlark Travel at
          <span class="mobile_link">555-555-0123</span>.</td>
        </tr>
      </table>
    </td>
  </tr>
</table>
```

> Because you can’t use localhost addresses in email, if your site isn’t live yet, you can use a placeholder service for any graphics. For example, http://placehold.it/100x100 dynamically serves a 100-pixel-square graphic you can use. This technique is used quite often for for-placement-only (FPO) images and layout purposes.

``` js
app.post('/cart/checkout', (req, res, next) => {
  const cart = req.session.cart
  if(!cart) next(new Error('Cart does not exist.'))
  const name = req.body.name || '', email = req.body.email || ''
  // input validation
  if(!email.match(VALID_EMAIL_REGEX))
    return res.next(new Error('Invalid email address.'))
  // assign a random cart ID; normally we would use a database ID here
  cart.number = Math.random().toString().replace(/^0\.0*/, '')
  cart.billing = {
    name: name,
    email: email,
  }
  res.render('email/cart-thank-you', { layout: null, cart: cart },
    (err,html) => {
        console.log('rendered email: ', html)
        if(err) console.log('error in email template')
        mailTransport.sendMail({
          from: '"Meadowlark Travel": info@meadowlarktravel.com',
          to: cart.billing.email,
          subject: 'Thank You for Book your Trip with Meadowlark Travel',
          html: html,
          text: htmlToFormattedText(html),
        })
          .then(info => {
            console.log('sent! ', info)
            res.render('cart-thank-you', { cart: cart })
          })
          .catch(err => {
            console.error('Unable to send confirmation: ' + err.message)
          })
    }
  )
})
```

Note that we’re calling res.render twice. Normally, you call it only once (calling it twice will display only the results of the first call). 

However, in this instance, we’re circumventing the normal rendering process the first time we call it: notice that we provide a callback. 

Doing that prevents the results of the view from being rendered to the browser. Instead, the callback receives the rendered view in the parameter html: all we have to do is take that rendered HTML and send the email!

We specify layout: null to prevent our layout file from being used, because it’s all in the email template (an alternate approach would be to create a separate layout file for emails and use that instead). Lastly, we call res.render again. This time, the results will be rendered to the HTML response as normal.


## Encapsulating Email Functionality

If you’re using email a lot throughout your site, you may want to encapsulate the email functionality. Let’s assume you always want your site to send email from the same sender (“Meadowlark Travel” <info@meadowlarktravel.com>) and you always want the email to be sent in HTML with automatically generated text. Create a module called `lib/email.js` (ch11/lib/email.js in the companion repo):

``` js
const nodemailer = require('nodemailer')
const htmlToFormattedText = require('html-to-formatted-text')

module.exports = credentials => {

  const mailTransport = nodemailer.createTransport({
    host: 'smtp.sendgrid.net',
    auth: {
      user: credentials.sendgrid.user,
      pass: credentials.sendgrid.password,
    },
  })

  const from = '"Meadowlark Travel" <info@meadowlarktravel.com>'
  const errorRecipient = 'youremail@gmail.com'

  return {
    send: (to, subject, html) =>
      mailTransport.sendMail({
        from,
        to,
        subject,
        html,
        text: htmlToFormattedText(html),
      }),
  }

}
```

Now all we have to do to send an email is the following (ch11/05-email-library.js in the companion repo):

``` js
const emailService = require('./lib/email')(credentials)

emailService.send(email, "Hood River tours on sale today!",
  "Get 'em while they're hot!")
```
