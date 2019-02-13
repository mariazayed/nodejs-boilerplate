# NodeJS-ExpressJS-MongoDB
Building RESTful Web APIs with NodeJS, ExpressJS, MongoDB and TypeScript

## Pre-requirements: 
* [NodeJS](https://nodejs.org/en/)
* install TypeScript and TypeScript Node using this command:
    ```sh
    npm install -g typescript ts-node
    ```
* [Postman](https://www.getpostman.com/downloads/) to test the HTTP requests
* [MongoDB](https://docs.mongodb.com/manual/administration/install-community/) then you have to install [Robo Mongo](https://robomongo.org/) or [Mongo Compass](https://docs.mongodb.com/compass/master/install/) for GUI interface

## Project Contents
### Part 1: Setting up the project
- Initiate a node project
- Install all the dependencies
- Configure the TypeScript configuration file
- Edit the running scripts in package.json
- Getting started with the base configuration
### Part 2: Implement routing and CRUD
- Create .ts file for routing
- Building CRUD for the Web APIs
### Part 3: Using Controller and Model for Web APIs
- Create a Model for your data
- Create your first Controller
### Part 4: Connect Web APIs to MongoDB 
- Create your first contact
- Get all contacts
- Get contact by Id
- Update an existing contact
- Delete a contact

# Part 1: Setting up the project
## Step1: Initiate a node project
Create a project folder and initiate the npm project. 
```sh
mkdir node-express-mongodb
cd node-express-mongodb
npm init 
```
## Step 2: Install all the dependencies
```sh
npm install --save @types/express express body-parser mongoose nodemon
```
## Step 3: Configure the TypeScript configuration file
Create a _“tsconfig.json”_ file in the root directory and copy this code
```javascript
// tsconfig.json
{
    "compilerOptions": {
        "module": "commonjs",
        "moduleResolution": "node",
        "pretty": true,
        "sourceMap": true,
        "target": "es6",
        "outDir": "./dist",
        "baseUrl": "./lib"
    },
    "include": [
        "lib/**/*.ts"
    ],
    "exclude": [
        "node_modules"
    ]
}
```


Create a _“lib folder”_ in the root directory<br>
**For development, we will put all the typescript files in the _“lib folder”_**<br>
**For production, we will save all the javascript files in the _“dist folder”_**<br>
So whenever we run the _“tsc”_ command, all the ts files in the lib folder will be compiled to js files in the dist folder
## Step 4: edit the running scripts in package.json
```javascript
// package.json
"scripts": {
    "build": "tsc",
    "dev": "ts-node ./lib/server.ts",        
    "start": "nodemon ./dist/server.js",
    "prod": "npm run build && npm run start"
}
```

So, for the development, we can run a test server by running
```
npm run dev
For production
npm run prod
```

## Step 5: getting started with the base configuration
Create _“app.ts”_ and _“server.ts”_ files in the _“lib”_ directory and copy those codes

```javascript
// lib/app.ts
import * as express from "express";
import * as bodyParser from "body-parser"; // for parsing incoming request data.

class App {

    public app: express.Application;

    constructor() {
        this.app = express();
        this.config();        
    }

    private config(): void{
        // support application/json type post data
        this.app.use(bodyParser.json());
        //support application/x-www-form-urlencoded post data
        this.app.use(bodyParser.urlencoded({ extended: false }));
    }

}

export default new App().app;
```

```javascript
// lib/server.ts
import app from "./app";
const PORT = 3000;

app.listen(PORT, () => {
    console.log('Express server listening on port ' + PORT);
})
```

Run ```“npm run dev”``` to test that everything is good
![capture](https://user-images.githubusercontent.com/27064594/52698253-cb4fd380-2f7b-11e9-9e32-eed66b48d954.PNG)

# Part 2: Implement routing and CRUD
## Step 6: Create .ts file for routing
Create _“/lib/routes/routes.ts”_ file and copy this code
```javascript
// /lib/routes/routes.ts
import {Request, Response} from "express";

export class Routes {       
    public routes(app): void {          
        app.route('/')
        .get((req: Request, res: Response) => {            
            res.status(200).send({
                message: 'GET request successfulll!!!!'
            })
        })               
    }
}
```

Then you need to import the _“routes”_ to the _“lib/app.ts”_
```javascript
// /lib/app.ts
...
import { Routes } from "./routes/routes";

class App {
    ....
    public routes: Routes = new Routes();
    
    constructor() {
    ...    
        this.routes.routes(this.app);     
    }
    ...
}
```

Now, you can send GET request to your application directly using this link ( http://localhost:3000 ) or by using Postman.

## Step 7: Building CRUD for the Web APIs
CRUD stands for Create, Read, Update, and Delete
>GET: for retrieving data<br>
>POST: for creating new data<br>
>PUT: for updating data<br>
>DELETE: for deleting data<br>

Copy the following code and update the _“/lib/routes/routes.ts”_ file

```javascript
import {Request, Response} from "express";

export class Routes {

    public routes(app): void {

        app.route('/').get((req: Request, res: Response) => {
            res.status(200).send({
                message: 'GET request successful !'
            })
        });

        app.route('/contact').get((req: Request, res: Response) => {
            res.status(200).send({
                message: 'Get all contacts !'
            })
        });

        app.route('/contact').post((req: Request, res: Response) => {
            res.status(200).send({
                message: 'Create new contact'
            })
        });

        app.route('/contact/:contactId').get((req: Request, res: Response) => {
            res.status(200).send({
                message: 'Get a specified contact details'
            })
        });

        app.route('/contact/:contactId').put((req: Request, res: Response) => {
            res.status(200).send({
                message: 'Update a specified contact details'
            })
        });

        app.route('/contact/:contactId').delete((req: Request, res: Response) => {
            res.status(200).send({
                message: 'Delete a specified contact'
            })
        });
    }
}
```

# Part3: Using Controller and Model for Web APIs
## Step 8: Create a Model for your data
Create a _“lib/models”_ folder for all the models, create a _“model.ts”_ file and copy the following code

```javascript
//   /lib/models/model.ts
import * as mongoose from 'mongoose';

const Schema = mongoose.Schema;

export const ContactSchema = new Schema({
    firstName: {
        type: String,
        required: 'Enter your first name'
    },
    lastName: {
        type: String,
        required: 'Enter your last name'
    },
    email: {
        type: String
    },
    company: {
        type: String
    },
    phone: {
        type: Number
    },
    created_date: {
        type: Date,
        default: Date.now
    }
});
```

## Step 9: Create your first Controller
Now we will apply the real logic to the route and controller. First of all create _“/lib/controllers/controller.ts”_<br>
### Create a new contact (POST request)
```javascript
//   /lib/controllers/controller.ts
import * as mongoose from 'mongoose';
import { ContactSchema } from '../models/model';
import { Request, Response } from 'express';

const contact = mongoose.model('Contact', ContactSchema);
export class ContactController {

    public addNewContact(req: Request, res: Response) {
        let newContact = new contact(req.body);

        newContact.save((err, contact) => {
            if (err) {
                res.send(err);
            }
            res.json(contact);
        });
    }
}
```


```javascript
// /lib/routes/routes.ts
import { ContactController } from "../controllers/controller";
public contactController: ContactController = new ContactController();

// Create a new contact
app.route('/contact')
    .post(this.contactController.addNewContact);
```

### Get all contacts (GET request)
```javascript
    //   /lib/controllers/controller.ts
    public getContacts (req: Request, res: Response) {
        contact.find({}, (err, contact) => {
            if(err){
                res.send(err);
            }
            res.json(contact);
        });
    }
```
```javascript
// /lib/routes/routes.ts
app.route('/contact').get(this.contactController.getContacts);
```

### View a single contact (GET method)
We need the ID of the contact in order to view the contact info.
```javascript
    //   /lib/controllers/controller.ts
    public getContactWithID (req: Request, res: Response) {
        contact.findById(req.params.contactId, (err, contact) => {
            if(err){
                res.send(err);
            }
            res.json(contact);
        });
    }
```
In the routes, we simply pass the _‘/contact/:contactId’_
```javascript
// /lib/routes/routes.ts
app.route('/contact/:contactId').get(this.contactController.getContactWithID);
```

### Update a single contact (PUT method)
```javascript
    //   /lib/controllers/controller.ts
    public updateContact (req: Request, res: Response) {
        contact.findOneAndUpdate({ _id: req.params.contactId }, req.body, { new: true }, (err, contact) => {
            if(err){
                res.send(err);
            }
            res.json(contact);
        });
    }
```
**Note** that, without _{new: true}_, the updated document will not be returned.
```javascript
// /lib/routes/routes.ts
app.route('/contact/:contactId').put(this.contactController.updateContact);
```

### Delete a single contact (DELETE method)
```javascript
    //   /lib/controllers/controller.ts
    public deleteContact (req: Request, res: Response) {
        contact.remove({ _id: req.params.contactId }, (err) => {
            if(err){
                res.send(err);
            }
            res.json({ message: 'Successfully deleted contact!'});
        });
    }
```
```javascript
// /lib/routes/routes.ts
app.route('/contact/:contactId').delete(this.contactController.deleteContact);
```


**NOTE:** for simplicity, you don’t have to call _app.route(‘/contact/:contactId’)_ every single time for GET, PUT or DELETE a single contact. You can combine them:

```javascript
        // /lib/routes/routes.ts
        app.route('/contact/:contactId')
            .get(this.contactController.getContactWithID)
            .put(this.contactController.updateContact)
            .delete(this.contactController.deleteContact)
```
From now, your model and controller are ready




# Part 4: Connect Web APIs to MongoDB 
All that you need to do is to import mongoose package and declare URL for your MongoDB in the _“app.ts”_ file. After that, you will connect your app with your database through mongoose.

```javascript
// lib/app.ts
...
import * as mongoose from "mongoose";

class App {

    ...
    public mongoUrl: string = 'mongodb://localhost/contactsDB';  

    constructor() {
        ...
        this.mongoSetup();
    }

    private mongoSetup(): void{
        mongoose.Promise = global.Promise;
        mongoose.connect(this.mongoUrl);    
    }

}

export default new App().app;
```


After this, your application is ready to launch ```npm run dev```
Now, we will test the CRUD feature through the postman 

## Step 10: Create your first contact
I will send a POST request to ( http://127.0.0.1:3000/contact ) with the information of a contact in the body.<br>

Remember to set the content-type in Headers<br>
```Content-Type: application/x-www-form-urlencoded```

After sending, the server return the status 200 with contact information in the database.
![post](https://user-images.githubusercontent.com/27064594/52714072-abcba180-2fa1-11e9-8066-6df025828a9e.PNG)

## Step 11:  Get all contacts
To get all contacts, we just need to send a GET request to ( http://127.0.0.1:3000/contact ) You will get an Array of all the contacts in the database.
![get](https://user-images.githubusercontent.com/27064594/52714114-c3a32580-2fa1-11e9-957c-3cd1da15627a.PNG)

## Step 12: Get contact by Id
If we want to get a single contact by Id, we will send a GET request to ( http://127.0.0.1:3000/contact/:contactId ) It will return an Object of your contact. Remember that the ID that we passed to the URL is the _"id”_ of the contact.
![get2](https://user-images.githubusercontent.com/27064594/52714210-f4835a80-2fa1-11e9-83d0-7f557223e052.PNG)

## Step 13: Update an existing contact
In case we want to update an existing contact, we will send a PUT request to the ( http://127.0.0.1:3000/contact/:contactId ) together with the detail. 
![put](https://user-images.githubusercontent.com/27064594/52714248-0bc24800-2fa2-11e9-9de5-fe877883cb32.PNG)

## Step 14: Delete a contact
To delete a contact, we will send a DELETE request to ( http://127.0.0.1:3000/contact/:contactId ). It will return a message saying that _“Successfully deleted contact!”_
![delete](https://user-images.githubusercontent.com/27064594/52714273-21377200-2fa2-11e9-93b0-3fbe84ed72de.PNG)


