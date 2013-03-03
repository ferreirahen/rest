Rest.js
=======

Just enough client, as you need it.  Make HTTP requests from a browser or Node.js applying the only the client features you need.  Configure a client once, and share it safely throughout your application.  Easily extend with interceptors that wrap the request and/or response, or MIME type converters for rich data formats.


Build Status
------------

<table>
  <tr><td>Master</td><td><a href="http://travis-ci.org/s2js/rest" target="_blank"><img src="https://secure.travis-ci.org/s2js/rest.png?branch=master" /></a></tr>
  <tr><td>Development</td><td><a href="http://travis-ci.org/s2js/rest" target="_blank"><img src="https://secure.travis-ci.org/s2js/rest.png?branch=dev" /></a></tr>
</table>


Usage
-----

Using Rest.js is easy.  The core clients provide limited functionality around the request and response lifecycle.  The input and response objects are normalized to support portability between browser and server environments.

The response from a client is a promise that is resolved when the remote request finishes.

The core client behavior can be augmented with interceptors.  An interceptor wraps the client and transforms the request and response.  For example: an interceptor may authenticate a request, or reject the promise if an error is encountered.  Interceptors may be combined to create a client with the desired behavior.  A configured interceptor acts just like a client.  The default client is dumb, it only know the low level mechanics of making a request and parsing the response.  All other behavior is applied and configurated with interceptors.

Interceptors are applied to a client by chaining.  To chain a client with an interceptor to a client, call the `chain` function on the client providing the interceptor behavior and optionally a configuration object.  A new client is returned containing the new behavior applied to the client.  It's important to note that the behavior of the original client is not modified, in order to use the new behavior, you must use the returned client.


### Making a basic request: ###

```javascript
var rest = require('rest');

rest({ path: '/' }).then(function(response) {
    console.log('response: ', response);
});
```

In this example, you can see that the request object is very simple, it just includes the path.  All of the attributes of a request are optional.

The response should look familiar as well, it contains all the fields you would expect, including the response headers (many clients ignore the headers).


### Working with JSON: ###

If you paid attention when executing the previous example, you may have noticed that the response.entity is a string.  Often we work with more complex data types.  For this, Rest.js supports a rich set of MIME type conversions with the `mime` interceptor.  The correct converter will automatically be chosen based on the `Content-Type` response header.  Custom converts can be registered for a MIME type, more on that later...

```javascript
var rest, mime, client;

rest = require('rest'),
mime = require('rest/interceptor/mime');

client = rest.chain(mime);
client({ path: '/data.json' }).then(function(response) {
    console.log('response: ', response);
});
```

Before an interceptor can be used, it needs to be configured.  In this case, we will accept the default configuration, and obtain a client.  Now when we see the response, the entity will be a JS object instead of a String.


### Composing Interceptors: ###

```javascript
var rest, mime, errorCode, client;

rest = require('rest'),
mime = require('rest/interceptor/mime');
errorCode = require('rest/interceptor/errorCode');

client = rest.chain(mime)
             .chain(errorCode, { code: 500 });
client({ path: '/data.json' }).then(
    function(response) {
        console.log('response: ', response);
    },
    function(response) {
        console.error('response error: ', response);
    }
);
```

In this example, we take the client create by the `mime` interceptor, and wrap it with the `errorCode` interceptor.  The errorCode interceptor accepts a configuration object that indicates what status codes should be considered an error.  In this case we override the default value of <=400, to only reject with 500 or greater status code.

Since the errorCode interceptor can reject the response promise, we also add a second handler function to receive the response for requests in error.

Clients can continue to be composed with interceptors as needed.  At any point the client as configured can be shared.  It is safe to share clients and allow other parts of your application to continue to compose other clients around the shared core.  Your client is protected from additional interceptors that other parts of the application may add.


### Declarative Interceptor Composition: ###

First class support is provided for declaratively composing interceptors using [wire.js](https://github.com/cujojs/wire).  Wire is an dependency injection container; you specify how the parts of your application interrelate and wire takes care of the dirty work to make it so.

Let's take the previous example and configure the client using a wire specification instead of imperative code.

```javascript
{
	...,
	client: {
		rest: [
			{ module: 'rest/interceptor/mime' },
			{ module: 'rest/interceptor/errorCode', config: { code: 500 } }
		]
	},
	plugins: [{ module: 'rest/wire' }]
}

```

There are a couple things to notice.  First is the 'plugins' section, by declaring the `rest/wire` module, the `rest` factory becomes available within the specification.  The second thing to notice is that we no longer need to individually `require()` interceptor modules; wire is smart enough to automatically fetch the modules.  The interceptors are then chained together in the order they are defined and provided with the corresponding config object, if it's defined.  The resulting client can then be injected into any other object using standard wire facilities.


### Custom MIME Converters: ###

```javascript
var registry = require('rest/mime/registry');

registry.register('application/vnd.com.example', {
    read: function(str) {
        var obj;
        // do string to object conversions
        return obj;
    },
    write: function(obj) {
        var str;
        // do object to string conversions
        return str;
    }
});
```

Registering a custom converter is a simple as calling the register function on the mime registry with the type and converter.  A converter has just two methods: `read` and `write`.  Read converts a String to a more complex Object.  Write converts an Object back into a String to be sent to the server.  HTTP is fundamentally a text based protocol after all.

Built in converters are available under `rest/mime/type/{type}`, as an example, JSON support is located at `rest/mime/type/application/json`.  You never need to know this as a consumer, but it's a good place to find examples.


Supported Environments
----------------------

Our goal is to work in every major JavaScript environment; Node.js and major browsers are actively tested and supported.

If your preferred environment is not supported, please let us know. Some features may not be available in all environments.

Tested environments:
- Node.js (0.8, should wok in earlier versions)
- Chrome (stable)
- Firefox (stable, ESR, should work in earlier versions)
- IE (6-10)
- Safari (5, 6, iOS 4-6, should work in earlier versions)
- Opera (11, 12, should work in earlier versions)

Specific browser test are provided by [Travis CI](https://travis-ci.org/s2js/rest) and [Sauce Labs' Open Sauce Plan](https://saucelabs.com/opensource). You can see [specific browser test results](https://saucelabs.com/u/s2js-rest), although odds are they do not reference this specific release/branch/commit.


Getting Started
---------------

Rest can be installed via [npm](https://npmjs.org/), [Bower](http://twitter.github.com/bower/), or from source.

To install without source:

    $ npm install rest

or

    $ bower install rest

From source:

    $ npm install

Rest.js is designed to run in a browser environment, utilizing [AMD modules](https://github.com/amdjs/amdjs-api/wiki/AMD), or within [Node.js](http://nodejs.org/).  [curl](https://github.com/cujojs/curl) is highly recommended as an AMD loader, although any loader should work.

An ECMAScript 5 compatible environment is assumed.  Older browsers, ::cough:: IE, that do not support ES5 natively can be shimmed.  Any shim should work, although we've tested against cujo's [poly](https://github.com/cujojs/poly)


Reporting Issues
----------------

Please report issues on [GitHub](https://github.com/s2js/rest/issues).  Include a brief description of the error, information about the runtime (including shims) and any error messages.

Feature requests are also welcome.


Running the Tests
-----------------

The test suite can be run in two different modes: in node, or in a browser.  We use [npm](https://npmjs.org/) and [Buster.JS](http://busterjs.org/) as the test driver, buster is installed automatically with other dependencies.

Before running the test suite for the first time:

    $ npm install

To run the suite in node:

    $ npm test

To run the suite in a browser:

    $ npm start
    browse to http://localhost:8282/ in the browser(s) you wish to test.  It can take a few seconds to start.


Contributors
------------

- Scott Andrews <andrewss@vmware.com>
- Jeremy Grelle <jgrelle@vmware.com>

Please see CONTRIBUTING.md for details on how to contribute to this project.


Copyright
---------

Rest.js is made available under the MIT license.  See LICENSE.txt for details.

Copyright (c) 2012-2013 VMware, Inc. All Rights Reserved.

VMware, Inc.
3401 Hillview Avenue
Palo Alto, CA 94304


Change Log
----------

.next
- Interceptor configuration chaining
- wire.js factory
- access to the originating client as request.originator
- hateoas and location interceptors default to use request.originator
- defaultRequest interceptor, provide default values for any portion of a request
- XDomainRequest support for IE 8 and 9
- allow child MIME registries and configurable registry for the mime interceptor
- shared 'this' between request/response phases of a single interceptor per request
- cleaned up interceptor's request handler for complex requests
- XHR fall back interceptor for older IE
- SimpleRestStore that provides the functionality of RestStore without Dojo's QueryResults
- mutli-browser testing with Sauce Labs

0.8.4
- Bower installable, with dependencies
- node client's response.raw includes ClientResquest and ClientResponse objects
- basicAuth interceptor correctly indicates auth method

0.8.3
- moving from the 'scothis' to the 's2js' organization, no functional changes

0.8.2
- requests may be canceled
- timeout incerceptor that cancels the request unless it finishes before the timeout
- retry interceptor handles error respones by retrying the request after an elapsed period
- error interceptor handlers may recover from errors, a rejected promise must be returned in order to preserve the error state
- response objects, with an error property, are used for client errors instead of the thrown value
- interceptor response handlers recieve the interceptor's client rather then the next client in the chain
- interceptor request handlers may provide a response
- convert modules to UMD format; no functional impact
- replaced rest/util/base64 with an MIT licenced impl; no functional impact

0.8.1
- fixed bug where http method may be overwritten

0.8.0
- npm name change 'rest-template' -> 'rest'
- introduced experimental HATEOAS support
- introduced 'location' interceptor which follows Location response headers, issuing a GET for the specified URL
- default method to POST when request contains an entity
- response handlers now have access to the request client to issue subsequent requests
- interceptors may specify their default client
- renamed `rest/interceptor/_base` to `rest/interceptor`

0.7.5
- Initial release, everything is new
