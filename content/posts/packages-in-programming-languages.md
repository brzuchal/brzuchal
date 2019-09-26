---
title: "Packages in programming languages"
date: 2019-08-12T19:00:20+02:00
description: |
    Condensed description about what are packages in programming languages
tags: 
- "package"
- "module"
categories:
- "Programming"
---
# Introduction

With modular programming, concerns are separated such that modules perform 
logically discrete functions, interacting through well-defined interfaces.

The module / package concept is formally supported by wide range of programming 
languages but many of them support package / module concept in different ways.

Some encapsulates symbols around single file, like for eg. *Rust*, *Python*;
other encapsulates symbols around wider scope like *Java* encapsulates modules 
around *Java Package*.

There are few key aspects around programming language support for 
module / package concept.

## Identity
Some languages use package name while resolving used / imported symbols, some not.
In *Rust* package name used by *Cargo* tool doesn't mean anything while in 
*Python* and *Java* it is a part of symbol name used while resolving symbols.

## Scope
There are programming languages differentiate meaning of module and package,
in *Python* scope of [module](https://docs.python.org/3/tutorial/modules.html#modules)
is a single file while in *Java* [module](https://www.oracle.com/corporate/features/understanding-java-9-modules.html)
adds a higher level of aggregation above packages - which are directory / namespace scoped units,
in *Rust* single [package](https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html)
is one or more crates that provide a set of functionality,
crates are library or binary and consist of
[modules](https://doc.rust-lang.org/rust-by-example/mod.html).

## Compiler directives
Language like *Rust* can compile source code written in two
different [editions](https://doc.rust-lang.org/edition-guide/editions/index.html) 
- more or less slightly different dialects through 
[`edition`](https://doc.rust-lang.org/cargo/reference/manifest.html#the-edition-field-optional) attribute.

## Symbol autoloading
In *Rust* there is a 
[`build`](https://doc.rust-lang.org/cargo/reference/manifest.html#the-build-field-optional), 
[`include` and `exclude`](https://doc.rust-lang.org/cargo/reference/manifest.html#the-exclude-and-include-fields-optional) 
package attribute, in *Python* there is [`__path__`](https://docs.python.org/3/reference/import.html#module-path)
attribute is used during imports of its subpackages.

## Exported symbol list
Used in *JavaScript* through 
[`export`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)
statement is used when creating JavaScript modules to export functions, objects, or primitive values from the module 
and *Java* through [`exports`](https://www.oracle.com/corporate/features/understanding-java-9-modules.html) 
module directive specifies one of the module’s `packages` whose `public` types (and their nested `public` and `protected` 
types) should be accessible to code in all other modules 
in *Python* namespaced module 
[`__all__`](https://docs.python.org/3/tutorial/modules.html#importing-from-a-package) 
attribute is used as a construct which instructs loading machinery which symbols to load.

## Required modules / packages
In *Rust* it is possible to specify 
[`dependencies`](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html) 
using package attribute, other languages like *Java* has 
a [`requires` and `requires transitive`](https://www.oracle.com/corporate/features/understanding-java-9-modules.html) 
module directive.

# Conclusion
Following above list of package / module key aspects might want to think of adopting 
similar concept in *PHP*.

At first what can benefit could potentially be:

* **identity** as a part of FQCN which allows identitying membership to explicit package;
* **scope** aggregating symbols with namespaces and in root package namespace;
* **compiler directives** this is the place where `declare` directives should exist;
* **symbol autoloading** could potentially be a good start point in restructuring current autoload machinery adding 
  support for other non-class symbols;
* **exported symbol list** this potentially is far future which could help providing package symbols visibility;
* **required packages** also far future, currently fulfilled by *Composer Packages*;
