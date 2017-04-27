# php simple router

a very lightweight single file of the router.

> referrer the porject [noahbuscher\macaw](https://github.com/noahbuscher/Macaw) , but add some feature.

- supported request method: `GET` `POST` `PUT` `DELETE` `HEAD` `OPTIONS`
- support event: `found` `notFound`. you can do something on the trigger event.
- support custom the founded route parser: `SRoute::setRouteParser()`. you can custom how to call matched route handler.
- allow some special config, by `SRoute::config()`. 

## install

```json
"require": {
    "inhere/sroute": "dev-master"
}
```

## usage

first, import the class

```php
use inhere\sroute\SRoute;
```

### add routes

```php
// match GET. handler use Closure
SRoute::get('/', function() {
    echo 'hello';
});

// access 'test/john'
SRoute::get('/test/(\w+)', function($arg) {
    echo $arg; // 'john'
});

// match POST
SRoute::post('/user/login', function() {
    var_dump($_POST);
});

// match GET or POST
SRoute::map(['get', 'post'], '/user/login', function() {
    var_dump($_GET, $_POST);
});

// match any method
SRoute::any('/home', function() {
    echo 'hello';
});
```

#### use controller action

```php
// if you config 'ignoreLastSep' => true, '/index' is equals to '/index/'
SRoute::get('/index', 'app\controllers\Home@index');
```

#### dynamic action

match dynamic action, config `'dynamicAction' => true`

> NOTICE: use dynamic action, should be use `any()`.

```php
// access '/home/test' will call 'app\controllers\Home::test()'
SRoute::any('/home/(\w+)', app\controllers\Home::class);

// can match '/home', '/home/test'
SRoute::any('/dynamic(/\w+)?', app\controllers\Home::class);
```

#### use action executor

if you config `'actionExecutor' => 'run'`

```php
// access '/user', will call app\controllers\User::run('')
// access '/user/profile', will call app\controllers\User::run('profile')
SRoute::get('/user', 'app\controllers\User');
SRoute::get('/user/profile', 'app\controllers\User');

// if config 'actionExecutor' => 'run' and 'dynamicAction' => true,
// access '/user', will call app\controllers\User::run('')
// access '/user/profile', will call app\controllers\User::run('profile')
SRoute::get('/user(/\w+)?', 'app\controllers\User');
```

### setting events(if you need)

```php
SRoute::any('/404', function() {
    echo "Sorry,This page {$_GET['path']} not found.";
});
```

```php
// on found
SRoute::on(SRoute::FOUND, function ($uri, $cb) use ($app) {
    $app->logger->debug("Matched uri path: $uri, setting callback is: " . is_string($cb) ? $cb : 'Unknown');
});

// on notFound, redirect to '/404'
SRoute::on('notFound', '/404');

// can also, on notFound, output a message.
SRoute::on('notFound', function ($uri) {
    echo "the page $uri not found!";
});
```

### setting config(if you need)

```php
// set config
SRoute::config([
    'stopOnMatch' => true,
    'ignoreLastSep' => true,
    'dynamicAction' => true,
]);
```

- default config

```php
// there are default config.
[
    // stop On Matched. only match one
    'stopOnMatch'  => true,
    
    // Filter the `/favicon.ico` request.
    'filterFavicon' => false,
    
    // ignore last '/' char. If is true, will clear last '/'.
    'ignoreLastSep' => false,

    // enable dynamic action.
    // e.g
    // if set True;
    //  SRoute::any('/demo/(\w+)', app\controllers\Demo::class);
    //  you access '/demo/test' will call 'app\controllers\Demo::test()'
    'dynamicAction' => false,

    // action executor. will auto call controller's executor method to run all action.
    // e.g
    //  `run($action)`
    //  SRoute::any('/demo/(?<act>\w+)', app\controllers\Demo::class);
    //  you access `/demo/test` will call `app\controllers\Demo::run('test')`
    'actionExecutor' => '', // 'run'
]
```

> NOTICE: you must call `SRoute::config()` on before the `SRoute::dispatch()`

### begin dispatch

```php
SRoute::dispatch();
```

## License 

MIT