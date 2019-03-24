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
* **Ensure that you can take advantage of a multi-core system**
* **Ensure that you can run “many” node-applications on a single droplet on the same port (80)**
