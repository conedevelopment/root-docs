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

## Writing Resources

### Inline Resources

Inline resources are useful when you don't want to make a custom resource class, or when you just want to override something in the "parent" resource.

```php
class Post extends Model implements Resourcable
{
    use InteractsWithResource {
        InteractsWithResource::toResource as toDefaultResource;
    }

    /**
     * Get the resource representation of the model.
     *
     * @return \Cone\Root\Resources\Resource
     */
    public static function toResource(): Resource
    {
        return static::toDefaultResource()
                    ->withFields([
                        ID::make(),
                    ])
                    ->authorize(static function (Request $request): bool {
                        return $request->user()->can('create', static::class);
                    });
    }
}
```

#### Fields
#### Filters
#### Actions
#### Widgets
#### Extracts

### Resource Classes
