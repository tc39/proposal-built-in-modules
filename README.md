# Standard Library Proposal

Proposal for adding a mechanism for enabling a more extensive standard library in JavaScript. With
this infrastructure in place it will be possible to start iterating on standard
library features as modules in the future.

<details>
  <summary><strong>Table of Contents</strong></summary>

  * [Scope](#scope)
  * [Motivation](#motivation)
  * [Benefits](#benefits)
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


## Benefits

### **Availability**

Pages and applications don't have to include shared libraries anymore which helps with download size and speed.

### **Standardization**

With the introduction of a standard library developers will get a well-defined API that does not have to be included with their pages or application. The functionality of the standard library will have gone through a standardization track and will have well-defined APIs and behavior.

### **Extensibility**

> FIXME This point needs work

The standard library provides a great location for future extensions to the language and the toolkit available to developers. Because the functionality is added to a reserved space there is less chance of conflicts and all additions have gone through a standardization track. It also provides a space where features can be developed that are not bound to the prototype, a generic `len` or `map` function for example that works on multiple types.

The standard library furthermore has versioning built into it to avoid conflicts.

### **Fallback Mechanism**

Importing features from the standard library has a fallback mechanism that can be used to override existing features if the version does not match or provide entire implementations when they are not available. The standard library can facilitate in feature detected to avoid cumbersome runtime checks (using error handling for example).

### **Speed**

Features need to be explicitly imported from the standard library when used, using the module syntax. This allows engines to only load parts of the standard library, setting up prototype chains when actually used. This allows engines to skip doing this at startup and only load parts that are actually used.

> Question: Is this a weak argument? Engines already do lazy startup shenanigans

Some standard library functions can be done in regular JavaScript, which is currently already the case. Moving functionality from external libraries (like Lodash or Ramda) allows them to tap into native code (C++) for heavy lifting and performance-conscious code.

Pages and applications furthermore don't have to include shared libraries anymore which helps with download speed and parse speed.

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
