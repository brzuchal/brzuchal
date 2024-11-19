---
title: "PHP Records vs Structs: Simplifying Immutability in PHP"
date: 2024-11-19T12:00:00+00:00
description: |
    PHP is exploring two approaches to immutability: *Records* for concise, lightweight value objects, and *Structs* with Copy-on-Write for performance. 
    *Records* reduce boilerplate, while *Structs* handle large data efficiently. 
    Both proposals, promise to enhance PHP's capabilities. Which approach suits your needs?
type: post
tags: 
- structs
- records
- data
categories:
- Programming
---
# PHP Records vs Structs: Simplifying Immutability in PHP

As a long-time PHP enthusiast and contributor, I’ve always been fascinated by how PHP can evolve to improve developer experience 
while maintaining its simplicity. Over the years, I’ve explored various ways to enhance the language, including my previously 
published [Object Initializer RFC](https://wiki.php.net/rfc/object-initializer) and a [blog post](https://brzuchal.com/posts/object-initializer-in-php/) 
discussing its potential impact. Additionally, I drafted an [early RFC on Structs](https://wiki.php.net/rfc/structs), 
which reflected my interest in improving PHP’s handling of immutable data.

The PHP Internals community is discussing two exciting proposals: **Records**, championed by Rob, and **Structs**, proposed by Ilija Tovilo. 
Both aim to bring value semantics into PHP, but with distinct approaches and goals. 
Ilija’s proposal has even progressed into a [Work-in-Progress PR on GitHub](https://github.com/php/php-src/pull/13800), showing active development. 
Let’s dive into these proposals, share my perspective, and highlight what they could mean for PHP.

---

## What Are Records and Structs?

At their core, both Records and Structs aim to simplify the creation of value objects in PHP. However, they differ in their goals and implementations.

- **Records** ([Discussion](https://externals.io/message/125975)): Focus on providing a concise syntax for defining value objects with value semantics, reducing boilerplate for common use cases.
- **Structs** ([Discussion](https://externals.io/message/122845)): Emphasize value semantics with **Copy-on-Write (CoW)** behavior, ensuring efficiency when dealing with large or nested structures. This approach is under active development, with a [WIP PR](https://github.com/php/php-src/pull/13800) available.

---

## Syntax Comparison

The syntax of Records is concise and minimal:
```php
record User(string $name, int $age);
```

In contrast, Structs use a class-like structure:
```php
struct User {
    public string $name;
    public int $age;
}
```

While both approaches define value objects, Records clearly reduce verbosity, making them ideal for simple use cases. Structs, on the other hand, offer more explicitness, which might be preferable for complex scenarios.

---

## Key Features of Records

### 1. Value Semantics
Records implement value-based semantics by default. This means two instances with the same properties are considered equal:
```php
$user1 = new User("Alice", 30);
$user2 = new User("Alice", 30);
var_dump($user1 === $user2); // true
```
This simplicity eliminates the need for manual equality methods.

### 2. Extensibility with Attributes
Attributes provide a flexible way to enhance functionality without adding complexity. For example:
```php
#[JsonSerializable]
record User(string $name, int $age);
```
This aligns well with modern PHP trends like enums and attributes.

### 3. Syntax Simplicity
The single-line definition of Records makes them perfect for small types like configuration objects or DTOs.

---

## Key Features of Structs

### 1. Copy-on-Write (CoW) Semantics
Structs optimize value semantics by sharing data until a modification is required. When a change occurs, only the modified data is copied:
```php
struct Line {
    public Point $start;
    public Point $end;
}

$line1 = new Line(start: new Point(0, 0), end: new Point(1, 1));
$line2 = $line1->with(end: new Point(2, 2));
```
This approach ensures performance efficiency for large or nested structures.

### 2. Explicit Equality
Unlike Records, Structs require developers to define their own equality logic, offering flexibility but at the cost of verbosity.

---

## Strengths and Weaknesses

| Feature                      | Records                  | Structs                   |
|------------------------------|--------------------------|---------------------------|
| **Syntax**                   | Concise and minimal      | Verbose but familiar      |
| **Equality**                 | Automatic by value       | Requires manual logic     |
| **Performance**              | Standard value semantics | Optimized with CoW        |
| **Extensibility**            | Attributes               | Traditional methods       |
| **Ideal Use Case**           | Small, lightweight types | Large, nested structures  |

---

## Community Opinions

The PHP community has provided valuable feedback on these proposals:

- **Larry Garfield**: Advocates for using the `new` keyword for Records, aligning them with existing PHP conventions. He also questions the need for a `\Record` interface, suggesting that instances could simply behave as regular objects.
- **Rowan Tommins**: Highlights the importance of supporting composition for more complex data structures, a capability that could be explored in future extensions.

For more details, you can follow the discussions on [Records](https://externals.io/message/125975) and [Structs](https://externals.io/message/122845).

---

## My Preferences

As someone deeply invested in PHP's evolution, I find myself leaning towards Rob's Records proposal for its simplicity, automatic equality, and alignment with modern PHP trends. However, I also recognize the value of Ilija’s Copy-on-Write semantics for performance-critical applications. A hybrid approach combining these strengths could offer the best of both worlds.

---

## Future Directions

The discussion around Records and Structs opens the door to exciting possibilities:

1. **Composition Support**: Enabling Records or Structs to support composition could allow for more reusable and complex data structures.
2. **Standard Attributes**: Attributes like `#[ValueObject]` or `#[Immutable]` could formalize and enhance the functionality of value types.
3. **Performance Optimizations**: Integrating CoW into Records for larger structures might strike a balance between simplicity and efficiency.

---

## What’s Next?

The PHP internals team and the community must weigh the trade-offs between these proposals. A **community vote** could help gauge overall preferences and provide clarity on the direction PHP should take.

As these discussions continue, it’s clear that both Records and Structs have the potential to significantly improve PHP’s handling of value data. The question now is not whether, but how, they will shape the future of the language.

---

**What are your thoughts?**  
Would you prefer the simplicity of Records or the performance benefits of Structs? Let me know in the comments or join the discussion on the PHP internals mailing list!
