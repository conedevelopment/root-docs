---
title: "Filters"
github: "https://github.com/conedevelopment/root-docs/blob/master/filters.md"
order: 8
---

Filters are responsible for transforming the current request to a database query.

## Creating Filters

You can generate new filter classes by calling the `root:filter` artisan command:

```sh
php artisan root:filter Category
```

## Registering Filters

You can register filters in resources and extracts, by using the `filters` method.

```php
use App\Root\Filters\Category;
use Illuminate\Http\Request;

/**
 * Define the filters for the resource.
 */
public function filters(Request $request): array
{
    return [
        Category::make(),
    ];
}
```

Alternatively, you can use `withFilters` method on an object that resolves filters. It can be useful when you just want to hook into the object for some reason.

```php
use App\Root\Filters\Category;
use Cone\Root\Support\Collections\Filters;
use Illuminate\Http\Request;

$resource->withFilters(static function (Request $request): array {
    return [
        Category::make(),
    ];
});
```

## Configuration

### Options

Filters have an `option` method, where you can define the selectable options:

```php
namespace App\Root\Filters;

use Cone\Root\Filters\Filter;
use Illuminate\Http\Request;
use Illuminate\Database\Eloquent\Builder;

class Category extends Filter
{
    /**
     * Apply the filter on the query.
     */
    public function apply(Request $request, Builder $query, mixed $value): Builder
    {
        return $query->where('category', $value);
    }

    /**
     * Get the filter options.
     */
    public function options(Request $request): array
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

### Functional Filters

Also, in some cases you may interact with the Query Builder in a filter, but you don't want to render any component for it on the front-end. In that case, you may use functional components. For functional components the `$component` property is set to `null`:

```php
use Cone\Root\Filters\Filter;

class FunctionalFilter extends Filter
{
    /**
     * The Vue component.
     *
     * @var string|null
     */
    protected ?string $component = null;
}
```

### Authorization

You may allow or disallow interaction with filters. To do so, you can call the `authorize` method on the filter instance:

```php
$filter->authorize(static function (Request $request): bool {
    return $request->user()->isAdmin();
});
```
