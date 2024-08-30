# After [![Build Status][1]][2]

Invoke callback after n calls

## Status: production ready

## Example

```js
var after = require("after")
var db = require("./db") // some db.

var updateUser = function (req, res) {
  // use after to run two tasks in parallel,
  // namely get request body and get session
  // then run updateUser with the results
  var next = after(2, updateUser)
  var results = {}
  
  getJSONBody(req, res, function (err, body) {
    if (err) return next(err)
    
    results.body = body
    next(null, results)
  })
  
  getSessionUser(req, res, function (err, user) {
    if (err) return next(err)
    
    results.user = user
    next(null, results)
  })
  
  // now do the thing!
  function updateUser(err, result) {
    if (err) {
      res.statusCode = 500
      return res.end("Unexpected Error")
    }
    
    if (!result.user || result.user.role !== "admin") {
      res.statusCode = 403
      return res.end("Permission Denied")
    }
    
    db.put("users:" + req.params.userId, result.body, function (err) {
      if (err) {
        res.statusCode = 500
        return res.end("Unexpected Error")
      }
      
      res.statusCode = 200
      res.end("Ok")  
    })   
  }
}
```

## Naive Example

```js
var after = require("after")
    , next = after(3, logItWorks)

next()
next()
next() // it works

function logItWorks() {
    console.log("it works!")
}
```

## Example with error handling

```js
var after = require("after")
    , next = after(3, logError)

next()
next(new Error("oops")) // logs oops
next() // does nothing

// This callback is only called once.
// If there is an error the callback gets called immediately
// this avoids the situation where errors get lost.
function logError(err) {
    console.log(err)
}
```

## Installation

`npm install after`

## Tests

`npm test`
