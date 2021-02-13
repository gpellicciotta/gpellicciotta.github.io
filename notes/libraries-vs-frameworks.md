# Libraries vs Frameworks

## Differences

The difference between the two are subtle but generally can be described as follows:
  - A library simply offers a bunch of functions that can be used at will
    It doesn't (typically) have any constraints on the order in which its functions (or classes) can be used.
    It doesn't (typically) rely on meta-data in the form of annotations/attributes, or the naming of classes and/or methods

  - A framework is a library that cannot just be used at will: more preparation is needed.
    This preparation can come in the form of annotations/attributes that need to get applied (e.g. springboot),
    or classes/methods that need to have a certain name.
    Often a framework is accompanied by tools that help setup the necessary 'infrastructure code'.

## Opinion

A library is always superior to a framework as:
  - It has a far smaller learning curve (as it _only_ relies on normal programming techniques)
  - It is easier to substitute for another library in the future (e.g. see the various logging libraries)
    as long as its API surface is not too large: there is far less _lock in_ than with a framework

Offering a framework often is proof that:
  - No good abstraction could be determined (that is representable in pure functions/classes)
  - A form of _lock in_ is sought after

## Notes

An SDK is different from a framework: an SDK really is a library in addition to tools that help in setting up a _normal_ development environment:
compilers, linkers, documentation generators, ...

An SDK hence is also limited to the situation where a programming language is being implemented/offered.

Note that nowadays some people call their frameworks SDKs (e.g. see various web frameworks): this really highlights the fact that programming
against a framework is almost programming against a different programming language, also needing additional
tools and additional languages (e.g. sass) to make the whole work.
