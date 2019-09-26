---
title: "Why Object Types are useful in Hydrators and Extractors"
date: 2016-08-29T07:48:07+00:00
description: |
    Object Type-hint described on examples based on Hydration pattern 
    from authored RFC 
type: post
tags: 
- "dependency-injection"
- "hydration"
- "object"
categories:
- "Programming"
---
There are many implementations of Hydrator and Extractor pattern. 
What is Hydration?

> Hydration is the act of populating an object from a set of data.

Hydrators have wide usage in ORM and Data Mappers their responsibility 
is to provide entity objects from relational database. 
Hydrators allow objects to be filled with data from an array. 
Extractors do the opposite, they take an object and return an array of data.

{{< figure src="/img/user-address-er-diagram.png" title="User Address ER diagram" >}}

General Hydration is described very well in _Zend/Std/Hydrator_ package 
which can be imported easily with _Composer_ by:

```bash
composer require zendframework/zend-hydrator:^3.0
```

## Hydration example

According to library implementation basic hydration has one method 
defined in `HydrationInterface`:

```php
/**
 * Zend Framework (http://framework.zend.com/)
 *
 * @link      http://github.com/zendframework/zf2 for the canonical source repository
 * @copyright Copyright (c) 2005-2015 Zend Technologies USA Inc. (http://www.zend.com)
 * @license   http://framework.zend.com/license/new-bsd New BSD License
 */
namespace Zend\Hydrator;
interface HydrationInterface
{
    /**
     * Hydrate $object with the provided $data.
     *
     * @param  array $data
     * @param  object $object
     * @return object
     */
    public function hydrate(array $data, $object);
}
```

As we can see method hydrate() declares two params array `$data` and 
`$object` which is used each time hydrator packs data from array into 
object.

## Extraction

Analog situation exists in `ExtractionInterface`:

```php
/**
 * Zend Framework (http://framework.zend.com/)
 *
 * @link      http://github.com/zendframework/zf2 for the canonical source repository
 * @copyright Copyright (c) 2005-2015 Zend Technologies USA Inc. (http://www.zend.com)
 * @license   http://framework.zend.com/license/new-bsd New BSD License
 */
namespace Zend\Hydrator;
interface ExtractionInterface
{
    /**
     * Extract values from an object
     *
     * @param  object $object
     * @return array
     */
    public function extract($object);
}
```

As we can see method `extract()` declares one param `$object` which 
is used each time hydrator extracts data from object into array.

## What about Object Types?

Currently in PHP version 7.1 doesn't support object type hinting and 
return type declarations. 
According to doc-block annotations valid input is object same as 
return type in case of `hydrate()` method and object type hint declaration 
in case of `extract()` method.

Providing object types as they're described in my 
[RFC:Object-typehint](https://wiki.php.net/rfc/object-typehint) could 
increase readability and provide runtime type checks in examined situations. 
Thats because in described use case it is common to handle arbitrary 
object types.

Having an object return type would allow some errors to be detected more 
quickly. 
The following function is meant to return an object. 
With the object return typ set for the function, failing to return an 
object would cause a `TypeError` to be thrown in the location where the 
bug is.

The extraction step can take an arbitrary object as the sole parameter. 
The hydration step can take an arbitrary object as the second parameter, 
and will return an arbitrary object. 
Having the type for the parameters and the return type be set as `object` 
will make the expected types be clearer to anyone using these functions, 
as well as detect incorrect types if there is an error in the code.

## Usecase improvement

Simple object type hint and return declaration can be provided 
in described interfaces like that:

```php
interface HydrationInterface {
  /**
   * Hydrate an object by populating data
   * @param array $data Data to populate
   * @param object $object Object to populate data
   * @return object
   */
  public function hydrate(array $data, object $object) : object;
}
interface ExtractionInterface {
  /**
   * Extract values from an object
   * @param object $object Object to extract data from
   * @return array
   */
  public function extract(object $object) : array;
}
class Hydrator implements HydrationInterface, ExtractionInterface {
  /**
   * {@inheritdoc}
   */
  public function hydrate(array $data, object $object) : object {
    // hydration implementation without need for `is_object` validation
  }
  /**
   * {@inheritdoc}
   */
  public function extract(object $object) : array {
    // extraction validation without need for `is_object` validation
  }
}
```