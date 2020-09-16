[Prev][prev]
|
[Table of Contents](../)
|
[Next][next]

[prev]: ../ch12
[next]: ../ch14

# Persistence

## Filesystem Persistence

One way to achieve persistence is to simply save data to so-called flat files (flat because there’s no inherent structure in a file; it’s just a sequence of bytes). Node makes filesystem persistence possible through the fs (filesystem) module.

Filesystem persistence has some drawbacks. In particular, it doesn’t scale well. The minute you need more than one server to meet traffic demands, you will run into problems with filesystem persistence, unless all of your servers have access to a shared filesystem.

Also, because flat files have no inherent structure, the burden of locating, sorting, and filtering data will be on your application. For these reasons, you should favor databases over filesystems for storing data. The one exception is storing binary files, such as images, audio files, or videos.

If you do need to store binary data, keep in mind that filesystem storage still has the problem of not scaling well. If your hosting doesn’t have access to a shared filesystem (which is usually the case), you should consider storing binary files in a database (which usually requires some configuration so the database doesn’t grind to a stop) or a cloud-based storage service, like Amazon S3 or Microsoft Azure Storage.

Now that we have the caveats out of the way, let’s look at Node’s filesystem support. We’ll revisit the vacation photo contest from Chapter 8. In our application file, let’s fill in the handler that processes that form (ch13/00-mongodb/lib/handlers.js in the companion repo):

``` js
const pathUtils = require('path')
const fs = require('fs')

// create directory to store vacation photos (if it doesn't already exist)
const dataDir = pathUtils.resolve(__dirname, '..', 'data')
const vacationPhotosDir = pathUtils.join(dataDir, 'vacation-photos')
if(!fs.existsSync(dataDir)) fs.mkdirSync(dataDir)
if(!fs.existsSync(vacationPhotosDir)) fs.mkdirSync(vacationPhotosDir)

function saveContestEntry(contestName, email, year, month, photoPath) {
  // TODO...this will come later
}

// we'll want these promise-based versions of fs functions later
const { promisify } = require('util')
const mkdir = promisify(fs.mkdir)
const rename = promisify(fs.rename)

exports.api.vacationPhotoContest = async (req, res, fields, files) => {
  const photo = files.photo[0]
  const dir = vacationPhotosDir + '/' + Date.now()
  const path = dir + '/' + photo.originalFilename
  await mkdir(dir)
  await rename(photo.path, path)
  saveContestEntry('vacation-photo', fields.email,
    req.params.year, req.params.month, path)
  res.send({ result: 'success' })
}
```

There’s a lot going on there, so let’s break it down. We first create a directory to store the uploaded files (if it doesn’t already exist). You’ll probably want to add the data directory to your .gitignore file so you don’t accidentally commit uploaded files. Recall from Chapter 8 that we’re handling the actual file upload in meadowlark.js and calling our handler with the files already decoded. What we get is an object (files) that contains the information about the uploaded files.

Since we want to prevent collisions, we can’t just use the filename the user uploaded (in case two users both upload portland.jpg). To avoid this problem, we create a unique directory based on the timestamp; it’s pretty unlikely that two users will both upload portland.jpg in the same millisecond! Then we rename (move) the uploaded file (our file processor will have given it a temporary name, which we can get from the path property) to our constructed name.

Finally, we need some way to associate the files that users upload with their email addresses (and the month and year of the submission). We could encode this information into the file or directory names, but we are going to prefer storing this information in a database.

Since we haven’t learned how to do that yet, we’re going to encapsulate that functionality in the vacationPhotoContest function and complete that function later in this chapter.

Even though filesystem persistence has its drawbacks, it’s frequently used for intermediate file storage, and it’s useful to know how to use the Node filesystem library. However, to address the deficiencies of filesystem storage, let’s turn our attention to cloud persistence.


## Cloud Persistence

Cloud storage is becoming increasingly popular, and I highly recommend you take advantage of one of these inexpensive, robust services.

(https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/getting-started-nodejs.html)

The good news is that once you get past this initial configuration, using cloud persistence is quite easy. Here’s an example of how easy it is to save a file to an Amazon S3 account:

``` js
const filename = 'customerUpload.jpg'

s3.putObject({
  Bucket: 'uploads',
  Key: filename,
  Body: fs.readFileSync(__dirname + '/tmp/ + filename),
})
```


## Database Persistence

Traditionally, the word database is shorthand for relational database management system (RDBMS). Relational databases, such as Oracle, MySQL, PostgreSQL, or SQL Server, are based on decades of research and formal database theory. It is a technology that is quite mature at this point, and the power of these databases is unquestionable. 

However, we now have the luxury of expanding our ideas of what constitutes a database. NoSQL databases have come into vogue in recent years, and they’re challenging the status quo of internet data storage.

The two most popular types of NoSQL databases are document databases and key-value databases. Document databases excel at storing objects, which makes them a natural fit for Node and JavaScript. Key-value databases, as the name implies, are extremely simple and are a great choice for applications with data schemas that are easily mapped into key-value pairs.

I feel that document databases represent the optimal compromise between the constraints of relational databases and the simplicity of key-value databases, and for that reason, we will be using a document database for our first example. MongoDB is the leading document database and is robust and established at this point.


## A Note on Performance

Relational databases have traditionally relied on their rigid data structures and decades of optimization research to achieve high performance. NoSQL databases, on the other hand, have embraced the distributed nature of the internet and, like Node, have instead focused on concurrency to scale performance (relational databases also support concurrency, but this is usually reserved for the most demanding applications).

Planning for database performance and scalability is a large, complex topic that is beyond the scope of this book. If your application requires a high level of database performance, I recommend starting with [Kristina Chodorow and Michael Dirolf’s MongoDB: The Definitive Guide (O’Reilly)](https://learning.oreilly.com/library/view/mongodb-the-definitive/9781449381578/).


## Abstracting the Database Layer

Whenever possible, there is value in abstracting your technology choices, which refers to writing some kind of API layer to generalize the underlying technology choices. If done right, it reduces the cost of switching out the component in question. However, it comes at a cost: writing the abstraction layer is one more thing you have to write and maintain.

Happily, our abstraction layer will be very small, as we’re supporting only a handful of features for the purposes of this book. For now, the features will be as follows:

* Returning a list of active vacations from the database
* Storing the email address of users who want to be notified when certain vacations are in season

We’re going to keep our abstraction layer simple for the purposes of this book. We’ll contain it in a file called db.js that will export two methods that we’ll start by just providing dummy implementations:

``` js
module.exports = {
  getVacations: async (options = {}) => {
    // let's fake some vacation data:
    const vacations = [
      {
        name: 'Hood River Day Trip',
        slug: 'hood-river-day-trip',
        category: 'Day Trip',
        sku: 'HR199',
        description: 'Spend a day sailing on the Columbia and ' +
          'enjoying craft beers in Hood River!',
        location: {
          // we'll use this for geocoding later in the book
          search: 'Hood River, Oregon, USA',
        },
        price: 99.95,
        tags: ['day trip', 'hood river', 'sailing', 'windsurfing', 'breweries'],
        inSeason: true,
        maximumGuests: 16,
        available: true,
        packagesSold: 0,
      }
    ]
    // if the "available" option is specified, return only vacations that match
    if(options.available !== undefined)
      return vacations.filter(({ available }) => available === options.available)
    return vacations
  },
  addVacationInSeasonListener: async (email, sku) => {
    // we'll just pretend we did this...since this is
    // an async function, a new promise will automatically
    // be returned that simply resolves to undefined
  },
}
```

Now that we have an abstraction foundation for our database layer, let’s look at how we can implement database storage with MongoDB.

## Setting Up MongoDB

The difficulty involved in setting up a MongoDB instance varies with your operating system. For this reason, we’ll be avoiding the problem altogether by using an excellent free MongoDB hosting service, mLab.

> mLab is not the only MongoDB service available. The MongoDB company itself is now offering free and low-cost database hosting through its product MongoDB Atlas. Free accounts are not recommended for production purposes, though. Both mLab and MongoDB Atlas offer production-ready accounts, so you should look into their pricing before making a choice. It will be less hassle to stay with the same hosting service when you make the switch to production.\


## Mongoose

While there’s a low-level driver available for MongoDB, you’ll probably want to use an object document mapper (ODM). The most popular ODM for MongoDB is Mongoose.

One of the advantages of JavaScript is that its object model is extremely flexible. If you want to add a property or method to an object, you just do it, and you don’t need to worry about modifying a class. Unfortunately, that kind of freewheeling flexibility can have a negative impact on your databases because they can become fragmented and hard to optimize.

Before we get started, we’ll need to install the Mongoose module:

``` sh
npm install mongoose
```

Then we’ll add our database credentials to our .credentials.development.json file:

``` json
"mongo": {
    "connectionString": "your_dev_connection_string"
  }
}
```

Notice that we could establish a second set of credentials for production by creating a .credentials.production.js file and using NODE_ENV=production; you’ll want to do this when it’s time to go live!


## Database Connections with Mongoose

We’ll start by creating a connection to our database. We’ll put our database initialization code in db.js, along with the dummy API we created earlier (ch13/00-mongodb/db.js in the companion repo):

``` js
const mongoose = require('mongoose')
const { connectionString } = credentials.mongo
if(!connectionString) {
  console.error('MongoDB connection string missing!')
  process.exit(1)
}
mongoose.connect(connectionString)
const db = mongoose.connection
db.on('error' err => {
  console.error('MongoDB error: ' + err.message)
  process.exit(1)
})
db.once('open', () => console.log('MongoDB connection established'))

module.exports = {
  getVacations: async () => {
    //...return fake vacation data
  },
  addVacationInSeasonListener: async (email, sku) => {
    //...do nothing
  },
}
```

Any file that needs to access the database can simply import db.js. However, we want the initialization to happen right away, before we need the API, so we’ll go ahead and import this from meadowlark.js (where we don’t need to do anything with the API):

``` js
require('./db')
```

## Creating Schemas and Models

Let’s create a vacation package database for Meadowlark Travel. We start by defining a schema and creating a model from it. Create the file models/vacation.js (ch13/00-mongodb/models/vacation.js in the companion repo):

``` js
const mongoose = require('mongoose')

const vacationSchema = mongoose.Schema({
  name: String,
  slug: String,
  category: String,
  sku: String,
  description: String,
  location: {
    search: String,
    coordinates: {
      lat: Number,
      lng: Number,
    },
  },
  price: Number,
  tags: [String],
  inSeason: Boolean,
  available: Boolean,
  requiresWaiver: Boolean,
  maximumGuests: Number,
  notes: String,
  packagesSold: Number,
})

const Vacation = mongoose.model('Vacation', vacationSchema)
module.exports = Vacation
```

Once we have the schema, we create a model using mongoose.model: at this point, Vacation is very much like a class in traditional object-oriented programming. Note that we have to define our methods before we create our model.

We are exporting the Vacation model object created by Mongoose. While we could use this model directly, that would be undermining our effort to provide a database abstraction layer. So we will choose to import it only from the db.js file and let the rest of our application use its methods. Add the Vacation model to db.js:

``` js
const Vacation = require('./models/vacation')
```


## Seeding Initial Data

Eventually, you may want to create a way to manage products, but for the purposes of this book, we’re just going to do it in code (ch13/00-mongodb/db.js in the companion repo):

``` js
Vacation.find((err, vacations) => {
  if(err) return console.error(err)
  if(vacations.length) return

  new Vacation({
    name: 'Hood River Day Trip',
    slug: 'hood-river-day-trip',
    category: 'Day Trip',
    sku: 'HR199',
    description: 'Spend a day sailing on the Columbia and ' +
      'enjoying craft beers in Hood River!',
    location: {
      search: 'Hood River, Oregon, USA',
    },
    price: 99.95,
    tags: ['day trip', 'hood river', 'sailing', 'windsurfing', 'breweries'],
    inSeason: true,
    maximumGuests: 16,
    available: true,
    packagesSold: 0,
  }).save()

  new Vacation({
    name: 'Oregon Coast Getaway',
    slug: 'oregon-coast-getaway',
    category: 'Weekend Getaway',
    sku: 'OC39',
    description: 'Enjoy the ocean air and quaint coastal towns!',
    location: {
      search: 'Cannon Beach, Oregon, USA',
    },
    price: 269.95,
    tags: ['weekend getaway', 'oregon coast', 'beachcombing'],
    inSeason: false,
    maximumGuests: 8,
    available: true,
    packagesSold: 0,
  }).save()

  new Vacation({
      name: 'Rock Climbing in Bend',
      slug: 'rock-climbing-in-bend',
      category: 'Adventure',
      sku: 'B99',
      description: 'Experience the thrill of climbing in the high desert.',
      location: {
        search: 'Bend, Oregon, USA',
      },
      price: 289.95,
      tags: ['weekend getaway', 'bend', 'high desert', 'rock climbing'],
      inSeason: true,
      requiresWaiver: true,
      maximumGuests: 4,
      available: false,
      packagesSold: 0,
      notes: 'The tour guide is currently recovering from a skiing accident.',
  }).save()
})
```

The first time this executes, though, find will return an empty list, so we proceed to create two vacations and then call the save method on them, which saves these new objects to the database.


## Retrieving Data

We’ve already seen the find method, which is what we’ll use to display a list of vacations. However, this time we’re going to pass an option to find that will filter the data.

Specifically, we want to display only vacations that are currently available.

Create a view for the products page, views/vacations.handlebars:

``` hbs
<h1>Vacations</h1>
{{#each vacations}}
  <div class="vacation">
    <h3>{{name}}</h3>
    <p>{{description}}</p>
    {{#if inSeason}}
      <span class="price">{{price}}</span>
      <a href="/cart/add?sku={{sku}}" class="btn btn-default">Buy Now!</a>
    {{else}}
      <span class="outOfSeason">We're sorry, this vacation is currently
      not in season.
      {{! The "notify me when this vacation is in season"
          page will be our next task. }}
      <a href="/notify-me-when-in-season?sku={{sku}}">Notify me when
      this vacation is in season.</a>
    {{/if}}
  </div>
{{/each}}
```

Now we can create route handlers that hook it all up. 

In lib/handlers.js (don’t forget to import ../db), we create the handler:

``` js
exports.listVacations = async (req, res) => {
  const vacations = await db.getVacations({ available: true })
  const context = {
    vacations: vacations.map(vacation => ({
      sku: vacation.sku,
      name: vacation.name,
      description: vacation.description,
      price: '$' + vacation.price.toFixed(2),
      inSeason: vacation.inSeason,
    }))
  }
  res.render('vacations', context)
}
```

We add a route that calls the handler in meadowlark.js:

``` js
app.get('/vacations', handlers.listVacations)
```

If you run this example, you’ll see only the one vacation from our dummy database implementation. That’s because we’ve initialized the database and seeded its data, but we haven’t replaced the dummy implementation with a real one. So let’s do that now. Open db.js and modify getVacations:

``` js
module.exports = {
  getVacations: async (options = {}) => Vacation.find(options),
  addVacationInSeasonListener: async (email, sku) => {
    //...
  },
}
```

That was easy! A one-liner. Partially this is because Mongoose is doing a lot of the heavy lifting for us, and the way we’ve designed our API is similar to the way Mongoose works. When we adapt this later to PostgreSQL, you’ll see we have to do a little more work.

Why did we map the products returned from the database to a nearly identical object? One reason is that we want to display the price in a neatly formatted way, so we have to convert it to a formatted string.

We could have saved some typing by doing this:

``` js
const context = {
  vacations: products.map(vacations => {
    vacation.price = '$' + vacation.price.toFixed(2)
    return vacation
  })
}
```

That would certainly save us a few lines of code, but in my experience, there are good reasons not to pass unmapped database objects directly to views. The view gets a bunch of properties it may not need, possibly in formats that are incompatible with it. Our example is pretty simple so far, but once it starts to get more complicated, you’ll probably want to do even more customization of the data that’s passed to a view. It also makes it easy to accidentally expose confidential information or information that could compromise the security of your website. For these reasons, I recommend mapping the data that’s returned from the database and passing only what’s needed onto the view (transforming as necessary, as we did with price).


## Adding Data

We’ve already seen how we can add data (we added data when we seeded the vacation collection) and how we can update data (we update the count of packages sold when we book a vacation), but let’s take a look at a slightly more involved scenario that highlights the flexibility of document databases.

When a vacation is out of season, we display a link that invites the customer to be notified when the vacation is in season again. Let’s hook up that functionality. First, we create the schema and model (models/vacationInSeasonListener.js):

``` js
const mongoose = require('mongoose')

const vacationInSeasonListenerSchema = mongoose.Schema({
  email: String,
  skus: [String],
})
const VacationInSeasonListener = mongoose.model('VacationInSeasonListener',
  vacationInSeasonListenerSchema)

module.exports = VacationInSeasonListener
```

Then we’ll create our view, views/notify-me-when-in-season.handlebars:

``` hbs
<div class="formContainer">
  <form class="form-horizontal newsletterForm" role="form"
      action="/notify-me-when-in-season" method="POST">
    <input type="hidden" name="sku" value="{{sku}}">
    <div class="form-group">
      <label for="fieldEmail" class="col-sm-2 control-label">Email</label>
      <div class="col-sm-4">
        <input type="email" class="form-control" required
          id="fieldEmail" name="email">
      </div>
    </div>
    <div class="form-group">
      <div class="col-sm-offset-2 col-sm-4">
        <button type="submit" class="btn btn-default">Submit</button>
      </div>
    </div>
  </form>
</div>
```

Then the route handlers:

``` js
exports.notifyWhenInSeasonForm = (req, res) =>
  res.render('notify-me-when-in-season', { sku: req.query.sku })

exports.notifyWhenInSeasonProcess = (req, res) => {
  const { email, sku } = req.body
  await db.addVacationInSeasonListener(email, sku)
  return res.redirect(303, '/vacations')
}
```

Finally, we add a real implementation to db.js:

``` js
const VacationInSeasonListener = require('./models/vacationInSeasonListener')

module.exports = {
  getVacations: async (options = {}) => Vacation.find(options),
  addVacationInSeasonListener: async (email, sku) => {
    await VacationInSeasonListener.updateOne(
      { email },
      { $push: { skus: sku } },
      { upsert: true }
    )
  },
}
```

What magic is this? How can we “update” a record in the VacationInSeasonListener collection before it even exists? The answer lies in a Mongoose convenience called an upsert

Basically, if a record with the given email address doesn’t exist, it will be created. If a record does exist, it will be updated. Then we use the magic variable $push to indicate that we want to add a value to an array.


## PostgreSQL

Fortunately, there is robust support for every major relational database in the JavaScript ecosystem, and if you want or need to use a relational database, you shouldn’t have any problem.

Let’s take our vacation database and reimplement it using a relational database. For this example, we’ll use PostgreSQL, a popular and sophisticated open source relational database. The techniques and principles we’ll use will be similar for any relational database.

However, since most readers interested in this topic are probably already familiar with relational databases and SQL, we’ll use a Node PostgreSQL client directly.

There are many options for online PostgreSQL; for this example, I’ll be using ElephantSQL. Getting started couldn’t be simpler: create an account (you can use your GitHub account to log in), and click Create New Instance. All you have to do is give it a name (for example, “meadowlark”) and select a plan (you can use their free plan). You’ll also specify a region (try to pick the one closest to you). Once you’re all set up, you’ll find a Details section that lists information about your instance. Copy the URL (connection string), which includes the username, password, and instance location all in one convenient string.

Put that string in your .credentials.development.json file:

``` json
"postgres": {
  "connectionString": "your_dev_connection_string"
}
```

One difference between object databases and RDBMSs is that you typically do more up-front work to define the schema of an RDBMS and use data definition SQL to create the schema before adding or retrieving data. In keeping with this paradigm, we’ll do that as a separate step instead of letting our ODM or ORM handle it, as we did with MongoDB.

First, we’ll have to install the pg client library (npm install pg). Then create db-init.js, which will be run only to initialize our database and is distinct from our db.js file, which is used every time the server starts up (ch13/01-postgres/db.js in the companion repo):

``` js
const { credentials } = require('./config')

const { Client } = require('pg')
const { connectionString } = credentials.postgres
const client = new Client({ connectionString })

const createScript = `
  CREATE TABLE IF NOT EXISTS vacations (
    name varchar(200) NOT NULL,
    slug varchar(200) NOT NULL UNIQUE,
    category varchar(50),
    sku varchar(20),
    description text,
    location_search varchar(100) NOT NULL,
    location_lat double precision,
    location_lng double precision,
    price money,
    tags jsonb,
    in_season boolean,
    available boolean,
    requires_waiver boolean,
    maximum_guests integer,
    notes text,
    packages_sold integer
  );
`

const getVacationCount = async client => {
  const { rows } = await client.query('SELECT COUNT(*) FROM VACATIONS')
  return Number(rows[0].count)
}

const seedVacations = async client => {
  const sql = `
    INSERT INTO vacations(
      name,
      slug,
      category,
      sku,
      description,
      location_search,
      price,
      tags,
      in_season,
      available,
      requires_waiver,
      maximum_guests,
      notes,
      packages_sold
    ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, $13, $14)
  `
  await client.query(sql, [
    'Hood River Day Trip',
    'hood-river-day-trip',
    'Day Trip',
    'HR199',
    'Spend a day sailing on the Columbia and enjoying craft beers in Hood River!',
    'Hood River, Oregon, USA',
    99.95,
    `["day trip", "hood river", "sailing", "windsurfing", "breweries"]`,
    true,
    true,
    false,
    16,
    null,
    0,
  ])
  // we can use the same pattern to insert other vacation data here...
}

client.connect().then(async () => {
  try {
    console.log('creating database schema')
    await client.query(createScript)
    const vacationCount = await getVacationCount(client)
    if(vacationCount === 0) {
      console.log('seeding vacations')
      await seedVacations(client)
    }
  } catch(err) {
    console.log('ERROR: could not initialize database')
    console.log(err.message)
  } finally {
    client.end()
  }
})
```

Let’s start at the bottom of this file. We take our database client (client) and call connect() on it, which establishes a database connection and returns a promise. 

When that promise resolves, we can take actions against the database.

The first thing we do is invoke client.query(createScript), which will create our vacations table (also known as a relation).

One thing you may note is that we use snake_case to name our fields instead of camelCase. That is, what was “inSeason” has become “in_season.” While it is possible to use camelCase to name structures in PostgreSQL, you have to quote any identifiers with capital letters, which ends up being more trouble than it’s worth. We’ll come back to that a little later.

In traditional database design, we would probably create a new table to relate vacations to tags (this is called normalization). And we could do that here. But here is where we might decide to strike some compromises between traditional relational database design and doing things in the “JavaScript way.”

We could make this a text field and separate our tags with commas, but then we would have to parse out our tags, and PostgreSQL gives us a better way in JSON data types. We’ll see shortly that by specifying this as JSON (jsonb, a binary representation that’s usually higher performance), we can store this as a JavaScript array, and a JavaScript array comes out, just as we had in MongoDB.

Finally, we insert our seed data into the database by using the same basic concept as before: if the vacations table is empty, we add some initial data; otherwise, we assume we’ve already done that.

You’ll note that inserting our data is a little more unwieldy than it was with MongoDB. There are ways to solve this problem, but for this example, I want to be explicit about the use of SQL. We could write a function to make insert statements more naturally, or we could use an ORM (more on this later). But for now, the SQL gets the job done, and it should be comfortable for anyone who already knows SQL.

Note that although this script is designed to be run only once to initialize and seed our database, we’ve written it in a way that it’s safe to run multiple times. We included the IF NOT EXISTS option, and we check to see whether the vacations table is empty before adding seed data.

We can now run the script to initialize our database:

``` sh
node db-init.js
```

Database servers can typically handle only a limited number of connections at a time, so web servers usually implement a strategy called connection pooling to balance the overhead of establishing a connection with the danger of leaving connections open too long and choking the server. 

Fortunately, the details of this are handled for you by the PostgreSQL Node client.

We’ll take a slightly different strategy with our db.js file this time. Instead of a file we just require to establish the database connection, it will return an API that we write that handles the details of communicating with the database.

Recall that when we created our model, we used snake_case for our database schema, but all of our JavaScript code uses camelCase.

We could write our own function to do that translation, but we’ll rely on a popular utility library called Lodash, which makes it extremely easy. Just run npm install lodash to install it.

Right now, our database needs are very modest. All we need to do is fetch all available vacation packages, so our db.js file will look like this (ch13/01-postgres/db.js in the companion repo):

``` js
const { Pool } = require('pg')
const _ = require('lodash')

const { credentials } = require('./config')

const { connectionString } = credentials.postgres
const pool = new Pool({ connectionString })

module.exports = {
  getVacations: async () => {
    const { rows } = await pool.query('SELECT * FROM VACATIONS')
    return rows.map(row => {
      const vacation = _.mapKeys(row, (v, k) => _.camelCase(k))
      vacation.price = parseFloat(vacation.price.replace(/^\$/, ''))
      vacation.location = {
        search: vacation.locationSearch,
        coordinates: {
          lat: vacation.locationLat,
          lng: vacation.locationLng,
        },
      }
      return vacation
    })
  }
}
```

Short and sweet! We’re exporting a single method called getVacations that does as advertised. It also uses Lodash’s mapKeys and camelCase functions to convert our database properties to camelCase.


## Adding Data with PostgreSQL

We’ll start by adding the following data definition to the createScript string in db-init.js:

``` sql
CREATE TABLE IF NOT EXISTS vacation_in_season_listeners (
  email varchar(200) NOT NULL,
  sku varchar(20) NOT NULL,
  PRIMARY KEY (email, sku)
);
```

Remember that we took care to write db-init.js in a nondestructive fashion so we could run it at any time. So we can just run it again to create the vacation_in_season_listeners table.

Now we can modify db.js to include a method to update this table:

``` js
module.exports = {
  //...
  addVacationInSeasonListener: async (email, sku) => {
    await pool.query(
      'INSERT INTO vacation_in_season_listeners (email, sku) ' +
      'VALUES ($1, $2) ' +
      'ON CONFLICT DO NOTHING',
      [email, sku]
    )
  },
}
```

PostgreSQL’s ON CONFLICT clause essentially enables upserts. In this case, if the exact combination of email and SKU is already present, the user has already registered to be notified, so we don’t need to do anything. If we had other columns in this table (such as the date of last registration), we might want to use a more sophisticated ON CONFLICT clause (see the PostgreSQL INSERT documentation for more information). 

Note also that this behavior is dependent on the way we defined the table. We made email and SKU a composite primary key, meaning that there can’t be any duplicates, which in turn necessitated the ON CONFLICT clause (otherwise, the INSERT command would result in an error the second time a user tried to register for a notification on the same vacation).


## Using a Database for Session Storage

As we discussed in Chapter 9, using a memory store for session data is unsuitable in a production environment. Fortunately, it’s easy to use a database as a session store.

While we could use our existing MongoDB or PostgreSQL database for a session store, a full-blown database is overkill for session storage, which is a perfect use case for a key-value database.

As I write this, the most popular key-value databases for session stores are Redis and Memcached. In keeping with the other examples in this chapter, we’ll be using a free online service to provide a Redis database.

Start by heading over to [Redis Labs](https://redislabs.com/) and create an account. Then create a free subscription and plan. Choose Cache for the plan and give the database a name; you can leave the rest of the settings at their defaults.

You’ll reach a View Database screen, and, as I write this, the critical information doesn’t populate for a few seconds, so be patient. What you’ll want is the Endpoint field and the Redis Password under Access Control & Security (it’s hidden by default, but there’s a little button next to it that will show it). Take these and put them in your .credentials.development.json file:

``` json
"redis": {
  "url": "redis://:<YOUR PASSWORD>@<YOUR ENDPOINT>"
}
```

Redis allows connection with a password only; the colon that separates username from password is still required, however.

We’ll be using a package called connect-redis to provide Redis session storage. 

Once you’ve installed it (npm install connect-redis), we can set it up in our main application file. We still use expression-session, but now we pass a new property to it, store, which configures it to use a database. Note that we have to pass expressSession to the function returned from connect-redis to get the constructor: this is a pretty common quirk of session stores (ch13/00-mongodb/meadowlark.js or ch13/01-postgres/meadowlark.js in the companion repo):

``` js
const expressSession = require('express-session')
const RedisStore = require('connect-redis')(expressSession)

app.use(cookieParser(credentials.cookieSecret))
app.use(expressSession({
  resave: false,
  saveUninitialized: false,
  secret: credentials.cookieSecret,
  store: new RedisStore({
    url: credentials.redis.url,
    logErrors: true,  // highly recommended!
  }),
}))
```

Imagine we want to be able to display vacation prices in different currencies. Furthermore, we want the site to remember the user’s currency preference.

We’ll start by adding a currency picker at the bottom of our vacations page:

``` js
<hr>
<p>Currency:
    <a href="/set-currency/USD" class="currency {{currencyUSD}}">USD</a> |
    <a href="/set-currency/GBP" class="currency {{currencyGBP}}">GBP</a> |
    <a href="/set-currency/BTC" class="currency {{currencyBTC}}">BTC</a>
</p>
```

Now here’s a little CSS (you can put this inline in your views/layouts/main.handlebars file or link to a CSS file in your public directory):

``` css
a.currency {
  text-decoration: none;
}
.currency.selected {
  font-weight: bold;
  font-size: 150%;
}

```

Lastly, we’ll add a route handler to set the currency and modify our route handler for /vacations to display prices in the current currency (ch13/00-mongodb/lib/handlers.js or ch13/01-postgres/lib/handlers.js in the companion repo):

``` js
exports.setCurrency = (req, res) => {
  req.session.currency = req.params.currency
  return res.redirect(303, '/vacations')
}

function convertFromUSD(value, currency) {
  switch(currency) {
    case 'USD': return value * 1
    case 'GBP': return value * 0.79
    case 'BTC': return value * 0.000078
    default: return NaN
  }
}

exports.listVacations = (req, res) => {
  Vacation.find({ available: true }, (err, vacations) => {
    const currency = req.session.currency || 'USD'
    const context = {
      currency: currency,
      vacations: vacations.map(vacation => {
        return {
          sku: vacation.sku,
          name: vacation.name,
          description: vacation.description,
          inSeason: vacation.inSeason,
          price: convertFromUSD(vacation.price, currency),
          qty: vacation.qty,
        }
      })
    }
    switch(currency){
      case 'USD': context.currencyUSD = 'selected'; break
      case 'GBP': context.currencyGBP = 'selected'; break
      case 'BTC': context.currencyBTC = 'selected'; break
    }
    res.render('vacations', context)
  })
}
```

You’ll also have to add a route for setting the currency in meadowlark.js:

``` js
app.get('/set-currency/:currency', handlers.setCurrency)
```

Another reader’s exercise would be to make the set-currency route general-purpose to make it more useful. Currently, it will always redirect to the vacations page, but what if you wanted to use it on a shopping cart page? See if you can think of one or two ways of solving this problem.

If you look in your database, you’ll find there’s a new collection called sessions. If you explore that collection, you’ll find a document with your session ID (property sid) and your currency preference.