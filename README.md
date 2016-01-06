## Nefit Easy™ client library <a href="https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=9ZUNREGDSZ45L"><img style="vertical-align:middle" src="https://www.paypalobjects.com/en_GB/i/btn/btn_donate_SM.gif"></a>

Node.js client library for the [Nefit Easy](http://www.nefit.nl/consument/service/easy/easy) smart thermostat.

[Frequently Asked Questions](https://github.com/robertklep/nefit-easy-client/wiki)

Work in progress!

### Synopsis

The Nefit Easy™ smart thermostat is sold in The Netherlands by [Nefit](http://www.welkombijnefit.nl/nl), a company owned by [Robert Bosch GmbH](http://www.bosch.com/).

The Easy can be controlled through apps for Android and iOS, which communicate with a Bosch-hosted backend using the [XMPP](https://en.wikipedia.org/wiki/XMPP) protocol. This library aims to implement the communication protocol used between the apps and the backend.

### Installation

_This library requires Node.js 4.0.0 or later!_

#### Install it as a Node module

``` javascript
$ npm install nefit-easy-client
# Or, alternatively:
$ npm install nefit-easy-client -g
```

Use `-g/--global` if you want the CLI tool (`easy`) to be installed in a well-known bin directory. The CLI tool can be used to get/set parameters.

Run `easy --help` to get a list of supported options.

#### Checkout the repository (in case you want to run the example code)

``` javascript
$ git clone https://github.com/robertklep/nefit-easy-client
$ cd nefit-easy-client
$ npm install
```

### Projects using this library

* Pepijn Goossens' [Node-RED node](https://github.com/pepijng/node-red-contrib-nefit-easy)

### Using the HTTP server

The `easy` CLI tool provides a built-in HTTP server that acts as a bridge between HTTP and XMPP.

To run it (see `easy --help` for more options):

```
$ easy serve
```

By default, the server will be accessible through `http://127.0.0.1:3000/`.

The server isn't meant to be opened in a browser. Its intended use is to be called using tools like `curl` or `httpie`, or from other programs/scripts, through HTTP calls. The following examples will use `curl`.

The HTTP server provides two actions:

* retrieving an [endpoint](https://github.com/robertklep/nefit-easy-client/wiki/List-of-endpoints);
* writing data to an endpoint;

To retrieve an endpoint (for example, `/ecus/rrc/uiStatus`):
```
$ curl http://127.0.0.1:3000/bridge/ecus/rrc/uiStatus
```

To write data to an endpoint:
```
$ curl -XPOST http://127.0.0.1:3000/bridge/heatingCircuits/hc1/temperatureRoomManual -d '{"value":20}'
```

For `POST` requests, the data should be a [valid JSON string](http://jsonlint.com/). There is no input validation.

### Demo

There's an example script provided that will show temperatures (current, set and outside) and system pressure.

You need to provide the following information before the client can connect to the backend:

* The Easy serial number, which can be found on the manual that was supplied with the device;
* The access key (a 16-character code, also on the manual);
* The application password (set the first time you run the iOS/Android app);

The example script will read these values from environment variables.

``` javascript
$ export NEFIT_SERIAL_NUMBER=...
$ export NEFIT_ACCESS_KEY=...
$ export NEFIT_PASSWORD=...
$ node example/current-status.js
Temperature is set to 16.8°C, current is 16.5°C.
Outside temperature is 10°C.
System pressure is 1.4 bar.
```

### Disclaimer

The implementation of this library is based on reverse-engineering the communications between the apps and the backend, plus various other bits and pieces of information. It is _not_ based on any official information given out by Nefit/Bosch, and therefore there are no guarantees whatsoever regarding the safety of your devices and/or their settings, or the accuracy of the information provided.

## API

### General information

All (asynchronous) methods return a ([bluebird](http://bluebirdjs.com/)) promise that resolves to a plain object.

`nefit-easy-client` uses [`debug`](https://github.com/visionmedia/debug) to provide some debug logging:

```
$ env DEBUG=nefit-easy-client node your-app.js
```

#### Constructor

```
const NefitEasyClient = require('nefit-easy-client');
const client          = NefitEasyClient({
  serialNumber : NEFIT_SERIAL_NUMBER,
  accessKey    : NEFIT_ACCESS_KEY,
  password     : NEFIT_PASSWORD,
});
```

#### System status

`client.status() : Promise`

Example:
```
{
  "user mode"                   : "clock",
  "in house temp"               : 18.3,
  "in house status"             : "ok",
  "boiler indicator"            : "No",
  "control"                     : "room",
  "temp override duration"      : 0,
  "current switchpoint"         : "0",
  "ps active"                   : false,
  "fp active"                   : false,
  "temp override"               : true,
  "holiday mode"                : false,
  "boiler block"                : null,
  "boiler lock"                 : null,
  "boiler maintainance"         : null,
  "temp setpoint"               : 17.5,
  "temp override temp setpoint" : 17.5,
  "temp manual setpoint"        : 17.5,
  "hed enabled"                 : null,
  "hed device at home"          : null,
  "outdoor temp"                : 13,
  "outdoor source type"         : "virtual"
}
```

#### System pressure

`client.pressure() : Promise`

Example:
```
{
  "pressure" : 1.4,
  "unit"     : "bar"
}
```

#### System location

`client.location() : Promise`

Example:
```
{
  "lat" : 52.2425755,
  "lng" : 6.1792625
}
```

#### Program data (active program and switchpoint data)

`client.program() : Promise`

Example:
```
{
  "active"   : 1,       // Currently active program
  "program1" : [
    {
      "dow"  : 0,       // 0 = Sunday, 6 = Saturday
      "time" : "07:00",
      "temp" : 20
    },
    {
      "dow"  : 0,
      "time" : "23:00",
      "temp" : 16
    },
    ...
  ],
  "program2" : [
    ...
  ]
}
```

#### Low-level `GET` operation

`client.get(uri : String) : Promise`

This allows retrieving specific URI's from the backend.

A non-exhaustive list of URI's can be found [here](https://github.com/robertklep/nefit-easy-client/wiki/List-of-endpoints).

#### Low-level `PUT` operation

`client.put(uri : String, data) : Promise`

This allows writing values to specific URI's.
