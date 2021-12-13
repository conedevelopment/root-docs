# Resources

Resources are basically extra layers around models. Using resources it's easy to build up a CRUD workflow for a model, using [fields](#), [filters](#), [actions](#), [extracts](#) or [widgets](#).

## Resource Models

You can easily make a model "resourceable" by implementing the `Cone\Root\Interfaces\Resourceable` interface on the model class.

```php
namespace App\Models;

use Cone\Root\Interfaces\Resourceable;
use Cone\Root\Traits\InteractsWithResource;
use Cone\Root\Resources\Resource;
use Illuminate\Database\Eloquent\Model;

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
                //
            ]);
    }
}
```

> The `Cone\Root\Traits\InteractsWithResource` trait might be a big help, but of course you can customize your behavior as you wish. Also, you can just simply override the trait methods.
