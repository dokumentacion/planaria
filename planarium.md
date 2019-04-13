# How to Create a Planarium

While [Planaria](/planaria) deals with the Bitcoin facing side of things, Planarium deals with external facing features such as:

- HTTP API endpoint
- Realtime SSE (Server Sent Events) endpoint

These endpoints can be consumed by both **humans** and **machines**.

## The Interface

No matter how cool a state machine is, if there's no way to observe what's going on, it would be meaningless.

Planarium is the **interface through which the outside world can read from Planaria**

![scope](scope.jpg)

## Anatomy of a Planarium

Every Planarium Service is a **Single node.js module object** that follows the following convention:

```
module.exports = {
  planarium: ...,
  query: {
    web: ...,
    api: ...,
    log: ...
  },
  socket: {
    web: ...
    api: ...,
    topics: ...
  },
  transform: {
    request: ...,
    response: ...
  },
  url: ...,
  port: ...
}
```

- `planarium`: planarium version. currently '0.0.1'.
- `query`: the HTTP API description
	- `web`: The default query to display in the explorer
	- `api`: The HTTP API configuration
		- `timeout`: how many milliseconds to wait before timing out a query
		- `sort`: The default sort order of a query
		- `concurrency`: The default level of concurrency allowed for certain queries (So far only supports `aggregate`)
	- `log`: logging option (verbose if true)
- `socket`: The SSE endpoint configuration
	- `web`: The default query to subscribe to in the socket explorer
	- `api`: doesn't do anything for now. reserved.
	- `topics`: A set of Planaria topics to subscribe to. You can publish any message to any topics from the planaria code. And planarium can listen to them (but not just Planarium, any other apps can listen to them through zeromq)
- `transform`: A pair of request/response transform logic that lets you create [viratual attributes](https://docs.oracle.com/en/middleware/idm/unified-directory/12.2.1.3/oudcr/virtual-attribute.html) By using this feature you can support all kinds of additional virtual attributes WITHOUT having to store them in the database, as long as these virtual attributes can be derived/computed from the minimal dataset stored in the database. For example, BitDB's `h-prefix` attributes which represent hex encoded push data are all virtual computed attributes, they don't exist on the database, but is computed at query time from the `b-prefixed` (base64 encoded) attributes which ARE stored on the database.
    - `request`:  a function to transform incoming [Bitquery](https://docs.bitdb.network/docs/query_v3) request into a different format, which then can be queried against the actual Planaria state machine database.
    - `response`: a function to transform the query result to present to the API consumer.

## Example

```
var bcode = require('bcode')
var timeago = require("timeago.js");
module.exports = {
  planarium: '0.0.1',
  query: {
    web: {
      v: 3,
      q: { find: {}, limit: 20 }
    },
    api: {
      timeout: 50000,
      sort: { "timestamp": -1 },
      concurrency: { aggregate: 7 },
    },
    log: true
  },
  socket: {
    web: {
      v: 3,
      q: { find: {} }
    },
    api: {},
    topics: ["t"]
  },
  transform: {
    request: bcode.encode,
    response: function(items) {
      let i = bcode.decode(items)
      if (Array.isArray(i)) {
        i.forEach(function(item) {
          item.timeago = timeago.format(item.timestamp)
        })
      } else {
        i.timeago = timeago.format(i.timestamp)
      }
      return i
    }
  },
  url: "mongodb://localhost:27020",
  port: 3000,
}
```
