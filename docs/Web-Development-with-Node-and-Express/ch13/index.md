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

