# API GoBarber

This API will be consumed by our application in ReactJS web and React Native. This documentation was builded to be a consult material for further developments.
___

## Get Started

This API will use the Express framework.

[Express - Documentation](https://expressjs.com/pt-br/)

- Install the express as dependency
  - `yarn add express`
- After that we create a folder `src` that will allocate all our application files.
  - Inside `src` create:
    - `app.js` - where we configurate our express server.
    - `server.js`
    - `routes.js` - place that all our routes stay.

Inside **src/app.js**

- Create a const to receive the Express
- Create `class App{}` and inside the `constructor(){}` an attribute called server.
- Create `middleware(){}`and `route(){}` methods.

```js
const express = require('express');
const routes = require('./routes');
class App {
  constructor(){
    this.server = express();

    this.middlewares();
    this.routes();
  }

  middlewares(){
    this.server.use(express.json());
  }

  routes(){
    this.server.use(routes);
  }
}

module.exports = new App().server;
```
- _`express.json()` allow our API to receive requests in JSON format._
- _routes were imported from our **routes.js** file to be used by our server_
- _we will export an instance of App with the only attribute that can be accessed_

Now inside **server.js**

- Import the App that was instanced on `app.js` file
  - `const app = require('./app');`
- And use the method `listen()` to put the port to be listened
```js
const app = require('./app');

app.liste(3333);
```
_We separate the application structure from the part that we create the server, but why?_ 

_**Because by doing that when we start the unitary test,functional tests and integration tests, the server won't need to be initialized on a port, the tests will run directly on the App class**_

At this time inside **routes.js**

- We will import only the Router from Express
  - `const { Router } = require('express');`
- Then we instance the router
  - `const routes = new Router();`
- To test we will create a route and return the response a simple message

```js
routes.get('/', (req, res) => {
  return res.json({ message: 'Hello World' })
});
```
- Now on the Browser put _https://localhost:3333_
___

## Nodemon and Sucrase

**Nodemon** is a utility that will monitor for any changes in your source and automatically restart your server. Perfect for development. Install it using

**Sucrase** is an alternative to Babel, it allow us to use the new syntax of JavaScript as: `import` and `export`.

- Install Nodemon and Sucrase by running
  - `yarn add nodemon -D`
  - `yarn add sucrase -D`

> Flag `-D` for development dependency.

Now make the changes to `import` and `export`. Check [app.js](app.js), [routes.js](routes.js), [server.js](server.js) to understand the changes. 

- Now if you try to run `node src/server.js` it will not work because the new syntax is not supported.
- Now you have to run `sucrase-node src/server.js`.
- We installed de Nodemon to reset automatically so we have to configure it first.
  - [nodemon.js](nodemon.json)
- And create a new script to be easier to start the nodemon 
  - [package.json](package.json)