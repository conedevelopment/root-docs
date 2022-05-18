---
title: "Filters"
github: "https://github.com/conedevelopment/root-docs/blob/master/filters.md"
order: 8
---

Filters are responsible for transforming the current request to a database query.

## Creating Filters

You can generate new filter classes by calling the `root:filter` artisan command:

```php
php artisan root:filter Category
```

## Registering Filters

You can register filters in resources and extracts, by using the `filters` method.

```php
use App\Root\Filters\Category;
use Cone\Root\Http\Requests\RootRequest;

/**
 * Define the filters for the resource.
 *
 * @param  \Cone\Root\Http\Requests\RootRequest  $request
 * @return array
 */
public function filters(RootRequest $request): array
{
    return [
        Category::make(),
    ];
}
```

Alternatively, you can use `withFilters` method on an object that resovles actions. It can be useful when you just want to hook into the object for some reason.

```php
use App\Root\Filters\Category;
use Cone\Root\Support\Collections\Filters;
use Cone\Root\Http\Requests\RootRequest;

$resource->withFilters(static function (RootRequest $request, Filters $filters): Filters {
    return $filters->merge([
        Category::make(),
    ]);
});
```

> You can also pass an `array` instead of a `Closure`. In that case the array will be merged into the collection.

## Configuration

### Select Filters

By default Root generates select filters, when calling the `root:filter` artisan command.

Select filters have an `option` method, where you can define the selectable options:

```php
namespace App\Root\Filters;

use Cone\Root\Filters\SelectFilter;
use Cone\Root\Http\Requests\RootRequest;
use Illuminate\Database\Eloquent\Builder;

class Category extends SelectFilter
{
    /**
     * Apply the filter on the query.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @param  mixed  $value
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function apply(RootRequest $request, Builder $query, mixed $value): Builder
    {
        return $query->where('category', $value);
    }

    /**
     * Get the filter options.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function options(RootRequest $request): array
    {
        return [
            'product' => 'Product',
            'service' => 'Service',
        ];
    }
}
```

Also, you may configure a select filter with multiple selectable choices. To do so, call the `multiple` method on the filter instance:

```php
Category::make()->multiple();
```

### Input Filters

### Custom Components

### Functional Filters

### Authorization

You may allow or disallow interaction with actions. To do so, you can call the `authorize` method on the action instance:

```php
$action->authorize(static function (RootRequest $request): bool {
    return $request->user()->can('batchPublish', Post::class);
});
```
