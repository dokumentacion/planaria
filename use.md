# Use Cases

## Blockchain Explorers

Bitcoin has been around for a decade, yet we have very small variety of "Explorer" services that lets people make sense of the blockchain. This is because these "explorers" were built with a very narrow use case in mind.

Planaria sees **Bitcoin as a computer**, and was designed to complement this vision.

With Planaria, you can build your own custom **"block explorer"** that functions whichever way you want.

## Public Utility Database

With Planaria, not only can you build these databases easily, but also you can turn them into a monetizable public utility, thanks to the tightly integrated Bitcoin system.

## Forensic Database

You can derive all kinds of databases using Planaria. You just need to write a program to instruct how to map and filter Bitcoin.

One cool use case is a forensic database for detecting anomaly or potentially malicious transaction patterns.

## Analytics Database

You can build a separate standalone analytics database for application usage, protocol usage, transaction pattern usage, etc. This database can run independently from whoever is actually running the application.

## Prunable Database

You can build a database that automatically prunes itself depending on certain conditions. See [Chronos](https://medium.com/@_unwriter/chronos-f0f751669fef).

## Ephemeral Database

You can build a database that updates itself over time.

## Web Application Backend

Remember, Planaria is essentially a development framework that lets you build any kind of state machine from Bitcoin.

This means you can recreate any existing real-world application, but instead of using the traditional cloud model, now you use Bitcoin.

## Long-tail Applications

Applications and databases can be as niche as possible.

For example, you should be able to run a database for just your family, or just for the neighborhood, completely on Bitcoin.

This is possible because Planaria acts as the "filter". Filtering and Transforming is what Planaria is designed to do, so you can build a filtered and transformed version of Bitcoin that fits very specific niche needs.

## Portable Applications

When you store your app on Bitcoin, you effectively have an app that can fork or evolve while preserving the state. This is simple because all you need to do to fork an app is to take the Planaria code, make a few modifications, and deploy it. Then get people to join your fork.

This can be used to implement:

1. **Protocol Upgrades:** An app can "upgrade" when the entire community moves onto the new fork.
2. **Protocol Governance:** Let the community vote by choosing which fork to use.
3. **Protocol Specialization:** Build more niche apps derived from the original app which is more generic.

## Multi-purpose Backends

Planaria is designed to facilitate the "long tail" of data storage.

A more generic database can be a superset of a more niche database.

For example, [Genesis](https://medium.com/@_unwriter/genesis-a25b121e0575), an append-only Planaria node that stores an entire transaction object, is a superset of [Babel](https://medium.com/@_unwriter/babel-230f73ed5dcb), an append-only Planaria node that stores just a subset which only contains OP_RETURN outputs.

## Ultimate Bitcoin Filter

The main strength of Planaria is it lets you `map` and `filter` Bitcoin in whichever way you want, to turn it into a database with HTTP and SSE (Server Sent Events) API right out of the box.

### Map

Mapping means you can **transform** Bitcoin into whichever format you want. You can build various useful applications this way. Here are some examples:

1. **A Censored Database:** Whenever you detect certain patterns of transactions, you may want to first transform them in a way that appears as "censored" (through mosaic, etc.) before storing and serving.
2. **A Preview Database:** You may want to build a separate database that stores a "preview" version of the original data, which you may want to monetize. The "preview database" would be publicly available, but the original content may be stored encrypted.
3. **A Compressed Database:** Instead of storing the original content, you may apply certain compression algorithm before storing.
4. **A Content-addressable storage:** For certain content, you may want to hash them to create a checksum, and store the checksum. This way you can implement content-addressable storage.

### Filter

Planaria node providers can also **transparently** program their nodes to exclude or include certain types of transactions in their database.

This includes examples such as:

1. **Transactions of low value:** For example Leave out stress test transactions from the database
2. **Specialized subset of transactions:** On the other hand you could create a very specialized database that ONLY crawls the said stress test transactions only. This may be useless for most people, but may be useful for those who run the stress test.
3. **Leave out Illegal Content:** You can filter Bitcoin in a way that all illegal contents are filtered out and not stored on the database (For example, child porn). The filtering algorithm itself can be programmed as a state machine and shared across multiple applications.

## Bitcoin as IPC (Inter Process Calls)

You can also use Bitcoin purely as an event to trigger **ANY** application you write in JavaScript.

For example, you can write a program that automatically sends out email or SMS, or sends a Tweet whenever there's a certain pattern of incoming Bitcoin event.

> **Anything you can build with Node.js, you can power with Bitcoin, through Planaria**

## Transparent Private Computing

You can even implement **"transparent private computing"**.

Just because everything is transparent doesn't mean everything must be public. Through various clever tricks you can build an "encrypted application". Here's an example:

1. Build a Planaria Node.
2. Use encryption in certain parts of the code for the Planaria node.
3. Publish the code publicly, but share the keys only with the stakeholders.

The benefit of this approach is that it takes the best of both worlds (Transparent computing powered by Bitcoin + private computing powered by Encryption).

1. **Auditable:** The "log" is transparent so the entire state transition history can be audited later if needed.
2. **Private:** While the log is transparent, thanks to the encryption, nobody outside of the stakeholders group can see what the events in the log mean. It can only be interpreted by the stakeholders who hold the decryption keys.

## ANYTHING

Remember, all modern applications are nothing more than **CRUD backend API** + **frontend user interface**.

And Planaria lets you build the "CRUD backend API" part from Bitcoin. So this means you can build ANY Database--therefore ANY Bitcoin-powered API--you can imagine.
