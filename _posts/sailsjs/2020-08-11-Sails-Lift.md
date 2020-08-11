---
layout: post
title: Sails Lifts
comments: false
bigimg: /img/path.jpg
tags: [sails]
---
Initial point of `sails lift` command  is node_modules/sails/lift/app/lift.js. The following code is run to setup everything before running
```javascript
async.series([
    function (next) {
        //load config override
      sails.load(configOverride, next);
    },
    function (next){
        //initialize
      sails.initialize(next);
    },

    ], function whenSailsIsReady(err) {
    if (err) {
      //others code

    }
```

Go to `sails.load` first, it requires to module node_modules/sails/lift/app/load.js. Following is the heart of `sails.load`

```javascript
async.auto({
    config: [Configuration.load],

      // Verify that the combination of Sails environment and NODE_ENV is valid
      // as early as possible -- that is, as soon as we know for sure what the
      // Sails environment is.
      verifyEnv: ['config', function(results, cb) {
        //...
      }],

      // Check if the current app needs the Grunt hook installed but doesn't have it.
      grunt: ['config', checkGruntConfig],

      // Load hooks into memory, with their middleware and routes
      hooks: ['verifyEnv', 'config', helpLoadHooks],

      // Load actions from disk and config overrides
      controller: ['hooks', function(results, cb) {
        __loadActionModules(sails, cb);
      }],

      // Populate the "registry"
      // Houses "middleware-esque" functions bound by various hooks and/or Sails core itself.
      // (i.e. `function (req, res [,next]) {}`)
      //
      // (Basically, that means we grab an exposed `middleware` object,
      // full of functions, from each hook, then make it available as
      // `sails.middleware.[HOOK_ID]`.)
      //
      // - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
      // FUTURE: finish refactoring to change "middleware" nomenclature
      // to avoid confusion with the more specific (and more common)
      // usage of the term.
      // - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
      registry: ['hooks',
        function populateRegistry(results, cb) {
            /**/
        }
      ],
      // Load the router and bind routes in `sails.config.routes`
      router: ['registry', sails.router.load]
})
```

The order of running task: config > verifyEnv, grunt > hooks > controller , registry > router

Main flow of Sails default configuration `Configuration.load`

```javascript
// Core (default) hooks
hooks: _.reduce(DEFAULT_HOOKS, function (memo, hookBundled, hookIdentity) {

    // if `true`, then the core hook is bundled in the `lib/hooks/` directory
    // as `lib/hooks/HOOK_IDENTITY`.
    if (hookBundled === true) {
        console.log("core hook identity " + hookIdentity);  //i put the log console at here
        memo[hookIdentity] = require('../../hooks/'+hookIdentity);
    }
    // if it's a string, then the core hook is an NPM dependency of sails,
    // so require it (which grabs it from `node_modules/`)
    else if (_.isString(hookBundled)) {
    var hook;
    try {
        hook = require(hookBundled);
    } catch (unusedErr) {
        // FUTURE: provide access to error details instead of swallowing
        throw new Error('Sails internal error: Could not require(\''+hookBundled+'\').');
    }
    memo[hookIdentity] = hook;
    }
    // otherwise freak out
    else {
    throw new Error('Sails internal error: `'+hookIdentity+'`, a core hook, is invalid!');
    }
    return memo;
}
/*The output debug
core hook identity moduleloader
core hook identity logger
core hook identity request
core hook identity views
core hook identity blueprints
core hook identity responses
core hook identity helpers
core hook identity pubsub
core hook identity policies
core hook identity services
core hook identity security
core hook identity i18n
core hook identity userconfig
core hook identity session
core hook identity http
core hook identity userhooks
*/
```
