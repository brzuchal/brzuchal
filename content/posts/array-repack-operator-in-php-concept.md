---
title: "Array Repack Operator in PHP Concept"
date: 2017-05-17T12:00:00+00:00
description: |
    Concept of new operator concept for PHP which helps constructing new 
    arrays with selective copy from source array
tags: 
- "Arrays"
categories:
- "Programming"
---
Presenting concept of array operator which repacks array by keys to 
newly created array. 
I've called it _Array Repacking_ because such functionality will 
produce new array based on original array just with some few improvements.

## Syntax

Proposed syntax doesn't break any current syntax upon _PHP_ 7.1.

```php
$strings = ['one' => 1, 'two' => 2, 'three' => 3];
$onlyThreeAndTwo = $strings['three', 'two'];

var_dump($onlyThreeAndTwo);
//array(2) {
//  ["three"]=>
//  int(3)
//  ["two"]=>
//  int(2)
//}
```

As we can observe array `$strings` is being repacked with only two keys 
three and two with different order and returns new array which is stored 
in `$onlyThreeAndTwo`.

## Benefits

This behaviour can be useful when we need to cut off some values from array. 
It's kinda shorthand for:

```php
$strings = ['one' => 1, 'two' => 2, 'three' => 3];
$onlyThreeAndTwo = array_intersect_key($strings, array_flip(['three', 'two']));

var_dump($onlyThreeAndTwo);
//array(2) {
//  ["two"]=>
//  int(2)
//  ["three"]=>
//  int(3)
//}
```

Results are different because there is original keys order as in `$strings` 
table. 
In most cases it doesn't matter but there is one specific feature in _PHP_ 
which would actually benefit from this - it's called variadic function.

Variadic function can accept undefined number of arguments passed by adding 
`...` in front of function argument, for eg.:

```php
function sum(...$numbers) {
    $acc = 0;
    foreach ($numbers as $n) {
        $acc += $n;
    }
    return $acc;
}

echo sum(1, 2, 3, 4);
```

In above example there can be undefined number of arguments passed into 
function and all became accessible by `$numbers` array. 
But that's not the whole point I'm referring to variadic. 
There is also possibility to pass function arguments by array with `...` 
at function call.

```php
function add($a, $b) {
    return $a + $b;
}

echo add(...[1, 2])."\n";

$a = [1, 2];
echo add(...$a);
```

As we can see calling add an array with 1 and 2 was passed and those became 
values of `$a` and `$b` variable. 
Now imagine quite longer example where _Array Repack_ would help:

```php
class Service {}

class ServiceFactory {
    public function createService($name, $type, $scheme, $host, $port, $path) : Service {
        return new Service($name, $type, $scheme, $host, $port, $path);
    }
}

// and now we're going to repack entities into `Service` instances
$data = [
    [
        'id' => 156789,
        'host' => 'localhost',
        'port' => 80,
        'type' => 'auth',
        'scheme' => 'http',
        'path' => '/auth',
        'name' => 'authorisation'
    ],
];

$serviceFactory = new ServiceFactory();
foreach ($data as $service) {
    $serviceFactory->createService(...$service['name', 'type', 'scheme', 'host', 'port', 'path']);
}

// and without repacking it'll be 
$serviceFactory = new ServiceFactory();
foreach ($data as $service) {
    $serviceFactory->createService($service['name'], $service['type'], $service['scheme'], $service['host'], $service['port'], $service['path']);
}
```
