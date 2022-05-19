---
title: "Extracts"
github: "https://github.com/conedevelopment/root-docs/blob/master/extracts.md"
order: 7
---

Extracts are layers on the top of the resource index. They are responsible to perform more complex queries and display relevant information for the query, that you may not want to show on the index page.

## Creating Extracts

You can generate new extracts classes by calling the `root:extract` artisan command:

```sh
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

### Customizing the Query

You may customize the query of the extract you are working with. While its base query is provided by its parent resource, it's very possible you want to perform a completely different custom database query.

```php
use Cone\Root\Http\Requests\RootRequest;
use Cone\Root\Extracts\Extract;

class LongPosts extends Extract
{
    /**
     * Resolve the query for the extract.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function resolveQuery(RootRequest $request): Builder
    {
        return parent::resolveQuery($request)
                    ->withoutEagerloads()
                    ->selectRaw('`id`, `title`, length(`content`) as `length`')
                    ->orderBy('length');
    }
}
```

### Fields

> For the detailed documentation visit the [fields](/docs/fields) section.

Fields are handlers for the model attributes. They are responsible for saving and displaying the given attribute of the parent resource model. You can easily define fields on your extract by using the `fields` method:

```php
use Cone\Root\Fields\ID;
use Cone\Root\Fields\Text;
use Cone\Root\Extracts\Extract;
use Cone\Root\Http\Requests\ExtractRequest;
use Cone\Root\Http\Requests\RootRequest;
use Illuminate\Database\Eloquent\Model;

class LongPosts extends Extract
{
    /**
     * Define the fields for the extract.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function fields(RootRequest $request): array
    {
        return array_merge(parent::fields($request), [
            Text::make('Title'),
            Number::make('Length')->format(function (ExtractRequest $request, Model $model, mixed $value): string {
                return __(':value characters', ['value' => $value]);
            }),
        ]);
    }
}
```

### Filters

> For the detailed documentation visit the [filters](/docs/filters) section.

Filters are responsible for transforming the current request to a database query. You can easily define filters on your extract by using the `filters` method:

```php
use App\Root\Filters\Category;
use Cone\Root\Extracts\Extract;
use Cone\Root\Http\Requests\RootRequest;

class LongPosts extends Extract
{
    /**
     * Define the filters for the extract.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function filters(RootRequest $request): array
    {
        return array_merge(parent::filters($request), [
            Category::make(),
        ]);
    }
}
```

> Note, by default there are two registered filters `Sort` and `Search`. If you don't want to register them by default, just define an array without the `array_merge()` function.

### Actions

> For the detailed documentation visit the [actions](/docs/actions) section.

Actions are responsible for performing a specific action on a set of models. You can easily define actions on your extract by using the `actions` method:

```php
use App\Root\Actions\Publish;
use Cone\Root\Extracts\Extract;
use Cone\Root\Http\Requests\RootRequest;

class LongPosts extends Extract
{
    /**
     * Define the actions for the resource.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function actions(Request $request): array
    {
        return array_merge(parent::actions($request), [
            Publish::make(),
        ]);
    }
}
```

### Widgets

> For the detailed documentation visit the [widgets](/docs/widgets) section.

Widgets are cards that display some information or any content you want to display. You can easily define widgets on your extract by using the `widgets` method:

```php
use App\Root\Widgets\TotalPosts;
use Cone\Root\Extracts\Extract;
use Cone\Root\Http\Requests\RootRequest;

class LongPosts extends Extract
{
    /**
     * Define the widgets for the extract.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function widgets(RootRequest $request): array
    {
        return array_merge(parent::widgets($request), [
            TotalPosts::make(),
        ]);
    }
}
```

### Authorization

You may allow or disallow interaction with extracts. To do so, you can call the `authorize` method on the action instance:

```php
$action->authorize(static function (RootRequest $request): bool {
    return $request->user()->can('viewLongPosts');
});
```
