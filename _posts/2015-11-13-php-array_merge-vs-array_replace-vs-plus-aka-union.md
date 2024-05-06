---
layout: post
title:  "Php arraymerge vs arrayreplace vs plus aka union"
date:   2015-11-13 12:00:00 +0800
categories: [coding, laravel, php]
---
# array_merge vs array_replace vs + (plus aka union) in PHP

If you wrote some PHP, most likely you have used [array_merge](https://www.php.net/manual/en/function.array-merge.php) here and there for your arrays. You may have met [array_replace](https://www.php.net/manual/en/function.array-replace.php) function, introduced in PHP5.3. Finally, you could notice [+ operator (aka union)](https://www.php.net/manual/en/language.operators.array.php).

Since all 3 do similar thing, and the docs don’t quite describe the difference between them, here’s a nice image of it

There are a few things to consider:
1. `array_merge` and `array_replace` work just the same for **keyed (associative) elements**, so they can be used interchangeably:
    ```php
    // associative arrays
    array_replace($a, $b) === array_merge($a, $b)
    ```
2. `array_replace` and `+` do the opposite always:
   ```php
    // numeric arrays
    array_replace($a, $b) === $b + $a

    // associative arrays
    array_replace($a, $b) == $b + $a // equal, but not identical ===
    ```
3. `array_merge` behaves differently to the other 2 with numeric arrays:
    ```php
    // numeric arrays
    array_replace($a, $b) != array_merge($a, $b)
    ```

That being said, we could think of the + operator as kinda redundant, since the same can be achieved with array_replace function.

However, there are cases, where it comes in handy: say you have an $options array being passed to a function/method, and there are also defaults to be used as fallback:

```php
// we could do it like this
function foo(array $options)
{
   $defaults = ['foo' => 'bar'];
   
   $options = array_replace($defaults, $options);

   // ...
}

// but + here might be way better:
function foo(array $options)
{
   $options += ['foo' => 'bar'];

   // ...
}
```
