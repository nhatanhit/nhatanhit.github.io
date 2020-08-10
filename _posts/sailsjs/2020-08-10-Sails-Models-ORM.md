---
layout: post
title: Sails Models-ORM
comments: false
bigimg: /img/path.jpg
tags: [sails-models,sails]
---

# Table of Contents
1. [Models And ORM](#models-and-orm)
2. [Handle Errors](#handle-errors)
3. [Negotiating Model Errors](#negotiating-model-errors)
4. [Model LifeCycle](#model-lifecycle)
5. [Waterline Query Language](#waterline-query-language)



# Models And ORM

By convention, models are defined by creating a file in a Sails app's api/models/ folder
API working with model: https://sailsjs.com/documentation/reference/waterline-orm/models

## Query methods: 
Since they usually have to send a query to the database and wait for a response, most model methods are asynchronous.
In Node.js, Sails, and JavaScript in general, the recommended way to handle this is by using async/await

## Define custom model methods
Custom model methods are most useful for extrapolating controller code that relates to a particular model. They allow code to be pulled out of controllers and inserted into reusuable functions that can be called from anywhere (independent of req or res).

## Many to Many relationship 
When Waterline detects that two models have collection attributes that point to each other through their via keys (see below), it will automatically build up a join table for you.
Note that adding or removing associated records from one side of a many-to-many relationship will automatically affect the other side. 

### Dominance
In most cases, Sails will be able to create the join table for a many-to-many association without any input from you. However, if the two models in the association use different datastores, you may want to choose which one should contain the join table. You can do this by setting dominant: true on one of the associations in the relationship.

### Choosing a "dominant"
If one side is a SQL database, placing the join table on that side will allow your queries to be more efficient, since the relationship table can be joined before the other side is communicated with. This reduces the number of total queries required from 3 to 2
If one datastore is much faster than the other, all other things being equal, it probably makes sense to put the join table on that side.
If you know that it is much easier to migrate one of the datastores, you may choose to set that side as dominant
Along the same lines, if one of your datastores is read-only, you won't be able to write to it, so you'll want to make sure your relationship data can be persisted safely on the other side

## One way  association (Belongs To)
## One to one (Has One)

## Reflexive association
Code example
```javaScript
module.exports = {
  attributes: {
    firstName: {
      type: 'string'
    },
    lastName: {
      type: 'string'
    },

    // Add a singular reflexive association
    bestFriend: {
      model: 'user',
    },

    // Add one side of a plural reflexive association
    parents: {
      collection: 'user',
      via: 'children'
    },

    // Add the other side of a plural reflexive association
    children: {
      collection: 'user',
      via: 'parents'
    },

    // Add another plural reflexive association, this one via-less
    bookmarkedUsers: {
      collection: 'user'
    }

  }
};
// Add User #12 as a parent of User #23
await User.addToCollection(23, 'parents', 12);
// Find User #12 and populate its children
var userTwelve = await User.findOne(12).populate('children');

```

# Handle Errors 
The basically, the calling of handling error has following format
```javascript
try {
  await Something.create({…});
} catch (err) {
  // err.name
  // err.code
  // …
}
```

# Negotiating Model Errors 
When you want to display more info of the error you received -> using function  intercept() or tolerate()  of Model
An error is described by name (The broad classification of the error.) and code(A narrower classification of the error that is sometimes included.)
When an error has name: 'UsageError', this indicates that a Waterline method was used incorrectly, or executed with invalid options (for example, attempting to create a new record that would violate one of your model's high-level validation rules.)
name: 'AdapterError' , indicate a problem in the underlying adapter, and not in the request itself. This can happen when a database goes offline, when there is a permission issue, because of some database-specific edge case, or (more rarely) a bug in the adapter.
err.code === 'E_UNIQUE' => A uniqueness error occurs when a value that should be unique matches that of another record in the database. While this is considered an adapter error, it has its own code to differentiate it from a normal adapter error: code: 'E_UNIQUE'

Demo code
```
await User.create({ emailAddress: inputs.emailAddress })
// Uniqueness constraint violation
.intercept('E_UNIQUE', (err)=> {
  return 'emailAlreadyInUse';
})
// Some other kind of usage / validation error
.intercept({name:'UsageError'}, (err)=> {
  return 'invalid';
});
// If something completely unexpected happened, the error will be thrown as-is.

return exits.success();
```

# Model LifeCycle

Each Model has a lifecycle, there are 2 lifecycle callbacks attach on this lifecycle
Example Use case of life cycle callback :hashing password before updating it into database

```javascript
// User.js
module.exports = {

  attributes: {

    username: {
      type: 'string',
      required: true
    },

    password: {
      type: 'string',
      minLength: 6,
      required: true
    }

  },


  beforeCreate: function (valuesToSet, proceed) {
    // Hash password
    sails.helpers.passwords.hashPassword(valuesToSet.password).exec((err, hashedPassword)=>{
      if (err) { return proceed(err); }
      valuesToSet.password = hashedPassword;
      return proceed();
    });//_∏_
  }

};
```

# Waterline Query Language 
The syntax supported by Sails' model methods is called Waterline Query Language. Waterline knows how to interpret this syntax to retrieve or mutate records from any supported database. Under the covers, Waterline uses the database adapter(s) installed in your project to translate this language into native queries and send those queries to the appropriate database.

Document:

https://sailsjs.com/documentation/reference/waterline-orm/queries -> there are a small note of handling query error in this article

https://sailsjs.com/documentation/concepts/models-and-orm/query-language

{: .box-note}
To do the unit test use StandAlone WaterLine: https://sailsjs.com/documentation/concepts/models-and-orm/standalone-waterline-usage




