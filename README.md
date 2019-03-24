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
