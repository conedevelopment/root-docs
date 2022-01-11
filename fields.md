# Fields

Fields are handlers for the model attributes. They are responsible for saving and displaying the given attribute of the resource model.

## Values

Field values are extracted from the given model's attributes. The attribute that is matching with the name of the field will be injected into the field when displaying or converting it into a form input.

### Defining Default Value

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
### Value Hydration

## Visibility

## Authorization

## Validation

## Searchable and Sortable Fields

## Available Fields

## Creating Fields
