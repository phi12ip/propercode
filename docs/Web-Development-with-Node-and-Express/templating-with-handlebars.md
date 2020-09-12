# Templating with Handlebars

Templating is a technique for constructing and formatting content sent to the user. 

This process of replacing fields is sometimes called interpolation, which is just a fancy word for “supplying missing information” in this context.

Using Javascript to emit HTML can be very problematic:

* You have to constantly worry about what characters need to be escaped and how to do that.

* Using JavaScript to generate HTML that itself includes JavaScript quickly leads to madness.

* You usually lose the nice syntax highlighting and other handy language-specific features your editor has.

* It can be much harder to spot malformed HTML.

* Your code is hard to visually parse.

* It can make it harder for other people to understand your code.


Templating makes this:

``` js
document.write('<h1>Please Don\'t Do This</h1>')
document.write('<p><span class="code">document.write</span> is naughty,\n')
document.write('and should be avoided at all costs.</p>')
document.write('<p>Today\'s date is ' + new Date() + '.</p>')
```

using handlebars into:

``` hbs
<h1>Much Better</h1>
<p>No <span class="code">document.write</span> here!</p>
<p>Today's date is {{today}}.</p>
```

I would err on the side of templates, however, and avoid generating HTML with JavaScript except for the simplest cases.

## Chosing a Template Engine


Performance

* Clearly, you want your templating engine to be as fast as possible. It’s not something you want slowing down your website.

Client, server, or both?

* Most, but not all, templating engines are available on both the server and client sides. If you need to use templates in both realms (and you will), I recommend you pick something that is equally capable in either capacity.

Abstraction

* Do you want something familiar (like normal HTML with curly brackets thrown in, for example), or do you secretly hate HTML and would love something that saves you from all those angle brackets? Templating (especially server-side templating) gives you some choices here.

## Pug: A Different Approach

``` hbs
doctype html                        <!DOCTYPE html>
html(lang="en")                     <html lang="en">
  head                              <head>
    title= pageTitle                <title>Pug Demo</title>
    script.                         <script>
      if (foo) {                        if (foo) {
         bar(1 + 5)                         bar(1 + 5)
      }                                 }
  body                              </script>
                                    <body>
    h1 Pug                         <h1>Pug</h1>
    #container                      <div id="container">
      if youAreUsingPug
        p You are amazing           <p>You are amazing</p>
      else
        p Get on it!
      p.                            <p>
        Pug is a terse and           Pug is a terse and
        simple templating             simple templating
        language with a               language with a
        strong focus on               strong focus on
        performance and               performance and
        powerful features.            powerful features.
                                    </p>
                                    </body>
                                    </html>
```

As much as I admire the Pug philosophy and the elegance of its execution, I’ve found that I don’t want the details of HTML abstracted away from me.

## Handlebars Basics

Handlebars is an extension of Mustache, another popular templating engine.

The key to understanding templating is understanding the concept of context. When you render a template, you pass the templating engine an object called the context object, and this is what allows replacements to work.

For example, if my context object is

``` js
{ name: 'Buttercup' }
```

and my template is

``` hbs
<p>Hello, {{name}}!</p>
```

then {{name}} will be replaced with Buttercup. What if you want to pass HTML to the template? For example, if our context was instead

``` js
{ name: '<b>Buttercup</b>' }
```

then using the previous template will result in <p>Hello, &lt;b&gt;Buttercup&lt;b&gt;</p>, which is probably not what you’re looking for. To solve this problem, simply use three curly brackets instead of two: {{{name}}}.

While we’ve already established that we should avoid writing HTML in JavaScript, the ability to turn off HTML escaping with triple curly brackets has some important uses. For example, if you were building a content management system (CMS) with what you see is what you get (WYSIWYG) editors, you would probably want to be able to pass HTML to your views. Also, the ability to render properties from the context without HTML escaping is important for layouts and sections, which we’ll learn about shortly.

## Comments

Comments in handlebars look `{{! comment goes here }}` 

``` hbs
{{! super-secret comment }}
<!-- not-so-secret comment -->
```

The super secret comment will never be sent to the browser.

## Blocks

Blocks provide flow control, conditional execution, and extensibility.

Consider the following:

``` js
{
  currency: {
    name: 'United States dollars',
    abbrev: 'USD',
  },
  tours: [
    { name: 'Hood River', price: '$99.95' },
    { name: 'Oregon Coast', price: '$159.95' },
  ],
  specialsUrl: '/january-specials',
  currencies: [ 'USD', 'GBP', 'BTC' ],
}
```

And the template: 

``` hbs
<ul>
  {{#each tours}}
    {{! I'm in a new block...and the context has changed }}
    <li>
      {{name}} - {{price}}
      {{#if ../currencies}}
        ({{../currency.abbrev}})
      {{/if}}
    </li>
  {{/each}}
</ul>
{{#unless currencies}}
  <p>All prices in {{currency.name}}.</p>
{{/unless}}
{{#if specialsUrl}}
  {{! I'm in a new block...but the context hasn't changed (sortof) }}
  <p>Check out our <a href="{{specialsUrl}}">specials!</p>
{{else}}
  <p>Please check back often for specials.</p>
{{/if}}
<p>
  {{#each currencies}}
    <a href="#" class="currency">{{.}}</a>
  {{else}}
    Unfortunately, we currently only accept {{currency.name}}.
  {{/each}}
</p>
```

`{{.}}` refers to the current context

> Accessing the current context with a lone period has another use: it can distinguish helpers (which we’ll learn about soon) from properties of the current context. For example, if you have a helper called foo and a property in the current context called foo, {{foo}} refers to the helper, and {{./foo}} refers to the property.

## Server-Side Templates

Server-side templates, in addition to hiding your implementation details, support template caching, which is important for performance.

To set caching to be on explicitly for development:

``` js
app.set('view cache', true)
```

We need to install an external package for handlebars support:

``` sh
npm install express-handlebars
```

Then in the Express app:

```js
const expressHandlebars = require('express-handlebars')
app.engine('handlebars', expressHandlebars({
  defaultLayout: 'main',
})
app.set('view engine', 'handlebars')
```

express-handlebars expects Handlebars templates to have the .handlebars extension. I’ve grown used to this, but if it’s too wordy for you, you can change the extension to the also common .hbs when you create the express-handlebars instance: 

``` js
app.engine('handlebars', expressHandlebars({ extname: '.hbs' })).
```

## Views and Layouts

A view usually represents an individual page on your website (though it could represent an Ajax-loaded portion of a page, an email, or anything else for that matter)

By default, Express looks for views in the views subdirectory. 

A layout is a special kind of view—essentially, a template for templates.

Layouts are essential because most (if not all) of the pages on your site will have an almost identical layout. For example, they must have an `<html>` element and a `<title>` element, they usually all load the same CSS files, and so on.

Bare bones layout file:

``` hbs
<!doctype html>
<html>
  <head>
    <title>Meadowlark Travel</title>
    <link rel="stylesheet" href="/css/main.css">
  </head>
  <body>
    {{{body}}}
  </body>
</html>
```

> It’s important to use three curly brackets instead of two: our view is most likely to contain HTML, and we don’t want Handlebars trying to escape it.

Specify the name of the default layout:

``` js
app.engine('handlebars', expressHandlebars({
  defaultLayout: 'main',
})
```

By default, Express looks for views in the views subdirectory, and layouts in views/layouts. So if you have a view views/foo.handlebars, you can render it this way:

``` js
app.get('/foo', (req, res) => res.render('foo'))
```

It will use views/layouts/main.handlebars as the layout. If you don’t want to use a layout at all (meaning you’ll have to have all of the boilerplate in the view), you can specify layout: null in the context object:

``` js
app.get('/foo', (req, res) => res.render('foo', { layout: null }))
```

You can also optionally supply the template name: 

``` js
app.get('/foo', (req,res) => res.render('foo', { layout: 'microsite'}))
```

Keep in mind that the more templates you have, the more basic HTML layout you have to maintain. On the other hand, if you have pages that are substantially different in layout, it may be worth it; you have to find a balance that works for your projects.

## Sections

One technique I’m borrowing from Microsoft’s excellent Razor template engine is the idea of sections. Layouts work well if all of your view fits neatly within a single element in your layout, but what happens when your view needs to inject itself into different parts of your layout?

A common example of this is a view needing to add something to the `<head>` element or to insert a `<script>`, which is sometimes the very last thing in the layout, for performance reasons.

Handlebars doesn't have anything out of the box to handle this, but we can use handlebars helpers to accomplish this.

``` js
app.engine('handlebars', expressHandlebars({
  defaultLayout: 'main',
  helpers: {
    section: function(name, options) {
      if(!this._sections) this._sections = {}
      this._sections[name] = options.fn(this)
      return null
    },
  },
}))
```

Now we can use the section helper in a view. Let’s add a view (views/section-test.handlebars) to add something to the `<head>` and a script:

``` hbs
{{#section 'head'}}
  <!-- we want Google to ignore this page -->
  <meta name="robots" content="noindex">
{{/section}}

<h1>Test Page</h1>
<p>We're testing some script stuff.</p>

{{#section 'scripts'}}
  <script>
    document.querySelector('body')
      .insertAdjacentHTML('beforeEnd', '<small>(scripting works!)</small>')
  </script>
{{/section}}
```

Now in the layout:

``` hbs

```
## Partials

Let’s imagine we want a Current Weather component that displays the current weather conditions in Portland, Bend, and Manzanita. We want this component to be reusable so we can easily put it on whatever page we want, so we’ll use a partial. First, we create a partial file, views/partials/weather.handlebars:

``` hbs
<div class="weatherWidget">
  {{#each partials.weatherContext}}
    <div class="location">
      <h3>{{location.name}}</h3>
      <a href="{{location.forecastUrl}}">
        <img src="{{iconUrl}}" alt="{{weather}}">
        {{weather}}, {{temp}}
      </a>
    </div>
  {{/each}}
  <small>Source: <a href="https://www.weather.gov/documentation/services-web-api">
    National Weather Service</a></small>
</div>
```

Note that we namespace our context by starting with `partials.weatherContext`. Since we want to be able to use the partial on any page, it’s not practical to pass the context in for every view, so instead we use `res.locals` (which is available to every view). But because we don’t want to interfere with the context specified by individual views, we put all partial context in the `partials` object.

> express-handlebars allows you to pass in partial templates as part of the context. For example, if you add partials.foo = "Template!" to your context, you can render this partial with {{> foo}}. This usage will override any .handlebars view files, which is why we used partials.weatherContext earlier, instead of partials.weather, which would override views/partials/weather.handlebars.

Our middleware will inject the weather data into the res.locals.partials object, which will make it available as the context for our partial.

To make our middleware more testable, we’ll put it in its own file, `lib/middleware/weather.js`:

``` js
const getWeatherData = () => Promise.resolve([
  {
    location: {
      name: 'Portland',
      coordinates: { lat: 45.5154586, lng: -122.6793461 },
    },
    forecastUrl: 'https://api.weather.gov/gridpoints/PQR/112,103/forecast',
    iconUrl: 'https://api.weather.gov/icons/land/day/tsra,40?size=medium',
    weather: 'Chance Showers And Thunderstorms',
    temp: '59 F',
  },
  {
    location: {
      name: 'Bend',
      coordinates: { lat: 44.0581728, lng: -121.3153096 },
    },
    forecastUrl: 'https://api.weather.gov/gridpoints/PDT/34,40/forecast',
    iconUrl: 'https://api.weather.gov/icons/land/day/tsra_sct,50?size=medium',
    weather: 'Scattered Showers And Thunderstorms',
    temp: '51 F',
  },
  {
    location: {
      name: 'Manzanita',
      coordinates: { lat: 45.7184398, lng: -123.9351354 },
    },
    forecastUrl: 'https://api.weather.gov/gridpoints/PQR/73,120/forecast',
    iconUrl: 'https://api.weather.gov/icons/land/day/tsra,90?size=medium',
    weather: 'Showers And Thunderstorms',
    temp: '55 F',
  },
])

const weatherMiddleware = async (req, res, next) => {
  if(!res.locals.partials) res.locals.partials = {}
  res.locals.partials.weatherContext = await getWeatherData()
  next()
}

module.exports = weatherMiddleware
```

Then in `views/home.handlebars`:

``` hbs
<h2>Home</h2>
{{> weather}}
```

The `{{> partial_name}}` syntax is how you include a partial in a view: express-handlebars will know to look in `views/partials` for a view called `partial_name.handlebars` (or `weather.handlebars`, in our example).

> express-handlebars supports subdirectories, so if you have a lot of partials, you can organize them. For example, if you have some social media partials, you could put them in the `views/partials/social` directory and include them using `{{> social/facebook}}`, `{{> social/twitter}}`, etc.

## Perfecting Your Templates

Your templates are at the heart of your website. A good template structure will save you development time, promote consistency across your website, and reduce the number of places that layout quirks can hide.

<https://html5boilerplate.com/>

