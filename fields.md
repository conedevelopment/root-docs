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
use Illuminate\Http\Request;

/**
 * Define the fields for the resource.
 */
public function fields(Request $request): array
{
    return [
        Text::make(__('Title'), 'title'),
    ];
}
```

### Configuration

### Value Resolution

Field values are extracted from the given model's attributes. The attribute that is matching with the name of the field will be injected into the field when displaying or converting it into a form input.

It may happen that the attribute does not exists on the model that matches with the field, or its value needs to be converted before using it from the field. In that case you may define a default value resolver on your field instance:

```php
use Cone\Root\Fields\Text;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;

Text::make('Title')
    ->value(static function (Request $request, Model $model, mixed $value): string {
        return is_null($value) ? 'foo' : $value;
    });
```

> Alternatively, you may do default value definitions on your model directly.

### Formatting Value

You may define custom value formatters for the field values. In that case you can easily define a format resolver with the `format` method:

```php
use Cone\Root\Fields\Number;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;
use Illuminate\Support\Number;

Number::make('Price')
    ->format(static function (Request $request, Model $model, mixed $value): string {
        return Number::currency($value);
    });
```

### Value Hydration

You may define custom value hydration logic on your field. To do so, use the `hydrate` method passing a `Closure`:

```php
use Cone\Root\Fields\Number;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;
use Illuminate\Support\Number;

Number::make('Price')
    ->hydrate(static function (Request $request, Model $model, mixed $value): void {
        $model->setAttribute('price', $value);
    });
```

## Authorization

You may allow/disallow interaction with fields. To do so, you can call the `authorize` method on the field instance:

```php
use Illuminate\Http\Request;

$field->authorize(static function (Request $request, Model $model): bool {
    return $request->user()->isAdmin();
});
```

### Visibility

You may show or hide fields based on the current resource view. For example, some fields might be visible on the `index` page, while others should be hidden. You can easily customize the action visibility logic using the `visibleOn` and `hiddenOn` methods:

```php
$field->visibleOn(['index', 'show']);

$field->hiddenOn(['update', 'create']);
```

### Validation

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

### Searchable and Sortable Fields

On index or extract views you may want to filter the results by sorting or searching any desired fields. To do so you can call the `sortable` and the `searchable` methods on your field instance:

```php
$field->sortable();
$field->sortable(false);

$field->searchable();
$field->searchable(false);
```

## Available Fields

### Boolean

The `Boolean` field is typically a handler for `boolean` model attributes:

> Don't forget to [cast](https://laravel.com/docs/master/eloquent-mutators#attribute-casting) you model attribute as a `boolean`.

```php
$field = Boolean::make(__('Enabled'), 'enabled');
```

### Checkbox

The `Checkbox` field is typically a handler for `json` model attributes (array of values):

> Don't forget to [cast](https://laravel.com/docs/master/eloquent-mutators#attribute-casting) you model attribute as a `json` or `array`.

```php
$field = Checkbox::make(__('Permissions'), 'permissions')
    ->options([
        'view' => 'view',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ]);
```

You may also pass a `Closure` to customize the resolution logic of the options:

```php
use App\Category;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;

$field = Checkbox::make(__('Categories'), 'categories')
    ->options(static function (Request $request, Model $model): array {
        return match (true) {
            $request->user()->isAdmin() => Category::query()->pluck('name', 'id')->all(),
            default => $request->user()->categories()->pluck('name', 'id')->all(),
        };
    });
```

### Color

The `Color` field is typically a handler for `color` model attributes:

```php
$field = Color::make(__('Background Color'), 'bg_color');
```

> You may use the `hex_color` validation rule for this field. By default no rules attached.

### Date

The `Date` field is typically a handler for `date` or `datetime` model attributes:

```php
$field = Date::make(__('Expires at'), 'expires_at');
```

> You may use the `date` validation rule for this field. By default no rules attached.

You can also apply modifiers on a `Date` field:

```php
// Adds/removes a time handler for the field
$field->withTime();
$field->withTime(false);

// Sets the displayed timezone for the input
// Note, the value is saved to the database in the app's timezone (normally UTC)
$field->timezone('Europe/Budapest');

// Adds the "step" HTML input attribute
$field->step(1);

// Adds the "min" HTML input attribute
$field->min('2024-01-01');
$field->min(now()->addDays(7));

// Adds the "max" HTML input attribute
$field->max('2024-01-01');
$field->max(now()->addDays(7));
```

### Editor

The `Editor` field is typically a handler for `text` model attributes (HTML). The `Editor` field is a [Tiptap](https://tiptap.dev/product/editor) editor combined with [alpine](https://alpinejs.dev/):

```php
$field = Editor::make(__('Content'), 'content');
```

The `root.php` config file contains the default configuration for each editor instance. You may edit the config file or you can also customize the configuration per instance:

```php
$field->withConfig(static function (array $config): array {
    return array_merge($config, ['autofocus' => true]);
});
```

It is also possible using the [Media Library](#media) with the `Editor` field to insert existing media files or upload new ones:

```php
$field->withMedia();

// or you may customize the media field
$field->withMedia(static function (Media $media): void {
    $media->withRelatableQuery(static function (Request $request, Builder $query): Builder {
        return $query->where('user_id', $request->user()->getKey());
    });
});
```

You can also apply modifiers on a `Editor` field:

```php
// Sets the editor height
$field->height('300px');
```

### File

### ID

### Media

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

### Relation Fields

Relation fields are representing Eloquent relation definitions on the resource models. Relation fields are highly customizable and provide a nice and detailed API.

#### BelongsTo

The `BelongsTo` field is typically a handler for a `Illuminate\Database\Eloquent\Relations\BelongsTo` relation:

```php
$field = BelongsTo::make(__('Author'), 'author');
```

> Root assumes that there is an already defined `author` relation on the Resource Model.

#### BelongsToMany

#### HasMany

#### HasOne

#### MorphMany

#### MorphOne

#### MorphToMany
