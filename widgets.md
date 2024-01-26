---
title: "Widgets"
github: "https://github.com/conedevelopment/root-docs/blob/master/widgets.md"
order: 9
---

Widgets are cards that hold some information or any content you want to display.

## Creating Widgets

You can generate new action classes by calling the `root:widget` artisan command:

```sh
php artisan root:widget PostCount
```

## Registering Widgets

You can register actions in resources and extracts, by using the `widgets` method.

```php
use App\Root\Widgets\PostCount;
use Illuminate\Http\Request;

/**
 * Define the widgets for the resource.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function widgets(Request $request): array
{
    return array_merge(parent::widgets($request), [
        PostCount::make(),
    ]);
}
```

Alternatively, you can use `withWidgets` method on an object that resolves widgets. It can be useful when you just want to hook into the object for some reason.

```php
use App\Root\Widgets\PostCount;
use Cone\Root\Support\Collections\Widgets;
use Illuminate\Http\Request;

$resource->withWidgets(static function (Request $request, Widgets $widgets): Widgets {
    return $widgets->merge([
        PostCount::make(),
    ]);
});
```

> You can also pass an `array` instead of a `Closure`. In that case the array will be merged into the collection.

## Configuration

### Authorization

You may allow or disallow interaction with widgets. To do so, you can call the `authorize` method on the widget instance:

```php
$widget->authorize(static function (Request $request): bool {
    return $request->user()->can('viewPostCount');
});
```

### Visibility

You may show or hide widgets based on the current resource view. For example, some widgets might be visible on the index page, while others should be hidden. You can easily customize the action visibility logic using the `visibleOn` and `hiddenOn` methods:

```php
$widget->visibleOn(static function (Request $request): bool {
    return $request->user()->can('viewPostCount');
});

$widget->hiddenOn(static function (Request $request): bool {
    return $request->user()->cannot('viewPostCount');
});
```

Also, you can use the built-in methods as well:

```php
$widget->visibleOnIndex();
$widget->visibleOnShow();

$widget->hiddenOnIndex();
$widget->hiddenOnShow();
```

### Blade Templates

A widget class must have a valid template property, that holds a real blade template:

```php
use Cone\Root\Widgets\Widget;

class PostCount extends Widget
{
    /**
     * The Blade template.
     *
     * @var string
     */
    protected string $template = 'widgets.post-count;
}
```

## Widget Data

You can customize the data passed to the blade template by using the `data` method:

```php
use App\Models\Post;
use Illuminate\Http\Request;
use Cone\Root\Widgets\Widget;

class PostCount extends Widget
{
    /**
     * Get the data.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function data(Request $request): array
    {
        return [
            'count' => Post::query()->count(),
        ];
    }
}
```

Alternatively, you can use `with` method on a widget. It can be useful when you just want to hook into the object for some reason.

```php
use App\Models\Post;
use Illuminate\Http\Request;

$widget->with(static function (Request $request): array {
    return [
        'published_count' => Post::query()->published()->count(),
    ];
});
```

> You can also pass an `array` instead of a `Closure`.

### Custom Components

By default, widget use the `Widget` component, however you can use any component you want. You may also register your own custom component as well:

```js
document.addEventListener('root:booting', ({ detail }) => {
    detail.app.component('CustomWidget', require('./Components/CustomWidget').defaul);
});
```

```php
use Cone\Root\Widgets\Widget;

class PostCount extends Widget
{
    /**
     * The Vue component.
     *
     * @var string|null
     */
    protected string $component = 'CustomWidget';
}
```

### Async Widgets

In some scenarios, you may want your widget to be asynchronously loaded. For example, when the rendered template generates a huge chunk of data, you may want to load the component in another HTTP request. To do so, call the `ąsyc` method on  the widget instance:

```php
PostCount::make()->async();
```

> Please note, all the routes for the asnyc widgets will be registered automatically.
