# quantm-instrumentation
A modification of `opentelemtery-instrumentation-http` package.

# Steps to make it work:
Compiling the package:
```
$ git clone https://github.com/zinh88/quantm-instrumentation
$ cd quantm-instrumentation
$ npm i
$ npm run compile
```
Make a directory for a node app (`my-app`) and install this package:
```
$ npm install --save /path/to/quantm-instrumentation
```
Create a file `instrumentation.js` to initialize the Node SDK in the `my-app` directory. Put `QuantmHttpInstrumentation` in the instrumentations and add the `csID` from `process.env.QTM_CHANGESETID` etc in its config. For example:
```js
// instrumentation.js
const { NodeSDK } = require("@opentelemetry/sdk-node")
const { ConsoleSpanExporter } = require("@opentelemetry/sdk-trace-node")
const { Resource } = require("@opentelemetry/resources")
const { SemanticResourceAttributes } = require("@opentelemetry/semantic-conventions")
const { CompositePropagator, W3CTraceContextPropagator, W3CBaggagePropagator } = require("@opentelemetry/core")
const { QuantmHttpInstrumentation } = require("quantm-instrumentation");

const csID = process.env.QTM_CHANGESETID || "000"
const wID = process.env.QTM_WORKLOADID || "000"


const sdk = new NodeSDK({
    resource: new Resource({
        [ SemanticResourceAttributes.SERVICE_NAME ]: "servicename",
        [ SemanticResourceAttributes.SERVICE_VERSION ]: "0.1.0",
    }),
    traceExporter: new ConsoleSpanExporter(),
    textMapPropagator: new CompositePropagator({
        propagators: [
            new W3CTraceContextPropagator(),
            new W3CBaggagePropagator(),
        ]
    }),
    instrumentations: [
        new QuantmHttpInstrumentation({
            quantmConfig: {
                changesetID: csID,
                workloadID: wID
            }
        })
    ]

})
sdk.start()
...
```

Run your app (maybe called `app.js`) with `require` and `instrumentation.js`:
```
$ node --require ./instrumentation.js app.js
```
