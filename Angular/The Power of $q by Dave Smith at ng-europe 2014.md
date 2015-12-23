
#▶[The Power of $q by Dave Smith at ng-europe 2014](https://www.youtube.com/watch?v=33kl0iQByME)
##The old way
Passing callbacks as function arguments
```
function getCurrentUser(callback){
  $http.get('/api/user/me')
  .success(function(user){ callback(user); })
}
```
Adding another request
```
function getPremission(callback){
  $http.get('/api/premissions')
  .success(function(premissions){ callback(premissions); })
}
```
Calling both...
```
getCurrentUser(function(user){ /*Do something with user*/ });
getPremissions(function(premission){ /*Do something with premissions*/ });
//Do something with user and premisions... How?
```
Serial?
```
getCurrentUser(function(user){
  getPremissions(function(premissions){
    //Do somethiong with premissions and user
  })
});
```
But id doen **not** scale. 
Problems:
* Not parallelizable
* Not composable
* Not dynamic

##With $q we can do better!
Two simple steps:
* Stop passing callbacks as parameters
* Start returning promises
```
function getCurrentUser(){
  var deferred = $q.defere();
  $http.get('/api/users/me')
  .success(function(user){
    deferred.resolve(user);
  });
  return deferred.promise;
}
```
```
getCurrentUser().then(function(user){ //Do something with user });
```
Suddenly, all your async operations become:
* Parallelizable
* Composable
* Dynamic

###Parallelizable
```
$q.all([getCurrentUser(), getPremissions()])
.then(function(responses){
  var user = responses[0];
  var premissions =  responses[1];
});
```
This can be even nicer...
```
function spread(func) {
  return function(array)
  {
    func.apply(void 0, array);
  }
}
```
```
$q.all([getCurrentUser(), getPremissions()])
.then(spread(function(user, premissions){ //Do something with user and premissions }));
```

###Composable
They can be chained.

###Dynamic
```
var promises = [getCurrentUser()];
if(shouldGetPermissions){
  promises.push(getPermissions());
}
$q.all(promises).then(function(){ /* All done */});
```
##Why this API separation?
```
deferred.resolve()
```
```
promise.then()
```

This forces a separation between notifier and receiver.

**Notifier** is entity of code that has access to the deferred object, that constructed it and is responsible for notifying callers when operation is complete.

**Receiver** is entity of code who is simply subscribing to these services by way of the promise.

Reciever does not have access to the notifier objects because they could accidentaly trigger the resolution before the operations is complete. The only entity that can trigger the resolution is the one that constructed the deferred object.

##What about errors?
The notifier can send errors like this: `deferred.reject()`
The reciever can recieve them like this:
```
promise.then(function(){
    /*success*/
  }, function(){
    /*failure*/
  });
```
##What about status?
The notifier can send progress updates like this: `deferred.notify(...)`
The receiver can receive them like this:
```
promise.then(function(){
  /* success */
}, function(){
  /* failure */
}, function(){
  /* progress (called 0 ot more times) */
})
```
##Deferreds: One time use
Once you resolve or reject a **deferred** object, it cannot be resolved or deferred again.

##But you can be late to the party
You **can** connect **.then()** to a promise **after** its deferred has been resoved, and your callback will be called at the next digest loop (this happens with $q).

##Last but not least: $q.when()
On abstract level: It's all about taking values of any kind wrapping them in promisses and returning them.
What can be passed: other promisses, literal values, objects,...

##[Easy client-side caching](https://youtu.be/33kl0iQByME?t=15m50s)
```
myApp.factory('movies', function($q, $http){
  var cachedMovies;
  return{
    getMovies: function(){
      return $q.when(cachedMovies || helper());
    };
  }
  
  function helper(){
    var deferred = $q.defer();
    $http.get('/movies').succcess(function(movies){
      cachedMovies = movies;
      deferred.resolve(movies);
    })
    return deferred.promise;
  }
});
```
But this one contains a bug. Bug fix:
```
myApp.factory('movies', function($q, $http){
  var cachedMovies, p;
  return{
    getMovies: function(){
      return $q.when(cachedMovies || p || helper());
    };
  }
  
  function helper(){
    var deferred = $q.defer();
    p = deferred.promise;
    $http.get('/movies').succcess(function(movies){
      cachedMovies = movies;
      deferred.resolve(movies);
    })
    return deferred.promise;
  }
});
```
**Side note** $http already does caching. Look at the "cache" argument.

##$q is Angular's Champagne
* All $http requests use $q
 * Request interceptors
 * REsponse interceptors
* $interval and $timeout
* ngAnimate
* ngModelController (async validators)
* $tamplateRequest

Why did Angular write $q? Why not use the original "Q"? Because $q is aware of the digest loop. Why we would want promises to wait until the next digest loop. Thats just to make sure that when they execute a dirty check will happen after your then() function gets called. Otherwise you would have to apply digest yourself.