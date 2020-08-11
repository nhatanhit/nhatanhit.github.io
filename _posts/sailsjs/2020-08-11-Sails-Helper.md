---
layout: post
title: Sails Built-in Helpers
comments: false
bigimg: /img/path.jpg
tags: [sails]
---

Below is the list of helpers that Sails provided 

-------------------------------------------------------

 sails.helpers

```text
 Available methods:
   .   
   ├── flow
   │   ├── build()
   │   ├── dive()
   │   ├── forEach()
   │   ├── pause()
   │   ├── simultaneously()
   │   ├── simultaneouslyForEach()
   │   └── until()
   │   
   ├── fs
   │   ├── cp()
   │   ├── ensureDir()
   │   ├── exists()
   │   ├── ls()
   │   ├── mkdir()
   │   ├── mv()
   │   ├── read()
   │   ├── readJson()
   │   ├── readStream()
   │   ├── rmrf()
   │   ├── write()
   │   ├── writeJson()
   │   └── writeStream()
   │   
   ├── gravatar
   │   └── getAvatarUrl()
   │   
   ├── http
   │   ├── del()
   │   ├── get()
   │   ├── getStream()
   │   ├── patch()
   │   ├── post()
   │   ├── put()
   │   └── sendHttpRequest()
   │   
   ├── mailgun
   │   └── sendHtmlEmail()
   │   
   ├── passwords
   │   ├── checkPassword()
   │   └── hashPassword()
   │   
   ├── process
   │   ├── executeCommand()
   │   └── killChildProcess()
   │   
   ├── sendgrid
   │   └── sendHtmlEmail()
   │   
   ├── strings
   │   ├── ensureUniq()
   │   ├── random()
   │   ├── template()
   │   ├── toStream()
   │   └── uuid()
   │   
   ├── stripe
   │   ├── chargeCustomer()
   │   └── saveBillingInfo()
   │   
   ├── twilio
   │   └── sendTextMessage()
   │   
   └── sendTemplateEmail()


 Example usage:
   var contents = await sails.helpers.fs.ls('./colorado/');
   var name = sails.helpers.strings.random('url-friendly');

 More info:
   https://sailsjs.com/support
```