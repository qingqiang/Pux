Phux
=============
The Fast PHP router.


Phux tries not to consume computation time to build all routes dynamically (like Symfony/Routing). Instead,
Phux compiles your routes to plain PHP array for caching, the compiled routes can be loaded from cache very fast.

There are two route dispatching strategy in Phux while Symfony/Routing only
provides PCRE pattern matching:

1. Plain string comparison.
2. PCRE pattern comparison.

You've already knew that PCRE pattern matching is slower than plain string comparison, although PHP PCRE caches the pattern.

The plain string comparison is designed for these static routing paths, it does
improve the performance while you have simple routes.

The PCRE pattern comparison is used when you have some dynamic routing paths,
for example, you can put the some place holder in your routing path, and pass
these path arguments to your controller later.

Phux sorts and compiles your route to single cache, we use longest matching so
we sort long patterns in descending order before compiling the routes to cache.

Phux uses indexed array as the data structure for storing route information so it's faster.


Synopsis
------------

```php
class ProductController {
    public function listAction() {
        return 'product list';
    }
    public function itemAction($id) { 
        return "product $id";
    }
}

$router = new Router([ 'cache_file' => 'mux.php' ]);

if ( false === $router->load() ) {
    $router->add('/product', ['ProductController','listAction']);
    $router->add('/product/:id', ['ProductController','itemAction'] , [
        'require' => [ ':id' => '\d+', ],
        'default' => [ ':id' => '1', ]
    ]);

    $router->save();
}
$route = $router->dispatch('/product/1');
```

Mux
-----
Mux is where you define your routes, and you can mount multiple mux to a parent one.

```php
$mainMux = new Mux;
$mainMux->expandSubMux = true;

$pageMux = new Mux;
$pageMux->add('/page1', [ 'PageController', 'page1' ]);
$pageMux->add('/page2', [ 'PageController', 'page2' ]);

$mainMux->mount('/sub', $pageMux);

foreach( ['/sub/page1', '/sub/page2'] as $p ) {
    $r = $mainMux->dispatch($p);
    ok($r, "Matched route for $p");
}
```

The `expandSubMux` option means whether to expand/merge submux routes to the parent mux.

When expandSubMux is enabled, it improves dispatch performance when you
have a lot of sub mux to dispatch.

### Different String Comparison Strategies

When expandSubMux is enabled, the pattern comparison strategy for 
strings will match the full string.

When expandSubMux is disabled, the pattern comparison strategy for 
strings will match the prefix.
