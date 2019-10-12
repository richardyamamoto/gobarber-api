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

## Concept of JWT (JSON Web Token)

We are going to use JWT to generate session tokens and provide permission to user in session.

The Token is separated in 3 parts

1. <span style="color: orange">Header</span> - Contents which kind of token we generate.
2. <span style="color: red">Payload</span> - It carries some extra informations that could be useful for backend or frontend (non-sensitive data) _Exemple: id_
3. <span style="color: cyan">Signature</span> - The prove that token was not modified.

Token exemple:
<span style="color: orange">eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9</span >.<span style="color: red">eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ</span>.<span style="color: cyan">SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c</span>
___

## JWT Authentication

- Install JSON Web Token
  - `yarn add jsonwebtoken`
- Then create `src/app/controllers/`[SessionController.js](src/app/controllers/SessionController.js)
- `import jwt from 'jsonwebtoken'`
- `import User from '../models/User'`
- Create the class SessionControler with the method asyncronous store
- Then verify if the user email exists
- Check if password matchs
  - We are going to create a new method inside our user Model, because this verification is basically an user rule.
- On `src/app/models/`[User.js](src/app/models/User.js)
- Create the method `checkPassword(password){}`
```js
checkPassword(password){
  return bcrypt.compare(password, this.password_hash);
}
```
- Now back to [SessionController.js](src/app/controllers/SessionController.js)
- Verify if the password from request body matches with password hash
```js
if(! (await user.checkPassoword(password))){
  return res.status(401).json({error: 'Passoword does not match'})
}
```
>_If the password does not match with password hash will return this error_

- Create the session
```js
const {id, name, email} = user;
return res.json({
  user:{
    id,
    name,
    email,
  },
  token: jwt.sign({id}, '25de0fdcb8a263d895d0fc5dac2c1e25', {
    expiresIn: '7d',
  })
})
```
- Organize better, create at `src/config/`[auth.js](src/config/auth.js)
- Then import it
  - `import authConfig from '../../config/auth'`
- And place the authConfig.secret and authConfig.expiresIn in the correct place.
___

## Authentication Middleware

We only will allow the user to update data if he is on a session

- First at `src/app/controllers/`[UserController.js](src/app/controllers/UserController.js)
- Create the method `async update() {}`
- Then on `src/`[routes.js](src/routes.js)
  - `routes.put('/users', UserController.update)`

We will have to intercept the route to verify if the user are in session, to make this we use `middlewares`
- At `src/app/`[middlewares](src/app/middlewares)`/`[auth.js](src/app/middlewares/auth.js)

>_The token that was provided by our session, is going to be passed to our auth middleware by the header of the requisition_
- Now export default the middleware
- Create a constant to receive the request header authorization.
  - Where our Bearer token was placed.
- Then using unstructuring we can extract only the token inside the array
```js
const authHeader = req.headers.authorization;
const [, token] = authHeader.split(' ');
```
- Now import the JWT and the authConfig
  - `import jwt from 'jsonwebtoken'`
  - `import authConfig from '../../config/auth'` - where is our secret
- We will use the `try catch` to handle the token verification.
- To verify, JWT has the method `verify()`, but it has two versions, the syncronous and asyncronous, although the asyncronous version uses the callback function style.
- We are going to solve this problem by using `promisify` from `util`, it transforms callback functions into async await functions.
  - `import { promosify } from 'util'`
- After decoded the token, the payload carries the user id, so we are going to mutate the requesition and input the userId.
```js
import jwt from 'jsonwebtoken';
import { promisify } from 'util';
import authConfig from '../../config/auth';

export default async (req, res, next) => {
  const authHeader = req.headers.authorization;
  if (!authHeader) {
    return res.status(401).json({error: 'Token not provided'});
  }

  const [, token] = authHeader.split(' ');

  try {
    const decoded = await promisify(jwt.verify)(token, authConfig.secret);
    req.userId = decoded.id;

    return next();
  }catch(error) {
    return res.status(401).json({error:'Invalid token'})
  }
}
```
>_`promisify()`: transforms functions into async/await functions. The return is another function which waits for the parameters of the original function._

By extracting the user id from the token, we can make alterations without pass the ID by query parameters.
___

## User Update

- Inside `src/app/controllers/`[UserController.js](src/app/controllers/UserController.js)
- At `async update() {}`
- Create a const to receive the user id from request
- Than declare another const to receive the user found by primary key (PK)
- We need to certify if the user change de email, this email must be unique.
- And check the password too.
- The file should looks like:
```js
async update(req, res) {
  const { userId } = req;
  const { email, oldPassword } = req.body;
  const user = await User.findByPk(userId);

  if (email !== user.email) {
    const userExists = await User.findOne({ where: { email } });

    if (userExists) {
      return res.status(401).json({ error: 'Email already taken' });
    }
  }

  if (oldPassword && !(await user.checkPassword(oldPassword))) {
    return res.status(401).json({ error: 'Password does not match' });
  }

  const { id, name, provider } = await user.update(req.body);
  return res.json({
    id,
    name,
    email,
    provider,
  });
}
```
___

## Validating Input Data

To validate input data, we will use the Yup that works with schema validation
- `yarn add yup`
- At `src/app/controller/`[UserController.js](src/app/controller/UserController.js)
- Import Yup
  - `import * as Yup from 'yup'`
- Declare a constant to receive the schema
```js
const schema = Yup.object().shape({
  name: Yup.string().required(),
  email: Yup.string().email()required(),
  password: Yup.string().min(6).required(),
})
```
- After that we are going to validade the Update
- At `async update() {}`
- We are going put some rules for password
```js
const schema = Yup.object().shape({
  name: Yup.string(),
  email: Yup.string().email(),
  oldPassword: Yup.string().min(6),
  password: Yup.string().when('oldPassword', (oldPassword, field) =>
    oldPassword ? field.required() : field
  ),
  confirmPassword: Yup.string()
    .min(6)
    .when('password', (password, field) =>
      password ? field.required().oneOf([Yup.ref('password')]) : field
    ),
});
```
Explanation of `.when()`
```js
.when(String:field_name, Function(field_name, field) => Options:boolean ? true : false)
```
___

## File Upload

We are going to use Multer to upload files in `multipart/form-data`

- Create
- `src`
  - `temp`
    - `uploads`

- Then

- `src`
  - `config`
    - [multer.js](src/config/multer.js)

- Now inside `multer.js`
- Import
  - `import multer from 'multer'`
  - `import crypto from 'crypto'`
  - `import { extname, resolve } from 'path'`

- Export default the object configuration
- Create attribute `store` that receives `multer.diskStorage({})`
- The first attribute of `diskStorage()` is the `destination` and second `filename`
```js
destination: resolve(__dirname, '..', '..', 'temp', 'uploads')
```
```js
filename: (req, file, cb) => {
  crypto.randomBytes(16, (err, res) => {
    if (err) return cb(err);

    return cb(null, res.toString('hex') + extname(file.originalname));
  });
},
```
- At `routes.js`
- Import
  - `import multer from 'multer'`
  - `import multerConfig from './config/multer'`
- Create
`const upload = multer(multerConfig);`
  - The middleware
- Create a route to upload file
  - Just for test
```js
routes.post('/file', upload.single('file'), (req, res) => {
  return res.json({message: 'ok'})
})
```
- Look at the temp folder the uploaded file.
___

## User Avatar

When Multer act on the route, it release an attribute -> `req.file`
> It happens on single file, if exists more than one it would be `req.files`

- Create `src/app/controllers/`[FileController.js](src/app/controllers/FileControllers.js)
- Then on `routes.js` import the FileController.js
  - `import FileController from '../app/controllers/FileController'`
- We must create a new table for files
  - `yarn sequelize migration:create --name=create-files`
- We need: `id`, `name`, `path`, `created_at`, `updated_at`
- Run the database table `yarn sequelize db:migrate`
- Now create the model of [File.js](src/app/models/File.js)
- On `src/database/index.js`, import the File model and put it on the array.

- On `FileController.js` import the File model
- Then unstructure the `req.file` to recover some data.
  - We need `originalname` and `filename`.
> To rename some attributes we can use `const { Attribute: NewName }`

```js
async store(req, res) {
  const { originalname: name, filename: path } = req.file;

  const file = await File.create({
    name,
    path,
  })

  return res.json(file)
}
```
The way we run our migrations, the file table does not have a connection with each other. Now to solve this problem create a new migration to add the table column.

- `yarn sequelize migration:create --name=add-avatar-field-to-users`
- This migrations is a little bit different.
- Instead of:
```js
return queryInterface.createTable({
  ...
})
```
- We will replace the default return for:
```js
return queryInterface.addColumn()
```
- The `addColumn()`:

- 1. First parameter is which table to add the column: String,
- 2. The name of the new columm: String,
- 3. Information: Object,
    - 3.1. Type,
    - 3.2. References (foreign key): Object
      - First attribute the table reference `model: 'table_name'`
      - Second attribute the reference `key: column_name`
    - 3.3. onUpdate
    - 3.4. onDelete
>All the `avatar_id` on users table will be an `id` on files table
- `yarn sequelize db:migrate`
- This way the avatar id will not appear at the user table.
- We have to create a relation with User model and File model.
- At `src/app/models/User.js`
- Create a static method `associate()`
```js
static associate(models){
  this.belongsTo(models.File, {
    foreignKey: 'avatar_id',
  })
}
```
- Then on `src/database/index.js`
- Put another `map()` with:
```js
.map(model => model.associate && model.associate(this.connection.models))
```
- Now put on the `req.body` JSON the `"avatar_id": 1`, it will appear at the user table.
___

## Provider Listing

- On `routes.js` create a new router and a new controller.
```js
routes.get('/providers', ProviderController.js)
```
- `src/app/controllers/`[ProviderController.js](src/app/controllers/ProvierController.js)
- At the controller create the method `async index()` to list the providers
- Import the User model and File model
  - `import User from '../models/User'`
  - `import File from '../models/File'`
- Use the findAll and where to find only providers
- The property `attributes` return only selected attributes
- Property `includes` is an array that can receive information of another models
- Each object can have a model and nickname
  - To change the nickname from model, go to [User.js](src/app/models/User.js).
  - On method `associate()` after the foreignKey, put `as: 'avatar'` that is the nickname.
  - Then at the controller, put `as: 'avatar'` below `model: File`.
```js
async index(req, res) {
  const provier = await User.findAll({
    where: {provider: true},
    attributes: ['id', 'name', 'email', 'avatar_id'],
    includes:[
      {
        model: File,
        as: 'avatar',
        attributes: ['name','path']
      },
    ]
  });
  return res.json(provider);
}
```
- Until now we don't have an Url to show the picture, it's a good practice to include it.
- On [File.js](src/app/models/File.js).
- Insert another field
  - Type virtual
  - Method `get() {}`
- The method `get()` allow to insert some functionalities
```js
get() {
  return `http://localhost:3333/files${this.path}`
}
```
- To show static media, Express has the method `static`
- At [app.js](src/app.js)
- Import `resolve` from path
- On `middlewares() {}` put
```js
this.server.use('files', express.static(resolve(__dirname, '..', 'temp', 'uploads')))
```
___

## Migration and Model of Appointments

Create a new migration for Appointments
- `yarn sequelize migration:create --name=create-appointments`
- Check the changes [create-appointments.js](src/database/migrations/20191009195657-create-appointments.js)
- Then `yarn db:migrate`
- Create an Appointment model -> [Appointment.js](src/app/models/Appointment.js)
- The Appointment model must have the association with User model
- To practice this again:
```js
static associate(models) {
  this.belongsTo(models.User, { foreignKey: 'user_id', as: 'user' });
  this.belongsTo(models.User, { foreignKey: 'provider_id', as: 'provider' });
}
```
___

## Service Scheduling

- Create the AppointmentController.js at `src/app/controllers`
- Create the method `async post() {}`
- We need to certify that the user is not a provider
- First import User model
  `import User from '../models/User'`
- Use Yup to validate data by declaring the schema
```js
const schema = Yup.object().shape({
      provider_id: Yup.number().required(),
      date: Yup.date().required(),
    });
```
- Use unstructure to extract the provider id and data from requisition body
```js
const { provider_id, date } = req.body;
```
- Then create a constant to receive the result of the search
```js
const isProvider = await User.findOne({where: {
  id: provider_id,
  provider: true,
}})
```
- Then if it is false, return an error
```js
if (isProvider) {
  return res.status(401).json({error: 'Can only create appointments with providers'})
}
```
- Then create the appointment
```js
const appointment = await Appointment.create({
  user_id: req.userId,
  provider_id,
  date: date,
})
```

## Validating Scheduling

We are going to use the `Date fns` to validate date.

[Documentation](https://date-fns.org/docs/Getting-Started)

Install running `yarn add date-fns@next`

- The first validation is if the date that user wants to create the appointment has already not passed.
- The second validation is if the provider has the date available to create appointments.
- Will be allowed only one appointment per hour.

**1st Verification - If date is not passed from current date.**

- Import some functions of date-fns
  - `import { startOfHour, parseISO } from 'date-fns'`
    - `startOfHour()` - Will get the hour and replace minutes and seconds to 00
    - `parseISO()` - Transform `YYYY-MM-DDTHH:MM:SS-GMT` into JavaScript date
- After check if the user is provider
- Create a constant to receive the hour of the appointment
```js
const hourStart = startOfHour(parseISO(date));
```
- Import the function `isBefore` from date-fns
  - `import { startOfHour, parseISO, isBefore} from 'date-fns'`
- Now compare the current date to the `hourStart`
- If the `new Date()`(current date) if before the `hourStart` from the requesition body, it means that's not possible to create an appointment.
```js
if(isBefore(hourStart, new Date())) {
  return res.status(400).json({error: 'Past dates are not allowed'})
}
```
**2nd Verification - Date availablility**

- Create a constant `checkAvailability`
```js
const checkAvailability = await Appointment.findOne({
  where: {
    provider_id,
    canceled_at: null,
    date: hourStart,
  }
})

if(checkAvailability) {
  return res.status(401).json({ error: 'Appointment date are not available' })
}
```
- If it returns true, it means that the date is not available.
___

## Listing User Appointments

- Create `async index(req, res) {}`
- A constant will receive the appointments
- We need to include the provider attributes from model User
- And include the file attributes from the File model
```js
const appointments = await Appointments.findAll({ where: {
  user_id: req.userId,
  canceled_at: null,
  order: ['date'],
  attributes: ['id', 'name'],
  include: [
    {
      model: User,
      as: 'provider'
      attributes: ['id', 'name'],
      include: [
        {
          model: File,
          as: 'avatar'
          attributes: ['id', 'path', 'url']
        }
      ]
    }
  ]
}})
```
- Do not forget to put the index method on routes

`routes.get('/appointments', AppointmentController.index)`
___

## Applying Pagination

We have the parameter query that is passed on the URL

Exemple: `http://localhost:3333/appointments?page=1`

This query can be recovered from `req.query`

- Create a constant with unstructuring to receive the query parameter.
```js
const { page } = req.query;
```
- There is an attribute called `limit` that limits the number of objects to be shown.
- There is another attribute called `offset` that jumps the number of objects.
- Put this attributes after `attributes: []`.
```js
{
  attributes: [],
  limit: 20,
  offset: (page - 1) * 20,
}
```
___

## Listing Provider Schedule

The provider must have a unique list. So let's create a controller for it

- Create `ScheduleCotroller.js` inside `src/app/controllers`
- Create the route inside `routes.js`
  - Import the `ScheduleController.js`
```js
router.get('/schedule', ScheduleController.index);
```
- Now on `ScheduleController.js` we will edit the index method.
- Create a constant to receive the the verification of the user
  - User must be a provider
- Import the User model.
  - `import User from '../models/User'`
```js
const isProvider = await User.findOne({ where: {
  id: req.userId,
  provider: true,
}})

if (!isProvider) {
  return res.status(401).json({ error: 'User is not provider' })
}
```
- After this verification we will pass the date of the search by query parameters
  - Exemple: `http://localhost:3333/schedule?date=2019-10-12T00:00:00-03:00`
- To recover this value using unstructuring
  - `const { date } = req.query;`
- Now that we have the date we want to search, we must import some functions from `date-fns` and `sequelize`.
  - `import { startOfDay, endOfDay, parseISO } from 'date-fns'`
  - `import { Op } from 'sequelize'`
- First of all we need to format the date
- Then lets search into Appointment model:
```js
const parsedDate = parseISO(date)

const appointments = await Appointments.findAll({ where: {
  provider_id: req.userId,
  canceled_at: null,
  date: {
    [Op.between]: [
      startOfDay(parsedDate),
      endOfDay(parsedDate)
    ],
  },
  order: ['date'],
  }
});

return res.json(appointments)
```
>The `Op.between` is passed through brackets because is the name of the key, and receive statements for comparison inside an array.
___

## Setting MongoDB

MongoDB is a non relational database, we are going to use them, because there are some data that does not have relation. This kind of database are really performatic.

- First of all, we need to run a new docker container
  - `docker run --name mongobarber -p 27017:27017 -d -t mongo`
    - [Docker-MongoDB](https://hub.docker.com/_/mongo)
  - At the browser put `http://localhost:27017`, it will show a message if the Mongo container is running.
- To connect our application with MongoDB, we are going to use [Mongoose](https://mongoosejs.com/docs/index.html)
- Install the Mongoose
  - `yarn add mongoose`
- Inside [database/index.js](src/database/index.js)
- We'll import mongoose
  - `import mongoose from 'mongoose'`
- Create a new method `mongo() {}` and call it on `constructor()`
```js
mongo() {
  this.mongoConnection = mongoose.connect(
    'mongodb://localhost:27017/gobarber',
    { useNewUrlParser: true, useFindAndModify: true, useUnifiedTopology: true }
  )
}
```
>The method `connect()` waits as 1st parameter the url from database, in our case as we do not have user and password, it will directly the host then port then the name of the database. As 2nd parameter(optional), it waits for an configuration object, we will use the "new url format", "find and modify" and  "unified topology" as true.
___

## Notificating New Appointment

Mongo use Schema instead of Models, so we'll have a folder for this. Schema free is the methodology used, it means that one registry can have the field title while another registry does not have.

Use [MongoDB Compass](https://www.mongodb.com/download-center/compass) as Interface.

- First at `src/app` create the folder `schemas`
- Then create the [Notification.js](src/app/schemas/Notification.js)
- Import the mongoose
  - `import mongoose from 'mongoose'`
- Create a constant to be an intance of mongoose schema
```js
const NotificationSchema = new mongoose.Schema({
    content: {
      type: String,
      required: true,
    },
    user: {
      type: Number,
      required: true,
    },
    read: {
      type: Boolean,
      required: true,
      default: false,
    }
  },
  {
    timestamps: true
  }
);

export default mongoose.model('Notification', NotificationSchema)
```
Now on [AppointmentController.js](src/app/controllers/AppointmentController.js), after create the appointment, we will send the notification to provider.

- We will use the data from User model
- Then use the method `format` from `date-fns` and the locale from `date-fns/locale/pt`
```js
import { format } from 'date-fns';
import pt from 'date-fns/locale/pt'
...
const { name } = await User.findByPk(req.userId);
const formattedDate = format(
  hourStart,
  "'dia 'dd' de 'MMMM', ás 'HH:mm'h'",
  {locale: pt}
)
await Notification.create({
  content: `Novo agendamento de ${name} para ${formattedDate}`,
  user: provider_id,
})
```

Now when the notification is created, we send to Mongo the data of the appointment for the provider.
___

## Listing User Notifications

- We will start creating a new route and a new Controller
- Create at `src/app/controllers/`[NotificationController.js](src/app/controllers/NotificationController.js)
- Import the Notifications from schema
  - `import Notifications from '../schemas/Notifications'`
- We need to verify is the user is a provider
  - `import User from '../models/User'`
- Then after user verification, we need to list the notifications
  - Schemas works a little bit different than Sequelize Models, the method to find all is `find()` and it waits as parameter directly the filter object.
  - And to put another filters, we use chaining
    - Concatenate the functions
```js
async update(req, res) {
  const isProvider = await User.findOne({
    where: {
      id: req.userId,
      provider: true,
  }});

  if(!isProvider) {
    return res.status(401).json({ error: 'Only allowed to providers' })
  }

  const notification = await Notification.find(
    user: user.reqId,
  )
  .sort({
    createdAt: 'desc',
  })
  .limit(20);
}

return res.json(notification);
```
- At `routes.js`
- Create
`routes.get('/notifications', NotificationController.index)`
___

