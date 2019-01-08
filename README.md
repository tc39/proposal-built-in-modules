# JavaScript Standard Library Proposal

Proposal for adding a mechanism for enabling a more extensive standard library in JavaScript. With
this infrastructure in place it will be possible to start iterating on standard
library features as modules in the future.

<details>
  <summary><strong>Table of Contents</strong></summary>

  * [Scope](#scope)
  * [Motivation](#motivation)
  * [Proposed Solution](#proposed-solution)
  * [Import Semantics](#import-semantics)
  * [Module Resolution](#module-resolution)
  * [Related Efforts](#related-efforts)

</details>

## Scope

The goal of this proposal is to define a mechanism for enabling a more extensive standard library in
JavaScript than is currently available. This proposal would not change the behavior of any existing code or
add any new syntax, except possibly the syntax for importing the standard library code.

The contents of the standard library is tangential to this proposal, and would be built and expanded upon in
later efforts. Such a library would only cover features which would be useful in JavaScript in general, not
things which are tied to the web platform. (A good heuristic: if something would make sense on a web browser
but not in node or on [embedded devices](https://www.moddable.com/) or [robots](http://johnny-five.io/), it
probably isn't in scope.) See [#16](https://github.com/tc39/proposal-javascript-standard-library/issues/16)
for discussion of the extent and contents of the library.


## Motivation

Most programming languages have core language features (syntax, operators, primitives etc.) and also
include a standard library with common functionality. Developers are able to use this in their programs
immediately because the library is bundled with the runtime.

The JavaScript language does not have a standard library resulting in common functionality being developed
into libraries that are included with JavaScript programs. Because these libraries need to be included in every
program instead of being provided by the runtime these programs are bigger with users paying the cost of
downloading and parsing the extra bits. These libraries are also harder to cache between programs.

JavaScript programs run on a variety of clients with different runtime implementations and versions. Libraries
have to account for these different environments by including code that is not required or even run on some
clients, but does incur a download and parse cost.


## Proposed Solution

To enable developers to access common functionality at runtime that is provided by the JavaScript engine the
notion of a standard library has to be created. This will allow code currently loaded through libraries as
part of the program to shift towards being bundled with JavaScript implementations in the future.

> Disclaimer: This proposal covers adding a mechanism for enabling a standard library in JavaScript
> implementations, it does not describe its contents. This is considered a tangential effort that would
> be built and expanded upon in the future when a mechanism for accessing it is in place.

<!-- Availability & Standardization -->
Having a standard library readily available at runtime means programs won't have to include the functionality
availabile in the standard library, reducing the download and parse cost of the program. The functionality will also be
standardized across implementations giving the developer guarentees about quality, behavior and speed.

<!-- Extensibility -->
Although JavaScript engines already have the notion of built-ins, the standard library will use modules and the
`import` syntax to access it. This mechanism should already be familiar to developers and will allow them to
opt-in to functionality they need. Using modules also allows more flexibility when designing the library
contents and helps avoiding conflicts with global APIs.

Modules for the standard library should be able to be written in plain JavaScript for the most part but for
hot pieces of code the engine could provide a native implementation.

## Import Semantics

To import modules from the standard library the engine has to be able to distinguish between standard library
modules and other (user) modules. To allow the engine to do this standard library modules should use a prefix
in the module identifier string. This is prefered over other alternatives because it is does not introduce new
syntax for loading standard library modules to the `import` statement developers should already be familiar with.

### Namespace

The `js::` prefix will be reserved as the namespace for the JavaScript language only and will be governed by
TC39, making the standard library a true JavaScript standard library. This will allow the committee to work
safely within this namespace when designing and developing the standard library.

> There were other prefixes considered, please see the Appendices for more detail.

By creating a namespaces specifically for the JavaScript standard library, developers will know what to expect
when importing from using the `js::` prefix across different implementations and can be assured the same
modules are available across these implementations (not considering implementation constraints, vendor
timelines or version differences).

It is completely feasable that more namespaces are introduced which are goverened by other standards bodies or
organizations.  However it is important that these namespaces stay independent of each other to avoid conflicts and
contrain development within namespaces due to outside pollution.

### Freezing Imports

All imported objects and classes from the standard library will have their prototype frozen. This will prevent
prototypes from imported objects to be modified outside of the module causing prototype pollution.

In the past the committee had to make concessions to maintain web compatibility when adding new functionality
to built-in objects. By freezing the prototype of standard library exports, it will no longer be possible for 
third party code to modify or extending library code in a possibly incompatible way. This will allow for more
flexibility when designing and developing the standard library. Extending standard library classes and objects
can still be done using `extend` or `Object.create`.

> TODO: Describe how prototypes will be frozen

## Module Resolution

The ECMAScript specification defines an _Abstract Operation_ for resolving and loading modules called
[**HostResolveImportedModule**](https://www.ecma-international.org/ecma-262/#sec-hostresolveimportedmodule).
This operation is handled by the embedding environment and has to result in a **ModuleRecord** or an
**Error**.

Because standard library modules are be bundled with the engine they have to be loaded in a different manner.
The engine needs to be able to see resolution requests in order to participate in the resolution of standard
library modules.

### Chained Loading

To allow the JavaScript engine to participate in module resolution the **HostResolveModuleIdentifier**
_Abstract Operation_ should be changed to allow more than one resolver. Taking inspiration from the [Python
import mechanism](https://docs.python.org/3/reference/import.html) there should be a chain of resolvers to
resolve modules and a way to register resolvers. This will allow the engine to have its own resolver alongside
any resolvers registered by the embedder.

Having more than one resolver is not very different than the current operation, each resolver in the chain
still has the same responsibilities when trying to resolve a _ModuleIdentifier_:

  * It must return an instance of ModuleRecord
  * It must throw an error if a ModuleRecord cannot be created
  * It must be idempotent for pair of (_referencingModule_, _specifier_)

The difference is that the engine is responsible for maninging the chaining of resolvers together. When a
resolver in the chain is unable to resolve a _ModuleRecord_, the _ModuleIdentifier_ should be passed to the
next resolver in the chain. An error should only be surfaced when all resolvers in the chain were consulted
and no _ModuleRecord_ was produced.

Multiple resolvers in a chain allow the engine to register its own resolver for standard library modules, but
would also allow embedders to register multiple resolvers, e.g. for different purposes.


### Polyfilling

In order to support polyfilling new resolvers should always be registered at the head of the resolver chain
and the standard library resolver used by the engine should always be the first resolver to register. This
will always make the standard library resolver the last in the chain, giving resolvers registered by the
embedder first chance of handling imports from the standard library.

For the web platform polyfilling could be done using the [Import Maps Proposal](https://github.com/domenic/import-maps).
A resolver registered by the embedder (a web browser in this case) could check the import map to see if a
standard library _ModuleIdentifier_ should be redirected to another implementation.

> TODO: How does recursion work? With a special flag maybe?


## Related Efforts

This subject has been talked about in the past and has related efforts:

- <https://github.com/tc39/ecma262/issues/395>
- <https://github.com/drufball/layered-apis>

<!-- There should be an updated link to the Import Maps proposal -->
