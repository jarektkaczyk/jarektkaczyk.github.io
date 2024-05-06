---
layout: post
title:  "Revisionable a quick example"
date:   2015-10-10 12:00:00 +0800
categories: [coding, laravel, php]
---

# Revisionable a quick example

A short reminder about my [revisionable](https://github.com/jarektkaczyk/revisionable) package (compatible with L4 & L5+) and quick example of how you can use it:

1. Controller:
```php

public function history($id)
{
  $ticket = Ticket::find($id);

  return view('tickets.revisions.timeline', compact('ticket'));
}
```

2. Model:
```php

use Sofa\Revisionable\Revisionable;
use Sofa\Revisionable\Laravel\RevisionableTrait;

class Ticket extends Model implements Revisionable 
{
  use RevisionableTrait;

  protected $revisionPresenter = 'App\Presenters\Revisions\Ticket';

  protected $revisionable = [
    'item_id', 'customer_id', 'status_id',
    'responsible_id', 'defect', 'note',
  ];

  // ...
}
```

4. Presenter:
```php

use Sofa\Revisionable\Laravel\Presenter;

class TicketPresenter extends Presenter {

  protected $labels = [
    'item_id'        => 'Przedmiot',
    'customer_id'    => 'Klient',
    'status_id'      => 'Status',
    'responsible_id' => 'Serwisant',
    'defect'         => 'Usterka',
    'note'           => 'Uwagi',
  ];

  protected $passThrough = [
    'item_id'        => 'item.name',
    'customer_id'    => 'customer.name',
    'responsible_id' => 'serviceman.name',
    'status_id'      => 'status.name',
  ];

  protected $actions = [
    'created'  => 'utworzony',
    'updated'  => 'edytowany',
    'deleted'  => 'usunięty',
    'restored' => 'przywrócony',
  ];

}
```

5. Views (only relevant parts)
```php

@unless (count($revisions))

  <p>Nie znaleziono historii zmian dla podanych kryteriów</p>

@else
  <section id="cd-timeline">

    @foreach ($ticket->revisions as $revision)

      @include('revisions.single', ['revision' => $revision])

    @endforeach

  </section>
@endif
```

```php
<div class="alert alert-warning no-margin">
  <caption>
    {{ $revision->created_at }} rekord <strong>{{ $revision->action }}</strong> przez: <strong>{{ $revision->user }}</strong>.
  </caption>
</div>

@if (count($revision->old))
<table class="table">
  <thead>
    <tr>
      <th>Pole</th>
      <th>Stara wartość</th>
      <th>Nowa wartość</th>
    </tr>
  </thead>

    @foreach ($revision->old as $key => $v)

    <tr>
      <td>{{ $revision->label($key) }}</td>
      <td class="{{ $revision->isUpdated($key) ? ' danger' : '' }}">{{ $revision->old($key) }}</td>
      <td class="{{ $revision->isUpdated($key) ? ' success' : '' }}">{{ $revision->new($key) }}</td>
    </tr>

    @endforeach

</table>
@endif
```

That’s it!

Any issues, please post to [https://github.com/jarektkaczyk/revisionable/issues](https://github.com/jarektkaczyk/revisionable/issues)
