# polymorphic testing 
Here we analyze how to best implement polymorphism. We're not answering whether there are alternatives to polyphormism that would potentially better fit the GNAP design's objective. 

I ask for your understanding, this is not a trivial problem to be resolved

> If you look at another engineer's work and think, "That's dumb. Why don't you just..." Take a breath. Find out why the problem is hard. _—Adrienne Porter Felt_

## the context and problem
It is envisaged within [IETF gnap](https://tools.ietf.org/id/draft-ietf-gnap-core-protocol-00.html) to use polymorphism for json request/responses.
The rationale has been explained by Justin Richer in various places. In my own words, I would say that the intent is to make the life of the developer easier, with client libraries not having to support all possible permutations. Just prepare your request and send it to the Authorization Server (AS), and you're good to go. 

Despite the advantages, the concern is that the conversion between objects and json might be difficult for strongly typed langages (see for instance [this discussion](https://medium.com/@aems/one-mans-struggle-with-typescript-class-serialization-478d4bbb5826) on how typescript makes it more complex) and potentially error prone.
Related discussions: 
- [HN discussion](https://news.ycombinator.com/item?id=24855750)
- IETF mailing list, see [discussion](https://mailarchive.ietf.org/arch/msg/txauth/0RRynhG7hwxpE341K-gSZhRXPIo/)

We should examine in details what it implies for each party in the protocol (AS, RS, client).
Note that at this stage, it may well be that the Working Group just decides on a different design altogether.

## review of possibilities

Ideally we'd want support for both self-description (for flexibility/evolutions) and schemas/types (to limit errors). But we should try to avoid reinventing the wheel. [Oh no! Not yet another serialization format!](https://scottlocklin.wordpress.com/2017/04/02/please-stop-writing-new-serialization-protocols/) 

JSON schema is the prefered option currently, but I still think worthwile to make a more systematic open review. 

The table below provides a comparison:

| Name             | Schema          | Self-Describing  | Spec               | Maturity        | Compact         |
|------------------|-----------------|------------------|--------------------|-----------------|-----------------|
| **JSON schema**  | :green_heart:   | :green_heart:    | :yellow_heart:†    | :green_heart:   | :broken_heart:  |
| capnproto        | :green_heart:   | :green_heart:    | :yellow_heart:     | :yellow_heart:  | :green_heart:   |
| msgpack          | :broken_heart:  | :green_heart:    | :yellow_heart:     | :green_heart:   | :green_heart:   |
| ion              | :green_heart:   | :green_heart:    | :yellow_heart:     | :green_heart:   | :green_heart:   |
| tjson            | :green_heart:   | :green_heart:    | :yellow_heart:†    | :broken_heart:  | :broken_heart:  |
(please shout if you think there's a mistake)

*†NOTE: unmaintained IETF draft*

Unfortunately none of these projects have been standardised (or even are in the process of being standardised), but they do provide specification documents.
The maturity evaluation criteria is subjective (based on number of available implementations, in various mainstream programming languages, and usage in various projects or support from a large organization). 

We should also have in mind potential extensions (like CBOR for constrained use cases in OAuth ACE).

### [json schema](https://json-schema.org)
JSON schema allows to specify the structure of a JSON document (here's a [link](https://json-schema.org/understanding-json-schema/reference/object.html#properties) to get a feeling of how this works). 
JSON schema has the ability to extend using $ref to other schemas (such as https://schema.org for instance) or may be combined with JSON-LD vocabularies (often used in the self sovereign identity space). 

People wouldn't be forced to use the schema so they can still use a regular json processor (which is both good and bad).

Of the entire list, it is the most widely used and is already supported by many tools. 

GNAP related resources:https://github.com/ietf-wg-gnap/core-protocol/pull/4

### [capnproto](https://capnproto.org/)
A fast RPC protocol that happens to also implement its own serialization scheme and provides a [schema langage](https://capnproto.org/language.html), including a union type. Many implementations but parent supporting company sandstorm was stopped, and the core of how it works is quite hairy (especially if one also wants to support dynamic languages).

### [msgpack](https://msgpack.org/)
Not meant to be easily readable by a human (just like CBOR, which is real fine to me), comes with many implementations. Doesn't provide a schema, so not a great fit here. 
But this highlights an issue: even if we have json schema, how can we include support for a compact format / CBOR too? (for constrainted cases that will likely occur).

### [ion](http://amzn.github.io/ion-docs/)
There comes the new kid on the block, backed by amazon, it provides a compact format (kind of a merge between json and cbor) which would be a good fit in an IAM context, to generalize to a wider audience of clients (web, mobile, iot). Related subprojects support [hash](https://amzn.github.io/ion-hash/) and, most importantly for our concern, a [schema](https://amzn.github.io/ion-schema/). Several implementations are available. 
Could be a real contender, as it removes the compact format / CBOR issue.

### experimental
By experimental, I mean new and interesting but rarely if ever seen in production. 

- [tjson](http://tjson.org) codifies the type through a tag, for instance `{"hello-object:O": {"hello-string:s": "Hello, world!"}}`. What I like about this is that it makes it explicit in the document itself, you don't need to rely on an external schema. Several implementations available.
- [typedJSON](https://github.com/JohnWeisz/TypedJSON): polymorphic object structures require type-annotations to be present in JSON, and benefit from a [reflection API for ECMAScript](https://github.com/rbuckton/reflect-metadata). Implemented in typescript only.
- [noproto](https://github.com/ClickSimply/NoProto): implemented in rust and wasm, with support of basic [option values](https://docs.rs/no_proto/0.1.2/no_proto/format/index.html#option-scalar).

### not considered
- XML (because I've done too much of that in a hopefully buried past).
- there are also various schemes out there that require some compiled schema (protobuf, avro, thrift, ASN1, DER, etc.), but this doesn't make it a great fit for our specific purpose.

## reasonably polymorphic tests - WIP 
We aim to provide concrete implementations of polymorphism on real gnap messages. As a result of the comparison, I will focus on JSON and ION.

The goal is to:
- test various alternatives (even if they're not going to be chosen in the spec, it should provide a more detailed justification on the choices made)
- provide guidance on how to best implement it in various mainstream languages, in the most idiomatic way possible 
- provide a test suite that includes various examples and ensure compatibility between implementations

By mainstream, I have in mind (eventually): typescript, java (kotlin?), python, go, rust. 

Help welcome. I guess it makes a fun coding exercice. For instance, will also probably try it in scala3, because I want to learn about the new features ;-)

### json schema
- [rust](https://github.com/fimbault/test_gnap_schema) : a very basic test of using a json schema validation
