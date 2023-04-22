# MEAN STACK IMPLEMENTATION

## INSTALL NODEJS

### UPDATE UBUNTU

`sudo apt update`

![sudo apt update](./images_4/sudo_apt_update.png)

### UPGRADE UBUNTU

`sudo apt upgrade`

![sudo_apt_upgrade](./images_4/sudo_apt_upgrade.png)

### ADD CERTIFICATES

`sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates`

![addition_of_certificates](./images_4/certificate_release.png)

`curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`

### INSTALL NODEJS

`sudo apt install -y nodejs`

![nodejs_install](./images_4/nodejs_install.png)

## INSTALL MONGODB

`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`

![apt_key](./images_4/apt-key.png)

`echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list`

`sudo apt install -y mongodb`

![mongodb_install](./images_4/mongodb_install.png)

### START THE SERVER

`sudo service mongodb start`


### VERIFICATION OF RUNNING SERVICE

`sudo systemctl status mongodb`

![mongodb_status_check](./images_4/starting_status_check_mongodb.png)

### NPM INSTALL

`sudo apt install -y npm`

![npm_install](./images_4/npm_install.png)

### BODY PARSER PACKAGE INSTALLATION

`sudo npm install body-parser`


### CREATION OF BOOKS FOLDER

`mkdir Books && cd Books`

### INITIALIZATION OF NPM PROJECT IN Books DIRECTORY

`npm init`

![npm_init](./images_4/npm_init.png)

`vi server.js`

`var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});`

## INSTALLATION OF EXPRESS AND SETTING UP ROUTES TO THE SERVER

### INSTALLATION OF MONGOOSE

`cd ..`

`sudo npm install express mongoose`

![install_express_mongoose](./images_4/installation_express_mongoose.png)

### CREATION OF APPS FOLDER IN THE BOOKS FOLDER AND OPENING routes.js IN APPS FOLDER

`mkdir apps && cd apps`

`vi routes.js`

`var Book = require('./models/book');
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
};`

### CREATION OF THE MODELS FOLDER IN THE APPS FOLDER

`mkdir models && cd models`

### CREATION OF FILE NAMED book.js IN MODELS FOLDER

`vi book.js`

`var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);`

## ACCESSING THE ROUTES WITH ANGULARJS

### CHANGE DIRECTORY BACK TO BOOKS

`cd ../..`

### CREATE FOLDER NAMED PUBLIC IN BOOKS FOLDER

`mkdir public && cd public`

### ADDITION OF FILE NAMED script.js IN PUBLIC FOLDER

`vi script.js`

`var app = angular.module('myApp', []);
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
});`

### CREATION OF index.html IN PUBLIC FOLDER

`vi index.html`

`<!doctype html>
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
</html>`

### CHANGE DIRECTORY TO Books FOLDER

`cd ..`

### STARTING SERVER

`node server.js`

![web_application](./images_4/web_Book%20_register_application.png)