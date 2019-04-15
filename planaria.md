# How to Create a Planaria

Planaria is the Bitcoin facing part of the Planaria framework.

This page will walk you through how you can write a Planaria program.

> Once you have built a Planaria, you will want to publish or run it.
>
> You can learn more about publishing, deploying, and running a Planaria program [here](/serve)


## Overview

**Every Planaria is a state machine, powered by Bitcoin.**

![fsm](fsm2.png)

1. **Input:** An event listener that listens to realtime events from Bitcoin.
2. **Transition Logic:** A program that handles state transition
3. **State Memory:** A "memory" to persist and update the state
4. **Output Logic:** An event producer that publishes custom programmable events for other modules

Planaria lets you express all of the above as a single portable JavaScript object.

## Anatomy of a Planaria

![anatomy](anatomy.png)

Every Planaria state machine is a **SINGLE Node.js module object** that follows the following convention:

```
module.exports = {
  planaria: ...,
  from: ...,
  name: ...,
  version: ...,
  description: ...,
  address: ...,
  index: { ... },
  onmempool: async function(m) { ...  },
  onblock: async function(m) { ...  },
  onrestart: async function(m) { ...  }
}
```

Here's what each attribute represents:

- `planaria`: Planaria schema version (Currently '0.0.1')
- `from`: The block height to start crawling from
- `name`: The name of state machine this state machine will be published as
- `version`: The state machine version (Your application version)
- `description`: The description of the state machine to be advertised to the Planaria Network
- `address`: Bitcoin address tied to this state machine (You can generate a unique address using the Planaria Computer)
- `index`: a declarative descrition of the database index for faster queries and retrievals
- `onmempool`: an event handler for handling realtime mempool transactions
- `onblock`: an event handler for handling new block events
- `onrestart`: an event handler for when the state machine restarts (used to do cleanup, etc. for handling unexpected crashes or restarts)


## API

There are largely four classes of APIs you need to understand:

1. **ATTRIBUTES:** Define various attributes declaratively.
2. **INPUT API:** Listen to Bitcoin events.
3. **STATE API:** Write to Planaria State Machine.
4. **OUTPUT API:** Publish events.

### Attributes

These are the attributes used to describe the state database. Most of these attributes are auto-generated for you as a scaffold when you run the `pc` command:

#### planaria

The schema version of Planaria engine itself. Currently `0.0.1`.

#### from

The block height to start crawling from.

#### name

The name of state machine this state machine will be published as.

#### version

The state machine version (Your application version)

#### description

The description of the state machine to be advertised to the Planaria Network

#### address

Bitcoin address tied to this state machine (You can generate a unique address using the Planaria Computer)

#### index

A declarative description of the database index for faster queries and retrievals.

The syntax:


```
{
  index: {
    <Collection Name>: {
      "keys": [<Index Key0>, <Index Key1>, <Indx Key2>, ...],
      "unique": [<Unique Index Key0>, ...],
      "fulltext": [<Fulltext Index Key0>, <Fulltext Index Key1>, ...]
    }
  }
}
```

**Example 1:**

- One collection: `block`
- Put index on keys `hash`, `confirmations`, ...
- Ensure that the key `hash` is unique

```
{
  ...
  index: {
    block: {
      keys: [ 'hash', 'confirmations', 'size', 'height', 'version', 'versionHex', 'merkleroot', 'tx', 'time', 'mediantime', 'nonce', 'bits', 'difficulty', 'chainwork', 'previousblockhash', 'coinbase' ],
      unique: ['hash'],
    },
  },
  ...
}
```

**Example 2:**

- Two collections: `c` and `u`
- Put index on `tx.h`, `blk.i`, ... 
- Ensure that `tx.h` is unique
- Create a single full text index for each collection based on keys `out.s0`, `out.s1`, ...

```
{
  ...
  index: {
    c: {
      keys: [
        'tx.h', 'blk.i', 'blk.t', 'blk.h',
        'in.e.a', 'in.e.h', 'in.e.i', 'in.i',
        'out.e.a', 'out.e.i', 'out.e.v', 'out.i',
        'in.b0', 'in.b1', 'in.b2', 'in.b3', 'in.b4', 'in.b5', 'in.b6', 'in.b7', 'in.b8', 'in.b9', 'in.b10', 'in.b11', 'in.b12', 'in.b13', 'in.b14', 'in.b15',
        'out.b0', 'out.b1', 'out.b2', 'out.b3', 'out.b4', 'out.b5', 'out.b6', 'out.b7', 'out.b8', 'out.b9', 'out.b10', 'out.b11', 'out.b12', 'out.b13', 'out.b14', 'out.b15',
        'out.s0', 'out.s1', 'out.s2', 'out.s3', 'out.s4', 'out.s5', 'out.s6', 'out.s7', 'out.s8', 'out.s9', 'out.s10', 'out.s11', 'out.s12', 'out.s13', 'out.s14', 'out.s15'
      ],
      unique: ['tx.h'],
      fulltext: ['out.s0', 'out.s1', 'out.s2', 'out.s3', 'out.s4', 'out.s5', 'out.s6', 'out.s7', 'out.s8', 'out.s9', 'out.s10', 'out.s11', 'out.s12', 'out.s13', 'out.s14', 'out.s15', 'in.e.a', 'out.e.a']
    },
    u: {
      keys: [
        'tx.h',
        'in.e.a', 'in.e.h', 'in.e.i', 'in.i',
        'out.e.a', 'out.e.i', 'out.e.v', 'out.i',
        'in.b0', 'in.b1', 'in.b2', 'in.b3', 'in.b4', 'in.b5', 'in.b6', 'in.b7', 'in.b8', 'in.b9', 'in.b10', 'in.b11', 'in.b12', 'in.b13', 'in.b14', 'in.b15',
        'out.b0', 'out.b1', 'out.b2', 'out.b3', 'out.b4', 'out.b5', 'out.b6', 'out.b7', 'out.b8', 'out.b9', 'out.b10', 'out.b11', 'out.b12', 'out.b13', 'out.b14', 'out.b15',
        'out.s0', 'out.s1', 'out.s2', 'out.s3', 'out.s4', 'out.s5', 'out.s6', 'out.s7', 'out.s8', 'out.s9', 'out.s10', 'out.s11', 'out.s12', 'out.s13', 'out.s14', 'out.s15'
      ],
      unique: ['tx.h'],
      fulltext: ['out.s0', 'out.s1', 'out.s2', 'out.s3', 'out.s4', 'out.s5', 'out.s6', 'out.s7', 'out.s8', 'out.s9', 'out.s10', 'out.s11', 'out.s12', 'out.s13', 'out.s14', 'out.s15', 'in.e.a', 'out.e.a']
    }
  },
  ...
}
```

---

### Input

There are three types of events:

1. `onmempool`: new realtime mempool transaction
2. `onblock`: new block event, including all the transactions
3. `onrestart`: triggered when the node restarts. important for cleaning up before restart crawling.

#### onmempool

The `onmempool` method gets triggered whenver there's a new incoming mempool transaction.

You can implement the event handler by making use of the machine objct that gets passed into the handler as argument.

```
{
  ...
  onmempool: function(m) {
    /*****************************************
    *
    *   Do something with m, where:
    *
    *   m := {
    *     input: <Transaction in TXO Format>,
    *     state: <Database API>
    *   }
    *
    *****************************************/
  },
  ...
}
```

The `m` variable consists of two attributes:

- `input`: The incoming transaction event object
- `state`: The CRUD API object

You can use the `input` to derive any data, and write it the `state` object through the CRUD API.


#### onblock

```
{
  ...
  onblock: function(m) {
    /********************************************************************************************************************
    *
    *   Do something with m, where:
    *
    *   m := {
    *     input: {
    *       block: {
    *         info: {
    *           "hash": "000000000000000000410544d7cfe8f90b81daa1f02469a40be99e455c671be7",
    *           "confirmations": 151461,
    *           "size": 853803,
    *           "height": 411501,
    *           "version": 4,
    *           "versionHex": "00000004",
    *           "merkleroot": "efeb98352db1ad0ff691a8d76a552134f464648e09cc598e9b850b9aebbb8fa6",
    *           "tx": [
    *             "d14bb27028ce113f26b1ddee282479c573729b500dd3eda9249ccca360cf576e",
    *             "5432ca41143c74dc6dbb5551c817a50901fe1559b963f8a9731656f9136f52d2",
    *             "3ac66bbf21ad279223cab8b166f4c74621f98eff6ffb4cbf3fd080da88b5bb23",
    *             "3865e7facad32626dcd45584f830ce0ae439008a5271aa828158a96cd38b5de4",
    *             "8b3244fb3cd991bc216d27d11f863f44e55ae8f1f5d9655714a0b4caa408f573",
    *             "0143601e1a1621e94e0802fe44112a6a7f6ed889128dab537531158b8c27d565",
    *           ],
    *           "time": 1463095538,
    *           "mediantime": 1463093385,
    *           "nonce": 4098576299,
    *           "bits": "1805a8fa",
    *           "difficulty": 194254820283.44,
    *           "chainwork": "00000000000000000000000000000000000000000019172cdab3f28b3ad7c0ec",
    *           "previousblockhash": "00000000000000000258adbacc58c8acd91d492943ad64ab740ca5fa74b7f2a6",
    *           "nextblockhash": "000000000000000000b66d9af7a90a4335c59384f0c7cef926a63d4303a65811",
    *           "coinbase": "036d47061e4d696e656420627920416e74506f6f6c20626a3020201a535520573510efe0030000ea5c5581"
    *         },
    *         items: [
    *           <Transaction in TXO Format>,
    *           <Transaction in TXO Format>,
    *           ...
    *           <Transaction in TXO Format>
    *         ]
    *       },
    *       mempool: {
    *         info: { },
    *         items: [
    *           <Transaction in TXO Format>,
    *           <Transaction in TXO Format>,
    *           ...
    *           <Transaction in TXO Format>
    *         ]
    *       }
    *     },
    *     state: <Database API>,
    *     clock: {
    *       bitcoin: <Bitcoin Height>,
    *       self: <Machine Height>
    *     }
    *   }
    *
    ********************************************************************************************************************/
  },
  ...
}
```

#### onrestart

### State

You can access the state CRUD API through:

-  `m.state.create`
-  `m.state.read`
-  `m.state.update`
-  `m.state.delete`


#### create

Insert a new document to the database (Create)

- `name`: the collection name. You can store to as many collections as you want, and thanks to the no-schema nature of MongoDB it will just store them all.
- `data`:
  - **one item:** if it's a regular object, it will be inserted as a single document
  - **an array of items:** if it's an array, it will be bulk inserted as multiple documents

```
await m.state.create({
  name: <COLLECTION_NAME>,
  data: <ITEM|ITEMS>
}).catch(function(e) {
  if (e.code != 11000) {
    console.log("# onmempool e = ", e)
  } else {
    console.log("duplicate", m.tx)
  }
})
```

#### read

You can determine the next state transition not only based on the incoming event, but also based on the current state, simply by querying the state database with `read`.

```
await m.state.read({
  name: <COLLECTION_NAME>,
  filter: {
    find: <MONGODB_FIND_FILTER>,
    project: <MONGODB_PROJECT>,
    sort: <MONGODB_SORT>,
    limit: <MONGODB_LIMIT>,
    skip: <MONGODB_SKIP>,
  }
})
```


#### update

In order to provide a simple interface for updating, the `update` API takes the following approach:

1. Delete all items that match the `filter.find` query
2. Transform the deleted items using the `map()` function
3. Re-insert the transformed items to the collection

> This approach works for most cases because the insertion order doesn't matter for most Planaria state machines, since everything can be ordered by block time.
>
> However, if you DO want to enforce certain order, you may want to take care of this manually, programmatically.

```
await m.state.update({
  name: <COLLECTION_NAME>,
  filter: {
    find: <MONGODB_FIND_FILTER>
  },
  map: function(item) {
    /********************************************************
    * A Transformer function that takes each filtered item
    * from "filter" and transforms it to a new item
    ********************************************************/
    let new_item = DO_SOME_TRANSFORM(item)
    return new_item
  }
}).catch(function(e) {
  if (e.code != 11000) {
    console.log("# onblock error = ", e)
    process.exit()
  }
})
```

#### delete

Delete all items that match certain MongoDB filter

This is especially handy inside `onrestart`, where you can use it to clean up any dangling documents since the last shutdown. 

```
let res = await m.state.delete({
  name: <COLLECTION_NAME>,
  filter: {
    find: <MONGO_FIND_FILTER>
  }
}).catch(function(e) {
  console.log("# onblock prune error = ", e)
})
```

### Output

Output APIs are for publishing events.

When you publish an event, you can consume it from:

1. **Planarium:** The `socket` API automatically turns the event you publish here into an SSE event.
2. **Any other modules:** Other modules can listen to the event through Zeromq.

#### publish

After taking care of all the state transition tasks, you can publish an event with zeromq by calling the `output.publish` method, which takes an object as an argument, which contains two attributes:

1. `name`: The topic name to publish to
2. `data`: The data to publish

Here's an example:

```
onblock: function(m) {
  ...
  m.output.publish({
    name: 'block',
    data: m.input.block.info
  })
}
```


## Examples

Let's take a look at some examples. 

Each example will demonstrate a new unique feature Planaria provides:

1. Appending
2. Block Event Handling
3. Transaction Event Handling

Note: These are not just theoretical code. These are actual state machines running in production:

### Appending

Appending is how Bitcoin natively operate. You simply create a new record based on Bitcoin transaction events.

For example, let's say you want to create a database of every **real world timestamp** of when a node encountered a mempool transaction:

```
module.exports = {
  ...
  onmempool: function(m) {
    await m.state.create({
      name: "timestamp",
      data: {
        hash: m.input.tx.h,
        timestamp: Date.now()
      }
    }).catch(function(e) { console.log("# onmempool error = ", e) })
  }
}
```

### Block Event

Now let's take a look at an example of handling new block events.

[Meta](https://medium.com/@_unwriter/meta-be3c18582ec7) is a database for crawling and serving Block header data. Here's the entirety of its code:

```
module.exports = {
  from: 0,
  name: 'meta',
  version: '0.0.3',
  description: 'Bitcoin Block Metadata Database',
  address: '18uQBWMhHgiieR6FnQtXDDba8T97LVu5uH',
  index: {
    block: {
      keys: [ 'hash', 'confirmations', 'size', 'height', 'version', 'versionHex', 'merkleroot', 'tx', 'time', 'mediantime', 'nonce', 'bits', 'difficulty', 'chainwork', 'previousblockhash', 'coinbase' ],
      unique: ['hash'],
    },
  },
  onblock: async function(m) {
    let info = m.input.block.info
    delete info.tx
    await m.state.create({
      name: "block",
      data: m.input.block.info,
      onerror: function(e) { if (e.code != 11000) { process.exit() } }
    })
    m.output.publish({name: 'block', data: m.input.block.info})
  },
  onrestart: async function(m) {
    let result = await m.state.delete({
      name: 'block',
      filter: { find: { "height": { $gte: m.clock.self.now } } }
    }).catch(function(e) { if (e.code && e.code !== 11000) { process.exit() } })
    await m.clock.self.set(m.clock.self.now)
  },
}
```

First some explanation on the metadata:

- `from`: the Meta database crawls all the way from the genesis block, so the "from" is set to 0
- `name`: This state machine will be published as the name "meta".
- `version`: The state machine may go through multiple upgrades in the future. The current version is `0.0.3` (follows semantic versioning)
- `description`: The description to publish to the planaria network
- `address`: The Bitcoin address tied to this state machine. Only the person who owns the private key to this address can publish to the Planaria network.
- `index`: Describes which collections and which attributes to attach index to. In this case the DSL is saying:
	- for the `block` collection, attach index to `hash`, `confirmations`, `size`, ... keys.
	- and ensure that the `hash` key is unique. This means no duplicate items with the same hash key will be inserted.


Now the actual program:

- `onblock`: This is the new block hander which gets triggered whenever there's a new block. The `m` variable contains everything required to parse a block and its transactions.
- `onrestart`: It is important to write `onrestart` to do cleanup task, because the machine may need to pause and restart at some point, or may even crash. And it's important to clean up dangling entries before restarting in order to make sure there's no discrepancy. In this case, it deletes all the block data after the last checkpoint, and starts over.


### Transaction Event

Also, by attaching a `onmempool` handler, you can handle realtime mempool transactions.

By default, Planaria ships with the [Genesis Bitdb](https://medium.com/@_unwriter/genesis-a25b121e0575) as stub, so it works right out of the box.

```
/***************************************
*
* Crawl Everything
*
***************************************/
module.exports = {
  planaria: '0.0.1',
  from: 525470,
  name: "Genesis",
  version: '0.0.1',
  description: 'Full BitDB',
  address: "1FnauZ9aUH2Bex6JzdcV4eNX7oLSSEbxtN",
  index: {
    c: {
      keys: [
        'tx.h', 'blk.i', 'blk.t', 'blk.h',
        'in.e.a', 'in.e.h', 'in.e.i', 'in.i',
        'out.e.a', 'out.e.i', 'out.e.v', 'out.i',
        'in.b0', 'in.b1', 'in.b2', 'in.b3', 'in.b4', 'in.b5', 'in.b6', 'in.b7', 'in.b8', 'in.b9', 'in.b10', 'in.b11', 'in.b12', 'in.b13', 'in.b14', 'in.b15',
        'out.b0', 'out.b1', 'out.b2', 'out.b3', 'out.b4', 'out.b5', 'out.b6', 'out.b7', 'out.b8', 'out.b9', 'out.b10', 'out.b11', 'out.b12', 'out.b13', 'out.b14', 'out.b15',
        'out.s0', 'out.s1', 'out.s2', 'out.s3', 'out.s4', 'out.s5', 'out.s6', 'out.s7', 'out.s8', 'out.s9', 'out.s10', 'out.s11', 'out.s12', 'out.s13', 'out.s14', 'out.s15'
      ],
      unique: ['tx.h'],
      fulltext: ['out.s0', 'out.s1', 'out.s2', 'out.s3', 'out.s4', 'out.s5', 'out.s6', 'out.s7', 'out.s8', 'out.s9', 'out.s10', 'out.s11', 'out.s12', 'out.s13', 'out.s14', 'out.s15', 'in.e.a', 'out.e.a']
    },
    u: {
      keys: [
        'tx.h',
        'in.e.a', 'in.e.h', 'in.e.i', 'in.i',
        'out.e.a', 'out.e.i', 'out.e.v', 'out.i',
        'in.b0', 'in.b1', 'in.b2', 'in.b3', 'in.b4', 'in.b5', 'in.b6', 'in.b7', 'in.b8', 'in.b9', 'in.b10', 'in.b11', 'in.b12', 'in.b13', 'in.b14', 'in.b15',
        'out.b0', 'out.b1', 'out.b2', 'out.b3', 'out.b4', 'out.b5', 'out.b6', 'out.b7', 'out.b8', 'out.b9', 'out.b10', 'out.b11', 'out.b12', 'out.b13', 'out.b14', 'out.b15',
        'out.s0', 'out.s1', 'out.s2', 'out.s3', 'out.s4', 'out.s5', 'out.s6', 'out.s7', 'out.s8', 'out.s9', 'out.s10', 'out.s11', 'out.s12', 'out.s13', 'out.s14', 'out.s15'
      ],
      unique: ['tx.h'],
      fulltext: ['out.s0', 'out.s1', 'out.s2', 'out.s3', 'out.s4', 'out.s5', 'out.s6', 'out.s7', 'out.s8', 'out.s9', 'out.s10', 'out.s11', 'out.s12', 'out.s13', 'out.s14', 'out.s15', 'in.e.a', 'out.e.a']
    }
  },
  onmempool: async function(m) {
    delete m.input.tx.r
    await m.state.create({
      name: "u",
      data: m.input
    }).catch(function(e) { console.log("# onmempool error = ", e) })
    m.output.publish("u", m.input)
  },
  onblock: async function(m) {
    await m.state.create({
      name: "c",
      data: m.input.block.items,
      onerror: function(e) {
        if (e.code != 11000) {
          console.log("# Error", e, m.input, m.clock.bitcoin.now, m.clock.self.now)
          process.exit()
        }
      }
    }).catch(function(e) { process.exit() })
    if (m.clock.bitcoin.now > m.clock.self.now) {
      await m.state.delete({
        name: "u",
        filter: { find: {} }
      }).catch(function(e) { console.log(e) })
      await m.state.create({
        name: "u",
        data: m.input.mempool.items,
        onerror: function(e) { if (e.code != 11000) { process.exit() } }
      }).catch(function(e) { console.log(e) })
    }
    m.output.publish("c", m.input)
  },
  onrestart: async function(m) {
    await m.state.delete({
      name: 'c',
      filter: { find: { "blk.i": { $gte: m.clock.self.now } } },
      onerror: function(e) { if (e.code !== 11000) { process.exit() } }
    }).catch(function(e) { console.log("# onrestart error = ", e) })
    await m.clock.self.set(m.clock.self.now).catch(function(e) { process.exit() })
    await m.state.delete({ name: "u", filter: { find: {} } }).catch(function(e) {
      console.log("# mempool delete error: ", e)
    })
  }
}
```
