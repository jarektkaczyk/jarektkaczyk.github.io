---
layout: post
title:  "20 Eloquent tricks - more"
date:   2018-04-14 12:00:00 +0800
categories: [coding, laravel, eloquent, php]
---
Recently I stumbled upon interesting article by [Povilas Korop](https://twitter.com/PovilasKorop) on laravel-news [20 eloquent tips and tricks](https://laravel-news.com/eloquent-tips-tricks). First of all thanks for sharing your knowledge Povilas – it helps many people out there, keep it up!

As you may know, I’ve worked with Eloquent for long and love its features, at the same time I’ve become very sensitive about some dangers lurking for inexperienced developers that come with all the magic of Eloquent :) 
Let’s use this opportunity to look at the tips from different perspective and learn even more by going through some of them in more details!

In order to make it valuable for you I’m keeping it as concise as possible:

1. `in/decrement`: **GOOD**

2. `findOrFail`: **GOOD**, but there’s a catch in `findOrFail`. Laravel internally handles `ModelNotFoundException` [here](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Foundation/Exceptions/Handler.php#L198) and it means that any call that fails here, will trigger a `404 NOT FOUND` response from your app. This works fine when called for example for something like this:

    ```php
    Route::get('products/{id}', function ($id) {

     return Product::findOrFail($id);

    });
    ````

    Laravel uses the same pattern for implicit route model binding described [here](https://laravel.com/docs/5.6/routing#route-model-binding)

    However, if you’re calling your query deeper in the code of your application, you do not want to automatically trigger `404` for any not-found record, as it might have nothing to do with the routing or even HTTP layer at all.  
    That being said, `findOrFail` is cool for top, HTTP layer. Other than that I recommend something between the lines of (yes, oldskool, even archaic, no magic involved… you get the idea):

    ```php
    $model = $someQuery->find($id);

    if (is_null($model)) {
     // handle logic when record is not found
    }
    ```

3. `boot` **GOOD**

4. `default ordering on relationships` – definitely no (unless the relationship itself explicitly states that):

    ```php 
    // Organization
    public function approvedUsers() {
     return $this->hasMany('App\\User')->where('approved', 1)->orderBy('email');
    }

    // then somewhere you need this
    $organization->approvedUsers()->latest()->get();
    ```

    Now you’re scratching your head and trying to figure out why you don’t get `latest` users. After a while you notice that they are ordered by email, so you’re tracking it down – relationship is found, so the question arises – can I change the relationship? Not really, it would break some functionality, probably untested, so it is not the way to go. Eventually it seems that the only way is a NEW relationhsip. Not good at all.

    You could make it better, if you really need default ordering, by creating custom and explicit relation in the first place – just like [here](https://twitter.com/SOFTonSOFA/status/966669189103599616)

5. `props`: **GOOD**. There’s one exception – I recommend not using `$appends` EVER as this one very easily leads to huge problems if abused. Imagine using accessor that does some heavy logic (which it should not), like querying relation and returning its value:

    ```php
    public function getOrganizationNameAttribute()
    {
     return $this->organization->name;
    }
    ```
     
    this is evil as it is, trust me. Now, imagine you have an API endpoint that serves your model directly, and some time later your colleague needs to add `organization_name` to the JSON result. Easy does it, right? Just added to `$appends` and it is automagically in the JSON!

    Only now you realize that your endpoint serves data 20 times slower now and you have no idea why. Here it is – magic has its cost. You endpoint used, say, 5 queries before, but now it uses 5 + N queries, because accessor runs the query under the hood. Finding it out is not trivial and even noticing it might take long, which can hurt the business. Don’t go this path, [be like new Leo and save yourself some headache](https://softonsofa.com/a-story-about-laravel-resources) 😉

6. `findMany`: **GOOD**. I’d only add note about different return values `find(int|string 1) : ?Model` while `find(array|Arrayable $ids) : Collection`. Which means the first returns `Model` or `null`, but the latter returns `Collection` **always** (even if empty).

7. `dynamic wheres`: **NEVER do this**. It is illogical, confusing, impossible to understand for anyone without decent knowledge about Eloquent.  
    Next thing is this: if you ever need to restructure your DB or just investigate usage and find all queries against a column, you’d find easily `where('some_column')`, `orderBy('some_column')` etc by simply grepping against `'some_column'`, but you won’t find `whereSomeColumn($value)`.  
    Another catch is this: you know already that Eloquent provides those magic calls (dynamic wheres), so when you see `whereDate` you can think it is the same. But wait, there’s no `date` column in the table, so WTF? Oh, sure, this is actually real method `whereDate($column, $value)`. You get the idea, trust me, you neighbor and their dog will hate you if you use it even once.

8. `order by relation`: Custom relation definitely **GOOD**, but ordering nope. While it is valid, it’s dangerous as well. You should **NEVER depend on the collection ordering** if you don’t know how many items collection has (and it must be rather small collection). Imagine this code when your page grows from 20 `Topics` to 20 thousands `Topics` – then every single pageload is looong seconds, sometimes memory exhausted errors occur etc. Databases are pretty good with ordering, use them.

9. `when`: I totally agree with the comment: _It may not feel shorter or more elegant_ – True, it is not shorter, nor more elegant, nor better in any way for me. Here’s what I recommend:
 
    ```php
    if ($request->has('role')) { // I know the intent already
     $query->where('role_d', $request->get('role')); // then following action
    }

    // vs

    // Reading this I am not sure what false means, no idea what $role will be
    // in the closure, I have to spend a few seconds analyzing it every time.
    // It adds up in big codebase and I love to avoid unnecessary overload
    $query->when(request('role', false), function ($q, $role) {
     return $q->where('role_d', $role);
    });
    ```

    One may argue that you HAVE TO avoid `if` statements, but here’s the thing – removing them doesn’t make you functional programmer, using `function () {}` rather than `if` doesn’t make you functional programmer, it doesn’t add any value to the business either.

10. `withDefault`: **OKAY** in general. I prefer explicit code, so don’t use it myself, but it’s matter of preference. However using it strictly for presentation purpose as in the example – no like.

11. `accessor ordering`: **Smart** – as in No. 8 be careful in cases, where you cannot be sure bout collection size.

12. `global scope`: **OKAY**. Personally I wouldn’t do that in a global scope, as global scopes are good for only so many cases, and only to apply some `where` constraints (`SoftDeletes` is a good example, `Active` might be applicable in many cases etc).

13. `raw`: **GOOD**

14. `replicate`: **GOOD**

15. `chunk`: VERY **GOOD**. I would add that you NEVER want to call `Model::all()` unless you have a dictionary-like model, say, a list of countries. A model that you can be sure will never grow (there won’t be more than 300 countries in our lifetime I guess).

16. `generators`: **GOOD**

17. `updated_at`: **GOOD**

18. `update` return value: **GOOD**

19. `orWhere`: This one is actually incorrect, both in the code and logically. I believe it comes from the bad example in the Laravel’s own docs. I corrected the example and added some notes to the docs recently, so feel free to look it up now [docs](https://laravel.com/docs/5.6/queries#parameter-grouping)

20. `orWhere` again: Same as No. 19.
