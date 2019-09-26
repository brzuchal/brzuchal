---
title: "Packages in PHP - Syntax conception"
date: 2019-08-12T19:01:12+02:00
description: |
    Concept of Packages in PHP - idea and thoughts about the way of 
    implementing them inside PHP
tags: 
- "package"
- "module"
categories:
- "Programming"
---
# Introduction

Following recent post 
[comparing package module concept supported by other languages](/posts/packages-in-programming-languages/)
I took an opportunity to gather my thoughts and write own vision of syntax for that concept in *PHP*.
 
Currently in *PHP* package term is used only by *Composer* - tool for dependency 
management - the only thing about such use is that the package itself is
a virtual thing living only for dependency management and has no meaning at 
runtime.

In languages like: *Python*, *Java*, *Rust*; module
has name which is also a part of symbol path name used to resolve specific
module symbols. 

* in *Rust* modules -
when `use foo::bar;` is used `bar` symbol from `foo` module is imported to local symbol table.
All symbols with no explicit visibility are considered private thus public symbols require 
explicit `pub` visibility modifier so they can be imported through `use` declaration.

* in *Python* modules -
when `from foo import bar` is used symbol `bar` from `foo` module is imported to local symbol table.
It is possible to import all desired symbol names using `__init__.py` with `__all__` 
table which includes all symbols to import using `from foo import *` statement.

* in *PHP* namespaces -
when `use Foo\Bar;` is used class `Foo\Bar` is used to resolve usage of `Bar` class in code.
Although *PHP* use declaration looks similar to other languages, namespaces in *PHP* are simple 
prefixes with no meaning.

## Module identity.

Looking at given other language examples can observe that part of import/use declaration uses 
module name as a prefix.
This doesn't happen in *PHP* cause namespaces are thin object prefixes to do that extracting
package from use declaration is needed and cannot be done without providing an additional
FQN delimiter which splits module / package name from symbol name while resolving symbol names
and additional package declaration in each file consisting with proper package name.

Using `package` declaration in source file could give parser desired package name 
and symbol name, for eg.:
```
<?php
package Foo;
class Bar {}
const FOO_BAR = 1;
function foo(Bar $bar) {}
```

Looking for proper delimiter a `:` could be a good example for that.
For eg.:
```
<?php
use Foo:Bar;
use const Foo:FOO_BAR;
use function Foo:foo;

$bar = new Bar();
foo($bar);
echo FOO_BAR;
```
This concept is nearly compatible with current *PHP* syntax and provides no breaking change.
There could be an issue when trying to use group `use` syntax simply looks weird.
```
<?php
use Foo: { Bar, Baz };
```

Other concept could follow Python semantics:
```
<?php
from Foo use Bar;
from Foo use const FOO_BAR;
from Foo use function foo;

$bar = new Bar();
foo($bar);
echo FOO_BAR;
```

Going further might wanna try with grouping `from` statement, for eg.:
```
<?php
from Foo {
    use Bar;
    use const FOO_BAR;
    use function foo;
}

$bar = new Bar();
foo($bar);
echo FOO_BAR;
```

Although using `from` keyword to indicate package to reflect on looks pretty neat we
still have an issue when trying to resolve FQN name without explicit `use` clause.
```
<?php
$bar = new Foo:Bar();
Foo:foo($bar);
echo Foo:FOO_BAR;
```
This is the case where a `:` delimiter may help and this is already considered syntax 
error so won't break already existing code.

# Conclusion
When looking for proper package syntax in use this looks quite easy to achieve.
The issue could be in a way of declaring package either it would be through a 
specified language construct like in *Python* or *Java*
or by sort of a separate configuration file like in *Rust*

IMHO it has very big potential and proposes almost non-invasive transformation way.
