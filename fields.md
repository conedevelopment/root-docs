# Fields

Fields are handlers for the model attributes. They are responsible for saving and displaying the given attribute of the resource model.

## Values

Field values are extracted from the given model's attributes. The attribute that is matching with the name of the field will be injected into the field when displaying or converting it into a form input.

### Default Value

It may happen that the attribute does not exists on the model that matches with the field, or its value needs to be converted before using it from the field. In that case you may define a default value resolver on your field instance:

```php
use Cone\Root\Fields\Text;
use Illuminate\Databsase\Eloquent\Model;
use Illuminate\Http\Request;

Text::make('Title')
    ->default(static function (Request $request, Model $model, mixed $value) {
        return is_null($value) ? 'foo' : $value;
    });
```

> Alternatively, you may do default value definitions on your model directly.

### Formatting Value

You may define custom value formatters for the field values. In that case you can easily define a format resolver with the `format` method:

```php
use Cone\Root\Fields\Number;
use Illuminate\Databsase\Eloquent\Model;
use Illuminate\Http\Request;

Number::make('Price')
    ->format(static function (Request $request, Model $model, mixed $value) {
        return sprintf('%d USD', $value);
    });
```

### Value Hydration

You may define custom value hydration logic on your field clasas. To do so, you can easily override the default `hydrate` method:

```php
namespace App\Root\Fields;

use Cone\Root\Fields\Field;
use Illuminate\Databsase\Eloquent\Model;
use Illuminate\Http\Request;

class CustomField extends Field
{
    /**
     * Hydrate the model.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @param  mixed  $value
     * @return void
     */
    public function hydrate(Request $request, Model $model, mixed $value): void
    {
        $model->saving(function (Model $model) use ($value): void {
            $model->setAttribute($this->name, $value);
        });
    }
}
```

## Visibility

You may show or hide fields based on the current resource view. For example, some fields might be visible on the index page, while others should be hidden. You can easily customize the field visibility logic using the `visible` method:

```php
use Illuminate\Http\Request;

$field->visibleOn(static function (Request $request): bool {
    return $request->user()->isAdmin();
});

$field->hiddenOn(static function (Request $request): bool {
    return ! $request->user()->isAdmin();
});
```

Also, you can use the built-in methods as well. They are enough in most of the cases:

```php
$field->visibleOnIndex();
$field->visibleOnCreate();
$field->visibleOnShow();
$field->visibleOnUpdate();
$field->visibleOnDisplay();
$field->visibleOnForm();

$field->hiddenOnIndex();
$field->hiddenOnCreate();
$field->hiddenOnShow();
$field->hiddenOnUpdate();
$field->hiddenOnDisplay();
$field->hiddenOnForm();
```

You can also pass a `Closure` to any of the listed methods, to fully customize the visibility logic:

```php
$this->visible(static function (Request $request): bool {
    return $request->user()->can('attachTag', Post::class);
});

$this->hidden(static function (Request $request): bool {
    return $request->user()->cannot('attachTag', Post::class);
});
```

## Authorization

You may allow/disallow interaction with fields. To do so, you can call the `authorize` method on the field instance:

```php
$field->authorize(static function (Request $request): bool {
    return $request->user()->can('create', Post::class);
});
```

Also, when authorizing fields, pivot fields or actions, you may have access to the models of the context:

```php
// Action or top level field
$field->authorize(static function (Request $request, ?Model $model = null): bool {
    if (is_null($model)) {
        return $request->user()->can('create', Post::class);
    }

    return $request->user()->can('view', $model);
});

// Pivot field
$field->authorize(static function (Request $request, ?Model $model = null, ?Model $related = null): bool {
    if (is_null($model) || is_null($related)) {
        return $request->user()->can('create', Tag::class);
    }

    return $request->user()->can('attachTag', $model, $related);
});
```

> The `$model` and the `$related` parameters can be null, since they are only passed to the callback when they are available in the current authorization context.

## Validation

You may define validation rules for your field instances. To do so, you can call the `rules`, `createRules` and `updateRules` on the field instance:

```php
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;
use Illuminate\Validation\Rule;

$field->rules(['string', 'required'])
    ->createRules(['unique:posts'])
    ->updateRules(static function (Request $request, Model $model): array {
        return [Rule::unique('posts')->ignoreModel($model)];
    });
```

> You can pass an `array` or a `Closure` to the rule methods.

The value passed to the `rules` method will be present for both creating and updating models.

## Searchable and Sortable Fields

On index or extract views you may want to filter the results by sorting or searching any desired fields. To do so you can call the `sortable` and the `searchable` methods on your field instance:

```php
$field->sortable();
$field->sortable(false);

$field->searchable();
$field->searchable(false);
```

## Available Fields

### Text
### Textarea
### Number
### Select
### Checkbox
### Radio
### Boolean
### Range
### Color
### Date
### Editor
### Media
### File
### BelongsTo
### BelongsToMany
### HasOne
### HasMany
### MorphOne
### MorphMany
### MorphToMany

## Creating Fields

In some cases, the default fields are not enough, or don't provide the flexibility you need. In that scenario you can create your own custom fields easily. To generate a field easily you can call the following artisan command:

```sh
php artisan root:field Autocomplete
```

### Custom Vue Component
### Registering Custom Field Routes
