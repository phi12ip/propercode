[Prev][prev]
|
[Table of Contents](../)
|
[Next][next]

[prev]: ../ch11
[next]: ../ch13

# Production Concerns

In this chapter, you’ll learn about Express’s support for different execution environments, methods to scale your website, and how to monitor your website’s health. You’ll see how you can simulate a production environment for testing and development and also how to perform stress testing so you can identify production problems before they happen.


## Execution Environments

Express supports the concept of execution environments: a way to run your application in production, development, or test mode. You could actually have as many different environments as you want. For example, you could have a staging environment, or a training environment. 

However, keep in mind that development, production, and test are “standard” environments, and both Express and third-party middleware often make decisions based on those environments. 

While it is possible to specify the execution environment by calling app.set('env', 'production'), it is inadvisable to do so; it means your app will always run in that environment, no matter what the situation.

It’s preferable to specify the execution environment by using the environment variable NODE_ENV. Let’s modify our app to report on the mode it’s running in by calling app.get('env’):

``` js
const port = process.env.PORT || 3000
app.listen(port, () => console.log(`Express started in ` +
  `${app.get('env')} mode at http://localhost:${port}` +
  `; press Ctrl-C to terminate.`))
```

If you start your server now, you’ll see you’re running in development mode; it’s the default if you don’t specify otherwise. Let’s try putting it in production mode:

``` sh
$ export NODE_ENV=production
$ node meadowlark.js
```

If you’re using Unix/BSD, there’s a handy syntax that allows you to modify the environment only for the duration of that command:

``` js
$ NODE_ENV=production node meadowlark.js
```

This will run the server in production mode, but once the server terminates, the NODE_ENV environment variable won’t be modified.

If you start Express in production mode, you may notice warnings about components that are not suitable for use in production mode. If you’ve been following along with the examples in this book, you’ll see that connect.session is using a memory store, which is not suitable for a production environment. 

Once we switch to a database store in Chapter 13, this warning will disappear.


## Environment-Specific Configuration

Mainly, the execution environment is a tool for you to leverage, allowing you to easily make decisions about how your application should behave in the different environments.

As a word of caution, you should try to minimize the differences between your development, test, and production environments. That is, you should use this feature sparingly. If your development or test environments differ wildly from production, you are increasing your chances of different behavior in production, which is a recipe for more defects.

Some differences are inevitable; for example, if your app is highly database driven, you probably don’t want to be messing with the production database during development, and that would be a good candidate for environment-specific configuration. Another low-impact area is more verbose logging. There are a lot of things you might want to log in development that are unnecessary to record in production.

Let’s add some logging to our server.

We’ll use morgan (don’t forget npm install morgan), which is the most common logging middleware (ch12/00-logging.js in the companion repo):

``` js 
const morgan = require('morgan')
const fs = require('fs')

switch(app.get('env')) {
  case 'development':
    app.use(morgan('dev'))
    break
  case 'production':
    const stream = fs.createWriteStream(__dirname + '/access.log',
      { flags: 'a' })
    app.use(morgan('combined', { stream }))
    break
}
```

If you start the server as you normally would (node meadowlark.js) and visit the site, you’ll see the activity logged to the console. To see how the application behaves in production mode, run it with NODE_ENV=production instead. 

Now if you visit the application, you won’t see any activity on the terminal (probably what we want for a production server), but all of the activity is logged in Apache’s Combined Log Format, which is a staple for many server tools.

We accomplished this by creating an appendable ({ flags: a }) write stream and passing it to the morgan configuration. Morgan has many options; to see them all, check out the [morgan](https://github.com/expressjs/morgan) documentation.

> In the previous example, we’re using __dirname to store the request log in a subdirectory of the project itself. If you take this approach, you will want to add log to your .gitignore file. Alternatively, you could take a more Unix-like approach and save the logs in a subdirectory of /var/log, as Apache does by default.

Whenever you’re tempted to make a development-specific modification, you should always think first about how that might have QA consequences in production.


## Running Your Node Process

Running the regular node program is fine for development and testing, but it has disadvantages for production. Notably, there are no protections if your app crashes or gets terminated. A robust process manager can address this problem.

Depending on your hosting solution, you may not need a process manager if one is provided by the hosting solution itself. That is, the hosting provider will give you a configuration option to point to your application file, and it will handle the process management.

But if you need to manage the process yourself, there are two popular options for process managers:

* Forever
* PM2

Forever is a little more straightforward and easy to get started, and PM2 offers more features.

If you want to experiment with a process manager without investing a lot of time, I recommend giving Forever a try. You can try it in two steps. First, install Forever:

``` sh
npm install -g forever
```

Then, start your application with Forever (run this from your application root):

``` sh 
forever start meadowlark.js
```

Your application is now running…and it will stay running even if you close your terminal window! You can restart the process with forever restart meadowlark.js and stop it with forever stop meadowlark.js.

Getting started with PM2 is a little more involved but is worth looking into if you need to use your own process manager for production.

## Scaling Your Website

These days, scaling usually means one of two things: scaling up or scaling out. Scaling up refers to making servers more powerful: faster CPUs, better architecture, more cores, more memory, etc. Scaling out, on the other hand, simply means more servers. 

With the increased popularity of cloud computing and the ubiquity of virtualization, server computational power is becoming less relevant, and scaling out is usually the most cost-effective method for scaling websites according to your needs.

The most important thing to remember when building a website designed to be scaled out is persistence. If you’re used to relying on file-based storage for persistence, stop right there. That way lies madness.

The moral of this story is that unless you have a filesystem that’s accessible to all of your servers, you should not rely on the local filesystem for persistence. The exceptions are read-only data, like logging, and backups. For example, I have commonly backed up form submission data to a local flat file in case the database connection failed. In the case of a database outage, it is a hassle to go to each server and collect the files, but at least no damage has been done.

## Scaling Out with App Clusters

Node itself supports app clusters which is a simple single server form of scaling out.

With app clusters, you can create an independent server for each core (CPU) on the system (having more servers than the number of cores will not improve the performance of your app).

Let’s go ahead and add cluster support to our website. While it’s quite common to do all of this work in your main application file, we are going to create a second application file that will run the app in a cluster, using the nonclustered application file we’ve been using all along. To enable that, we have to make a slight modification to meadowlark.js first (see ch12/01-server.js in the companion repo for a simplified example):

``` js
function startServer(port) {
  app.listen(port, function() {
    console.log(`Express started in ${app.get('env')} ` +
      `mode on http://localhost:${port}` +
      `; press Ctrl-C to terminate.`)
  })
}

if(require.main === module) {
  // application run directly; start app server
  startServer(process.env.PORT || 3000)
} else {
  // application imported as a module via "require": export
  // function to create server
  module.exports = startServer
}
```

Then, we create a new script, meadowlark-cluster.js (see ch12/01-cluster in the companion repo for a simplified example):

``` js
const cluster = require('cluster')

function startWorker() {
  const worker = cluster.fork()
  console.log(`CLUSTER: Worker ${worker.id} started`)
}

if(cluster.isMaster){

  require('os').cpus().forEach(startWorker)

  // log any workers that disconnect; if a worker disconnects, it
  // should then exit, so we'll wait for the exit event to spawn
  // a new worker to replace it
  cluster.on('disconnect', worker => console.log(
    `CLUSTER: Worker ${worker.id} disconnected from the cluster.`
  ))

  // when a worker dies (exits), create a worker to replace it
  cluster.on('exit', (worker, code, signal) => {
    console.log(
      `CLUSTER: Worker ${worker.id} died with exit ` +
      `code ${code} (${signal})`
    )
    startWorker()
  })

} else {

    const port = process.env.PORT || 3000
    // start our app on worker; see meadowlark.js
    require('./meadowlark.js')(port)

}
```

Assuming you’re on a multicore system, you should see some number of workers started. If you want to see evidence of different workers handling different requests, add the following middleware before your routes:

``` js
const cluster = require('cluster')

app.use((req, res, next) => {
  if(cluster.isWorker)
    console.log(`Worker ${cluster.worker.id} received request`)
  next()
})
```

Now you can connect to your application with a browser. Reload a few times and see how you can get a different worker out of the pool on each request. (You may not be able to; Node is designed to handle large numbers of connections, and you may not be able to stress it sufficiently simply by reloading your browser; later we’ll explore stress testing, and you’ll be able to better see the cluster in action.)


## Handling Uncaught Exceptions

In the asynchronous world of Node, uncaught exceptions are of particular concern. Let’s start with a simple example that doesn’t cause too much trouble (I encourage you to follow along with these examples):

``` js
app.get('/fail', (req, res) => {
  throw new Error('Nope!')
})
```

When Express executes route handlers, it wraps them in a try/catch block, so this isn’t actually an uncaught exception. This won’t cause too much of a problem: Express will log the exception on the server side, and the visitor will get an ugly stack dump. However, your server is stable, and other requests will continue to be served correctly. If we want to provide a “nice” error page, create a file views/500.handlebars and add an error handler after all of your routes:

``` js
app.use((err, req, res, next) => {
  console.error(err.message, err.stack)
  app.status(500).render('500')
})
```

It’s always a good practice to provide a custom error page; it not only looks more professional to your users when errors do occur, but also allows you to take action when errors occur.

Let’s try something worse:

``` js
app.get('/epic-fail', (req, res) => {
  process.nextTick(() =>
    throw new Error('Kaboom!')
  )
})
```

Go ahead and try it. The result is considerably more catastrophic: it brings your whole server down! In addition to not displaying a friendly error message to your user, now your server is down, and no requests are being served. This is because setTimeout is executing asynchronously; execution of the function with the exception is being deferred until Node is idle. The problem is, when Node is idle and gets around to executing the function, it no longer has context about the request it was being served from, so it has no recourse but to unceremoniously shut down the whole server, because now it’s in an undefined state. (Node can’t know the purpose of the function or its caller, so it can no longer assume that any further functions will work correctly.)

> process.nextTick is similar to calling setTimeout with an argument of 0, but it’s more efficient. We’re using it here for demonstration purposes; it’s not something you would generally use in server-side code. However, in coming chapters, we will be dealing with many things that execute asynchronously, such as database access, filesystem access, and network access, to name a few, and they are all subject to this problem.

There is action that we can take to handle uncaught exceptions, but if Node can’t determine the stability of your application, neither can you. In other words, if there is an uncaught exception, the only recourse is to shut down the server. 

The best we can do in this circumstance is to shut down as gracefully as possible and have a failover mechanism. The easiest failover mechanism is to use a cluster. If your application is operating in clustered mode and one worker dies, the master will spawn another worker to take its place.

So with that in mind, how can we shut down as gracefully as possible when confronted with an unhandled exception? Node’s mechanism for dealing with this is the uncaughtException event.

``` js
process.on('uncaughtException', err => {
  console.error('UNCAUGHT EXCEPTION\n', err.stack);
  // do any cleanup you need to do here...close
  // database connections, etc.
  process.exit(1)
})
```

It’s unrealistic to expect that your application will never experience uncaught exceptions, but you should have a mechanism in place to record the exception and notify you when it happens, and you should take it seriously. Try to determine why it happened so you can fix it. Services like Sentry, Rollbar, Airbrake, and New Relic are a great way to record these kinds of errors for analysis.

For example, to use Sentry, first you have to register for a free account, at which point you will receive a data source name (DSN), and then you can modify your exception handler:

``` js
const Sentry = require('@sentry/node')
Sentry.init({ dsn: '** YOUR DSN GOES HERE **' })

process.on('uncaughtException', err => {
  // do any cleanup you need to do here...close
  // database connections, etc.
  Sentry.captureException(err)
  process.exit(1)
})
```


## Scaling Out with Multiple Servers

To achieve this kind of parallelism, you need a proxy server. (It’s often called a reverse proxy or forward-facing proxy to distinguish it from proxies commonly used to access external networks, but I find this language to be confusing and unnecessary, so I will simply refer to it as a proxy.)

If you do configure a proxy server, make sure you tell Express that you are using a proxy and that it should be trusted:

``` js
app.enable('trust proxy')
```

Doing this will ensure that req.ip, req.protocol, and req.secure will reflect the details about the connection between the client and the proxy, not between the client and your app. Also, req.ips will be an array that indicates the original client IP and the names or IP addresses of any intermediate proxies.


## Monitoring Your Website

Monitoring your website is one of the most important—and most often overlooked—QA measures you can take.

There’s nothing you can do about failures: they are as inevitable as death and taxes. However, if there is one thing you can do to convince your boss and your clients that you are great at your job, it’s to always know about failures before they do.


## Third-Party Uptime Monitors

UptimeRobot is free for up to 50 monitors and is simple to configure. Alerts can go to email, SMS (text message), 

You can monitor for the return code from a single page (anything other than a 200 is considered an error) or to check for the presence or absence of a keyword on the page.


## Stress Testing

Stress testing (or load testing) is designed to give you some confidence that your server will function under the load of hundreds or thousands of simultaneous requests. 

Let’s add a simple stress test using Artillery. First, install Artillery by running npm install -g artillery; then edit your package.json file and add the following to the scripts section:

``` json
  "scripts": {
    "stress": "artillery quick --count 10 -n 20 http://localhost:3000/"
  }
```

Make sure your application is running (in a separate terminal window, for example) and then run npm run stress. You’ll see statistics like this:

``` log
Started phase 0, duration: 1s @ 16:43:37(-0700) 2019-04-14
Report @ 16:43:38(-0700) 2019-04-14
Elapsed time: 1 second
  Scenarios launched:  10
  Scenarios completed: 10
  Requests completed:  200
  RPS sent: 147.06
  Request latency:
    min: 1.8
    max: 10.3
    median: 2.5
    p95: 4.2
    p99: 5.4
  Codes:
    200: 200

All virtual users finished
Summary report @ 16:43:38(-0700) 2019-04-14
  Scenarios launched:  10
  Scenarios completed: 10
  Requests completed:  200
  RPS sent: 145.99
  Request latency:
    min: 1.8
    max: 10.3
    median: 2.5
    p95: 4.2
    p99: 5.4
  Scenario counts:
    0: 10 (100%)
  Codes:
    200: 200
```

If you stress test your application regularly and benchmark it, you’ll be able to recognize problems. If you just finished a feature and you find that your connection times have tripled, you might want to do some performance tuning on your new feature!

