---
layout: post
title:  "How to use ngCordova sqlite service and Cordova-SQLitePlugin with Ionic Framework"
date:   2014-11-15 12:18:15
categories: ionic angularjs cordova sqlite
---
So let me set the scene.....

I started with a basic Ionic app which I created using:
```
ionic start myApp sidemenu
```

Then I install the sqlite plugin:
```
ionic plugin add https://github.com/brodysoft/Cordova-SQLitePlugin
```
and ngCordova
```
bower install ngCordova
```
which gave me the following options:
Unable to find a suitable version for angular, please choose one:
```bash
  1.) - angular#1.2.0 which resolved to 1.2.0 and is required by ngCordova#0.1.4-alpha 
  2.) - angular#>= 1.0.8 which resolved to 1.2.0 and is required by angular-ui-router#0.2.10 
  3.) - angular#1.2.25 which resolved to 1.2.25 and is required by angular-animate#1.2.25, angular-sanitize#1.2.25 
  4.) - angular#~1.2.17 which resolved to 1.2.25 and is required by ionic#1.0.0-beta.13Prefix the choice with ! to persist it to bower.json
```

I picked option 3) and I included the scripts in the file as follows:
```html    
<script src="lib/ionic/js/ionic.bundle.js"></script>
<script src="lib/ngCordova/dist/ng-cordova.js"></script>
<script src="cordova.js"></script>
<script src="js/app.js"></script>
<script src="js/controllers.js"></script>
```
I then added a controller to the search view:
```js
.controller('SearchCtrl', function ($scope, $cordovaSQLite){
  console.log('Test');
   var db = $cordovaSQLite.openDB({ name: "my.db" });

        // for opening a background db:
        var db = $cordovaSQLite.openDB({ name: "my.db", bgType: 1 });

        $scope.execute = function() {
          console.log('Test');
          var query = "INSERT INTO test_table (data, data_num) VALUES (?,?)";
          $cordovaSQLite.execute(db, query, ["test", 100]).then(function(res) {
            console.log("insertId: " + res.insertId);
          }, function (err) {
            console.error(err);
          });
     };
})
```
This caused the error:
```bash
    > TypeError: Cannot read property 'openDatabase' of undefined
    >     at Object.openDB  (http://localhost:8100/lib/ngCordova/dist/ng-cordova.js:2467:36) 
```
Next I tried manually including the SQLitePlugin.js by:
copying from ***plugins/com.brodysoft.sqlitePlugin/www*** to main ***www/*** and adding it to the index.html page

I tried including before everything:
```html
    <script src="SQLitePlugin.js"></script>
    <script src="lib/ionic/js/ionic.bundle.js"></script>
    <script src="lib/ngCordova/dist/ng-cordova.js"></script>
    <script src="cordova.js"></script>
    <script src="js/app.js"></script>
    <script src="js/controllers.js"></script>
```
But I got Error **ReferenceError: cordova is not defined**

### Solution
So Turns out that it is because Cordova is platform specific and doesn't work when you run ionic serve. I was able to figure this out by run the same code on an android device without an issue when I built and deployed. So thanks to a suggetion from another developer I inlcuded the following:
```js
if (window.cordova) {
      db = $cordovaSQLite.openDB({ name: "my.db" }); //device
} else {
  db = window.openDatabase("my.db", '1', 'my', 1024 * 1024 * 100); // browser
}
```
