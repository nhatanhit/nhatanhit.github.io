---
layout: post
title: Sails Lifts
comments: false
bigimg: /img/path.jpg
tags: [sails]
---

# Overview

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

# The sails.load

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

## The configuration load(config)

Main flow of Sails default configuration `Configuration.load` in function configLoaded in file load.js

```javascript
function configLoaded(err, results) {
    if (err) {
        sails.log.error('Error encountered loading config ::\n', err);
        return cb(err);
    }

    // Override the previous contents of sails.config with the new, validated
    // config w/ defaults and overrides mixed in the appropriate order.
    sails.config = results.mixinDefaults;
    console.log("Config Loaded");
    console.log(sails.config);
    return cb();
}
/*
Output Config Loaded
{ environment: undefined,
  hooks:
   { moduleloader: [Function],
     logger: [Function],
     request: [Function],
     views: [Function],
     blueprints: [Function],
     responses: [Function],
     helpers: [Function],
     pubsub: [Function],
     policies: [Function],
     services: [Function],
     security: [Function],
     i18n: [Function],
     userconfig: [Function],
     session: [Function],
     http: [Function],
     userhooks: [Function] },
  appPath: '/Users/anhtran/projects/test-project',
  paths: { tmp: '/Users/anhtran/projects/test-project/.tmp' },
  routes: {},
  middleware: {},
  implementationSniffingTactic: 'analogOrClassical',
  generators: { modules: {} },
  _generatedWith: { sails: '1.2.4', 'sails-generate': '1.17.2' },
  _: [ 'lift' ],
  configs: [ '/Users/anhtran/projects/test-project/.sailsrc' ],
  config: '/Users/anhtran/projects/test-project/.sailsrc' 
}
*/
```

Here we see that we already had attributes: environment, hooks, apPath, paths, routes, middleware, and implementationSniffingTactic. Where they come from ?. Answer: In the function `function defaultConfig` in file configuration/index.js. 

```javascript
//...
return {

        environment: defaultEnv,

        // Note: to avoid confusion re: timing, `hooks` configuration may eventually be removed
        // from `sails.config` in favor of something more flexible / obvious, e.g. the `app` object
        // itself (i.e. because you can't configure hooks in `userconfig`-- only in `overrides`).

        // Core (default) hooks
        hooks: _.reduce(DEFAULT_HOOKS, function (memo, hookBundled, hookIdentity) {

          // if `true`, then the core hook is bundled in the `lib/hooks/` directory
          // as `lib/hooks/HOOK_IDENTITY`.
          if (hookBundled === true) {
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
        }, {}) || {},

        // Save appPath in implicit defaults
        // appPath is passed from above in case `sails lift` was used
        // This is the directory where this Sails process is being initiated from.
        // (  usually this means `process.cwd()`  )
        appPath: appPath,

        // Built-in path defaults
        paths: {
          tmp: path.resolve(appPath, '.tmp')
        },

        // Start off `routes` and `middleware` as empty objects
        routes: {},
        middleware: {},

        // Set implicit default for sniffing tactic.
        // Respected by helpers and actions, when running the configured
        // bootstrap function, and when invoking the `initialize` method
        // in hooks.
        implementationSniffingTactic: 'analogOrClassical'

};
    
```

Now we go to `verifyEnv`

## The Environment Verification(verifyEnv)
As described in the comment, the mainly task in this function is checking the env and process.env.node

```javascript
function verifyEnvironment() {
    // At this point, the Sails environment is set to its final value,
    // whether it came from the command line or a config file. So we
    // can now compare it to the NODE_ENV environment variable and
    // act accordingly.  This may involve changing NODE_ENV to "production",
    // which we want to do as early as possible since dependencies might
    // be relying on that value.

    // If the Sails environment is production, but NODE_ENV is undefined,
    // log a warning and change NODE_ENV to "production".
    if (sails.config.environment === 'production' && process.env.NODE_ENV !== 'production' ) {
      if (_.isUndefined(process.env.NODE_ENV)) {
        sails.log.debug('Detected Sails environment is "production", but NODE_ENV is `undefined`.');
        sails.log.debug('Automatically setting the NODE_ENV environment variable to "production".');
        sails.log.debug();
        process.env.NODE_ENV = 'production';
      } else {
        throw flaverr({ name: 'userError', code: 'E_INVALID_NODE_ENV' }, new Error('When the Sails environment is set to "production", NODE_ENV must also be set to "production" (but it was set to "' + process.env.NODE_ENV + '" instead).'));
      }
    }
  }
```
## The Grunt Config
The code for this task is located in this file `app/private/checkGruntConfig`. It only checks  the appearance of module: `sails-hook-grunt`. Don't need to aware to this module 

##  Hooks 
Function `helpLoadHooks` plays important role to load hooks for sails. Let take a look



