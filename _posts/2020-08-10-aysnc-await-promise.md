---
layout: post
title: Async, Await, Promises
comments: false
bigimg: /img/path.jpg
tags: [javascript]
---

#  Async
Async always return a promise
```javascript
async function fetchUrl(url) {
    return url;
}
//call
fetchUrl("www.google.com"); //return Promise {<fulfilled>: "www.gooogle.com"}
```

# Async => Promises => then / catch
After Async return a promise , we can get the result by appending to promise chain 

```javascript
async function fetch(url) {
    try {
        //....
        return url;
    } catch(ex) {
        throw Error(ex);
        //or alternatively 
        return Promise.reject(new Error(ex));
    }
    
}
function logFetch(url) {
    return  fetch(url)
        .then(data => { return data + ".vn";})
        .then(text => {console.log(text)})  //output: www.google.com.vn
        .catch(err => {
            console.log(err);
        })
}
logFetch("wwww.google.com");
```

# Await
To make the code of  the Promise chains (then, then ,catch) become shorter, using await

```javascript
async function fetch(url) {
    return url; 
}
async function logFetch(url) {
    try {
        a =  await fetch(url);
        //or on alternative way
        a = await fetch(url)
            .then((x) => console.log(x))
            .catch(e => console.log(e)); 
    } catch(ex) {   //this is for await without then/cache
        console.log(ex);
    }
    console.log(a + ".vn");
}
logFetch("wwww.google.com");    //output: www.google.com
```