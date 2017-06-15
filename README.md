Base router for your JS framework or frameworkless app

[![Build Status](https://travis-ci.org/apsavin/router-base.svg?branch=master)](https://travis-ci.org/apsavin/router-base) [![Coverage Status](https://coveralls.io/repos/apsavin/router-base/badge.svg)](https://coveralls.io/r/apsavin/router-base) [![NPM version](https://badge.fury.io/js/router-base.svg)](http://badge.fury.io/js/router-base) [![Bower version](https://badge.fury.io/bo/router-base.svg)](http://badge.fury.io/bo/router-base)

It's abstract and knows nothing about http. It's just matches and generates urls.

##How to include it in my app or framework?

Router supports:

1. [node.js](http://nodejs.org) modules,
2. [requirejs](http://requirejs.org) modules
3. and, of course, awesome [ym](https://github.com/ymaps/modules) modules.
4. You can just use the `<script>` tag also, RouterBase will export the global variable then.

You can install it with npm:

```bash
$ npm install router-base
```

or bower:

```bash
$ bower install router-base
```

##How to use it?

###Very basic example

At first, you need to create an instance:

```javascript
var myRouter = new RouterBase({

    routes: myRoutes // routes config

});
```

Where routes config is an Array with objects, each object is a configuration for a route.
For example:

```javascript
var myRoutes = [{
    id: 'simplest_route',
    path: '/a/route'
}];

var myRouter = new RouterBase({
    routes: myRoutes
});

// You can generate routes by id:
myRouter.generate('simplest_route'); 
// returns '/a/route'

// You can find a route by path and method:
myRouter.match({path: '/a/route', method: 'GET'}); 
// returns {id: 'simplest_route', parameters: {}, definition: { id: 'simplest_route',
                                                                   path: '/a/route',
                                                                   defaults: {},
                                                                   requirements: {},
                                                                   host: undefined,
                                                                   methods: [ 'GET', 'POST', 'PUT', 'DELETE' ],
                                                                   schemes: [ 'http', 'https' ] } }
}

// You can get full info about a route by id:
myRouter.getRouteInfo('simplest_route');
// for answer see [tests](https://github.com/apsavin/router-base/blob/master/test/data/getRouteInfo-data.js#L4-L41)
```

You can use access to the `definition` if you want to set additional fields to the route.
For example, you can mark route as secure specifying `secure: true` in the route config
and then check this field like this:

```javascript
var route = myRouter.match({path: '/a/route', method: 'GET'}); 

if (route && route.definition.secure) // do something

```

I will omit the `definition` field in the next examples.

###Beautiful urls examples

####You can use named parameters in paths of the routes.
```javascript
var myRoutes = [{
    id: 'route_with_parameter_in_path',
    path: '/some/path/{parameter}'
}];

myRouter.generate('route_with_parameter_in_path'); 
// will throw an Error, because parameter is needed for the route
myRouter.generate('route_with_parameter_in_path', {parameter: 1}); 
// returns '/some/path/1'
myRouter.generate('route_with_parameter_in_path', {parameter: 'value'}); 
// returns '/some/path/value'

myRouter.match({path: '/some/path/', method: 'GET'}); 
// returns null
myRouter.match({path: '/some/path/to', method: 'GET'}); 
// returns {id: 'route_with_parameter_in_path', parameters: {parameter: 'to'}, definition: {...}}
```
####Optional parameters in paths
```javascript
var myRoutes = [{
    id: 'route_with_parameter_in_path',
    path: '/some/path/{parameter}',
    defaults: {parameter: 1}
}];

myRouter.generate('route_with_parameter_in_path'); 
// returns '/some/path'
myRouter.generate('route_with_parameter_in_path', {parameter: 1}); 
// returns '/some/path/1'
myRouter.generate('route_with_parameter_in_path', {parameter: 'value'}); 
// returns '/some/path/value'

myRouter.match({path: '/some/path', method: 'GET'}); 
// returns  {id: 'route_with_parameter_in_path', parameters: {parameter: 1}, definition: {...}}
myRouter.match({path: '/some/path/to', method: 'GET'}); 
// returns {id: 'route_with_parameter_in_path', parameters: {parameter: 'to'}, definition: {...}}
```
####Restricted parameters in paths
```javascript
var myRoutes = [{
    id: 'route_with_parameter_in_path',
    path: '/some/path/{parameter}',
    defaults: {parameter: 1},
    requirements: {parameter: '\\d+'}
}];

myRouter.generate('route_with_parameter_in_path'); 
// returns '/some/path'
myRouter.generate('route_with_parameter_in_path', {parameter: 1}); 
// returns '/some/path/1'
myRouter.generate('route_with_parameter_in_path', {parameter: 'value'}); 
// throws an Error, because parameter is not numeric

myRouter.match({path: '/some/path', method: 'GET'}); 
// returns {id: 'route_with_parameter_in_path', parameters: {parameter: 1}, definition: {...}}
myRouter.match({path: '/some/path/to', method: 'GET'}); 
// returns null
```
####Generate custom query string
```javascript
var myRoutes = [{
    id: 'route_with_parameters',
    path: '/some/path/{parameter}'
}];

var myRouter = new RouterBase({
    routes: myRoutes,
    generateQuery: function (params) {
        var query = '';
        
        // generate custom query (in this example, convert array parameters as separate string parameters with brackets)
        
        return query;
    }
});

myRouter.generate('route_with_parameters', {parameter: 1, fruits: ['apple', 'banana']}); 
// returns '/a/route/1?fruits[]=apple&fruits[]=banana'
```

##All possible routes parameters

1. id - String, required. You can use it to link a route to your controller.
2. path - String, required. Can include named parameters in `{parameterName}` form.
3. host - String, optional. Can include named parameters in `{parameterName}` form (For example, '{sub}.example.com').
4. defaults - Object, optional. Keys are parameters names, values are parameters default values.
5. requirements - Object, optional. You can use it to restrict parameters. Keys are parameters names, values are strings. Strings from values are for regular expressions, router uses it to test parameters.
6. methods - Array, optional. For example, `['GET']` to allow only `GET` methods.
7. schemes - Array, optional. For example, `['https']` to force https.

##RouterBase parameters

1. routes - an Array of routes configs, the only required parameter
2. generateQuery - overrides default string query generate with your own custom function
3. defaultMethods - what methods available for routes, `['GET', 'POST', 'PUT', 'DELETE']` by default
4. defaultSchemes - what schemes available for routes, `['http', 'https']` by default
5. symbols.parametersDelimiters. By default, `.` and `/` can be used as parameters delimiters in paths
6. symbols.parameterStart, default value is '\{'
7. symbols.parameterMatcher, default value is '.*?'
8. symbols.parameterEnd, default value is '\}'

##Where is tests?
Of course, in `test` folder. Use `npm test` to run.
