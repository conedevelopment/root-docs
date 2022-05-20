---
title: "Actions"
github: "https://github.com/conedevelopment/root-docs/blob/master/actions.md"
order: 6
---

Actions are runnable tasks on a given collection of models. Actions can be registered in resources and extracts. Also, actions are easily configurable and customizable for the current user.

## Creating Actions

You can generate new action classes by calling the `root:action` artisan command:

```sh
php artisan root:action Publish
```

## Registering Actions

You can register actions in resources and extracts, by using the `actions` method.

```php
use App\Root\Actions\Publish;
use Cone\Root\Http\Requests\RootRequest;

/**
 * Define the actions for the resource.
 *
 * @param  \Cone\Root\Http\Requests\RootRequest  $request
 * @return array
 */
public function actions(RootRequest $request): array
{
    return array_merge(parent::actions($request), [
        Publish::make(),
    ]);
}
```

Alternatively, you can use `withActions` method on an object that resovles actions. It can be useful when you just want to hook into the object for some reason.

```php
use App\Root\Actions\Publish;
use Cone\Root\Support\Collections\Actions;
use Cone\Root\Http\Requests\RootRequest;

$resource->withActions(static function (RootRequest $request, Actions $actions): Actions {
    return $actions->merge([
        Publish::make(),
    ]);
});
```

> You can also pass an `array` instead of a `Closure`. In that case the array will be merged into the collection.

## Configuration

### Customizing the Query

You may customize the query of the action you are working with. While its base query is provided by its parent resource or extract, it's very possible you want to perform a completely different custom database query.

```php
use Cone\Root\Actions\Action;
use Cone\Root\Http\Requests\RootRequest;

class Publish extends Action
{
    /**
     * Resolve the query for the action.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function resolveQuery(RootRequest $request): Builder
    {
        return parent::resolveQuery($request)->whereNull('published_at');
    }
}
```

### Fields

> For the detailed documentation visit the [fields](/docs/fields) section.

You may define fields for any action:

```php
use Cone\Root\Actions\Action;
use Cone\Root\Fields\Date;
use Cone\Root\Http\Requests\RootRequest;

class Publish extends Action
{
    /**
     * Define the fields for the action.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function fields(RootRequest $request): array
    {
        return array_merge(parent::fields($request), [
            Date::make('Published at')->withTime(),
        ]);
    }
}
```

> When running the action, the submitted form data will be accessible in the `Request` object passed to the action's `handle` method.

### Authorization

You may allow or disallow interaction with actions. To do so, you can call the `authorize` method on the action instance:

```php
$action->authorize(static function (RootRequest $request): bool {
    return $request->user()->can('batchPublish', Post::class);
});
```

### Visibility

You may show or hide actions based on the current resource view. For example, some actions might be visible on the index page, while others should be hidden. You can easily customize the action visibility logic using the `visibleOn` and `hiddenOn` methods:

```php
$action->visibleOn(static function (RootRequest $request): bool {
    return $request->user()->can('batchPublish', Post::class);
});

$action->hiddenOn(static function (RootRequest $request): bool {
    return $request->user()->cannot('batchPublish', Post::class);
});
```

Also, you can use the built-in methods as well:

```php
$action->visibleOnIndex();
$action->visibleOnShow();

$action->hiddenOnIndex();
$action->hiddenOnShow();
```

> Note, actions are using the `ResolvesVisiblity` trait, but methods like `visibleOnUpdate()` have no effect.

### Destructive Actions

You may mark an action as destructive. That means, the `Run` button that performs the action becomes red, indicating it is a dangerous action to run.

```php
$action->destructive();
```

### Confirmable Actions

You may mark an action as confirmable. That means, when pressing the `Run` button, the user receives a confirmation popup. If it's been cancelled, the action won't run.

```php
$action->confirmable();
```
