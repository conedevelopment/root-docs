---
title: "Resources"
github: "https://github.com/conedevelopment/root-docs/blob/master/resources.md"
order: 3
---

Resources are basically extra layers around models. Using resources it's easy to build up a CRUD workflow for a model, using [fields](/docs/fields), [filters](/docs/filters), [actions](/docs/actions), [extracts](/docs/extracts) or [widgets](/docs/widgets).

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

You may register your resources by using the `RootServiceProvider` which is published and automatically registered when the `root:install` artisan command is called.

```php
namespace App\Providers;

use App\Models\User;
use App\Models\Post;
use Cone\Root\Root;
use Cone\Root\RootApplicationServiceProvider;

class RootServiceProvider extends RootApplicationServiceProvider
{
    /**
     * The resources.
     *
     * @return array
     */
    protected function resources(): array
    {
        return [
            User::toResource(),
            Post::toResource(),
        ];
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

> For the detailed documentation visit the [fields](/docs/fields) section.

Fields are handlers for the model attributes. They are responsible for saving and displaying the given attribute of the resource model. You can easily define fields on your resource by using the `fields` method:

```php
use Cone\Root\Fields\ID;
use Cone\Root\Fields\Text;
use Cone\Root\Http\Requests\RootRequest;
use Cone\Root\Resources\Resource;

class PostResource extends Resource
{
    /**
     * Define the fields for the resource.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function fields(RootRequest $request): array
    {
        return [
            ID::make(),
            Text::make('Title'),
        ];
    }
}
```

### Filters

> For the detailed documentation visit the [filters](/docs/filters) section.

Filters are responsible for transforming the current request to a database query. You can easily define filters on your resource by using the `filters` method:

```php
use App\Root\Filters\Category;
use Cone\Root\Http\Requests\RootRequest;
use Cone\Root\Resources\Resource;

class PostResource extends Resource
{
    /**
     * Define the filters for the resource.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function filters(RootRequest $request): array
    {
        return [
            Category::make(),
        ];
    }
}
```

### Actions

> For the detailed documentation visit the [actions](/docs/actions) section.

Actions are responsible for performing a specific action on a set of models. You can easily define actions on your resource by using the `actions` method:

```php
use App\Root\Actions\Publish;
use Cone\Root\Resources\Resource;
use Cone\Root\Http\Requests\RootRequest;

class PostResource extends Resource
{
    /**
     * Define the actions for the resource.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function actions(Request $request): array
    {
        return array_merge(parent::actions($request), [
            Publish::make(),
        ]);
    }
}
```

### Widgets

> For the detailed documentation visit the [widgets](/docs/widgets) section.

Widgets are cards that display some information or any content you want to display. You can easily define widgets on your resource by using the `widgets` method:

```php
use App\Root\Widgets\TotalPosts;
use Cone\Root\Http\Requests\RootRequest;
use Cone\Root\Resources\Resource;

class PostResource extends Resource
{
    /**
     * Define the widgets for the resource.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function widgets(RootRequest $request): array
    {
        return [
            TotalPosts::make(),
        ];
    }
}
```

### Extracts

> For the detailed documentation visit the [extracts](/docs/extracts) section.

Extracts are layers on the top of the resource index. They are responsible to perform more complex queries and display relevant information for the query, that you may not want to show on the index page. You can easily define extracts on your resource by using the `extracts` method:

```php
use App\Root\Extracts\PostLengths;
use Cone\Root\Http\Requests\RootRequest;
use Cone\Root\Resources\Resource;

class PostResource extends Resource
{
    /**
     * Define the extracts for the resource.
     *
     * @param  \Cone\Root\Http\Requests\RootRequest  $request
     * @return array
     */
    public function extracts(Request $request): array
    {
        return [
            PostLengths::make(),
        ];
    }
}
```
