[Table of Contents](README.md)

# Saving Time with Express

## Scaffolding

An idea borrowed from Ruby on Rails for quickly setting up and creating project components. A lot of magic, so not recommended for beginners... or others.

For more information, see the express-generator documentation.

## Express Getting Started

First you need to initialize the project and add express as a dependency:

``` sh
npm init -y
npm install express
```

If you are using Git, make sure to add the following to your `.gitignore` file:

``` sh
# ignore packages installed by npm
node_modules
```


Here is an example started template for Express:

``` js
const express = require('express')

const app = express()

const port = process.env.PORT || 3000

// custom 404 page
app.use((req, res) => {
  res.type('text/plain')
  res.status(404)
  res.send('404 - Not Found')
})

// custom 500 page
app.use((err, req, res, next) => {
  console.error(err.message)
  res.type('text/plain')
  res.status(500)
  res.send('500 - Server Error')
})

app.listen(port, () => console.log(
  `Express started on http://localhost:${port}; ` +
  `press Ctrl-C to terminate.`))

```

## Simple Express routes

``` js
app.get('/', (req, res) => {
  res.type('text/plain')
  res.send('Meadowlark Travel');
})

app.get('/about', (req, res) => {
  res.type('text/plain')
  res.send('About Meadowlark Travel')
})

// custom 404 page
app.use((req, res) => {
  res.type('text/plain')
  res.status(404)
  res.send('404 - Not Found')
})
```

The function you provide will get invoked when the route is matched.

Instead of using Node’s low-level res.end, we’re switching to using Express’s extension, res.send. We are also replacing Node’s res.writeHead with res.set and res.status. Express is also providing us a convenience method, res.type, which sets the Content-Type header. While it’s still possible to use res.writeHead and res.end, it isn’t necessary or recommended.

## Views and Layouts

Express supports many different view engines that provide different levels of abstraction. Express gives some preference to a view engine called Pug (which is no surprise, because it is also the brainchild of TJ Holowaychuk). The approach Pug takes is minimal: what you write doesn’t resemble HTML at all, which certainly represents a lot less typing (no more angle brackets or closing tags). The Pug engine then takes that and converts it to HTML.

Pug is appealing, but that level of abstraction comes at a cost. If you’re a frontend developer, you have to understand HTML and understand it well, even if you’re actually writing your views in Pug. Most frontend developers I know are uncomfortable with the idea of their primary markup language being abstracted away. For this reason, I am recommending the use of another, less abstract templating framework called Handlebars.

To add handlebars support to the project, run the following:

``` sh
npm install express-handlebars
```

And to configure the handlebards engine:

``` js
const express = require('express')
const expressHandlebars = require('express-handlebars')

const app = express()

// configure Handlebars view engine
app.engine('handlebars', expressHandlebars({
  defaultLayout: 'main',
}))
app.set('view engine', 'handlebars')
```

Create a file called views/layouts/main.handlebars:

``` html
<!doctype html>
<html>
  <head>
    <title>Meadowlark Travel</title>
  </head>
  <body>
    {{{body}}}
  </body>
</html>
```

The only thing that you probably haven’t seen before is this: {{{body}}}. This expression will be replaced with the HTML for each view. When we created the Handlebars instance, note we specified the default layout (defaultLayout: \'main'). That means that unless you specify otherwise, this is the layout that will be used for any view.

Now lets create the home page, `views/home.handlebars`:

``` html
<h1>Welcome to Home</h1>
```

and the about page, `views/about.handlebars`:

``` html
<h1>Welcome to About</h1>
```

Then our Not Found page, `views/404.handlebars`:

``` html
<h1>404 - Not Found</h1>
```

Server Error page, `views/500.handlebars`:

``` html
<h1>500 - Server Error</h1>
```

Now replace the routes in app:

``` js
app.get('/', (req, res) => res.render('home'))

app.get('/about', (req, res) => res.render('about'))

// custom 404 page
app.use((req, res) => {
  res.status(404)
  res.render('404')
})

// custom 500 page
app.use((err, req, res, next) => {
  console.error(err.message)
  res.status(500)
  res.render('500')
})
```

## Static Files and Views

The static middleware allows you to designate one or more directories as containing static resources that are simply to be delivered to the client without any special handling. This is where you would put things such as images, CSS files, and client-side JavaScript files.

In your project directory, create a subdirectory called public (we call it public because anything in this directory will be served to the client without question). Then, before you declare any routes, you’ll add the static middleware

``` js
app.use(express.static(__dirname + '/public'))
```

Now we can simply reference /img/logo.png (note, we do not specify public; that directory is invisible to the client), and the static middleware will serve that file, setting the content type appropriately. Now let’s modify our layout so that our logo appears on every page:

``` html
<body>
  <header>
    <img src="/img/logo.png" alt="Meadowlark Travel Logo">
  </header>
  {{{body}}}
</body>
```

## Dynamic Content in Views

Let's say you wanted to show a random fortune on the about page.

Create a fortunes array in the your app file:

``` js
const fortunes = [
  "Conquer your fears or they will conquer you.",
  "Rivers need springs.",
  "Do not fear what you don't know.",
  "You will have a pleasant surprise.",
  "Whenever possible, keep it simple.",
]
```

and then modify the about template :

``` html
<h1>About Meadowlark Travel</h1>
{{#if fortune}}
  <p>Your fortune for the day:</p>
  <blockquote>{{fortune}}</blockquote>
{{/if}}
```

and modify the route:

``` js
app.get('/about', (req, res) => {
  const randomFortune = fortunes[Math.floor(Math.random()*fortunes.length)]
  res.render('about', { fortune: randomFortune })
})
```