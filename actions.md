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
use Illuminate\Http\Request;

/**
 * Define the actions for the resource.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function actions(Request $request): array
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
use Illuminate\Http\Request;

$resource->withActions(static function (Request $request, Actions $actions): Actions {
    return $actions->merge([
        Publish::make(),
    ]);
});
```

> You can also pass an `array` instead of a `Closure`. In that case the array will be merged into the collection.

## Configuration

### Authorization

### Visibility

### Destructive Actions

### Confirmable Actions
