Osprey
======

The [RAML](raml.org) based [Node JS](http://nodejs.org/) framework for building robust and self-documented APIs, quick and painlessly.

### Important
**Osprey current release is still a work in progress. Osprey development team and other teams from MuleSoft have been working on Osprey based
implementations, and even though it has been a great experience and it has been properly enhanced, it doesn't meet our criteria for a first stable version yet. **

**We encourage you to use it and contribute raising issues, providing feedback or pushing your own code.**

#### Not supported features (yet)
Please, check the issues for [Osprey 1.0 Milestone](https://github.com/mulesoft/osprey/issues?milestone=1&state=open) to be up-to-date with the future features to be implemented.

### Fundamentals
Take advantage of the power of REST and its out of the box features, such as:
- Automatic Validations:
 - Form, uri and query parameters.
 - Headers.
 - JSON and XML Schemas.
- Default Parameters.
- Exception Handling support.
- Auto-generated mocks for your APIs (or APIs your team will be interacting with) by just defining sample responses in the RAML file.
- [API Console](https://github.com/mulesoft/api-console): Auto-generated documentation displayed on an interactive web application, allowing the developer to run the actual logic behind the APIs.

### Related projects
Check [Osprey-CLI](https://github.com/mulesoft/osprey-cli). The scaffolding tool to generate Osprey-based applications with just a command.

### Contributing
If you are interesting in contributing by submitting your code to this project, please read the [Contributors Agreement](https://api-notebook.anypoint.mulesoft.com/notebooks#bc1cf75a0284268407e4)

### Prerequisites

To start using Osprey you'll need the following:

* [Node JS](http://nodejs.org/)
* [NPM](https://npmjs.org/)

### Getting started

`npm install git+https://github.com/mulesoft/osprey.git`

Optionally, you can use [Bower](http://bower.io/) - `bower install git@github.com:mulesoft/osprey.git`

Note: You can ignore warnings appearing during osprey installation. Most of these are thrown by libraries being used. You can always review the warnings in case the installation is not successful.

#### Option A (Recommended)

1. Scaffold an aplication by using the [Osprey-CLI](https://github.com/mulesoft/osprey-cli).
2. Check the resulting directories structure.
3. If you are working with an empty RAML file, you need to start writing it.
4. Find /[output_folder]/src/app.js to start registering resources ([check this under "Key Concept"](https://github.com/mulesoft/osprey/edit/master/README.md#resources-registration) section on this same document).

#### Option B
You can check the [example](https://github.com/mulesoft/osprey/tree/master/examples) included on Osprey to see a fully functional application, and try to create one from the scratch.

#### Run your Osprey application
From your terminal run:
`grunt` (recommended: It will set up the proper listeners so changes in the code are automatically refreshed in runtime).

**OR** you can always run: `node src/app.js`

##### Accessing the API Console
Open a browser and navigate to http://localhost:3000/api/console/ to display the API Console.

### Key concepts
No matter which option you go through, it's imporant for you to recognize the following sections in your code.

#### Osprey Initialization
You can intialize Osprey as follow:
```javascript

api = osprey.create('/api', app, {
  ramlFile: path.join(__dirname, '/assets/raml/api.raml'),
  enableConsole: true,
  enableMocks: true,
  enableValidations: true,
  logLevel: 'debug'
});
```
#####Options
* `/api` represents the basePath of the API
* `app` represents the reference of an express App

#####Parameters
| Name         | Default Value  | Description  |
|:------------------|:---------------|:---------------|
| ramlFile          | null           | Indicates where the RAML file is being stored|
| enableConsole     | true           | Enables or disables the [API console](https://github.com/mulesoft/api-console) |
| consolePath       | /console       | Defines the url for the API-console relative to the apiPath |
| enableMocks       | true           | Enables or disables the mocks routes |
| enableValidations | true           | Enables or disables the validations |
| exceptionHandler  | {}             | Gives you the possibility to reuse exception handlers|
| logLevel          | off            | Sets the logging level. ['off', 'info', 'debug'] |

#### Resources registration
Register a resource is as easy as follow:
```javascript
api.get('/teams/:teamId', function(req, res) {
  //// Your business logic here!
  res.send({ name: 'test' })
});
```

`osprey.get` is always relative to the basePath defined in `osprey.create`.

#####Other supported methods

* api.get
* api.post
* api.put
* api.delete
* api.head
* api.patch

#### Exception Handling

Osprey gives you the posibility of handling exceptions in a very reusable way.

First you have to setup the exceptionHandler module.

```javascript
api = osprey.create('/api', app, {
  ramlFile: path.join(__dirname, '/assets/raml/api.raml'),
  exceptionHandler: {
    InvalidUriParameterError: (err, req, res) ->
      // Overwriting the default implementation
      res.send 400
    CustomError: (err, req, res) ->
      //// Do something here!
      res.send 400
  }
});
```

If a resource throws an error of type CustomError, the exception handler module will handle it.

```javascript
api.get('/teams', function (req, res) {
  throw new CustomError 'some exception'
});
```
##### Default Errors
| Name                       | HTTP Status| Description  |
|:---------------------------|:----|:---------------|
| InvalidAcceptTypeError     | 406 | It will be thrown when the Accept type is not supported by the API |
| InvalidContentTypeError    | 415 | It will be thrown when the Content type is not supported by the API |
| InvalidUriParameterError   | 400 | It will be thrown if a URI parameter is invalid according to the validation rules |
| InvalidFormParameterError  | 400 | It will be thrown if a Form parameter is invalid according to the validation rules |
| InvalidQueryParameterError | 400 | It will be thrown if a Query parameter is invalid according to the validation rules |
| InvalidHeaderError         | 400 | It will be thrown if a Header is invalid according to the validation rules |
| InvalidBodyError           | 400 | It will be thrown if a body is invalid according to the validation schemas |

#### Validations

You can enable or disable validations by using the option `enableValidations` in `osprey.create`.

##### Supported Validations

* Form Parameters
* Query Parameters
* URI Parameters
* Headers
* JSON Schema
* XML Schema

###### Notes

In order to support XML schema validation, you have to setup the following middleware in your application
[express-xml-bodyparser](https://www.npmjs.org/package/express-xml-bodyparser).

#### Example



```javascript
  var express = require('express');
  var path = require('path');
  var osprey = require('osprey');

  var app = module.exports = express()

  app.use(express.bodyParser());
  app.use(express.methodOverride());
  app.use(express.compress());
  app.use(express.logger('dev'));

  app.set('port', process.env.PORT || 3000));

  var api = osprey.create('/api', app, {
    ramlFile: path.join(__dirname, '/assets/raml/api.raml')
  });

  api.get('/resource', function(req, res) {
    //// Your business logic here!
  });

  if (!module.parent) {
    var port = app.get('port');
    app.listen(port);
    console.log('listening on port ' + port);
  }
```
