# OpenWhisk Client for JavaScript

[![Build Status](https://travis-ci.org/apache/incubator-openwhisk-client-js.svg?branch=master)](https://travis-ci.org/apache/incubator-openwhisk-client-js)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![codecov](https://codecov.io/gh/apache/incubator-openwhisk-client-js/branch/master/graph/badge.svg)](https://codecov.io/gh/apache/incubator-openwhisk-client-js)
[![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)

JavaScript client library for the [Apache OpenWhisk](https://github.com/apache/incubator-openwhisk) platform.
Provides a wrapper around the [OpenWhisk APIs](https://github.com/apache/incubator-openwhisk/blob/fb001afa237476eda0c0f6494ee92702e5986538/core/controller/src/main/resources/apiv1swagger.json) (Swagger JSON).

## installation

```bash
$ npm install openwhisk
```

## usage

### within openwhisk platform

This client library can use environment parameters to automatically configure the authentication credentials, platform endpoint and namespace. These parameters are defined within the Node.js runtime environment in OpenWhisk. Unless you want to override these values, you can construct the client instance without further configuration.

```javascript
var openwhisk = require('openwhisk');

function action() {
  var ow = openwhisk();
  return ow.actions.invoke('sample')
}

exports.main = action
```

_All methods return a Promise resolved asynchronously with the results. Failures are available through the catch method._

```javascript
ow.resource.operation()
  .then(function () { /* success! */ })
  .catch(function (err) { /* failed! */ })
```

Users can override default constructor parameters by passing in explicit options as shown in the example below.

_**Please note**: Due to [an issue](https://github.com/openwhisk/openwhisk/issues/1751) with the Node.js runtime in OpenWhisk, environment variables used by the constructor are not available until the invocation function handler is called. If you want to define the client instance outside this function, you will need to manually pass in the constructor options ._

```javascript
var openwhisk = require('openwhisk');
// DOES NOT WORK! Environment parameters not set.
var ow = openwhisk();

function action() {
  return ow.actions.invoke('sample')
}

exports.main = action
```

### outside openwhisk platform

```javascript
var openwhisk = require('openwhisk');
var options = {apihost: 'openwhisk.ng.bluemix.net', api_key: '...'};
var ow = openwhisk(options);
ow.actions.invoke('sample').then(result => console.log(result))
```

### constructor options

_Client constructor supports the following mandatory parameters:_

- **apihost.** Hostname and optional port for openwhisk platform, e.g. `openwhisk.ng.bluemix.net` or `my_whisk_host:80`. Used with API URL template `${protocol}://${apihost}/api/v1/`. If port is missing or port value is 443 in the apihost string, protocol is HTTPS. Otherwise, protocol is HTTP.
- **api_key.** Authorisation key for user account registered with OpenWhisk platform.

*Client constructor supports the following optional parameters:*

- **api.** Full API URL for OpenWhisk platform, e.g. `https://openwhisk.ng.bluemix.net/api/v1/`. This value overrides `apihost` if both are present.
- **namespace**. Namespace for resource requests, defaults to `_`.
- **ignore_certs**. Turns off server SSL/TLS certificate verification. This allows the client to be used against local deployments of OpenWhisk with a self-signed certificate. Defaults to false.
- **apigw_token**. API Gateway service authentication token. This is mandatory for using an external API Gateway service, rather than the built-in api gateway.
- **apigw_space_guid**. API Gateway space identifier. This is optional when using an API gateway service, defaults to the authentication uuid.

### environment variables

Client constructor will read values for the `apihost`, `namespace`, `api_key`, `ignore_certs`, `apigw_token` and `apigw_space_guid` options from the environment if the following parameters are set. Explicit options have precedence over environment values.

- *__OW_API_HOST*
- *__OW_NAMESPACE*
- *__OW_API_KEY*
- *__OW_IGNORE_CERTS*
- *__OW_APIGW_TOKEN*
- *__OW_APIGW_SPACE_SUID*



## Examples

### invoke action, blocking for result

```javascript
const name = 'reverseWords'
const blocking = true, result = true
const params = {msg: 'these are some words to reverse'}

ow.actions.invoke({name, blocking, result, params}).then(result => {
  console.log('here is the reversed string', result.reversed)
}).catch(err => {
  console.error('failed to invoke actions', err)
})
```

### fire trigger

```javascript
const name = 'eventTrigger'
const params = {msg: 'event trigger message string'}
ow.triggers.invoke({name, params}).then(result => {
  console.log('trigger fired!')
}).catch(err => {
  console.error('failed to fire trigger', err)
})
```

### create action from source file

```javascript
const name = 'reverseWords'
const action = fs.readFileSync('source.js', {encoding: 'utf8'})

ow.actions.create({name, action}).then(result => {
  console.log('action created!')
}).catch(err => {
  console.error('failed to create action', err)
})
```

### create action from zip package

```javascript
const name = 'reverseWords'
const action = fs.readFileSync('package.zip')

ow.actions.create({name, action}).then(result => {
  console.log('action created!')
}).catch(err => {
  console.error('failed to create action', err)
})
```

### retrieve action resource

```javascript
const name = 'reverseWords'
ow.actions.get(name).then(action => {
  console.log('action resource', action)
}).catch(err => {
  console.error('failed to retrieve action', err)
})
```

### chaining calls

```javascript
ow.actions.list()
  .then(actions => ow.actions.invoke(actions))
  .then(result => { /* ... */ })
```

### list packages

```javascript
ow.packages.list().then(packages => {
  packages.forEach(package => console.log(package.name))
}).catch(err => {
  console.error('failed to list packages', err)
})
```

### update package parameters

```javascript
const name = 'myPackage'
const package = {
  parameters: [
    {key: "colour", value: "green"},
    {key: "name", value: "Freya"}
  ]
}

ow.packages.update({name, package}).then(package => {
  console.log('updated package:', package.name)
}).catch(err => {
  console.error('failed to update package', err)
})
```

### create trigger feed from alarm package

```javascript
// alarmTrigger MUST already exist in default namespace
const params = {cron: '*/8 * * * * *', trigger_payload: {name: 'James'}}
const name = '/whisk.system/alarms/alarm'
const trigger = 'alarmTrigger'
ow.feeds.create({name, trigger, params}).then(package => {
  console.log('alarm trigger feed created')
}).catch(err => {
  console.error('failed to create alarm trigger', err)
})
```



## API Details

### resource identifiers + namespaces

When passing resource identifiers as parameters you can either use a short name, without an explicit namespace, or a fully-qualified identifier, including namespace and package details.

If the namespace is missing from the resource identifier, the client will use the namespace from configuration options following this ordering.

- `namespace` from method parameter options OR
- `namespace` from options passed into client constructor OR
- `namespace` from environment variable (`__OW_NAMESPACE`) OR
- default namespace: `_`

### list resources

```javascript
ow.actions.list()
ow.activations.list()
ow.triggers.list()
ow.rules.list()
ow.namespaces.list()
ow.packages.list()
```

Query parameters for the API calls are supported (e.g. limit, skip, count etc.) by passing an object with the named parameters as the first argument.

```javascript
ow.actions.list({skip: 100, limit: 50})
```

To count the number of resources without retrieving the resources you can use `count` query parameter.
```javascript
ow.actions.list({count:true})
```

The following optional parameters are supported:
- `namespace` - set custom namespace for endpoint

### retrieve resource

```javascript
ow.actions.get({name: '...'})
ow.activations.get({name: '...'})
ow.triggers.get({name: '...'})
ow.rules.get({name: '...'})
ow.packages.get({name: '...'})
ow.feeds.get({name: '...', trigger: '...'})
```

The following optional parameters are supported for all resource retrievals:
- `namespace` - set custom namespace for endpoint

Optional parameters for action resource retrievals are shown below:
- `code` - set to `true` or `false` depending on whether action code should be included or excluded respectively

This method also supports passing the `name` property directly without wrapping within an object.
```javascript
const name = "actionName"
ow.actions.get(name)
```

If you pass in an array for the first parameter, the `get` call will be executed for each array item. The function returns a Promise which resolves with the results when all operations have finished.

```javascript
ow.actions.get(["a", {name: "b"}])
```

### delete resource

```javascript
ow.actions.delete({name: '...'})
ow.triggers.delete({name: '...'})
ow.rules.delete({name: '...'})
ow.packages.delete({name: '...'})
ow.feeds.delete({name: '...', trigger: '...'})
```

The following optional parameters are supported:
- `namespace` - set custom namespace for endpoint

This method also supports passing the `name` property directly without wrapping within an object.

```javascript
const name = "actionName"
ow.actions.delete(name)
```

If you pass in an array for the first parameter, the `delete` call will be executed for each array item. The function returns a Promise which resolves with the results when all operations have finished.

```javascript
ow.actions.delete(["a", {name: "b"}])
```

### invoke action

```javascript
ow.actions.invoke({name: '...'})
```

The `actionName` parameter supports the following formats: `actionName`, `package/actionName`, `/namespace/actionName`, `/namespace/package/actionName`.

If `actionName` includes a namespace, this overrides any other `namespace` properties.

The following optional parameters are supported:
- `blocking` - delay returning until action has finished executing (default: `false`)
- `result` - return function result (`obj.response.result`) rather than entire API result (default: `false`)
- `params` - JSON object containing parameters for the action being invoked (default: `{}`)
- `namespace` - set custom namespace for endpoint

This method also supports passing the `name` property directly without wrapping within an object.

```javascript
const name = "actionName"
ow.actions.invoke(name)
```

If you pass in an array for the first parameter, the `invoke` call will be executed for each array item. The function returns a Promise which resolves with the results when all operations have finished.

```javascript
ow.actions.invoke(["a", {name: "b", blocking: true}])
```

### create & update action

```javascript
ow.actions.create({name: '...', action: 'function main() {};'})
ow.actions.update({name: '...', action: 'function main() {};'})
```

The following mandatory parameters are supported:
- `name` - action identifier
- `action` - String containing JS function source code, Buffer [containing package action zip file](https://github.com/openwhisk/openwhisk/blob/master/docs/actions.md#packaging-an-action-as-a-nodejs-module) or JSON object containing full parameters for the action body

The following optional parameters are supported:
- `namespace` - set custom namespace for endpoint
- `params` - object containing default parameters for the action (default: `{}`)
- `annotations` - object containing annotations for the action (default: `{}`)
- `limits` - object containing limits for the action (default: `{}`)
- `kind` - runtime environment parameter, ignored when `action` is an object (default: `nodejs:default`)
- `version` - set semantic version of the action. If parameter is empty when create new action openwisk generate 0.0.1 value when update an action increase the patch version.

If you pass in an array for the first parameter, the `create` call will be executed for each array item. The function returns a Promise which resolves with the results when all operations have finished.

```javascript
ow.actions.create([{...}, {...}])
```

### fire trigger

```javascript
ow.triggers.invoke({name: '...'})
```

The following optional parameters are supported:
- `params` - JSON object containing parameters for the trigger being fired (default: `{}`)
- `namespace` - set custom namespace for endpoint

This method also supports passing the `name` property directly without wrapping within an object.

```javascript
const name = "actionName"
ow.triggers.invoke(name)
```

If you pass in an array for the first parameter, the `invoke` call will be executed for each array item. The function returns a Promise which resolves with the results when all operations have finished.

```javascript
ow.triggers.invoke(["a", {name: "b", blocking: true}])
```

### create & update trigger

```javascript
ow.triggers.create({name: '...'})
ow.triggers.update({name: '...'})
```

The following optional parameters are supported:
- `trigger` - JSON object containing parameters for the trigger body (default: `{}`)
- `namespace` - set custom namespace for endpoint

### create & update packages

```javascript
ow.packages.create({name: '...'})
ow.packages.update({name: '...'})
```

The following optional parameters are supported:
- `package` - JSON object containing parameters for the package body (default: `{}`)
- `namespace` - set custom namespace for endpoint

### create & update rule

```javascript
ow.rules.create({name: '...', action: '...', trigger: '...'})
ow.rules.update({name: '...', action: '...', trigger: '...'})
```

`trigger` and `action` identifiers will have the default namespace (`/_/`)
appended in the request, unless a fully qualified name is passed in
(`/custom_ns/action_or_trigger_name`).

The following optional parameters are supported:
- `namespace` - set namespace for rule

### enable & disable rule

```javascript
ow.rules.enable({name: '...'})
ow.rules.disable({name: '...'})
```

The following optional parameters are supported:
- `namespace` - set custom namespace for endpoint

### create & update feeds

```javascript
ow.feeds.create({feedName: '...', trigger: '...'})
ow.feeds.update({feedName: '...', trigger: '...'})
```

The following optional parameters are supported:
- `namespace` - set custom namespace for endpoint
- `params` - JSON object containing parameters for the feed being invoked (default: `{}`)

## api gateway

OpenWhisk supports a [built-in API gateway service](https://github.com/apache/incubator-openwhisk/blob/master/docs/apigateway.md) and external third-party providers.

This client library defaults to using the platform service. If the `apigw_token` parameter is passed into the client constructor, the implementation will switch to the [IBM Bluemix API Gateway](https://console.ng.bluemix.net/docs/openwhisk/openwhisk_apigateway.html#openwhisk_apigateway).

*The interface for managing routes through the library does not change between providers.*

### retrieve route

```javascript
ow.routes.get({basepath: '...'})
ow.routes.get({name: '...'})
```

*This method is a wrapper for the list method. It throws an error if the base path or name parameter is missing.*

### list routes

```javascript
ow.routes.list()
```

The following optional parameters are supported to filter the result set:
- `relpath` - relative URI path for endpoints
- `basepath` - base URI path for endpoints
- `name` - identifier for API
- `operation` - HTTP methods
- `limit` - limit result set size
- `skip` - skip results from index

*`relpath` is only valid when `basepath` is also specified. `name` and `basepath` cannot be used together.*

### delete routes

```javascript
ow.routes.delete({basepath: '...'})
ow.routes.delete({name: '...'})
```

The following optional parameters are supported to filter the result set:
- `relpath` - relative URI path for endpoints
- `operation` - HTTP methods

### add route
```javascript
ow.routes.create({relpath: '...', operation: '...', action: '...'})
```

*`action` supports normal (actionName) and fully-qualified (/namespace/actionName) formats.*

The following optional parameters are supported:
- `responsetype` - content type returned by web action, possible values: `html`, `http`, `json`, `text` and `svg` (default: `json`).
- `basepath` - base URI path for endpoints (default: `/`)
- `name` - identifier for API (default: `basepath`)
- `secure_key` - auth key for secure web action

### add route (swagger)

```javascript
ow.routes.create({swagger: '{...}'})
```

Swagger parameter must be a well-formed JSON string, containing a valid Swagger API definition, which follows the [OpenWhisk API Gateway route schema](https://github.com/apache/incubator-openwhisk-apigateway/blob/master/doc/v2/management_interface_v2.md#post-v2tenant_idapis).

*No other parameters are supported when creating the route from a JSON Swagger document.*

## Debugging

Setting an environment parameter (`DEBUG=needle`) will dump the HTTP requests from the client library and responses received to `stderr`.

```bash
DEBUG=needle node script.js
```

This parameter can also be set dynamically at runtime, provided this happens before the `openwhisk` module is required.

```javascript
process.env.DEBUG='needle';
var openwhisk = require('openwhisk');
```

## Development

### unit tests

```bash
$ npm test
```

### integration tests

*Please [see the instructions](https://github.com/openwhisk/openwhisk-client-js/tree/master/test/integration) for setting up the integration test environment prior to running these tests.*

```bash
$ npm run-script test-integration
```

**Note:** The test integration runs in secure mode by default, which means that all trusted signers must be present and available to the client process.
If your local environment is using self-signed certificates, you can use the following command to start the script in insecure mode:

`npm run test-integration -i`

This will disable SSL/TLS verification for all SSL communication.

Alternatively, you can run the `prepIntegrationTests.sh` script using guest credentials or by specifying specific credentials.
Run the script with openwhisk credentials:
```bash
$ ./test/integration/prepIntegrationTests.sh <your key in the form of ABCD:EFGH> <openwhisk instance hostname> <openwhisk namespace> <api gatewaytoken>
```
The `prepIntegrationTests.sh` script is designed to give you feedback if it detects a setting that is not correct on your machine. ex: `node 6 or above is not detected`

## Code-Coverage:

You can customize how comprehensive the tests are over the code, and generate reports to view the results by using
the provided `code-coverage` commands below.

**Note:** Ensure that you use guest credentials with the wsk CLI.

To compile down to ECMA5 run the following command:
1 `$ npm run code-coverage-build`

To generate combined reports of both the unit and integration tests, run the following command:
2 `$ npm run code-coverage-run <key> <host> <namespace> <token> <options>`

The report is viewable under `/coverage`. Click **`/coverage/index.html`** to view the full report.
