# Tidying Up

## File and Directory Structure

It’s typical to try to restrict the number of files in your project root. Typically, you’ll find configuration files (like package.json), a README.md file, and a bunch of directories. Most source code goes under a directory often called src.

For real-world projects, you’ll probably eventually find that your project root gets cluttered if you’re putting source code there, and you’ll want to collect those files under a directory like src.

I’ve also mentioned that I prefer to name my main application file (sometimes called the entry point) after the project itself (meadowlark.js) as opposed to something generic like index.js, app.js, or server.js.

## Version Control

Documentation

* Being able to go back through the history of a project to see the decisions that were made and the order in which components were developed can be valuable documentation. Having a technical history of your project can be quite useful.

Attribution

* If you work on a team, attribution can be hugely important. Whenever you find something in code that is opaque or questionable, knowing who made that change can save you many hours. It could be that the comments associated with the change are sufficient to answer your questions, and if not, you’ll know who to talk to.

Experimentation

* A good version control system enables experimentation. You can go off on a tangent, trying something new, without fear of affecting the stability of your project. If the experiment is successful, you can fold it back into the project, and if it is not successful, you can abandon it.

## npm Packages

The npm packages that your project relies on reside in a directory called node_modules.

Two of the main purposes of the package.json file are to describe your project and to list its dependencies.

``` json
{
  "dependencies": {
    "express": "^4.16.4",
    "express-handlebars": "^3.0.0"
  }
}
```

As of version of 5 of npm, an additional file, package-lock.json, will be created. Whereas package.json can be “loose” in its specification of dependency versions (with the ^ and ~ version modifiers), package-lock.json records the exact versions that were installed, which can be helpful if you need to re-create the exact dependency versions in your project.

## Project Metadata

The other purpose of the package.json file is to store project metadata, such as the name of the project, authors, license information, and so on.

## Node Modules

Node modules, as the name implies, offer a mechanism for modularization and encapsulation. npm packages provide a standardized scheme for storing, versioning, and referencing projects (which are not restricted to modules).

For example, we import Express itself as a module in our main application file:

``` js
const express = require('express')
```

Let’s see how we can modularize the fortune cookie functionality we implemented in the previous chapter.

First let’s create a directory to store our modules. You can call it whatever you want, but lib (short for “library”) is a common choice. In that folder, create a file called fortune.js (ch04/lib/fortune.js in the companion repo):

``` js
const fortuneCookies = [
  "Conquer your fears or they will conquer you.",
  "Rivers need springs.",
  "Do not fear what you don't know.",
  "You will have a pleasant surprise.",
  "Whenever possible, keep it simple.",
]

exports.getFortune = () => {
  const idx = Math.floor(Math.random()*fortuneCookies.length)
  return fortuneCookies[idx]
}
```

The important thing to note here is the use of the global variable exports. If you want something to be visible outside of the module, you have to add it to exports.

It is traditional (but not required) to specify imports at the top of the file, so at the top of the meadowlark.js file, add the following line (ch04/meadowlark.js in the companion repo):

``` js
const fortune = require('./lib/fortune')
```
