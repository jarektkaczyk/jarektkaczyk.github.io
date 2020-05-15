# Interface HowTo

Great intro: https://stackoverflow.com/a/14244705/784588

In short: an interface defines what functionality you need in a class you depend on (while not caring about the implementation).
It helps in designing the code and allows deferring implementation details so they don't get in the way.

**TLDR;**

1. [define functional methods, you **MUST NEVER** define implementation details as part of the interface](#why-not-setters-amp-fluent-api)
    - you **MUST NEVER** define setters on an interface - they are ALWAYS implementation detail
    - you **MUST NEVER** define interface for so-called *fluent api/interface* (object returning itself) - *fluent api* is ALWAYS implementation detail
4. [you **SHOULD** avoid using associative arrays in both input & output - prefer VOs/POPOs/DTOs and Entities](#why-types-rather-than-associative-arrays)


---

Let's think of an example feature request *Integrate 3rd party API to allow our users to make transfers*

### Usual approach - DON'T:
1. review API documentation
2. pull or write SDK for it
3. write configuration for the API connection
4. write the actual functionality that serves the users


### The right way:
1. write the actual functionality that serves the users (based on mockups & specs)
2. write implementation to support that functionality
3. pull or write SDK that will be necessary for the implementation
4. write configuration for the API connection

---

Why is the usual approach not what you want?

- Because the most important thing (value for the user) comes last. By the time we had a chance to actually design how it works, we already put a lot of constraints on ourselves by whole implementation.
- If we discover at that point that we made wrong assumptions, it is hard and takes time to make changes.
- We may spend significant amount of time implementing things that we don't actually need


The right might not feel natural initially, and so requires a bit different approach.
Let's see an example of the code in the order we would write it:


```php
// 1 entrypoint for user's input
class TransferController
{
    function makeTransfer(TransferRequest $request, TransferService $transferService)
    {
        try {
            $transferService->send($request->getTransfer());

            return new OkResponse; // 200 OK
        } catch (ClientTransferException $e) {
            // API validation error caused by the Client (balance, invalid bank account number etc)
            return new ClientErrorResponse($e->getMessage()); // 400 Bad Request
        } catch (ServerTransferException|Throwable $e) {
            // API validation error caused by Server implementation (bug, runtime error etc)
            return new ServerErrorResponse($e->getMessage()); // 500 Internal Server Error
        }
    } 
}

// 2 input validation, parsing, sanitizing and turning it into domain-specific objects
class TransferRequest extends ApiRequest
{
    function rules(): array
    {
        return [
            'amount' => 'required|int',
            'holder*name' => 'required|string',
            'currency*code' => new ValidCurrencyRule($this),
            'account*number' => new ValidAccountNumberRule($this),
        ];
    }

    function getTransfer(): TransferEntity
    {
        return new TransferEntity(
            Money::make($this->input('amount'), $this->input('currency*code')),
            TransactionParty::make($this->input('account*number', $this->input('holder*name')))
        );
    }
}

// 3 expected low-level functionality that we will need
interface TransferService
{
    function send(TransferEntity $transfer): void;
}
```


At this point we:
1. are done with *write the actual functionality that serves the users*
2. don't have to get back to this part of the code again
3. can easily write automated tests (we may already have them - TDD comes naturally with such code)
4. know exactly what **the next step** is

The next step is driven by the interface here: `$transferService->send(TransferEntity $transfer)` - let's write the implementation then!

```php
// 1
class Transferwise implements TransferService
{
    function send(TransferEntity $transfer): void
    {
        // we need to call API somehow - let's assume there's SDK that we will use
    }
}
```

```php
// 2
class Transferwise implements TransferService
{
    function __construct(TransferWise\Client $client)
    {
        $this->client = $client;
    }

    function send(TransferEntity $transfer): void
    {
        $this->client->sendTransfer(
            // 1 pass data from the $transfer
            $transfer->getAmount() / 100,
            $transfer->getCurrencyCode(),
            ...
            // 2 let's assume we need more than just that, eg. some configuration
        );
    }
}
```

```php
// 3
class Transferwise implements TransferService
{
    function __construct(TransferWise\Client $client, Transferwise\Config $config)
    {
        $this->client = $client;
        $this->config = $config;
    }

    function send(TransferEntity $transfer): void
    {
        $this->client->sendTransfer(
            $transfer->getAmount() / 100,
            $transfer->getCurrencyCode(),
            $this->config->default_transfer_type,
            $this->config->markup_amount,
        );
    }
}
```

This pretty much covers [points 2 & 3](#the-right-way). Last thing would be to sprinkle the class with standard things like LOGGING, ERROR HANDLING etc - those should come last, as it's low level stuff that should never get in the way of functionality/value that we provide to the user.


```php
// 4
class Transferwise implements TransferService
{
    function __construct(TransferWise\Client $client, Transferwise\Config $config)
    {
        $this->client = $client;
        $this->config = $config;
        $this->logger = new NullLogger;
    }

    function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    function send(TransferEntity $transfer): void
    {
        $this->logger->info('useful log');

        try {
            $this->client->sendTransfer(
                $transfer->getAmount() / 100,
                $transfer->getCurrencyCode(),
                $this->config->default_transfer_type,
                $this->config->markup_amount,
            );
        } catch (Transferwise\SomeError $e) {
            $this->logger->info('useful log');
            throw new ClientTransferException('Transfer Failed', 0, $e);
        } catch (Throwable $e) {
            $this->logger->info('useful log');
            throw new ServerTransferException('Transfer Failed', 0, $e);
        }
    }
}
```


And finally we *write configuration for the API connection*:

```php
class TransferwiseServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->bind(Transferwise\Config::class, fn () => Config::fromArray([
            'default_transfer_type' => config('...'), // or env(...) if different in dev/prod env
            'markup_amount' => config('...'), // or env(...) if different in dev/prod env
        ]));
    }
}
```

---

### Why not setters & fluent api?

**It might be tempting** to define methods for fluent interface:

```php
interface TransferService
{
    // NEVER DO THIS:
    function setSomething(SomeGenericType $something): self;
    function setAmount(Money $amount): self;
    function setTransfer(TransferEntity $transfer): self;
    function send(): void
}

// NEVER DO THIS:
$transferService
    ->setSomething($something)
    ->setAmount($amount)
    ->setTransfer($transfer)
    ->send();
```

Such approach breaks rule 1 completely and creates very rigid structure.
That is because the ONLY thing we care about from user's perspective is `send()` method, and having *fluent api* defined on the interface binds us to this approach only for all implementations.

You can still use *fluent api* if this is your preferred way. It's great for **builder** objects, but not for functional services.

An example of proper interface and fluent builder - both achieve exactly the same goal and are valid implementions from user's perspective:

```php
// Standard approach
$transferService->send(
    new TransferEntity($amount, $recipient, $description)
);

// Fluent api approach
$transferService->send(
    TransferEntity::builder()
        ->setRecipient($recipient)
        ->setAmount($amount)
        ->setDescription($description)
);
```

    
### Why types rather than (associative) arrays?

Associative arrays in general SHOULD only be used within private scope but in public api prefer types.
Standard arrays are perfectly fine, as long as they represent a simple list and we don't depend on the keys.

Example:

```php
public function getTransfer(): TransferEntity
{
    $transfer = new TransferEntity;
    $transfer->property_a = 'value for a';
    $transfer->property_b = 12345;
    $transfer->property_c = 'value for c';
}

// VS

public function getTransferData(): array 
{
    return [
        'property_a' => 'value for a',
        'property_b' => 12345,
        'property_c' => 'value for c',
    ];
}
```

##### Why array is really BAD

1. consuming array as output from a class - you HAVE TO KNOW the keys inside -> you need to waste time to find the definition and/or usage for the array. As array is just a primitive type, IDE won't be able to offer FIND REFERENCES to help you.
2. usually it requires `if` and `fallback` values: `$array['key'] ?? null` or `$array['key'] ?? 'default'`
3. it's easy to make typo that goes unnoticed - you can easily ship code to production without spotting because of the point 2
4. static analysis tools in CI won't catch errors just like IDE
5. if you need to change/add key to the array, you waste time finding where it is used - again, no FIND REFERENCES in IDE

##### Why object is helpful

By implementing object, even simple POPO, you get rid of ALL of those problems:

```php
class TransferEntity
{
    public ?string property_a = null;
    public ?int property_b = null;
    public ?string property_c = 'default';
}
```

1. consuming the object is fast and easy -> IDE has your back and completes properties + their types
2. no need for `if` / `isset` checks
3. typos will be caught as early as in the IDE when you type
4. if something still went unnoticed, static analysis will be able to catch it during commit or later in the CI
5. changing payload means that IDE helps you with FIND REFERENCES


There is no downside to this approach:

- performance-wise we don't need to worry about primitives vs objects
- creating a class takes just a few seconds more that an array
- we can write less tests, having strict types on the objects
