# polymorphic-json
A systematic review and some tests

> If you look at another engineer's work and think, "That's dumb. Why don't you just..."
> Take a breath. Find out why the problem is hard. _—Adrienne Porter Felt_

## the context and problem
It is envisaged within [IETF gnap](https://tools.ietf.org/id/draft-ietf-gnap-core-protocol-00.html) to use polymorphism for json request/responses.
The rationale has been explained by Justin Richer in various places. In my own words, I would say that the intent is to make the life of the developer easier, with client libraries not having to support all possible permutations. Just prepare your request and send it to the Authorization Server (AS), and you're good to go. 

Despite the advantages, the concern is that the conversion between objects and json might be difficult for strongly typed langages (for instance [this discussion](https://medium.com/@aems/one-mans-struggle-with-typescript-class-serialization-478d4bbb5826)) and potentially error prone.
Related discussions: 
- [HN discussion](https://news.ycombinator.com/item?id=24855750)
- IETF mailing list, see [discussion](https://mailarchive.ietf.org/arch/msg/txauth/0RRynhG7hwxpE341K-gSZhRXPIo/)

We should examine in details what it implies for each party in the protocol (AS, RS, client).
Note that at this stage, it may well be that the Working Group just decides on a different design altogether.

## review of possibilities

Ideally we'd want support for both self-description (for flexibility/evolutions) and schemas (to limit errors). But we should try to avoid reinventing the wheel. [Oh no! Not yet another serialization format!](https://scottlocklin.wordpress.com/2017/04/02/please-stop-writing-new-serialization-protocols/). 

JSON schema is the prefered option currently, but I think worthwile to make a more systematic open review. 

The table below provides a comparison:

| Name             | Schemas         | Self-Describing  | Standardisation    | Maturity        |
|------------------|-----------------|------------------|--------------------|-----------------|
| **JSON schema**  | :green_heart:   | :green_heart:    | IETF†              | Good            |
(to be completed)

*†NOTE: Work in progress!*

The maturity evaluation criteria is subjective (based on number of available implementations, in various mainstream programming languages, and usage in various projects). 

We should also have in mind potential extensions (like CBOR for constrained use cases in OAuth ACE).

### [json schema](https://json-schema.org)
JSON schema allows to specify the structure of a JSON document (here's a [link](https://json-schema.org/understanding-json-schema/reference/object.html#properties) to get a feeling of how this works). 
JSON schema has the ability to extend using $ref to other schemas (such as https://schema.org for instance) or may be combined with JSON-LD vocabularies (often used in the self sovereign identity space). 

People wouldn't be forced to use the schema so they can still use a regular json processor (which is both good and bad).

Of the entire list, it is the most widely used and is already supported by many tools. 

GNAP related resources:https://github.com/ietf-wg-gnap/core-protocol/pull/4

### [capnproto](https://capnproto.org/)
A fast RPC protocol that happens to also implement its own serialization scheme and provides a [schema langage](https://capnproto.org/language.html), including a union type.

### [msgpack](https://msgpack.org/)
Not meant to be easily readable by a human (just like CBOR), comes with many implementations. Doesn't provide a schema.

### [ion](http://amzn.github.io/ion-docs/)
New kid on the block, backed by amazon, it provides a compact format (kind of a merge between json and cbor) which would be a good fit in an IAM context, to generalize to a wider audience of clients (web, mobile, iot). Related subprojects support [hash](https://amzn.github.io/ion-hash/) and, most importantly for our concern, a [schema](https://amzn.github.io/ion-schema/). Several implementations are available. 

### experimental
By experimental, I mean new and interesting but rarely if ever seen in production. 

- [tjson](http://tjson.org) codifies the type through a tag, for instance `{"hello-object:O": {"hello-string:s": "Hello, world!"}}`. What I like about this is that it makes it explicit in the document itself, you don't need to rely on an external schema. Several implementations available.
- [typedJSON](https://github.com/JohnWeisz/TypedJSON): polymorphic object structures require type-annotations to be present in JSON, and benefit from a [reflection API for ECMAScript](https://github.com/rbuckton/reflect-metadata). Implemented in typescript only.
- [noproto](https://github.com/ClickSimply/NoProto): implemented in rust and wasm, with support of basic [option values](https://docs.rs/no_proto/0.1.2/no_proto/format/index.html#option-scalar).

### not considered
- XML (because I've done too much of that in a hopefully buried past).
- there are also various schemes out there that required some compiled schema (protobuf, avro, thrift, ASN1, DER, etc.), but this doesn't make it a great fit for our specific purpose.

## reasonably polymorphic tests - WIP 
Here will be to implement polymorphism on real gnap messages. 
The goal is to:
- test various alternatives discussed here (even if they're not going to be chosen in the spec, it's a good exercice)
- provide guidance on how to best implement it in various mainstream languages, in the most idiomatic way. Maybe even provide a set of libraries for it someday. 
- provide a test suite that includes various examples and ensure compatibility between implementations

By mainstream, I have in mind: typescript, java (kotlin? scala3?), python, go, rust. Help welcome.

### json schema
- [rust](https://github.com/fimbault/test_gnap_schema) : a very basic test of using a json schema validation
