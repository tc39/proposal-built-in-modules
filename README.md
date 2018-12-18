# Standard Library Proposal

**Status:** Stage 1

This proposal describes adding a standard library to JavaScript that holds a set of APIs that can be used at runtime. It also describes how the standard library can be used by developers, what APIs could be part of the standard library and how it can be extended/evolved over time.

## Motivation

When using other programming languages the developer gets the core features defined by the language (syntax, operators, primitive types etc.) and usually a (sometimes exhaustive) standard library that can be leveraged for common functionality. These languages can make certain assumptions on the environment the code is run in like which language features are supported and what APIs are available in the standard library.

Because of the distributed nature of the Web Platform and JavaScript code running on varying clients, these assumptions cannot be made when programming for the Web. Code has to be downloaded to clients, incurring a network and parse cost and only after the code is running can features be detected in a (sometimes) cumbersome way. This results in pages or applications including libraries with abstracted common functionality (jQuery, Lodash, Ramda), libraries to do feature detection and polyfilling (core-js, et al) or compiling down to the lowest common feature set for their targeted users (using TypeScript or Babel).

These are all detrimental for both the developer and the end user. Evolving the JavaScript language through the prototype chain adding new (standard library) methods can become problematic over time forcing a fallback to less ideal features because of web compatibility issues.

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
const mp = new Map([["e", 8], ["n", 9], ["z", 0], ["o", 1], ["t", 2]);

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
