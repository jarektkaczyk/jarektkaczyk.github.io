---
layout: post
title:  "How to add custom casters in Laravel Tinker"
date:   2018-03-04 12:00:00 +0800
categories: [laravel, tinker, psysh, php, repl]
---
I made a series about psysh and Laravel’s tinker – powerful REPL where you can play with your whole application.

ICYMI check it out here: [tinker like a boss](https://softonsofa.com/tinker-like-a-boss-in-psysh/) and here’s example of what we can achieve:


```php
=> Carbon\Carbon @1519569240 {#745
 +date: "2018-02-25 14:34:00.000000",
 +weekday: "Sunday",
 +day_of_year: 55,
 +unix: 1519569240,
 +diff_for_humans: "6 days ago",
 }
```

There is, however, missing piece of the puzzle in the series. It describes how you can custimze presenters in REPL (called casters in PsySH), but I describe generic, global casters that reside in PsySH config. This is very handy, but you cannot distribute them among your team members (or anyone else for that matter) in the repo.

Today we are going to add that piece to the puzzle and see **how to customize and override psysh casters in Laravel app**.  

We need just a few simple steps:

1.  Override tinker command
2.  Override service provider
3.  Create our casters and define them in the application config

### Override original ServiceProvider and Command

First let’s create our customized Command, which will allows us to provide array with customized casters:

```php
namespace App\Console;

class TinkerCommand extends \Laravel\Tinker\Console\TinkerCommand
{
    protected array $casters = [];

    /**
     * Set custom casters for this tinker session.
     *
     * @param array $casters
     */
    public function setCasters(array $casters)
    {
        $this->casters = $casters;

        return $this;
    }

    /**
     * Get an array of custom and Laravel's own casters.
     *
     * @return array
     */
    protected function getCasters()
    {
        return array_merge(parent::getCasters(), $this->casters);
    }
}
```

Next step is to use this command instead of the original – we can achieve that by replacing the service provider:

```php
// in config/app.php replace the line:
Laravel\Tinker\TinkerServiceProvider::class,

// with:
App\Providers\TinkerServiceProvider::class,

// and create the your own provider:
class TinkerServiceProvider extends \Laravel\Tinker\TinkerServiceProvider
{
    /**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(
            'command.tinker',
            fn () => (new \App\Console\TinkerCommand)->setCasters(
                config('tinker.casters', [])
            )
        );

        $this->commands(['command.tinker']);
    }
}
```

### Create and register casters

Finally, let’s create example caster and set it up in the `config/tinker.php` configuration file:

```php
// config/tinker.php
return [
    'casters' => [
        // Either inline function:
        Carbon\Carbon::class => fn (Carbon\Carbon $carbon) => [
            'date' => $carbon->date,
            'weekday' => $carbon->format('l'),
            'day_of_year' => (int) $carbon->format('z'),
            'unix' => (int) $carbon->format('U'),
            'diff_for_humans' => $carbon->diffForHumans(),
        ],
        // OR class-based static method:
        Carbon\Carbon::class => 'App\Console\Casters::carbon',
    ],
];

// app/Console/Casters.php
class Casters
{
    public static function carbon(\Carbon\Carbon $carbon)
    {
        return [
            'date' => $carbon->date,
            'weekday' => $carbon->format('l'),
            'day_of_year' => (int) $carbon->format('z'),
            'unix' => (int) $carbon->format('U'),
            'diff_for_humans' => $carbon->diffForHumans(),
        ];
    }
}
```

And we are up & running!

Let’s see that in action before we dive into the code:

```
=> Carbon\Carbon @1519569240 {#745
 +date: "2018-02-25 14:34:00.000000",
 +weekday: "Sunday",
 +day_of_year: 55,
 +unix: 1519569240,
 +diff_for_humans: "6 days ago",
}
```

PS. Soon you will be able to customize casters without hassle, as I’ve already pushed a PR, which provides this functionality out-of-the-box:

[https://github.com/laravel/tinker/pull/39](https://github.com/laravel/tinker/pull/39)
