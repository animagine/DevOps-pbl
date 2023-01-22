# MEAN stack implementation on Ubuntu in AWS

**STEP 1: Install Nodejs** 

Spin up an ubuntu server EC2 instance in aws and update it.

`sudo apt update`

`sudo apt upgrade`

<img width="886" alt="Screenshot 2023-01-20 at 2 09 10 PM" src="https://user-images.githubusercontent.com/1076924/213704118-c8c28f02-ef0b-45eb-9b86-1fdceb9fb1f6.png">


Once this is completed, proceed to adding certificates and then install nodejs

`sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates`

`curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`

<img width="890" alt="Screenshot 2023-01-20 at 2 17 56 PM" src="https://user-images.githubusercontent.com/1076924/213704019-08a80759-d8c4-4106-a0ce-9656df0b76ae.png">

**Install Nodejs**

`sudo apt install -y nodejs`

---

**STEP 2: Install MOngoDB**

`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`

`echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

`sudo apt install -y mongodb`

running the script above may flag an error:

> The following packages have unmet dependencies:
 mongodb-org-mongos : Depends: libssl1.1 (>= 1.1.1) but it is not installable
 mongodb-org-server : Depends: libssl1.1 (>= 1.1.1) but it is not installable
 mongodb-org-shell : Depends: libssl1.1 (>= 1.1.1) but it is not installable
E: Unable to correct problems, you have held broken packages.


but should be easily fixed following the instructions below

**N.B - As at writing this, MongoDb has no official build for ubuntu 22.04 at the moment
Ubuntu 22.04 has upgraded libssl to 3 and does not propose libssl1.1
You can force the installation of libssl1.1 by adding the ubuntu 20.04 source:**

```
echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
sudo apt-get update
sudo apt-get install libssl1.1

```

Install mongodb again

`sudo apt install -y mongodb`

Delete the focal security that was created

`sudo rm /etc/apt/sources.list.d/focal-security.list`

once this is done, start the server

`sudo service mongod start`

verify that the service is up and running

`sudo systemctl status mongod`

<img width="882" alt="Screenshot 2023-01-20 at 3 19 22 PM" src="https://user-images.githubusercontent.com/1076924/213721154-8dfba814-f750-4cda-8b63-e9ec7fd727e1.png">

Install npm (Node Package Manager)

`sudo apt install -y npm`

Install body-parser package - the body-parser helps with the processing of JSON files passed in requests to the server

`sudo npm install body-parser`

Create a folder named 'Books'

`mkdir Books && cd Books`
<img width="880" alt="Screenshot 2023-01-20 at 3 53 04 PM" src="https://user-images.githubusercontent.com/1076924/213728619-09c73407-01b8-42fc-9fa2-21d0b739e56e.png">

`npm init`


`vi server.js`


```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});

```
---

**STEP 3 - Install Express and set up routes to the server**

Install express and mongoose

`sudo npm install express mongoose`

In the 'Books' folder, create a folder named 'apps' then also create a file named routes.js

`mkdir apps && cd apps`

`vi routes.js`

```
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};

```

In the 'apps' folder, create a folder named 'models'

`mkdir models && cd models`

Create a file named book.js

`vi book.js`

---

**STEP 4 access routes with AngularJS

change directory back to 'Books'

`cd ../..`

Create a folder named 'Public'

`mkdir public && cd public`

add a file named scripts.js

`vi scripts.js`

```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});

```

Create a file named index.html as well

`vi index.html`

insert the folowing lines of code

```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>

```

Change back to the 'Books directory

`cd ..`

start the node server

`node server.js`

Go to your browser, using your EC2 public address followed by port 3300

<img width="713" alt="Screenshot 2023-01-20 at 4 34 33 PM" src="https://user-images.githubusercontent.com/1076924/213740314-47568e10-a534-4be5-9a06-df717db140b3.png">

<img width="746" alt="Screenshot 2023-01-20 at 4 40 53 PM" src="https://user-images.githubusercontent.com/1076924/213741454-862fe5e4-2600-4931-a9b5-347ee9afc5ab.png">

<img width="692" alt="Screenshot 2023-01-20 at 4 40 37 PM" src="https://user-images.githubusercontent.com/1076924/213741463-3e1b9faa-63e5-47f7-8276-216dc199373b.png">


