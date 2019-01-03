# Standard Library Proposal

Proposal for adding a mechanism for enabling a more extensive standard library in JavaScript. With
this infrastructure in place it will be possible to start iterating on standard
library features as modules in the future.

<details>
  <summary><strong>Table of Contents</strong></summary>

  * [Scope](#scope)
  * [Motivation](#motivation)
  * [Proposed Solution](#proposed-solution)
  * [Requirements](#requirements)
  * [Strawman Solution](#strawman-solution)
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

To enable developers to access common functionality at runtime that is provided by the JavaScript engine a
notion of a standard library has to be created. This will allow code currently loaded through libraries as
part of the program to shift towards being bundled with JavaScript implementations in the future.

> Disclaimer: This proposal covers adding a mechanism for enabling a standard library in JavaScript
> implementations, it does not describe its contents. This is considered a tangential effort that would
> be built and expanded upon in the future when a mechanism for accessing it is in place.

<!-- Availability & Standardization -->
Having a standard library readily available at runtime means programs won't have to include the functionality
in their libraries reducing the download and parse cost of the program. The functionality will also be
standardized across implementations giving the developer guarentees about quality, behavior and speed.

<!-- Extensibility -->
Although JavaScript engines already have the notion of built-ins the standard library will use modules and the
`import` syntax to access it. This mechanism should already be familiar to developers and will allow them to
opt-in to parts they need. Using modules also allows more flexibility when designing the library contents and
helps avoiding conflicts with global APIs.

## Requirements

- A namespace must be reserved
- Prototypes of imported objects are frozen

Still exploring:

- Some form of versioning must be supported to avoid conflicts
- There must be a way to provide fallback implementation (polyfilling)
  - For missing features
  - For older implementations

## Strawman Solution

### Protected Namespace

A protected namespace is reserved for the standard library that can be used to import functionality. Some examples could be:

- `"std:foo:bar"`
- `"std://foo/bar"`
- `std.Foo.Bar`

> Question: How does versioning fit into this?

### Import Semantics

Using a feature from the standard library can be done through a regular import from the protected namespace:

```js
import { Date } from "std:Date";
import { Date } from "std:Date+2.1.6-alpha.1";


const d = new Date(2018, 7, 1);
```

> Describe that the prototype is frozen, and what we mean by that

It's our intent that modules in the standard library are pure (free from side-effects) as a policy, even though there are no limitations in the spec to enforce this.

### Generic Functions

The advantage of using regular imports is that it allows the development of more generic functions in the standard library. Examples that could be thought of (and exist in other languages like Python) could be:

- `len`
- `map`

```js
import { len, map } from "std:builtins";

const ar = [1, 2, 3];
const st = new Set([4, 5, 6, 7]);
const mp = new Map([["e", 8], ["n", 9], ["z", 0], ["o", 1], ["t", 2]]);

len(ar); // => 3
len(st); // => 4
len(mp); // => 5

map(ar, (val, idx) => val * 2); // => Array {2, 4, 6}
map(st, (val, idx) => val * 2); // => Set {8, 10, 12, 14}
map(mp, (val, key) => val * 2); // => Map { e: 16, n: 18, z: 0, o: 2, t: 4 }
```

### Polyfilling and Versioning

Acknowledge that this is something that needs to be incorporated.

> Question: How would this work exactly?

## Related Efforts

This topic has been talked about a lot in the past. There has been some recent activity around it as well:

- <https://github.com/drufball/layered-apis>
- <https://github.com/tc39/ecma262/issues/395>
