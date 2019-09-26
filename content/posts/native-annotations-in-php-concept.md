---
title: "Native Annotations in PHP concept"
date: 2019-05-20T15:37:57+02:00
draft: true
type: post
---

Annotations has been used in PHP since long time but they're still
not supported natively by _PHP_ language itself.

PHP offers only a single form of such metadata - doc-comments. 
In user-land, there exist some annotation reader libraries like 
_Doctrine Annotations_ which is widely used for eg. to express 
object-relational mapping metadata.


```php
class Foo
{
    /**
     * @MyAnnotation(myProperty="value")
     */
    private $bar;
}
```

The only thing needed to start working with them is installing package
through _Composer_ like this:

```bash
composer require doctrine/annotations
```

## Proposal

```php
use Uuid;
use ORM\Entity;
use ORM\Id;
use ORM\Column;
 
@Entity
@Table("foo")
class Foo {
    @Id @Column("id", type="uuid")
    private Uuid $id;
}
```

```php
use MVC\Route;
 
class FooController {
    @Route("/api/foo", methods=["POST"], name="foo_create")
    public function create(Request $request): Response
    {
        // specific implementation
    }
}
```
