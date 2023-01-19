MERN stack implementation

=============
Creating and hosting a TO-DO list application.

Dependencies - MongoDb to serve as database for our application.
             - Epressjs serving as a server for our application.
             - Reactjs to build the front-end of out application.
             - Nodejs javascript runtime environment that would run on the machine rather than the browser.
             
# Once virtual server is setup on AWS proceed to do the followwing

**update and upgrade ubuntu**
sudo apt update
sudo apt upgrade


## install nodejs on the server from Ubuntu repo
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v
npm -v


## Setup application
mkdir Todo

# check that directory was created successfully
ls 

# change into the application directory and initialise the node server
`cd Todo`

` npm init`

## Installing Expressjs
`npm install express`

`touch index.js`

`vim index.js`

-install dotenv

`npm install dotenv`

- update index.js file with the following code
- 
```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

```
## Start node server

`node index.js`

**N.B:** In your EC2 instance, open port 5000 in your security group under inbound rules.

Open up your browser and try to access your server's public address followed by port 5000

`http://<PublicIP-or--PublicDNS>:5000

## Routes, Models , 

The Todo application will have the following functions

- Creating a new taask
- Displaying list of all tasks
- Deleting completed tasks

Each task will be associated with a particular endpoint and will use different standard HTTP request methods:POST,GET,DELETE.

We will be creaating a route for eaach task that will define various endpoints that the aapp will depend on.

`mkdir routes`

`cd routes`

`touch api.js'

`nano api.js`

copy the code below and paste into the api.js file
```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;

```

## Setting up Mongodb database using MLAB

sign up for an account on MLAB [here](https://www.mongodb.com/atlas-signup-from-mlab)

Follow up the process and select AWS as the cloud provider and choose a region near you.

After setting it up, in the index.js file created earlier we specified  process.env to access environment variables. Now we will create this file in the Todo directory.

`touch .env`

`vi .env`

add the connection string to access the database

`DB = 'mongodb+srv://<username>:<password>@cluster0.yipanoa.mongodb.net/?retryWrites=true&w=majority'`

save and exit.

- Update index.js file with the code below

`vi index.js`

`const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});`

exit and save.

In Todo root directory start server using the following command

`node index.js`

## Frontend creation

- create react app in the Todo directory 

`npx create-react-app cleint`

- install dependencies

`npm install concurrently --save-dev`

`npm install nodemon --save-dev`

In the Todo folder, open the package.json file. replace the scripts declaration with the one below

`"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
`

-Change directory to the 'Client' directory

`cd client`

open the package.json file

`vi package.json`

Add the following value pair in the package.json file

`"proxy": "http://localhost:5000"`

Doing the above proxy configuration makes it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 

return to root directory

`cd ../ ..`

type the following command

`npm run dev'

the application should open and start running on localhost:3000

** NB ** open TCP port 3000 on EC2 instance by adding a ew rule to the security group.


## Creating react components

We will create the react app components in the 'Client' directory

`cd client`

`cd src`

`mkdir components`

- create 3 components in this directory

` touch Input.js ListTodo.js Todo.js `

`vi Input.js`

`
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input
`

- Next, install [Axios](https://github.com/axios/axios)
Axios is a Promised based HTTP cleint for the browser and node.js. This should be installed in the 'client' directory.

`cd .. /client`

`npm install axios`

- go to 'components' directory

`cd src/components`

- Open the TodoList.js file and paste in the following code

`vi ListTodo.js`

`
 import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo

`
- Open the Todo.js file and paste the following code in as well

` vi Todo.js'

` import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;
`

- Edit the default react code in the App.js file in the src directory

` cd ..`

` vi App.js`

`
import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;
`
- also in the same directory, open and edit the App.css file

`vi App.css`

`
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
`

- In the src directory open the index.css file

`vi index.css`

`body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
`
- Return to the Todo directory

`cd ../ ..`

- run the application using the following

`npm run dev`

- if everything runs without any erros, you should be able to view the application in your browser using http://localhost:3000
- 
 
<img width="1112" alt="Screenshot 2023-01-17 at 1 00 42 PM" src="https://user-images.githubusercontent.com/1076924/213542026-b584f976-c7ea-4bf4-81b8-0140c2d76599.png">



<img width="1402" alt="Screenshot 2023-01-17 at 2 23 59 PM" src="https://user-images.githubusercontent.com/1076924/213541881-ee3ffb52-abdc-4e5c-9d90-147d9afa5e9e.png">

