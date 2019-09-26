---
title: "Object Initializer for PHP"
date: 2019-09-26T19:23:14+02:00
description: |
    Concept of the Object Initializer Expression as a simplification of object 
    instantiation and properties initialization 
type: post
tags: 
- "class"
- "object"
- "initializer"
categories:
- "Programming"
---

In this article, we're gonna demystify 
[RFC: Object Initializer](https://wiki.php.net/rfc/object-initializer)
which was proposed by me up for discussion on PHP Internals.

## Motivation

The motivation behind Object Initializer RFC was dictated by a large number of 
repetitions when creating simple objects like 
_<abbr title="Data Transfer Object">DTO</abbr>_ and 
a lot of noise appearing by constantly repeating object instance variable 
name in statements which tends to assign public properties only.

## Current PHP language state

Let's have a look at the following example which uses 
[typed properties](https://wiki.php.net/rfc/typed_properties_v2):

```php
class Customer
{
    public string $firstname;
    public string $surname;
    public string $phoneNo;
    public string $emailAddress;
    public DateTimeImmutable $dateOfBirth;
}
```

Above example represents `Customer` _<abbr title="Data Transfer Object">DTO</abbr>_.

To create an instance of `Customer` and initialize all properties values we need
to make few statements to property initialize object.

```php
$customer = new Customer();
$customer->firstname = "John";
$customer->surname = "Doe";
$customer->phoneNo = "+1 (555) 333 222";
$customer->emailAddress = "john.doe@example.com";
$customer->dateOfBirth = DateTimeImmutable::createFromFormat("Y-m-d", "1983-01-01");
```

As we all can observe in above snippet there are many repetitions of `$customer->`
part. This also requires to instantiate an object first which is in the invalid state 
from the beginning. Why? Cause our properties are typed and not accepting `null` 
values which mean their default value cannot be `null` like in non-typed properties.
This requires 6 statements, one for instantiation and 5 for properties initialization. 

In current PHP version, we'll have to write all property assignment statements 
as a separate right after object instantiation and the only reason for that 
is a verbose language which lacks any construct reducing that boilerplate.

Right now this is a responsibility of the programmer to initialize that kind of 
object properly. 
There is no support from PHP language which tells programmer:

> Hey, you've created an object of class Foo and you're missing bar 
> and baz properties values!

## What is the cure?

The Object Initializer is basically a simplification of object instantiation 
and properties initialization. 
This means that Object Initializer itself creates an instance of a class 
and right after that assigns all visible and required properties values.

With a proposed solution we'll be notified by proper `RuntimeException` that 
we've not fulfilled all class properties needs.

```php
$customer = new Customer {
    firstname = "John",
    surname = "Doe",
    phoneNo = "+1 (555) 333 222",
    emailAddress = "john.doe@example.com",
    dateOfBirth = DateTimeImmutable::createFromFormat("Y-m-d", "1983-01-01")
};
```

Above snippet shows the more or less equivalent of the previous example with 
repetitions.

Why less? Well, it ensures that after the given expression evaluates 
_(which is one expression)_ resulting `Customer` object is in the right state
and all properties which require values are filled.

Let's take a look at what happens in details under the hood:

1. it creates `Customer` class instance calling zero-args
   constructor - the restriction on constructor says there must not be any required
   arguments which have no default value;
2. it assigns all listed properties with values standing on the opposite side of
   `=` assign operator.
   
The order of calling constructor before or after object initialization 
was discussed and calling constructor after initialization is just unusual 
to most of the programming languages.

The feature is known in other programming languages like Java and C# 
and the order of calling constructor there is the same.

The restriction put on Object Initializer requires to initialize all properties 
which are required and don't have a default value. 
In other words, if a property is untyped (there is no type declared within 
the property) it's default value would be `null`, but if the same property has 
a type declaration and doesn't accept `null` as a proper value, it has to be 
initialized through Object Initializer.

In general, The Object Initializer works with public properties cause we often use 
instantiation and initialization in outer scope of class which is used.

But, as noted earlier it is a simplification over separate statements which assign 
values, so it is possible to initialize "visible" properties. 
This means if used from class scope we can initialize all properties which 
implementing class has access to either `protected` or `private`.

The syntax is directly borrowed from other referencing programming languages. 
For arrays, it is square brackets but for objects, it was natural to choose 
curly braces.

There was a question on PHP Internals about assign token used either it 
should be a single equal sign `=` as it is already used to initialize property 
values in assign statement or it should be double arrow `=>` used in arrays syntax.
That question will be taken up for additional voting.

## Conclusion

Opinions differ in reception. There is support in favour of accepting this 
feature but there were also some comments mostly objecting the Object Initializer 
in favour of:

* _named arguments_ which are a different feature allowing to pass constructor 
  arguments followed by their names - this feature would potentially be very much 
  promising but not in this specific case which we're trying to solve here, 
* _constructor argument promotion_ - the feature known from Hack which allows 
  promoting constructor arguments as properties automatically, 
  
They're both good when the number of arguments in the constructor is limited but 
potentially will increase the noise introduced when used in simple "structs" 
like objects known as _<abbr title="Data Transfer Object">DTO</abbr>_'s.

Object Initializer tries to solve the issue of instantiating and 
initializing properties values were protecting them from change is not 
a real requirement. 
In objects which only transfers data, these are just typed containers for 
a bunch of values and the only requirement for them is type safety. 
This can be assured in different manners either it is a simple scalar value 
or specified _<abbr title="Value Object">VO</abbr>_ which ensures data 
correctness.
