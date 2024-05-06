---
layout: post
title:  "Sofa revisionable nice easy way to keep track of db changes"
date:   2014-12-18 12:00:00 +0800
categories: [coding, laravel, php]
---

It is essential for every application to keep track of the data revisions. As an admin or sort of supervising user, you want to know how, when and who is responsible for changing the data.

Here’s something you should try out, because it makes this kind of work stupid-simple:

[Sofa/Revisionable](https://packagist.org/packages/sofa/revisionable)

In order to get going with the package in Laravel4 app, you need to follow just a few simple steps:

1. get the package from the packagist

2. add service provider to the `app/config/app.php`

3. publish and adjust config if necessary (like changing revisions table name to something else or using alternative authentication service instead of default illuminate)

4. run the migration `php artisan migrate --package=sofa/revisionable`

5. add `RevisionableTrait` and setup revisioned/nonrevisionaed fields and/or custom connection to use for the models you want to keep track of

 

And.. that’s all!
