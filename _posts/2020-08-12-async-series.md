---
layout: post
title: Async Series
comments: false
bigimg: /img/path.jpg
tags: [nodejs,async]
---

Stuffs need to aware when using `async.series`

Check below code

```javascript
async.series([
    function(cb) {
        console.log("begin load system hooks definition");
        loadHookDefinitions(sails.hooks, cb);
        console.log("finish load system hooks definitions");
        
    },
    function(cb) {
        console.log("begin initialize system hooks");
        initializeHooks(sails.hooks, cb);
        console.log("finish initialize system hooks definitions");
      }
    ], function(err) {
            if (err) { 
                return cb(err); 
            }
            return cb();
    }
);
function loadHookDefinitions(hooks, cb) {
    return cb();
}
function initializeHooks(hooks, cb) {
    return cb();
}
```
The output response i got from console

```text
begin load system hooks definition
begin initialize system hooks
finish initialize system hooks definitions
finish load system hooks definitions
```

But `async.series` run tasks with order, the expected result should be

```text
begin load system hooks definition
finish load system hooks definitions
begin initialize system hooks
finish initialize system hooks definitions
```

The root cause: Because function loadHookDefinitions() already calling cb() , so that's trigger the next function to run 

So to ensure that tasks run with right order. The code should be


```javascript
async.series([
    function(cb) {
        console.log("begin load system hooks definition");
        console.log("finish load system hooks definitions");
        loadHookDefinitions(sails.hooks, cb);
        
        
    },
    function(cb) {
        console.log("begin initialize system hooks");
        console.log("finish initialize system hooks definitions");
        initializeHooks(sails.hooks, cb);
      }
    ], function(err) {
            if (err) { 
                return cb(err); 
            }
            return cb();
    }
);
function loadHookDefinitions(hooks, cb) {
    return cb();
}
function initializeHooks(hooks, cb) {
    return cb();
}
```
