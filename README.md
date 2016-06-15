# deppie

## Disclaimer
deppie is just a draft for now. You can play with it to see how it works, but don't use it a real project just yet. If you're interested, follow the repo for the beta release very soon.
<!-- deppie is still in an experimental phase. Use at your own risk, and expect breaking changes. However, owing to it's minimal API, you could argue that it will be relatively easy to modify your code for such changes. -->

## Introduction
The simplest, smallest Dependency Injection framework for javascript.

I tried to find a DI framework in JS that I could wrap my head around and get my team to use effortlessly, but they all included writing complex code to define modules and start the application, and they dictate the way you write your files.

deppie uses minimal code to set up an IoC container.

If you're not familiar with DI you can read [this article by Martin Fowler][1], but the basic idea is that you write your code that depends on a module, without knowing (or caring) where that module comes from as long as it has the interface you expect.

[This thread][2] has is a nice debate on whether we need DI or not, but, obviously, I believe we do for a few simple reasons

1. "require" dependencies without knowing their paths. This means that if you change a module location on disk or swap it with another, you don't need to touch every file that depends on it. You only have to change it's definition once.

2. It makes it a lot easier to mock dependencies for testing without the need for something like proxyquire to override the "require" function.

## How to use

There are 2 steps to use deppie

1. Declare which dependencies your module needs. Thanks to ES6/ES2015, we have a very elegant way to declare dependencies without any extra code or config. Simply wrap your module in a function that takes an object of all the dependencies in the application, and use destructuring to pick the ones your module needs. Think of it as a constructor function that you don't have to worry about calling with the right parameters. deppie will do that for you.

    eg: `myModule.js`

    ```
    module.exports = ({ dependency1, dependecy2 }) => {
        // return your object or function here
    };
    ```
    deppie will parse the signature of your function and figure out that this module depends on `dependency1` and `dependecy2` only. No verbose declaration needed.

2. Locate dependencies. This is the part where you tell deppie where to find the actual modules and what to call them when they are injected. It takes an object where a key is a module name that you can use to inject it, and the value is the constructor function for that module. At the root of your project, you should have a file that requires all the modules in your application, and tells deppie to wire them together.

    eg: `index.js`

    ```
    const deppie = require('deppie');

    // each of these files exports a constructor function as explained above
    const dependency1 = require('path/to/dependency1');
    const dependecy2 = require('path/to/dependecy2');
    const myModule = require('path/to/myModule');

    // the keys of this object are what the dependencies will be named for injection
    const moduleDefinitions = {
        dependency1,
        dependecy2,
        myModule,
    };

    // this will invoke all the constructor functions of all the modules passed, making sure the right dependencies are injected
    deppie(moduleDefinitions);
    ```

    Again, deppie understands that `myModule` relies on `dependency1` and `dependecy2`, so it will call their constructors first. And so on for all modules.

## How it works
Unlike most other DI frameworks, deppie works in a very straight-forward way. In fact, the core of it was written in one sitting for a few hours.

- Construct all dependencies

    deppie just goes through all the modules defined in `moduleDefinitions` and constructs them one by one, passing all the required dependencies. If a dependency has not been constructed yet, deppie will construct it first. This goes on recursively until the base case of recursion, which is a module with no dependencies at all.

- Entry point(s)

    You don't need to define any entry points explicitly to deppie. It can be any one of your modules, the only difference is that an entry point module will have no return naturally (void module). deppie will construct these module just like any other module, except it will not allow you to inject it in other modules.

    eg: `app.js`

    ```
    const bodyParser = require('body-parser');
    const express = require('express');

    module.exports = ({ getUsersRoute, config }) => {
        const app = express()
        .use(bodyParser.text())
        .use(bodyParser.json())
        .use(bodyParser.urlencoded({ extended: true }));

        // Add routes
        app.get('/api/users', getUsersRoute);

        app.listen(config.port);
        console.log(`listening on port ${config.port}`);
        // don't return anything
    };
    ```

### Validations
deppie will enforce some rules for your modules
- no missing dependencies
- no circular dependencies
- no self dependencies
- can't depend on void modules

and warn you about some other rules
- no unused modules (TODO)

<!--
TODO:
## Design decisions

## Partial adoption

## Good Practices
### index, app, routes -->


[1]: http://martinfowler.com/articles/injection.html
[2]: http://stackoverflow.com/questions/9250851/do-i-need-dependency-injection-in-nodejs-or-how-to-deal-with
