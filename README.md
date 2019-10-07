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

## Docker

### Docker concept

[Docker](Docker-concept.pdf)

- Creating isolated ambience(container)
  - Are ambiences that will not interfere on the behavior of other technologies and other tools of our server.
- If we install the PosgreSQL as default it will change a bunch of files on our system. To prevent this problems we use Docker.

**Image**
- A service or technologies that we can put inside the container as:
  - PostgreSQL, MySQL, MongoDB, Redis.

**Container**
- Intance of one Image
- We can have 3 images running at the same container.

**Docker Registry (DockerHub)**
- A place that we can put our own images

**Dockerfile**
- The recipe to build an image.
___
### Docker configuration

[Installation](https://docs.docker.com/v17.09/engine/installation/linux/docker-ce/ubuntu/)

- After installed to run create the Postgres image
  - `docker run --name <database-name> -e POSTGRES_PASSWORD=<database_password> -p 5432:5432 -d postgres`

> Flag `-p` will redirect the port, `machinePort:containerPort`
___

### Basic Command list

Stop container
```bash
docker stop <container_name>
```
___
Start container
```bash
docker start <container_name>
```
___
List container running
```bash
docker ps
```
___
List all containers at the machine
```bash
docker ps -a
```
___
### Postbird
Is a Graphic Interface to preview the database. [Installation](https://electronjs.org/apps/postbird)

- After connecting, create a database specific to the application.
___
## Sequelize
[Documentation](https://sequelize.org/)

Sequelize is a promise-based Node.js ORM for Postgres, MySQL, MariaDB, SQLite and Microsoft SQL Server.

Abstraction of database language, instead of using SQL we are going to use only JavaScript. Tables become Models

### Manipulating data

Using SQL

```sql
INSERT INTO users (name, email)
  VALUE (
    'Richard Yamamoto',
    'rybolteri@gmail.com'
  )
```

Now using Sequlize

```js
User.create({
  name: 'Richard Yamamoto',
  email: 'rybolteri@gmail.com'
})
```

Sequelize "translate" the SQL language to JavaScript.

>_User is our Model of user. Then we are going to import the User from `User.js`_

Another example is a query to search for an user

Using SQL

```sql
SELECT *
FROM users
WHERE email = 'rybolteri@gmail.com'
LIMIT 1
```

Using Sequelize will looks like

```js
User.findOne({
  where: {
    email: 'rybolteri@gmail.com'
  }
})
```

This syntax works fine with other Database

### Migrations

Is a version control for base of database for development and production environment.

Each migration file content instructions for creation, alterations or removal of table or columns.

Each migration happens by date. If we create a migration now, it can not depend of a unexistent migration.

Migration exemple

```js
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      name: {
        allowNull: false,
        type: Sequelize.STRING
      },
      email: {
        allowNull: false,
        unique: true,
        type: Sequelize.STRING
      }
    })
  },

down: (queryInterface, Sequelize) => {
  return queryInterface.dropTable('users')
}
```

Methods:

- `up` : When executing the migration.

- `down`: Intruction to rollback and delete the table, in case of something goes wrong.

**When the migration leaves the machine and goes to another developer or production environment, we can't edit it, we have to create another migration with the changes**

Each migration changes only one table.

### Seeds

Population of data for development and never going to be used in production environment. If we need the table with some default data, it must be add with migration.

### MVC Architecture

- Model: It keeps the abstractions of the database. Used to manipulate the data and don't have the responsibility by the rules of our application.

- Controller: It's the entry point of requisitions. Routes generally will be directly associated with a controller method.

- View: Is the return for the client.

>Controller going to be a class it will return a JSON and never call another controller.

When we create another Controller?

- When we have a new entity of the application

> Entity is not the same thing as Model. One Model can have more than one entity.
- Controller must have 5 methods or less, never more than 5.
  - `index` : Listing users
  - `show` : Exhibit one user
  - `store` : Save user data
  - `update` : Update user data
  - `delete` : Delete user
___

## Eslint and Prettier

- Installation:
  - `yarn add eslint -D`
  - `yarn add prettier eslint-config-prettier eslint-plugin-prettier -D`
- After installation run `yarn eslint --init`
- Paste the configurations [.eslintrc.js](.eslintrc.js)
- Create [.prettierrc](.prettierrc)
- Create [.editorconfig](.editorconfig)

>Automatic fix `yarn eslint --fix <folder_name> --ext <extension>`
___

## Sequelize configuration

Create the structure:

- `src`
  - `config`
    - `database.js`
  - `database`
    - `migrations`
  - `app`
    - `controllers`
    - `models`

Install dependency of sequelize
  - `yarn add sequlize`
  - `yarn add sequelize-cli -D`

Create [.sequelizerc](.sequelizerc)
  - This file will resolve the path to our Models, Migrations and Seeds.

Now on `src/config/`[database.js](src/config/database.js)
- Paste this configurations of database
- For PostgreSQL we need to add
  - `yarn add pg@^7.0.0 pg-hstore`
___

## User Migration

We will create our User migration using the `sequelize-cli` and it will turn things a lot easier.

First run `yarn sequelize migration:create --name=create-users`

Now we have to put the column of our migration (table)

For more details -> [User Migration](src/database/migrations/20191006222131-create-users.js).

- To execute the migration run
```bash
yarn sequelize db:migrate
```
- In case of mistakes, to rollback the last migration
```bash
yarn sequelize db:migrate:undo
```
- Or if you want to undo all the migrations
>Becareful because this command will delete the data from the tables.
```bash
yarn sequelize db:migrate:undo:all
```
- Inside Postbird we have the Sequelize Meta that is a registry of migrations. It contents all the migrations that was executed.
___

## User Model

Inside `src/app/models` create `User.js`.

- Inside `User.js` import
  - `import { Model } from 'sequelize';`.
- Then create `User` class inheriting from `Model`.
- Inside the class there is a `static init()` method that will initialize the `super init()` method.
- Inside this `super.init()`, will be placed an object with our table columns.
>_We can ignore the primary key, Foreign keys, created_at and updated_at_.
- We put only the columns that will be filled by the user.
- To put the type of the field we must import Sequelize
  - `import Sequelize, {...} from 'sequelize';`
- The first parameter of the `super.init()` is the object of the user table.
- The second parameter waits for and object with `sequelize` and could be filled with other configurations.
- For further details -> [User.js](src/app/models/User.js)
___

## Model Loader

We are going to create the file that will make the connection with Postgres database and load models for all the application.

- Create at `src/database` the `index.js`
- Import the Sequelize that will make this connection
  - `import Sequelize from 'sequelize';`
- Create `class Database {}`
- Inside `Database{}` we'll have the `constructor() {}` and `init() {}`
- The `constructor` will receive `this.init();`
- And `init` will receive a parameter `this.connection` that will be an intance of `Sequelize`
  - Import the database configuration
    - `import databaseConfig from '../config/database'`.
- The attribute `connection` will connect with our database.
```js
class Database {
  constructor() {
    this.init();
  }
  init(){
    this.connection = new Sequelize(databaseConfig)
  }
}
```
- Then import the User model
  - `import User from '../app/models/User'`
- Create a constant to receive an array of models.
  - `const models = [User,...]`
- Inside `init() {}` map the models array
  - Each Model has an static init method that waits for the connection
```js
models.map(model => model.init(this.connection));
```
- At **app.js** import the database
  - `import ./database`
    - It will recognize the index.js
- To test, at `routes.js`
  - `import User from './app/models/User'`
- Then create the user

```js
routes.get('/', async (req, res) => {
  const user = await User.create({
    name: 'Richard Yamamoto',
    email: 'rybolteri@gmail.com',
    password_hash: '123456',
  });
  return res.json(user);
})
```
>_By accessing the `https://localhost:3333/` the user must be seen with all the fields filled. And if you check in Postbird,the new user must be there too._
___

## User Registration

- First at `src/app/controllers` create the [UserController.js](src/app/controllers/UserController.js)
- It is basically a class.
- Import the User model

Basic form of a controller:
```js
import User from '../models/User'

class UserController() {
  async store (req, res) {

    return res.json({});
  }
}

export default new UserController();
```
- To create an user we will use the User model
- All models has the method `create()` and it waits for the requisition.
  - For us requisiton = JSON body.

- Inside the method `store(){}`
```js
async store (req, res){
  const user = await User.create(req.body);
  return res.json(user);
}
```
- In this case we have to verify if the user already exists.
- We can treat this flux with one `if`
```js
const { email } = req.body;
const userExists = await User.findOne({
  where: {
    email,
  }
})
if(userExists){
  return res.status(400).json({error: 'User already exists'});
}
```
>-   `.findOne()` search for only one result
>-  `where:{}` is analog to a filter
- The search will result an user that has the email equals of the requisition body email and return an error (400: Bad request)
- If nothing was found, the user is created.
___

## Generating Password Hash

To generate the hash we are going to use the Bcrypt

- Install
  - `yarn add bcryptjs`
- At `src/app/models/`[User.js](src/app/models/User.js)
- Import bcrypt
  - `import bcrypt from 'bcryptjs'`
- Below the closure of `super.init()`
- Create `this.addHook()`
  - `addHook()` is a functionality of Sequelize which is basically code snippets that are executed automatically based on actions that happens on the Model.
    - Exemple: `addHook('beforeSave', () => {})` - will execute the function (second parameter) before data been saved on database.
      - The function waits for the created model as parameter
- Ok, now the user will fill the password field and the hook will generate the hash.
- Bcrypt has a method `hash(<attribute>, <cryptography_strengh:integer>)`
```js
this.addHook('beforeSave', async (user) => {
  if(user.password){
    user.password_hash = await bcrypt.hash(user.password, 8);
  }
})
return this;
```
>DO NOT FORGET!!! -> **`return this`**
___