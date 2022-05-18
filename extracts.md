---
title: "Extracts"
github: "https://github.com/conedevelopment/root-docs/blob/master/extracts.md"
order: 7
---

Extracts are layers on the top of the resource index. They are responsible to perform more complex queries and display relevant information for the query, that you may not want to show on the index page.

## Creating Extracts

You can generate new extracts classes by calling the `root:extract` artisan command:

```php
php artisan root:extract LongPosts
```

## Registerin Extracts

You can register extracts in resources, by using the `extracts` method.

```php
use App\Root\Extracts\LongPosts;
use Cone\Root\Http\Requests\RootRequest;

/**
 * Define the extracts for the resource.
 *
 * @param  \Cone\Root\Http\Requests\RootRequest  $request
 * @return array
 */
public function extracts(RootRequest $request): array
{
    return [
        LongPosts::make(),
    ];
}
```

Alternatively, you can use `withExtracts` method on an object that resovles extracts. It can be useful when you just want to hook into the object for some reason.

```php
use App\Root\Extracts\LongPosts;
use Cone\Root\Support\Collections\Extracts;
use Cone\Root\Http\Requests\RootRequest;

$resource->withExtracts(static function (RootRequest $request, Extracts $extracts): Extracts {
    return $extracts->merge([
        LongPosts::make(),
    ]);
});
```

> You can also pass an `array` instead of a `Closure`. In that case the array will be merged into the collection.

## Configuration

### Actions

### Fields

### Filters

### Widgets

### Customizing the Query

### Authorization

### Routes
