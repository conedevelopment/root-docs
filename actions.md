---
title: "Actions"
github: "https://github.com/conedevelopment/root-docs/blob/master/actions.md"
order: 6
---

Actions are runnable tasks on a given collection of models. Actions can be registered in resources and extracts. Also, actions are easily configurable and customizable for the current user.

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
    return [
        Publish::make(),
    ];
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
use Cone\Root\Http\Requests\RootRequest;

$field->visibleOn(static function (RootRequest $request): bool {
    return $request->user()->isAdmin();
});

$field->hiddenOn(static function (RootRequest $request): bool {
    return ! $request->user()->isAdmin();
});
```

Also, you can use the built-in methods as well:

```php
$field->visibleOnIndex();
$field->visibleOnShow();

$field->hiddenOnIndex();
$field->hiddenOnShow();
```

> Note, actions are using the same `ResolvesVisiblity` trait as fields, but methods like `visibleOnUpdate()` have no effect.

You can also pass a `Closure` to any of the listed methods, to fully customize the visibility logic:

```php
$this->visible(static function (RootRequest $request): bool {
    return $request->user()->can('batchPublish', Post::class);
});

$this->hidden(static function (RootRequest $request): bool {
    return $request->user()->cannot('batchPublish', Post::class);
});
```

### Routes

Actions can also register routes for themselves and their fields. The routes are automatically registered. If you wish to register a custom route for an action you can easily do that:

```php
/**
 * The routes that should be registerd.
 *
 * @param  \Illuminate\Routing\Router  $router
 * @return void
 */
public function routes(Router $router): void
{
    parent::rotues($router);

    $router->post(sprintf('%s/%s', $this->getKey(), 'custom-action'), function (ActionRequest $request) {
        // Handle request...
    });
}
```

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
