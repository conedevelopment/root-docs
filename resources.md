# Resources

Resources are basically extra layers around models. Using resources it's easy to build up a CRUD workflow for a model, using [fields](#), [filters](#), [actions](#), [extracts](#) or [widgets](#).

## Resource Models

You can easily make a model "resourceable" by implementing the `Cone\Root\Interfaces\Resourceable` interface on the model class.

```php
namespace App\Models;

use App\Root\Resources\PostResource;
use Cone\Root\Interfaces\Resourceable;
use Cone\Root\Traits\InteractsWithResource;
use Cone\Root\Resources\Resource;
use Illuminate\Database\Eloquent\Model;

class Post extends Model implements Resourcable
{
    use InteractsWithResource;

    /**
     * Get the resource representation of the model.
     *
     * @return \Cone\Root\Resources\Resource
     */
    public static function toResource(): Resource
    {
        return new PostResource(static::class);
    }
}
```

### Registering Resources

You may register your resources using the model that it corresponds to. Typcally you may use a service provider for that:

```php
namespace App\Providers;

use App\Models\Post;
use Cone\Root\Root;
use Illuminate\Support\ServiceProvider;

class ResourceServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot(): void
    {
        Root::running(static function (): void {
            Post::registerResource();
        });
    }
}
```

## Writing Resources

You may create a Resource class using the `root:resource` artisan command. It requires only a name as its parameter and generates the resource class for the model:

```sh
php artisan root:resource PostResource

# or

php artisan root:resource CustomPostResource --model=Post
```

### Fields

> For the detailed documentation visit the [fields](#) section.

Fields are handlers for the model attributes. They are responsible for saving and displaying the given attribute of the resource model. You can easily define fields on your resource by using the `fields` method:

```php
class PostResource extends Resource
{
    /**
     * Define the fields for the resource.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function fields(Request $request): array
    {
        return [
            ID::make(),
            Text::make('Title'),
        ];
    }
}
```

Alternatively, you can use `withFields` method on an initialized resoure instance. It can be useful when you just want to hook into the resource instance for some reason. Typically you may do that in your model's `toResource` method:

```php
use Cone\Root\Support\Collections\Fields;
use Illuminate\Http\Request;

class Post extends BasePost
{
    /**
     * Get the resource representation of the model.
     *
     * @return \Cone\Root\Resources\Resource
     */
    public static function toResource(): Resource
    {
        return parent::toResource()
            ->withFields(static function (Request $request, Fields $fields): Fields {
                return $fields->merge([
                    Textarea::make('Excerpt'),
                ]);
            });
    }
}
```

> You can also pass an `array` instead of a `Closure`. In that case the array will be merged to the collection.

### Filters

> For the detailed documentation visit the [filters](#) section.

Filters are used on resource indexes and extract views. They are responsible for transforming the current request to a database query. You can easily define filters on your resource by using the `filters` method:

### Actions

> For the detailed documentation visit the [actions](#) section.

### Widgets

> For the detailed documentation visit the [widgets](#) section.

### Extracts

> For the detailed documentation visit the [extracts](#) section.
