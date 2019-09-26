---
title: "Simple Dic With Object Typehint"
date: 2016-08-21T22:00:00+00:00
description: |
    Simple DIC implementation based on proposed RFC:Object-typehint 
    implementation to providing a proof-of-concept and real life example 
    of usage
tags: 
- "dependency-injection"
- "object"
categories:
- "Programming"
---

Following the raging success of _PHP_ 7.0 additions scalar type hints and 
return types, there is also place for object typehint and return type. 
That feature can replace double `is_object()` validation inside 
function/method. I decided to write an 
[RFC:Object-typehint](https://wiki.php.net/rfc/object-typehint).

## Why do we need this?
Function and method typehints allow to declare expected sort of scalar 
type or class/interface but there is no way to declare any object type. 
Function and method return type allow to declare expected sort of scalar 
type or class/interface but there is same situation impossible to declare 
any object type. 
Those two (typehint and return type) should exists both for consistency, 
allowing typehint should able to declare same return type.

## How does it help in DIC

In all well known IoC pattern and it's implementations like Dependency Injection 
Container - The DependencyInjection component allows you to standardize 
and centralize the way objects are constructed in your application.

There are also many other implementations like Pimple or Zend\DI.

The main functions of simple DIC is setting service definitions and 
retrieving already prepared services, even complex instances with their 
internal dependencies.

Let's have a look on a simple interface.

```php
interface ServiceContainer {
  /**
   * Set service definition
   * @param string $id Service identifier
   * @param object $service Service object or closure
   */
  public function set(string $id, object $service);
  /**
   * Retrieve service object from container
   * @param string $id Service identifier
   * @return
   */
  public function get(string $id) : object;
}
```

I've putted here additional type-hint and return-type to protect any 
implementation to avoid implementing non-object services.

Why an object type-hint? Because in PHP we can catch here any objects 
which also are closures. They are simply instances of `\Closure` class 
and this kind of magic is made under the hood when we type for eg.:

```php
$outerScopeVariable = true;
$closure = function () use ($outerScopeVariable) {
    // my impl...
};
```

As we know many DIC implementations use closures for lazy 
loading/instantiating our services.

With additional return-type we're sure that PHP will do the magic of 
result validation - those results are validated against function/method 
declaration return-type on runtime with returned value.

Simple DIC implementation may look like this.

```php
class Container implements ServiceContainer {
  /** @var object[] */
  private $services = [];
  /**
   * {@inheritdoc}
   */
  public function set(string $id, object $service) {
    $this->services[$id] = $service;
  }
  /**
   * {@inheritdoc}
   */
  public function get(string $id) : object {
    // some assertion
    $service = $this->services[$id];
    if ($service instanceof Closure) {
      return $service();
    }
    return $service;
  }
}
```

This peace of code gives us partial functionality because any `get()` 
method call can return new new instances each time, but that was not 
purpose of this example. The significant improvement is object type-hint 
and return-type.

## Usage

Simple usage could look like this one.

```php
$container = new Container();
$container->set('std', new stdClass());
$container->set('lazy', function () use ($container) : object {
  return $container->get('std');
});

var_dump(
    $container->get('std'),
    $container->get('lazy')
);
```