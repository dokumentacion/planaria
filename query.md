# Bitquery

Planaria comes with **a portable programmable query language** called Bitquery:

![bitquery](bitquery.png)

Because Planaria is essentially a framework for building any imaginable Bitcoin powered database, it is important to have a uniform query language that can query them all. And this is what Bitquery does.

# Overview

Bitquery is a combination of two query languages:

- [MongoDB Query Language](https://docs.mongodb.com/manual/tutorial/query-documents/): For querying the database
- [JQ](https://stedolan.github.io/jq/): For filtering and transforming the response before returning

Every bitquery consists of 3 top level attributes:

1. **v:** is for "version". (The current version is '3'.)
2. **q:** is for "query".
    - **find:** MongoDB query filter object. [Learn more about MongoDB find command](https://docs.mongodb.com/manual/reference/method/db.collection.find/)
    - **aggregate:** MongoDB aggregationg pipeline stages array. [Learn more about Mongodb aggregate stages](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#db.collection.aggregate)
    - **project:** MongoDB project operator for selectively returning attributes. [Learn more about Mongodb projection](https://docs.mongodb.com/manual/tutorial/project-fields-from-query-results/)
    - **sort:** MongoDB sort operator. [Learn more about Mongodb sort operator](https://docs.mongodb.com/manual/reference/method/cursor.sort/)
    - **limit:** MongoDB limit operator. Limit the number of results to return. [Leanr more about MongoDB limit operator](https://docs.mongodb.com/manual/reference/method/cursor.limit/)
    - **skip:** MongoDB skip operator. Combining the "limit" and "skip" you can implement pagination. [Learn more about MongoDB skip operator](https://docs.mongodb.com/manual/reference/method/cursor.skip/)
    - **db:** An array of the collections to query against. By default it runs the query on all existing collections for the API endpoint.
3. **r:** is for "response". This is the response processor.
    - **f:** is for "function". It's a single-line program for processing the DB response before sending back to the client. The supported language is [jq](https://en.wikipedia.org/wiki/Jq_(programming_language)), **a Turing complete, stack based JSON processing language.**

# Attributes

## q

### Native MongoDB query

The first step of Bitquery is the actual query.

1. You can declare the query under the `"q"` attribute of the query language.
2. Then the Bitquery engine translates this `"q"` object into a native MongoDB API call at runtime.

You can query anything that's been crawled by Planaria.

Here's an example:

```
{
  "v": 3,
  "q": {
    "find": {
      "$text": {
        "$search": "hello"
      },
      "out.h1": "6d02",
      "out.b2": "hello"
    },
    "skip": 5,
    "limit": 10,
    "sort": { "blk.i": 1 }
  }
}
```

The currently supported query API methods are: `find`, `aggregate`, `project`, `sort`, `limit`, and `skip`, but this can be extended according to demand.

To learn how to query BitDB, you can just think of this as a regular MongoDB instance.

- You can learn more about MongoDB queries [here](https://docs.mongodb.com/manual/tutorial/query-documents/)
- You can check out some example queries [here](quickstart_v3)

### Collection Specific Query

Using the `db` attribute, you can specify the collections to query against. If you don't specify, the query is executed against all existing collections on the API endpoint.

For example, if the machine contained two collections `c` and `u`, you can choose to query ONLY against `c` like this:

```
{
  "v": 3,
  "q": {
    "db": ["c"],
    "find": {}
  }
}
```

which would return the response in the following format:

```
{
  "c": [
    ...
  ]
}
```

or ONLY against `u`:

```
{
  "v": 3,
  "q": {
    "db": ["u"],
    "find": {}
  }
}
```

which would return the response in the following format:

```
{
  "u": [
    ...
  ]
}
```

or against both:

```
{
  "v": 3,
  "q": {
    "db": ["c", "u"],
    "find": {}
  }
}
```

which would result in:

```
{
  "c": [
    ...
  ],
  "u": [
    ...
  ]
}
```

---

## r

`r` is a response handler function.

Currently there is only one type of response handler, named **"f"** (Response processing **function**). Here's an example (Pay attention to the **".r.f"** object):

```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "534c5000", "out.s3": "GENESIS" },
    "limit": 20,
    "project": { "out.$": 1, "_id": 0 }
  },
  "r": {
    "f": "[.[] | .out[0] | { token_symbol: .s4, token_name: .s5, document_url: .s6} ]"
  }
}
```

Let's zoom in to the response processor function `r.f`:

```
"[.[] | .out[0] | { token_symbol: .s4, token_name: .s5, document_url: .s6} ]"
```

This is written in [jq](https://en.wikipedia.org/wiki/Jq_(programming_language)), a Turing complete data processing language.

![jq](jq.png)

And here's how to read the program:

1. All jq programs are stack based, therefore you read from left to right.
2. All jq programs assume an incoming input object and produces an output in the end.
3. The `|` (pipe) works like [unix pipes](https://en.wikipedia.org/wiki/Pipeline_(Unix)). It passes the computed output from the left side as input to the right.

So following these rules (learn more about the syntax [here](https://stedolan.github.io/jq/manual/#Basicfilters)), here's what the code says:

1. for each item in the array (`.[]`)
2. find the first output (`.out[0]`)
3. and extract out its `.s4`, `.s5`, and `.s6` attributes to assign them to the custom attributes `"token_symbol"`, `"token_name"`, and `"document_url"`
4. and the entire expression is wrapped inside the root `[ ]`, which means it turns the resulting list into a JSON array

And here's the result:

```
{
  "u": [{
    ...
  }],
  "c": [{
    "token_symbol": "TEST",
    "token_name": "TEST",
    "document_url": "bitcoinfiles:b86b4bcbab7cd787b1c893ca101250c8c467dbba4df229b118218bd8a9e85a92"
  }, {
    "token_symbol": "VOTE",
    "token_name": "An Election",
    "document_url": "bitcoinfiles:a90e59ef7ca66b25b6ba98d028198ae222a8229804c4b0b3bc0b1bafe104738a"
  }, {
    "token_symbol": "WuCash",
    "token_name": "Wu Tang Cash",
    "document_url": "http://wu.cash"
  }, {
    "token_symbol": "bb23n",
    "token_name": "bb23",
    "document_url": "bb23n.com"
  }, {
    "token_symbol": "DBOOK01",
    "token_name": "Digital Book Example",
    "document_url": "https://digitalbookexampletokenurl.com"
  }, {
    "token_symbol": ""
    "token_name": ""
    "document_url": ""
  }, {
    "token_symbol": "MTT",
    "token_name": "MyTestToken",
    "document_url": ""
  }]
}
```

> Note: The processing function gets executed for each database the query is run against.
>
> This means `"u"` and `"c"` in this case, but if you specify the db using the "q.db" attribute, the function will only be executed for that db collection

Notice how instead of returning all the raw attributes straight from the DB, the API now returns the response in the **EXACT format you want**:

As you can see, this effectively closes the loop in the entire query lifecycle by letting you create a query that describes **NOT only the request, but also the response**, which means a query is completely self-contained for instant usage.

> When we end up with a query language that is 100% self-contained for instant usage,
>
> It is no longer just a query.
>
> It is effectively an **Open API** (Application Programming Interface) for your application.

Let's go through the detailed benefits of this approach:

### Human Readable Response

The default attribute names may not be what you want to use for the end users, you may want to process the result before returning to the requesting application.

In case your DB attribute names are not very human readable, you probably want a more human readable attribute name for the protocol API, such as "title", "document_uri", "username", etc.

### Turing Complete Query Language

The processing function is written in [jq](https://en.wikipedia.org/wiki/Jq_(programming_language)), a Turing complete, stack based functional programming language. Bitquery parser uses jq internally to process the response, so any syntax supported by jq is automatically possible inside the response processing function.

> [Learn jq syntax](https://stedolan.github.io/jq/manual/#Basicfilters)

Here are the benefits:

1. **Powerful:** jq is Turing complete, meaning you can transform any JSON into any format you want. Using the stack based paradigm, jq provides full range of programming capabilities such as: loops, map, filter, variable assignment, etc.
2. **Efficient:** It offloads the work from the database by doing all the filtering outside of the DB engine AFTER the DB responds, but BEFORE the result is sent back to the client. This makes the query much more performant.
3. **Portable:** Most importantly, the language fits into JSON format since jq is a single line programming language (stack based).
4. **Programmable:** jq has been a popular language of choice for manipulating data in the command line interface. People use it along with other programs on their computer through the power of [unix piping](https://en.wikipedia.org/wiki/Pipeline_(Unix)), which makes it infinitely extensible and programmable.

### Create a meta API!

Now that it's possible to include the processing logic directly inside the query itself, you can create Meta APIs. Here's an example for [Genesis](https://planaria.network/@1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN):

**"Find 100 transactions with an output OP_RETURN script that starts with `0x6d02` (memo.cash). And then return only its block index, block time, and content."**

```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "6d02" },
    "limit": 100,
    "sort": {"blk.i": 1}
  },
  "r": {
    "f": "[.[] | { block: .blk.i?, timestamp: .blk.t?, content: .out[1].s2 }]"
  }
}
```

By the MongoDB query `"q"`, the first 100 transactions of Memo posts are returned. After filtered by the response processing function `r.f`, only the fileds of block index, block time and the 3rd push data `s2` of the 2nd output `.out[1]` are returned. 

However, the Memeo post is not necessarily inclueded in the 2nd output of a transaction. Furthermore, after the [Genisis upgrade](https://github.com/bitcoin-sv-specs/protocol/blob/master/updates/genesis-spec.md) of BSV at Feb 4th 2020, one transaction can have multiple outputs containing the OP_RETRUN code and subsequent data, which could all be Memo posts. So, a more robust query may be:

```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "6d02" },
    "limit": 100
  },
  "r": {
    "f": "[.[] | { block: .blk.i?, timestamp: .blk.t?, content: [.out |.[] |if .h1==\"6d02\" then .s2 else empty end] | .[] }]"
  }
}
```

More complex example, also using [Genesis](https://planaria.network/@1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN):

1. Find 100 OP_RETURN transactions that start with `0x6d02` (memo.cash)
2. Group by block index (blk.i)
3. Map the items to only return the message (.out[1].s2) and transaction hash (.tx.h)

```
{
  "v": 3,
  "q": {
    "db": ["c"],
    "find": { "out.h1": "6d02" },
    "limit": 100
  },
  "r": {
    "f": "[ group_by(.blk.h)[] | { blocks: { (.[0].blk.i | tostring): [.[] | {message: .out[1].s2, tx: .tx.h} ] } } ]"
  }
}
```

# Usage

The same query language is used for both the "pull" (Query) and the "push" (Socket) APIs.

## Query

For the HTTP[S] API, you simply make a GET request with the Bitquery JSON object as a payload.

Then it will return the corresponding result from the node.

```html
<html>
<body>
<pre>loading...</pre>
<script>
var query = {
  v: 3,
  q: { find: {}, limit: 5 }
};
var b64 = btoa(JSON.stringify(query));
var url = "https://genesis.bitdb.network/q/1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN/" + b64;
var header = { headers: { key: "1KJPjd3p8khnWZTkjhDYnywLB2yE1w5BmU" } };
fetch(url, header).then(function(r) {
  return r.json()
}).then(function(r) {
  var result = JSON.stringify(r, null, 2);
  document.querySelector("pre").innerHTML = result;
})
</script>
</body>
</html>
```

## Socket

For Socket, it uses SSE (Server Sent Events).

You subscribe with the same Bitquery, and it will send you notifications when there's a new item that matches the pattern.

Because you can describe the pattern for anything including:

- Collection name
- MongoDB query

You can create a laser-focused subscription query to listen to.

```html
<html>
<body>
<script>
var query = {
  "v": 3, "q": { "find": {} }
}
// Base64 encoded query
var b64 = btoa(JSON.stringify(query))
// Subscribe to EventSource
var socket = new EventSource('https://bitsocket.org/s/'+b64)
// Handle new messages
socket.onmessage = function(e) {
  document.write("<pre>" + JSON.stringify(JSON.parse(e.data), null, 2) + "</pre>")
}
</script>
</body>
</html>
```

> You can learn more about the Socket feature here: [Learn Bitsocket](/socket)
