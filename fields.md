---
title: "Fields"
github: "https://github.com/conedevelopment/root-docs/blob/master/fields.md"
order: 5
---

Fields are handlers for the model attributes. They are responsible for saving and displaying the given attribute of the resource model.

## Creating Fields

In some cases, the default fields are not enough, or don't provide the flexibility you need. In that scenario you can create your own custom fields easily. To generate a field easily you can call the following artisan command:

```sh
php artisan root:field Autocomplete
```

## Registering Fields

You can register fields in resources, extracts, actions and other fields by using the `fields` method.

```php
use Cone\Root\Fields\Text;
use Cone\Root\Http\Requests\RootRequest;

/**
 * Define the fields for the resource.
 *
 * @param  \Cone\Root\Http\Requests\RootRequest  $request
 * @return array
 */
public function fields(RootRequest $request): array
{
    return array_merge(parent::fields($request), [
        Text::make(__('Title'), 'title'),
    ]);
}
```

Alternatively, you can use `withFields` method on an object that resovles actions. It can be useful when you just want to hook into the object for some reason.

```php
use App\Root\Fields\Text;
use Cone\Root\Http\Requests\RootRequest;
use Cone\Root\Support\Collections\Fields;

$resource->withFields(static function (RootRequest $request, Fields $fields): Fields {
    return $fields->merge([
        Text::make(__('Title'), 'title'),
    ]);
});
```

> You can also pass an `array` instead of a `Closure`. In that case the array will be merged into the collection.

### Configuration

### Value Resolution

Field values are extracted from the given model's attributes. The attribute that is matching with the name of the field will be injected into the field when displaying or converting it into a form input.

It may happen that the attribute does not exists on the model that matches with the field, or its value needs to be converted before using it from the field. In that case you may define a default value resolver on your field instance:

```php
use Cone\Root\Fields\Text;
use Illuminate\Databsase\Eloquent\Model;
use Illuminate\Http\Request;

Text::make('Title')
    ->default(static function (Request $request, Model $model, mixed $value): string {
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
    ->format(static function (Request $request, Model $model, mixed $value): string {
        return sprintf('%d USD', $value);
    });
```

### Value Hydration

You may define custom value hydration logic on your field clasas. To do so, you can easily override the default `hydrate` method:

```php
namespace App\Root\Fields;

use Cone\Root\Fields\Field;
use Cone\Root\Http\Requests\RootRequest;
use Illuminate\Databsase\Eloquent\Model;

class CustomField extends Field
{
    /**
     * Hydrate the model.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @param  mixed  $value
     * @return void
     */
    public function hydrate(RootRequest $request, Model $model, mixed $value): void
    {
        $model->setAttribute($this->name, $value);
    }
}
```

## Authorization

You may allow/disallow interaction with fields. To do so, you can call the `authorize` method on the field instance:

```php
use Cone\Root\Http\Requests\RootRequest;

$field->authorize(static function (RootRequest $request): bool {
    return $request->user()->can('create', Post::class);
});
```

Also, when authorizing fields, pivot fields or actions, you may have access to the models of the context:

```php
use Cone\Root\Http\Requests\RootRequest;
use Illuminate\Database\Eloquent\Model;

// Action or top level field
$field->authorize(static function (RootRequest $request, ?Model $model = null): bool {
    if (is_null($model)) {
        return $request->user()->can('create', Post::class);
    }

    return $request->user()->can('view', $model);
});

// Pivot field
$field->authorize(static function (RootRequest $request, ?Model $model = null, ?Model $related = null): bool {
    if (is_null($model) || is_null($related)) {
        return $request->user()->can('create', Tag::class);
    }

    return $request->user()->can('attachTag', $model, $related);
});
```

> The `$model` and the `$related` parameters can be null, since they are only passed to the callback when they are available in the current authorization context.

### Visibility

You may show or hide fields based on the current resource view. For example, some fields might be visible on the index page, while others should be hidden. You can easily customize the field visibility logic using the `visible` method:

```php
use Cone\Root\Http\Requests\RootRequest;

$field->visibleOn(static function (RootRequest $request): bool {
    return $request->user()->isAdmin();
});

$field->hiddenOn(static function (RootRequest $request): bool {
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
use Cone\Root\Http\Requests\RootRequest;

$this->visible(static function (RootRequest $request): bool {
    return $request->user()->can('attachTag', Post::class);
});

$this->hidden(static function (RootRequest $request): bool {
    return $request->user()->cannot('attachTag', Post::class);
});
```

### Validation

You may define validation rules for your field instances. To do so, you can call the `rules`, `createRules` and `updateRules` on the field instance:

```php
use Cone\Root\Http\Requests\RootRequest;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Validation\Rule;

$field->rules(['string', 'required'])
    ->createRules(['unique:posts'])
    ->updateRules(static function (RootRequest $request, Model $model): array {
        return [Rule::unique('posts')->ignoreModel($model)];
    });
```

> You can pass an `array` or a `Closure` to the rule methods.

The value passed to the `rules` method will be present for both creating and updating models.

### Searchable and Sortable Fields

On index or extract views you may want to filter the results by sorting or searching any desired fields. To do so you can call the `sortable` and the `searchable` methods on your field instance:

```php
$field->sortable();
$field->sortable(false);

$field->searchable();
$field->searchable(false);
```

## Available Fields

### BelongsTo

### BelongsToMany

### Boolean

### Checkbox

### Color

### Date

### Editor

### File

### HasMany

### HasOne

### ID

### Json

### Media

### MorphMany

### MorphOne

### MorphToMany

### Number

### Radio

### Range

### Select

### Tag

### Text

The `Text` field is typically a handler for `string` model attributes:

```php
$field = Text::make(__('Title'), 'title');
```

You can also apply modifiers on a text field:

```php
// Adds the "size" HTML input attribute
$field->size(40);

// Adds the "minlength" HTML input attribute
$field->minlength(40);

// Adds the "maxlength" HTML input attribute
$field->maxlength(40);
```

### Textarea

The `Textarea` field is typically a handler for longer `string` model attributes:

```php
$field = Textarea::make(__('Bio'), 'bio');
```

You can also apply modifiers on a text field:

```php
// Adds the "rows" HTML input attribute
$field->rows(20);

// Adds the "cols" HTML input attribute
$field->cols(100);
```
