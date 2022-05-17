---
title: "Filters"
github: "https://github.com/conedevelopment/root-docs/blob/master/filters.md"
order: 8
---

Filters are responsible for transforming the current request to a database query.

## Registering Filters

## Configuration

### Authorization

You may allow or disallow interaction with actions. To do so, you can call the `authorize` method on the action instance:

```php
$action->authorize(static function (RootRequest $request): bool {
    return $request->user()->can('batchPublish', Post::class);
});
```
