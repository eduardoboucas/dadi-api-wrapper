# DADI API wrapper

> A high-level library for interacting with DADI API

[![npm (scoped)](https://img.shields.io/npm/v/@dadi/api-wrapper.svg?maxAge=10800&style=flat-square)](https://www.npmjs.com/package/@dadi/api-wrapper)
![coverage](https://img.shields.io/badge/coverage-93%25-brightgreen.svg?style=flat-square)
[![build](http://ci.dadi.technology/dadi/api-wrapper/badge?branch=feature/test-suite&service=shield)](http://ci.dadi.technology/dadi/api-wrapper)
[![JavaScript Style Guide](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat-square)](http://standardjs.com/)

## Overview

[DADI API](https://github.com/dadi/api) is a high performance RESTful API layer designed in support of API-first development and the principle of COPE.

This library provides a high-level abstraction of the REST architecture style, exposing a set of chainable methods that allow developers to compose complex read and write operations using a simplistic and natural syntax.

## Getting started

1. Install the `@dadi/api-wrapper` module:

   ```shell
   npm install @dadi/api-wrapper --save
   ```

2. Add the library and configure the API settings:

   ```js
   var DadiAPI = require('@dadi/api-wrapper');
   var api = new DadiAPI({
     uri: 'http://api.example.com',
     port: 80,
     credentials: {
       clientId: 'johndoe',
       secret: 'f00b4r'
     },
     version: 'vjoin',
     database: 'testdb'
   });
   ```

3. Make a query:

   ```js
   // Example: getting all documents where `name` contains "john" and age is greater than 18
   api.in('users')
       .whereFieldContains('name', 'john')
       .whereFieldIsGreaterThan('age', 18)
       .find()
       .then(function (response) {
      	 // Use documents here
       });
   ```

## Methods

Each query consists of a series of chained methods to form the request, always terminated by an operation method. There are 5 terminating operations that return a Promise with the result of one or more requests to the database: [`create()`](#create), [`delete()`](#delete), [`find()`](#find), [`apply()`](#apply) and [`update()`](#update).

These operations (with the exception of `create()`) can make use of a series of [filtering methods](#filters) to create the desired subset of documents to operate on.

### Operations

#### `.apply(callback)`

Updates a list of documents with the result of individually applying `callback` to them.

```js
api.in('users')
   .whereFieldExists('gender')
   .apply(function (document) {
      document.name = (document.gender === 'male') ? ('Mr ' + document.name) : ('Mrs ' + document.name);

      return document;
   });
```

#### `.create()`

Creates a document.

```js
// Example
api.in('users')
   .create({
      name: 'John Doe',
      age: 45,
      address: '123 Fake St'
   })
   .then(function (doc) {
      console.log('New document:', doc);
   })
   .catch(function (err) {
      console.log('! Error:', err);
   });
```

#### `.delete()`

Deletes one or more documents.

```js
api.in('users')
   .whereFieldDoesNotExist('name')
   .delete();
```

#### `.find(options)`

Returns a list of documents.

```js
api.in('users')
   .whereFieldIsGreaterThan('age', 21)
   .useFields(['name', 'age'])
   .find();
```

`options` is one of the following:

- `extractResults` (Boolean): Selects whether just the results array should be returned, rather than the entire API response.
- `extractMetadata` (Boolean): Selects whether just the metadata object should be returned, rather than the entire API response.

#### `.getConfig()`

Gets the config for a collection or for the API.

```js
// Gets the collection config
api.in('users')
   .getConfig();

// Gets the API config
api.getConfig();
```

#### `.getStats()`

Gets collection stats.

```js
api.in('users')
   .getStats();
```

#### `.setConfig()`

Sets the config for a collection or for the API.

```js
// Sets the collection config
var collectionConfig = {};

api.in('users')
   .setConfig(collectionConfig);

// Sets the API config
var apiConfig = {};

api.setConfig(apiConfig);
```

#### `.update(update)`

Updates a list of documents.

```js
api.in('users')
   .whereFieldIsLessThan('age', 18)
   .update({
      adult: false
   });
```

### Filters

Filtering methods are used to create a subset of documents that will be affected by subsequent operation methods.

#### `.goToPage(page)`

Defines the page of documents to be used.

```js
// Example
api.goToPage(3);
```

#### `.limitTo(limit)`

Defines a maximum number of documents to be retrieved.

```js
// Example
api.limitTo(10);
```

#### `.sortBy(field, order)`

Selects a field to sort on and the sort direction. Order defaults to ascending (`asc`).

```js
// Example
api.sortBy('age', 'desc');
```

#### `.useFields(fields)`

Selects the fields to be returned in the response. Accepts array format.

```js
// Example
api.useFields(['name', 'age']);
```

#### `.where(query)`

Filters documents using a MongoDB query object or a Aggregation Pipeline array. The methods above are ultimately just syntatic sugar for `where()`. This method can be used for complex queries that require operations not implemented by any other method.

```js
// Example
api.where({name: 'John Doe'});
```

#### `.whereFieldBeginsWith(field, text)`

Filters documents where `field` begins with `text`.

```js
// Example
api.whereFieldBeginsWith('name', 'john');
```

#### `.whereFieldContains(field, text)`

Filters documents where `field` contains `text`.

```js
// Example
api.whereFieldContains('name', 'john');
```

#### `.whereFieldDoesNotContain(field, text)`

Filters documents `field` does not contain `text`.

```js
// Example
api.whereFieldDoesNotContain('name', 'john');
```

#### `.whereFieldEndsWith(field, text)`

Filters documents where field starts with `text`.

```js
// Example
api.whereFieldEndsWith('name', 'john');
```

#### `.whereFieldExists(field)`

Filters documents that contain a field.

```js
// Example
api.whereFieldExists('name');
```

#### `.whereFieldDoesNotExist(field)`

Filters documents that do not contain a field.

```js
// Example
api.whereFieldDoesNotExist('address');
```

#### `.whereFieldIsEqualTo(field, value)`

Filters documents where `field` is equal to `value`.

```js
// Example
api.whereFieldIsEqualTo('age', 53);
```

#### `.whereFieldIsGreaterThan(field, value)`

Filters documents where `field` is greater than `value`.

```js
// Example
api.whereFieldIsGreaterThan('age', 18);
```

#### `.whereFieldIsGreaterThanOrEqualTo(field, value)`

Filters documents where `field` is greater than or equal to `value`.

```js
// Example
api.whereFieldIsGreaterThanOrEqualTo('age', 19);
```

#### `.whereFieldIsLessThan(field, value)`

Filters documents where `field` is less than `value`.

```js
// Example
api.whereFieldIsLessThan('age', 65);
```

#### `.whereFieldIsLessThanOrEqualTo(field, value)`

Filters documents where `field` is less than or equal to `value`.

```js
// Example
api.whereFieldIsLessThanOrEqualTo('age', 64);
```

#### `.whereFieldIsOneOf(field, matches)`

Filters documents where the value of `field` is one of the elements of `matches`.

```js
// Example
api.whereFieldIsOneOf('name', ['John', 'Jack', 'Peter']);
```

#### `.whereFieldIsNotEqualTo(field, value)`

Filters documents where `field` is not equal to `value`.

```js
// Example
api.whereFieldIsEqualTo('age', 53);
```

#### `.whereFieldIsNotOneOf(field, matches)`

Filters documents where the value of `field` is not one of the elements of `matches`.

```js
// Example
api.whereFieldIsNotOneOf('name', ['Mark', 'Nathan', 'David']);
```

#### `.withComposition(value)`

Defines whether nested documents should be resolved using composition. Defaults to `false`.

```js
// Example
api.withComposition();
api.withComposition(true); // same as above
api.withComposition(false);
```

### Other methods

#### `.fromEndpoint(endpoint)`

Selects a custom endpoint to use. Please note that unlike collections, custom endpoints do not have a standardised syntax, so it is up to the authors to make sure the endpoint complies with standard DADI API formats, or they will not function as expected.

```js
// Example
api.fromEndpoint('custom-endpoint');
```

#### `.in(collection)`

Selects the collection to use.

```js
// Example
api.in('users');
```

#### `.useDatabase(database)`

Selects the database to use. Overrides any database defined in the initialisation options, and is reset when called without arguments.

```js
// Example
api.useDatabase('testdb');
```

#### `.useVersion(version)`

Selects the version to use. Overrides any version defined in the initialisation options, and is reset when called without arguments.

```js
// Example
api.useVersion('1.0');
```

### Debug mode

With debug mode, you'll be able to see exactly how the requests made to API look like. This functionality is enabled by setting a `debug` property in the config:

```js
var DadiAPI = require('@dadi/api-wrapper');
var api = new DadiAPI({
  uri: 'http://api.example.com',
  port: 80,
  credentials: {
    clientId: 'johndoe',
    secret: 'f00b4r'
  },
  version: 'vjoin',
  database: 'testdb',
  debug: true
});
```

```
[@dadi/api-wrapper] Querying URI: http://api.example.com:80/vjoin/testdb/articles?count=0
```
