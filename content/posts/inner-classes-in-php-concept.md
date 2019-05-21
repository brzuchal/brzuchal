---
title: "Inner Classes in PHP Concept"
date: 2016-11-24T22:37:35+00:00
description: |
    Concept of Inner Classes in PHP - idea and thoughts about the way of 
    implementing them inside PHP
tags: 
- "Classes"
- "Object"
categories:
- "Programming"
---
I'd like to present my concept of inner classes for PHP based on other languages and _PHP_ limitations.

What are those inner classes?

{{< figure src="/img/nested-inner-class-uml.jpg" title="Nested Inner Class UML" >}}

> In object-oriented programming (OOP), an inner class or nested class is a class declared entirely within the body of another class or interface. It is distinguished from a subclass. 
> -- <cite>Wikipedia</cite>

So basically inner class is a class that is declared inside the scope of another class. 
A nested class has access to the variables and other symbols of the classes it is nested inside.

In other languages there are examples of inner classes adoptions. 
They all have little different point of view. 
Firstable and most complex inner classes exists in _Java_. 
There are _Static Nested Classes_ and _Inner Classes_. 
Also there are two special kinds of inner classes: 

* `Local Classes`
* `Anonymous Classes` (they exists in _PHP_ from version 7.0). 

Similar to _Java_ there are also inner classes in _D_ language. 
Other non significant adoptions are _Ruby_ inner classes which only adds class prefix in name 
exactly same as using module and in _Python_ they are somewhat uncommon and doesn't automatically 
imply any sort of special relationship between the classes.

## Are there any benefits?

Compelling reasons for using nested classes include the following:

It is a way of logically grouping classes that are only used in one place: 
If a class is useful to only one other class, then it is logical to embed it in that class and 
keep the two together. 
Nesting such helper classes makes their package more streamlined.

It increases encapsulation: Consider two top-level classes, `A` and `B`, where `B` needs access to 
members of `A` that would otherwise be declared private. 
By hiding class `B` within class `A`, 
A's members can be declared private and `B` can access them. 
In addition, `B` itself can be hidden from the outside world.

It can lead to more readable and maintainable code: Nesting small classes within top-level 
classes places the code closer to where it is used.

The main advantage is organization and encapsulation. 
That allows your code to be even more object-oriented than it would be without inner classes. 
Anything that can be accomplished with inner classes can be accomplished without them using 
traditional classes with little more boilerplate in case of publicly accessible inner classes.

## How could it look like?

Given example presents inner class example as it may be implemented in _PHP_.

```php
class Person
{
  private interface Name
  {
    public function __construct(string $name);
    public function __toString() : string;
  }

  private class FirstName implements self\Name
  {
    private $name;

    public function __construct(string $name) {
      // validation
      if (strlen($name) < 3) {
        throw new InvalidargumentException(
          "Firstname requires min 3 letters long, given: {$name}"
        );
      }
      $this->name = $name;
    }

    public function __toString() : string {
      return $this->name;
    }
  }

  private class SecondName implements self\Name
  {
    private $name;

    public function __construct(string $name) {
      // it would be reasonable to have access to owning object
      // but `parent` already taken by super class
      // shouldn't `parent` be renamed to `super`!?
      // then `$parent` could be owning object `$this`
      if (is_null($parent->firstName)) {
        throw new RuntimeException(
          "It is impossible to have secondname without firstname"
        );
      }
      // validatiojn
      if (strlen($name) < 1) {
        throw new InvalidargumentException(
          "Secondname requires min 1 letter long, given: {$name}"
        );
      }
      $this->name = $name;
    }

    public function __toString() : string
    {
      return $this->name;
    }
  }

  /** @var self\FirstName */
  private $firstName;
  /** @var self\SecondName */
  private $secondName;

  public function __construct(string $fname, string $sname) {
    $this->firstName = new self\FirstName($fname);
    $this->secondName = new self\SecondName($sname);
  }

  public function getName() : string {
    return "{$this->firstName} {$this->secondName}";
  }
}

$me = new Person("Michał", "Marcin");
echo $me->getName(); // "Michał Marcin"
```

In above example we can see an interface called Name as all names are constructed from string 
and need `__toString()` method implemented.

Most people have more than one name and their validation rules may differ. Let's say firstname 
should be more than 3 letters and secondname more than one which is implemented inside inner 
class `FirstName` and `SecondName` implementing `Name` interface. 
As is observable the interface was declared as `self\Name` and such syntax ensures clear type 
name which points that this interface belong to owning class `Person`. 
Additionally self is special keyword inside class and it is in this case. 
Note self should then be restricted namespace name to avoid ambiguity.

Top class Person use inner classes in it's `__constructor()` to instantiate `Name` objects and 
store them in properties for usage in `getName()` method. 
Note tat inner classes and interface have private modifier in their declarations - that means 
any instance used or returned outside Person object should cause sort of fatal error at runtime.

That implies our inner classes and interfaces need to have PPP(public, private, protected) 
visibilities so they can be useful. 
When extending `Person` class it is reasonable to change private modifier to protected one so 
extending class will be able to deal with inner class instances.

## What about publicly accessible inner classes?

They may have wider use. For example presenting some related objects like stock and options. 
This example presents only concept.

```php
class Stock implements IteratorAggregate
{
  public class Option
  {
    private $name;
    private $value;

    public function __construct(string $name, int $value)
    {
      if (strlen($name) < 3) {
        throw new InvalidArgumentException(
          "Stock option name requires min. 3 letters long, given: {$name}"
        );
      }
      if ($value <= 0) {
        throw new InvalidArgumentException(
          "Stock option must have positive value, given: {$value}"
        );
      }
      $this->name = $name;
      $this->value = $value;
      // if there were access to owning `$this` by `$parent`
      // then could automatically add to options table
      $parent->options[] = $this;
    }
  }

  private $options = [];

  public function getIterator() {
    return new ArrayIterator($this->options);
  }
}
$stock = new Stock();
$option = new $stock\Option("MAD", 1500);
var_dump($stock);
// class Stock#1 (1) {
//   private $options =>
//   array(1) {
//     [0] =>
//     class Stock#1Option#2 (2) {
//       private $name =>
//       string(3) "MAD"
//       private $value =>
//       int(1500)
//     }
//   }
// }
$incorectlyNamedOption = new $stock\Option("a", 1);
// InvalidArgumentException: Stock option name requires min. 3 letters long, given: a
$incorectValueOption = new $stock\Option("ORA", -1);
// InvalidArgumentException: Stock option must have positive value, given: -1"
```

In above example Option have public visibility but cannot be instantiated without owning class instance that's why syntax creating new object out of Stock scope is little weird:

```php
$stock = new Stock();
$option = new $stock\Option("MAD", 1500);
```

Giving `$stock` in front of class name indicates that `Option` class belongs to Stock as inner class.

## Type hint and return type

Every inner class and interface should be accessible to use on type hint and as return type limited to owning class with restrictions on returning non-public type in public methods.

Little weird syntax plays on our favor because it is not possible to address `self\Option` outside `Stock` class as it would point to different scope because of self prefix. Also syntax with variable prefix `$stock\Option` do the same job so we can avoid autoloading the same way. The reason of inner classes need to have such syntax is avoiding non predictable behaviour outside owning class.

Restrictions:

* forbid serialization exactly the same as anonymous classes (`PHP Fatal error: Uncaught Exception: Serialization of 'class@anonymous' is not allowed`) - there is need for serialization in case of serializing whole `Person` or whole `Stock` class instance
* forbid type hint and return type on inner classes or interfaces outside owning class - they may not be stored as normal classes in `class_table` internally.