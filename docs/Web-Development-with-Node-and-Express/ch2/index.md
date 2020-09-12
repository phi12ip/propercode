[Prev][prev]
|
[Table of Contents](../)
|
[Next][next]

[prev]: ../ch1
[next]: ../ch3

# Getting Started with Node

## Getting Node

You can download Node for Windows, Mac OS, or Linux from <https://nodejs.org/en/>

## Using the Terminal

If you’re not friends with your terminal, I highly recommend you spend some time familiarizing yourself with your terminal of choice.

Everything you do with node with be though a CLI and not a GUI.

> Check out the [Linux Command Line][LCL] course if you're a CLI beginner


## Editors

Use Visual Studio Code for development.

## npm

npm is the ubiquitous package manager for Node packages.

Yarn was created to implement features not available in npm, but those features have since been added to npm so there is no reason to use Yarn on a new project. 


## A Simple Web Server with Node

``` js
const http = require('http')
const port = process.env.PORT || 3000

const server = http.createServer((req,res) => {
  // normalize url by removing querystring, optional
  // trailing slash, and making it lowercase
  const path = req.url.replace(/\/?(?:\?.*)?$/, '').toLowerCase()
  switch(path) {
    case '':
      res.writeHead(200, { 'Content-Type': 'text/plain' })
      res.end('Homepage')
      break
    case '/about':
      res.writeHead(200, { 'Content-Type': 'text/plain' })
      res.end('About')
      break
    default:
      res.writeHead(404, { 'Content-Type': 'text/plain' })
      res.end('Not Found')
      break
  } })

server.listen(port, () => console.log(`server started on port ${port}; ` +
  'press Ctrl-C to terminate....'))
```

## Serving Static Resources

``` js 
const http = require('http')
const fs = require('fs')
const port = process.env.PORT || 3000

function serveStaticFile(res, path, contentType, responseCode = 200) {
  fs.readFile(__dirname + path, (err, data) => {
    if(err) {
      res.writeHead(500, { 'Content-Type': 'text/plain' })
      return res.end('500 - Internal Error')
    }
    res.writeHead(responseCode, { 'Content-Type': contentType })
    res.end(data)
  })
}

const server = http.createServer((req,res) => {
  // normalize url by removing querystring, optional trailing slash, and
  // making lowercase
  const path = req.url.replace(/\/?(?:\?.*)?$/, '').toLowerCase()
  switch(path) {
    case '':
      serveStaticFile(res, '/public/home.html', 'text/html')
      break
    case '/about':
      serveStaticFile(res, '/public/about.html', 'text/html')
      break
    case '/img/logo.png':
      serveStaticFile(res, '/public/img/logo.png', 'image/png')
      break
    default:
      serveStaticFile(res, '/public/404.html', 'text/html', 404)
      break
  }
})

server.listen(port, () => console.log(`server started on port ${port}; ` +
  'press Ctrl-C to terminate....'))

```


<!--- Links --->
[LCL]: ../Linux-Command-Line/