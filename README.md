# Laravel Notes (Certification)

## Routing

### Basic Routing

Route interface implement all the HTTP verbs:

- GET
- POST
- PUT
- PATCH
- DELETE
- OPTIONS

You can create routes that responds to many HTTP verbs:
```
Route::match(['get', 'post'], '/home', function() {
	// Some code...
});
```

The *any* method registers a route that responds to **all** HTTP verbs. The *redirect* method redirects to another URI (third parameter adds a status, default is 301). If returning a view, use the *view* method.

### Route Parameters

You can catch a route parameter as a function argument, like this:
```
Route::get('/user/{id}', function (string $id) {
    return 'User '.$id;
});
```

Add a '?' at the end of the parameter to specify that it's optional. Remember to give the argument a default value. Using the *where* method, you may restrain the format of the route parameter using **regex**, as such: 
```
->where('id', '[0-9]+');
```

### Named Routes

Use the *name* method to give a reusable name to a route:
```
->name('profile');
```

You can then redirect using the route name instead of URI:
```
redirect()->route('profile');
```

### Route Grouping

If some routes are all using the same middleware, group them using the *middleware* method:
```
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second middleware...
    });
 
    Route::get('/user/profile', function () {
        // Uses first & second middleware...
    });
});
```

Same goes for *controllers*, *prefix*, *name* and *domain* methods.

### Route Model Binding

When using a route parameter, if the route parameter matches the function argument's name and type hint, Laravel will automatically perform the dependency injection:
```
Route::get('/users/{user}', function (User $user) {
    return $user->email; // The $user variable is the User model.
});
```

Note that by default the function argument used to retrieve the Eloquent model is supposed to be the *id*. To use another column, specify it in the route parameter definition:
```
Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```

## Middlewares

### Defining Middlewares

To create a new middleware, use the artisan CLI:
```
php artisan make:middleware MiddlewareName
```

The middleware will be created in the *Middleware* directory. Pass the request after performing some code by calling the $next callback with the request as an argument:

```
class Middleware
{
	public function handle(Request $request, Closure $next)
	{
		// Some code here...
		
		return \$next(\$request);
	}
}
```

A middleware can perform tasks before or after the request is passed on deeper in the application (onward to the web routes & controllers). The example above would perform its task **before** passing on the request. The following controller would run its code **after**:
```
class Middleware
{
	public function handle(Request $request, Closure $next)
	{
		\$response =  \$next(\$request);
		
		// Some code here...
		
		return \$response;
	}
}
```

### Registering Middlewares

To make a middleware run at every request, register it in the \$middleware property in the **app/Http/Kernel.php** class.

### Assigning Middleware to Routes

Use the *middleware* method after the route definition:
```
Route::get('/profile', function () {
    // ...
})->middleware(Authenticate::class);
```

Use an array to register several middlewares. 

Add aliases for middlewares in the \$middlewareAliases property in the **app/Http/Kernel.php** class. You can then call it using the alias string:
```
->middleware('auth');
```

To groups middlewares, register them in the \$middlewareGroups property in the **app/Http/Kernel.php** class.

### Middleware Parameters

Parameters come after the Request and Closure parameters in the *handle* function. You can define them when assigning the middleware to a route (the example will grant the value 'editor' to the \$role variable):
```
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware('role:editor');
```

### Terminable Middleware

By adding a *terminate* function, you can handle a task **after** it has been sent to the browser: 
```
public function terminate(Request $request, Response $response) {
	// Some code here...
}
```

The instance of the middleware that will be called will be a fresh one, meaning without the data saved on the first instantiation. To use the same middleware for both method, register it as a singleton in the **AppServiceProvider** *register* method.

## Controllers

### Defining Controllers

Use the `artisan make:controller` command to create a new controller. Controllers are accessed via the *web.php* routing file, by binding them to a route with a corresponding method, like so:
```
Route::get('/user/{id}', [UserController::class, 'show']);
```

### Single Action Controller

If a controller action is complex, you can dedicate it to this action only. Define an *\_\_invoke()* method on the controller that will be called every time the controller is ran. You do not need to specify the method in your route file, only the controller's name. Generate an invokable controller using the `--invokable` option of the `make:controller` artisan commad.

### Controller Middleware

Middleware is usually assigned in the routes file:
```
Route::get('profile', [UserController::class, 'show'])->middleware('auth');
```

However, you can also use the *middleware* method in the constructor of your controller: 
```
```
```
public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('log')->only('index');
        $this->middleware('subscribed')->except('store');
    }
```

You can also register a middleware as a closure, that is, define an anonymous function inside the *middleware* method, to avoid creating an entire middleware class.

### Resource Controller

You can create controllers with all the CRUD associated (create, read, update, delete) methods using the `--resource` option of the `make:controller`. Create an associated resource route that points to the controller using the *resource* method on the *Route* object:
```
Route::resource('photos', PhotoController::class);
```

The *resource* method will automatically create multiple routes to handle all the actions defined in the controller (see in the documentation for a list of all routes). If the model automatically associated with the routes and controller isn't found, a 404 method will be returned. You can override this behavior by using the *missing* method on the resource route definition:
```
Route::resource('photos', PhotoController::class)
        ->missing(function (Request $request) {
            return Redirect::route('photos.index');
        });
```

For *soft deleted models*, use the *withTrashed* method.

The *resources* method on the *Route* object allows you to define several resource controllers all at once by specifying an array:
```
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

If using route model binding, specify the model type-hinted using the `--model` option on the artisan `make:controller` command. Use the dotted notation to nest resources controllers.

## Views

### Create a View

Create a view by placing a `blade.php` file inside the *resources/views* folder. Return a view from the route file, like so (the array is data passed on to the view):
```
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```



