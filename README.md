# Period-2 Node.js, Express + JavaScript Backend Testing, NoSQL, MongoDB and Mongoose
## Why would you consider a Scripting Language as JavaScript as your Backend Platform?
It is good to have it as both front- and backend as you will have everything together instead of having to setup both parts seperatly.
Furthermore it's easy/fast to setup, as everything is made the same place, and can be setup using a template with node.js / express

## Explain Pros & Cons in using Node.js + Express to implement your Backend compared to a strategy using, for example, Java/JAX-RS/Tomcat
**Pros**
* Fast and easy setup, because it requires less code.
* Setting up middleware / new endpoints for REST calls is also pretty simple, and does not require as much as in Java.
* Express is a very lightweight, and can run on its own, whereas a javabackend has to be setup using e.g a tomcat server.
* If you want to go for a NoSQL database, it is also pretty simple to setup using Node.js / Express
* Fast at executing non-heavy CPU tasks with asynchronous code and the non blocking thread

**Cons**
* JS is single threaded even, which means Java is better at executing very CPU heavy and long tasks.
* It is easier to find errors in Java due to e.g the types, even though you could implement typescript in JS.
* Asynchronous code can to some extend be annoying to implement.

## Node.js uses a Single Threaded Non-blocking strategy to handle asynchronous task. Explain strategies to implement a Node.js based server architecture that still could take advantage of a multi-core Server.
There are 2 potential ways of doing this.
Either you implement a cluster module to help you doing this, or you run several instances of the node application, which can either be run on a multi-core system or across different machines.

**Cluster**

A single instance of Node.js runs in a single thread. To take advantage of multi-core systems, the user will sometimes want to launch a cluster of Node.js processes to handle the load. The cluster module allows easy creation of child processes that all share server ports.

**Sharding**

We can split the application into multiple instances where each instance is responsible for only a part of the application’s data. This strategy is often named **horizontal partitioning**, or **sharding**, in databases. Data partitioning requires a lookup step before each operation to determine which instance of the application to use. For example, maybe we want to partition our users based on their country or language. We need to do a lookup of that information first.

## Explain briefly how to deploy a Node/Express application including how to solve the following deployment problems:
To deploy the application, you would first have to download Node.js, and have a node project.
If you download the project from e.g GitHub, you have to remember to run npm install to get all of the node modules.
Now you can run the node project with npm start or node "name of node file", but if it crashes you have to restart everything manually.
* **Ensure that you Node-process restarts after a (potential) exception that closed the application**

    For this you can use a process manager like PM2, forever and StrongLoop process manager
* **Ensure that you Node-process restarts after a server (Ubuntu) restart**

    The process manager can also handle this part
* **Ensure that you can take advantage of a multi-core system**

    A process manager can also take care of this with the Cluster module (PM2)
* **Ensure that you can run “many” node-applications on a single droplet on the same port (80)**

    A reverse proxy like Nginx can be used for this.

## Explain the difference between “Debug outputs” and application logging. What’s wrong with console.log(..) statements in our backend-code.
Console.log() blocks the thread while writing to the console, whereas debugging statements do not and will only show in the console in a development environment. Logging is text written to files, to see what happens in the application over time.
This way it is way easier for the developers to track errors or possible security flaws.

## Demonstrate a system using application logging and “coloured” debug statements.
```javascript
const winston = require('winston')

//Remember: npm install winston 

//This is inspired by this article: http://tostring.it/2014/06/23/advanced-logging-with-nodejs/
//but slightly changed and updated to use the API of the newest version of Winston

var logger = winston.createLogger({
  transports: [
      new winston.transports.File({
          level: 'info',
          filename: './logs/all-logs.log',
          handleExceptions: true,
          json: true,
          maxsize: 5242880, //5MB
          maxFiles: 5,
          colorize: false
      }),
      new winston.transports.Console({
          level: 'debug',
          handleExceptions: true,
          json: false,
          colorize: true
      })
  ],
  exitOnError: false
})

logger.stream = {
  write: function(message, encoding){
      logger.info(message);
  }
};

module.exports=logger;
```
```javascript
var debug = require('debug')('loggingdemo:app');
const appLogger = require('./logger-setup')
...
app.use(require("morgan")("combined", { stream: appLogger.stream }));
appLogger.log('error', "127.0.0.1 - there's no place like home");
appLogger.log('warn', "127.0.0.1 - there's no place like home");
appLogger.log('info', "127.0.0.1 - there's no place like home");
appLogger.log('verbose', "127.0.0.1 - there's no place like home");
appLogger.log('debug', "127.0.0.1 - there's no place like home");

debug("Before middlewares")
```
These would both make an output in the terminal, but log would also log to a given file

## Explain, using relevant examples, concepts related to testing a REST-API using Node/JavaScript + relevant packages
To test a in Node/JS you can make use of Mocha and chai, and for rest purposes you make use of a http handler like fetch
```javascript
const expect = require('chai').expect;
const calc = require('../calc')
const fetch = require('node-fetch')
const PORT = 2345
const URL = `http://localhost:${PORT}/api/calc/add/`
let server
const filterDir = require('../module')
const fs = require('fs-extra')

describe("Calculator API", function () {

    describe("Testing the basic Calc API", function () {
        it("5 + 2 should return 7", function () {
            const result = calc.add(5, 2)
            expect(result).to.be.equal(7)
        })
    })

    describe("Testing the REST Calc API", function () {
        before(function (done) {
            calc.calcServer(PORT, function (s) {
                server = s
                done()
            })
        })
        after(function () {
            server.close()
        })
        it("Add 4 + 3 should return 7", async function () {
            const json = await fetch(URL + "4/3").then(r => r.json())
            expect(json.result).to.be.equal(7)
        })
    })

    describe("Testing makeitmodular", function () {
        before(function() {
            fs.mkdirSync("./1234test5678")
            fs.writeFileSync("./1234test5678/1.js")
            fs.writeFileSync("./1234test5678/2.js")
            fs.writeFileSync("./1234test5678/3.js")
        })

        it("Should return a list of length 4 and not throw an error", function (done) {
            filterDir("./1234test5678", "js", (err, data) => {
                if (err) {
                    done(err)
                }
                console.log(data.join("\n"));
                done()
            })
        })

        after(function() {
            fs.removeSync("./1234test5678")
        })
    })
})
```

## Explain, using relevant examples, the Express concept; middleware.
Middleware functions are functions that have access to the request object (req), the response object (res), and the next function in the application’s request-response cycle. The next function is a function in the Express router which, when invoked, executes the middleware succeeding the current middleware.

**Middleware functions can perform the following tasks:**
* Execute any code.
* Make changes to the request and the response objects.
* End the request-response cycle.
* Call the next middleware in the stack.

If the current middleware function does not end the request-response cycle, it must call next() to pass control to the next middleware function. Otherwise, the request will be left hanging.

**Example**
```Javascript
var express = require('express')
var app = express()

var myLogger = function (req, res, next) {
  console.log('LOGGED')
  next()
}

app.use(myLogger)

app.get('/', function (req, res) {
  res.send('Hello World!')
})

app.listen(3000)
```

## Explain, using relevant examples, how to implement sessions and the legal implications of doing this.
```Javascript
const cookieSession = require('cookie-session')
...
app.use(cookieSession({
  name: 'session',
  keys: [/* secret keys */],
  maxAge: 24 * 60 * 60 * 1000 // 24 hours
}))
```
Using session and cookies without user consent is very illegal due to the new GDPR enforced by the European Union. The law states that any information, which can be linked to a european citizen is personal data. If consent is not an option, the data has to be anonymous.

## Compare the express strategy toward (server side) templating with the one you used with Java on second semester.
Both JSP and EJS is using tags to embed either Java or JavaScript in HTML code.
Furthermore both have a "Frontcontroller" to control which modules should be used in the given situation.

## Demonstrate a simple Server Side Rendering example using a technology of your own choice (pug, EJS, ..).
```Javascript
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  var model = {
    title: "Site with a simple JOKE API",
    username: req.session.username,
    howToUse: "Get a random joke like this: /api/random",
    linkToAllJokes: "/api/all",
    linkToAddJoke: "/api/addJoke"
  }

  res.render('index', model);
});

module.exports = router;
```
```ejs
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
    <h1><%= title %></h1>
    <% if(typeof howToUse != 'undefined') { %>
    <p><%= howToUse %></p>
    <a href="<%= linkToAllJokes %>">All jokes</a><br>
    <a href="<%= linkToAddJoke %>">Add new joke</a>
    <% } else { %>
      <p><%= joke %></p>
      <a href="<%= link %>">Back to main</a>
   <% } %>
  </body>
</html>
```

## Explain, using relevant examples, your strategy for implementing a REST-API with Node/Express and show how you can "test" all the four CRUD operations programmatically using, for example, the Request package.
Look at the testing project

## Explain, using relevant examples, about testing JavaScript code, relevant packages (Mocha etc.) and how to test asynchronous code.
Testing project

## Explain, using relevant examples, different ways to mock out databases, HTTP-request etc.
You can mock using nock.
"Look at the testing project"

## Explain, preferably using an example, how you have deployed your node/Express applications, and which of the Express Production best practices you have followed.
Nginx has been used with logging enabled.
It is also possible to do with a process manager like PM2

# NoSQL, MongoDB and Mongoose
## Explain, generally, what is meant by a NoSQL database.
A NoSQL (originally referring to "non SQL" or "non relational") database provides a mechanism for storage and retrieval of data that is modeled in means other than the tabular relations used in relational databases. ... NoSQL databases are increasingly used in big data and real-time web applications.

## Explain Pros & Cons in using a NoSQL database like MongoDB as your data store, compared to a traditional Relational SQL Database like MySQL.
**Pros**
1. **Flexible Scalability**

Unlike rational database management model that is difficult to scale out when it come to commodity clusters NoSQL models make use of new nodes which makes them transparent for expansion. The model is designed to be used even with low cost hardwares. In this current world where outward scalability is replacing upwards scalability, NoSQL models are the better option.

2. **Stores Massive Amounts Of Data**

Given the fact that transaction rates are rising due to recognition, huge volumes of data need to be stored. While rational models have grown to meet this need it is illogical to use such models to store such large volumes of data. However these volumes can easily be handled by NoSQL models

3. **Database Maintenance**

The best rational models need the service of an expert to design, install and maintain. However, NoSQL models need much less expert management as it already has auto repair and data distribution capabilities, fewer administration and turning requirements as well as simplified data designs.

4. **Economical**

Rational models require expensive proprietary servers and storage systems whereas NoSQL models are easy and cheap to install. This means that more data can be processed and stored at a very minimal cost.

**Cons**
1. **Not Mature**

Rational models have been around for some time now compared to NoSQL models and as a result they have grown to be more functional and stable systems over the years.

2. **Less Support**

Every business should be reassured that in case a key function in their database system fails, they will have unlimited competent support any time. All rational model vendors have gone the extra mile to provide this assurance and made it sure that their support is available 24 hours which is not a step yet guaranteed by NoSQL vendors.

3. **Business Analytics And Intelligence**

NoSQL models were created because of the modern-day web 2.0 web applications in mind. And because of this, most NoSQL features are focused to meeting these demands ignoring the demands of apps made without these characteristics hence end up offering fewer analytic features for normal web apps.

Any businesses looking to implement NoSQL model needs to do it with caution, remembering the above mentioned pros and cons they posse in contrast to their rational opposites.

## Explain reasons to add a layer like Mongoose, on top on of a schema-less database like MongoDB
MongoDB by itself is schemaless, so we wanna add a layer like Mongoose, so we're able to handle the data in schemas and documents.

Mongoose is an object modeling tool for MongoDB and Node.js, somehow similar to a ORM tool as we know
Mongoose provides a straight-forward, schema-based solution to modeling your application data and includes, out of the box:
* Schemas
* Built-in type casting
* Validation (also included with plain MongoDB as of v. 3.2)
* Query building
* Business logic hooks (middleware)

## Explain about indexes in MongoDB, how to create them, and demonstrate how you have used them.
Period 3

## Explain, using your own code examples, how you have used some of MongoDB's "special" indexes like TTL and 2dsphere
Period 3

## Demonstrate, using a REST-API you have designed, how to perform all CRUD operations on a MongoDB
```Javascript
var mongoose = require('mongoose')

const mongoDB = "mongodb+srv://test:test@gettingstarted-zaepr.mongodb.net/mongodemo1?retryWrites=true"
mongoose.connect(mongoDB, { useNewUrlParser: true, useCreateIndex: true })
    .then((con) => console.log("Connected to Mongo"))
    .catch((err) => console.log("UPS DER OPSTOD EN FEJL!: " + err))

setTimeout(() => mongoose.disconnect(() => console.log("Disconnected")), 3000)

var userSchema = new mongoose.Schema({
    userName: String,
    email: { type: String, unique: true },
    created: { type: Date, default: Date.now }
});

var User = mongoose.model('User', userSchema);

async function addUser(userName, email) {
    var user = new User({ userName, email })
    await user.save()
}

async function findUser(fields, projection) {
    return await User.find({ userName: /Wonnegut/i }, { _id: 0, userName: 1, email: 1 })
        .sort({ username: 1 })
        .collation({ locale: "da" })
        .limit(2)
}

async function editUser() {
    var user = await User.findOneAndUpdate(
        { userName: "Kurt Wonnegut" },
        { email: "kurtIsHawt@wonnegut.dk" },
        { new: true }
    )
    console.log(user);
}

async function deleteUser() {
    await User.findOneAndDelete({userName: "Hanne Wonnegut"})
    var user = await User.findOne({userName: "Hanne Wonnegut"})
}
```

## Explain the benefits of using Mongoose, and demonstrate, using your own code, an example involving all CRUD operations
Already explained above

## Explain the “6 Rules of Thumb: Your Guide Through the Rainbow” as to how and when you would use normalization vs. denormalization.
* **One:** favor embedding unless there is a compelling reason not to.
* **Two:** needing to access an object on its own is a compelling reason not to embed it.
* **Three:** Arrays should not grow without bound. If there are more than a couple of hundred documents on the “many” side, don’t embed them; if there are more than a few thousand documents on the “many” side, don’t use an array of ObjectID references. High-cardinality arrays are a compelling reason not to embed.
* **Four:** Don’t be afraid of application-level joins: if you index correctly and use the projection specifier (as shown in part 2) then application-level joins are barely more expensive than server-side joins in a relational database.
* **Five:** Consider the write/read ratio when denormalizing. A field that will mostly be read and only seldom updated is a good candidate for denormalization: if you denormalize a field that is updated frequently then the extra work of finding and updating all the instances is likely to overwhelm the savings that you get from denormalizing.
* **Six:** As always with MongoDB, how you model your data depends – entirely – on your particular application’s data access patterns. You want to structure your data to match the ways that your application queries and updates it.

[Refrence link](https://www.mongodb.com/blog/post/6-rules-of-thumb-for-mongodb-schema-design-part-3)

## Demonstrate, using your own code-samples, decisions you have made regarding → normalization vs denormalization
https://techdifferences.com/difference-between-normalization-and-denormalization.html

```Javascript
var mongoose = require("mongoose");
var Schema = mongoose.Schema;

var JobSchema = new Schema({
    type: String,
    company: String,
    companyUrl: String
})

var UserSchema = new Schema({
    firstName: String,
    lastName: String,
    username: { type: String, unique: true, required: true },
    password: { type: String, required: true },
    email: { type: String, required: true },
    job: [JobSchema],
    created: { type: Date, default: Date.now },
    lastUpdated: Date
})
```

## Explain, using a relevant example, a full JavaScript backend including relevant test cases to test the REST-API (not on the production database)
Mini project
