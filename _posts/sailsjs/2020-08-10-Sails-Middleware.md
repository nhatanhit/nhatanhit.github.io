---
layout: post
title: Sails Middlware
comments: false
bigimg: /img/path.jpg
tags: [sails-middleware,sails]
---
Four types of middleware

HTTP middleware—to apply code before every HTTP request

Policies—to apply code before one or more controller actions

Hooks with the routes feature implemented—to apply code before one or more route handlers

Custom responses—to apply code after one or more controller actions


# Http Middleware
Defined in config/http.js
See detail : [Middleware](https://sailsjs.com/documentation/concepts/middleware)

# Policies
To make Express middleware apply to only a particular action, you can also include Express middleware as a policy—just be sure that you actually want it to run for both HTTP and virtual socket requests.

Policies in Sails are versatile tools for authorization and access control: they let you execute some logic before an action is run in order to determine whether or not to continue processing the request. The most common use-case for policies is to restrict certain actions to logged-in users only.

{: .box-note}
policies apply only to controllers and actions, not to views. If you define a route in your routes.js config file that points directly to a view, no policies will be applied to it.

# Hooks with routes function
[Hook with route function](https://sailsjs.com/documentation/concepts/extending-sails/hooks/hook-specification/routes)
# Custom Response
[Custom Response](https://sailsjs.com/documentation/concepts/extending-sails/custom-responses)
Usage: 

Create new .js file in folder api/response

Call with `res.thatFileName`.

For example: api/responses/insufficientFunds.js can be executed with a call to res.insufficientFunds().

```javascript
return res.insufficientFunds(err, { happenedDuring: 'signup' });
```

File api/responses/insufficientFunds.js
```javascript
module.exports = function insufficientFunds(err, extraInfo){

  var req = this.req;
  var res = this.res;
  var sails = req._sails;

  var newError = new Error('Insufficient funds');
  newError.raw = err;
  _.extend(newError, extraInfo);

  sails.log.verbose('Sent "Insufficient funds" response.');

  return res.badRequest(newError);

}
```

