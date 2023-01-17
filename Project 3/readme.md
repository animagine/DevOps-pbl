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
npm install dotenv

- update index.js file with the following code
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
