---
layout: post
title:  "Php how to use this in closure context matters"
date:   2015-11-21 12:00:00 +0800
categories: [coding, laravel, php]
---
# PHP how to use $this in Closure – context matters

Anonymous functions, in php known as [Closures](https://www.php.net/manual/en/class.closure.php), come in handy very often. One application in particular is very useful – extending classes, thanks to the capability to bind the closure to an instance and class scope.

However, binding to instance may prove a bit tricky, so let’s go through it together.

Let’s start with a simple example:

```php
<?php
 
class Foo
{
  private $prop = 'bar';
}
 
$closure = function () { 
  return $this->prop;
};
 
$closureInFooScope = $closure->bindTo($foo, get_class($foo));
```

Unfortunately above code will show following error Warning: Cannot bind an instance to a static closure.

While it’s rather unlikely that you use it in a static context like above, but chances are you do something like this:

```php
// a trait making class extendable
trait Macroable
{
  public static function macro($name, $callback) { ... }
  public function __call($method, $params) { 
    // $callback->bindTo($this) is called here
  }
}
 
class Foo
{
  use Macroable;
 
  private $firstName;
  private $lastName;
 
  public static function boot()
  {
    static::macro('fullName', function () {
      return $this->firstName . ' ' . $this->lastName;
    });
  }
}
```

OK, it’s not very sophisticated example, but you get the idea already (you can find real use-case in eg. Laravel Metable). Here, the closure is created again in a static context and it makes perfect sense in this case, but it means we’ll  get the same error as soon as macro is called: $foo->fullName().

Important thing to remember though is that php doesn’t care about the context, in which we invoke the process but only where the closure is created. That being said, a workaround for this is as simple as having additional class, providing instance context:

```php
2
3
4
5
6
7
8
9
10
11
	
 
class ClosureFactory
{
  public function getClosure()
  {
    return function () {
      return $this->firstName . ' ' . $this->lastName;
    };
  }
}
```

then we can rewrite our boot method:

```php
public static function boot()
{
  static::macro('fullName', (new ClosureFactory)->getClosure());
}
```

and make it work!

 

Having a bindTo method that in fact can’t do its job is a huge drawback in php 5.x, but we can easily work around the problem.

Wait, did I say php5? Right, because awaited and upcoming PHP7 fixes it. 
Speaking of, worth reading walkthrough to the Seven: [oreilly.com/web-platform/free/files/upgrading-to-php-seven.pdf](https://oreilly.com/web-platform/free/files/upgrading-to-php-seven.pdf)
