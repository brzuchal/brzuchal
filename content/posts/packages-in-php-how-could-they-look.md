---
title: "Packages in PHP - How could they look?"
date: 2019-08-13T12:00:22+02:00
draft: true
description: |
    Concept of Packages in PHP - idea and thoughts about the way of 
    implementing them inside PHP
type: post
tags: 
- "inner-class"
- "class"
- "object"
categories:
- "Programming"
---

A new topic of package/module concept has arrived at PHP Internals 
mailing list once again. The concept of modules is known from other
programming languages like for eg. _Java_, _JavaScript_, _Python_ and _C#_.
Each of them has own implementation of scoping declared symbols but they
all follow basic concept of isolating symbols for internal scoped use.

In this article I'm gonna gather some principles and try to describe 
a proposal based on my own thoughts about that.

## What is a package/module?

According to Wikipedia modular programming is:

> __Modular programming__ is a software design technique that emphasizes 
> separating the functionality of a program into independent, 
> interchangeable __modules__, such that each contains everything necessary 
> to execute only one aspect of the desired functionality.
> -- <cite>Wikipedia</cite> 

This gives first though that it tends to be a thing which encapsulates
logically related and complete implementation of a program/library.
Going along this thesis a package should express:

* encapsulated and logically related implementation.
 
Looking for examples we may want to look at 
[doctrine/collections](https://packagist.org/packages/doctrine/collections) 
_Composer Package_ (in this context package meaning differs we'll ge to 
that later).
It has no dependencies and implements well known collections pattern.
The root namespace of this library is _Doctrine\Common\Collections_ and
we can see some classes and interfaces possible to use in PHP.

Going further with Wikipedia description we can read:

> A module interface expresses the elements that are provided and 
> required by the module. The elements defined in the interface are 
> detectable by other modules. The implementation contains the working 
> code that corresponds to the elements declared in the interface.
> -- <cite>Wikipedia</cite> 

This leads to two additional features which package may implement. 
Different programming languages implement them or not.
According to what Wikipedia says package express:

* publicly provided symbols which are detectable,
* required dependencies from other packages.








## Autoload

Great so we've figured out how to use package but we're missing the way to load them.

## Rust packages

Some languages like *Rust* use a separate file named `Cargo.toml` which declares a package.
It is used by *Cargo* tool shipped with *Rust* compiler.

```
[package]
name = "hello_world" # the name of the package
version = "0.1.0"    # the current version, obeying semver
authors = ["Alice <a@example.com>", "Bob <b@example.com>"]
edition = '2018'
build = "build.rs"
```

Following given example it's easy to find package name `hello_world` and 
some additional package metadata. While `version` and `authors` are easy to grasp
interesting is `edition` which is a compiler declare used in compilation process
indicating which *Rust* edition rules to follow on compile-time.

## Python module packages

On the other hand looking at *Python* there is a separate file as well but nothing
more than a set of automatically exported symbols and include path used to locate 
sources.
Python provides a very straightforward packaging system, which is simply an extension 
of the module mechanism to a directory.
A file `modu.py` in the directory `pack/` is imported with the statement 
`import pack.modu`. 
This statement will look for an `__init__.py` file in `pack` and execute 
all of its top-level statements. 
Then it will look for a file named `pack/modu.py` and execute all of its 
top-level statements. 
After these operations, any variable, function, or class defined in `modu.py` 
is available in the `pack.modu` namespace.

```
__all__ = ['modu', 'vehicle']
import modu
import vehicle
```

By definition, if a module has a `__path__` attribute, it is a package.

A package’s `__path__` attribute is used during imports of its subpackages. 
Within the import machinery, it functions much the same as `sys.path`, i.e. 
providing a list of locations to search for modules during import. 
However, `__path__` is typically much more constrained than `sys.path`.
It is no longer required attribute cause import machinery automatically sets
it for the namespace package.

That shows a language being used here to identify public symbols and 
a sort of autoloading feature.

## PHP concept

Currently cause of lack of packages *PHP* uses only class autoloading machinery.
It allows to load classes at run-time and there is a wide set of known approach
like PEAR style, PSR-0, PSR-2 etc.

Proposing package concept could use a similar approach triggering autoload callable
put on stack of autoloaders but for loading package definition.

This is a missing part which needs clarification should it be populated using 
a language construct or a separate file definition.

Many *PHP* libraries use *Composer Packages* concept which in a separate file
holds information about package name _( not related to included class names )_
and a metadata describing autoloader configuration used to generate an autolaoder
for all vendor packages.

There is an RFC for [Namespace scoped declares](https://wiki.php.net/rfc/namespace_scoped_declares)
authored by Nikita Popov with proper [PR](https://github.com/php/php-src/pull/2972)
proposing to introduce new function which tries to bind package name and declare
compiler directives like `strict_types`.

```
namespace_declare('Foo', ['strict_types' => 1]);
```

Although given approach describe an internal `zend_package` struct it is creating
an unrelated to symbol names package name.

In my opinion using a *Composer Package* names instead of FQCN part as a package name
will not be an obvious solution for packages.

On one hand at this point a way to declare package may be not so important cause
it can evolve and change until next *PHP* major version will be released, one thing
I disagree. Providing a way to explicit package name use in FQCN would be more 
clean and understandable way solving the way how package can be designed.

Some languages like *Rust* use a package path passed by *Cargo* tool directly to compiler
which directs compiler to locate sources od package. So introducing separate file
for package definition could use similar machinery, then all what we need is a way to 
load package definition by extracted from FQCN package name.

Even if not by sort of package path IMO is important to direct *PHP* interpreter 
where are all the package symbols, either it would be through package own autoloader
or by general purpose autoloader.

A purpose of that could be to help future features like package private/protected symbols
to simply recognise a context of use: either the same package or different or sort of 
a feature which groups all publicly available symbols in a package like export declaration does
in *Python* through `__all__` package attribute or like in *Java* export directive which
instructs compiler what symbols can be used out of package.