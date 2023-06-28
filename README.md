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
		
		return $next($request);
	}
}
```

A middleware can perform tasks before or after the request is passed on deeper in the application (onward to the web routes & controllers). The example above would perform its task **before** passing on the request. The following controller would run its code **after**:
```
class Middleware
{
	public function handle(Request $request, Closure $next)
	{
		$response =  $next($request);
		
		// Some code here...
		
		return $response;
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

If a controller action is complex, you can dedicate it to this action only. Define an *\_\_invoke()* method on the controller that will be called every time the controller is ran. You do not need to specify the method in your route file, only the controller's name. Generate an invokable controller using the `--invokable` flag of the `make:controller` artisan commad.

### Controller Middleware

Middleware is usually assigned in the routes file:
```
Route::get('profile', [UserController::class, 'show'])->middleware('auth');
```

However, you can also use the *middleware* method in the constructor of your controller: 
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

You can create controllers with all the CRUD associated (create, read, update, delete) methods using the `--resource` flag of the `make:controller` artisan command. Create an associated resource route that points to the controller using the *resource* method on the *Route* object:
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

If using route model binding, specify the model type-hinted using the `--model` flag on the artisan `make:controller` command. Use the dotted notation to nest resources controllers.

## Views

### Create a View

Create a view by placing a `blade.php` file inside the *resources/views* folder. Return a view from the route file, like so (the array is data passed on to the view):
```
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

### Share Data with All Views

Use the *View* object's *share* method to share data with all views. Place the calls to the *share* method inside a service provider's *boot* method.

### View Composers

View composers are callbacks or class methods called when a view is rendered. They are useful when a view is rendered by several routes or controllers and and needs specific data. View composers are usually registered inside a service provider.

## Blade Templating

### Displaying Data

In blades, data is rendered like this: `Hello, {{ $name }}`. You do not have to only send variables to blades, you can pass any PHP statements to views: `The current UNIX timestamp is {{ time() }}`.
If using a Javscript framwork, use the `@` symbol to inform the Blade engine that an expression should remain untouched.
```
Hello, @{{ name }}.
```

The `@` symbol will be removed by Blade, however, the `{{ name }}` symbol will remain untouched. Use the `@verbatim` directive to avoid using the `@` before each blade directive. You may send a PHP array to your view with the intention to use it as JS object. In order to do so, you can use the `Illuminate\Support\Js::from` method:
```
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

### Blade Directives

Blade offers convenient shortcuts for common PHP statements, such as loops and conditionals. You may construct if statements using the `@if`, `@elseif`, `@else`, and `@endif` directives. They are working the same as their PHP counterparts:
```
@if (count($records) === 1)
    I have one record!
@endif
```

### Blade Components

Components can either be class based or autonomous. To create a class based component, use the `artisan make:component` command. It will create a class file in the `app/View/Components` directory, and the view will be placed in the `resources/views/components` directory. Use the `--view` flag to create an anonymous component, that is, a view without a class.

To display components, use the Blade component tag. Tags are starting with the string 'x-' followed by the kebab case name of the component class: `<x-user-profile/>`. If nested deeper inside the directory structure, use the dot notation: `<x-inputs.button/>`. If the components is to be rendered conditionally, define a 'shouldRender' method on the component class. If the method return false, the component will not render.

#### Passing Data to the Component

You may pass data to Blade components using HTML attributes. Like in Vue, simple strings are appended using vanilla attributes, and dynamic PHP values are prefixed with a ':' character:
```
<x-alert type="error" :message="$message"/>
```

You should define all of the components data attributes in the class constructor. Use the `camelCase` casing on the class, and the `kebab-case` casing for the HTML attributes. Additional dependencies should be declared in the constructor, before any of the component's properties. If you wish to prevent properties to be available to the view, add them to a protected '$except' array property on the component object. All properties and methods will be made available automatically to the view and do not need to be specified in the component's render method:
```
class Alert extends Component
{
    public function __construct(
        public string $type,
        public string $message,
		public AlertCreator $creator,
    ) {}
 
    public function render(): View
    {
        return view('components.alert');
    }
}
```

Once the component is rendered, display the content of the variables by echoing the variables name:
```
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

Since some JS frameworks use the same notation, you may write a double-colon '::' to inform Blade that the attribute is not a PHP expression: `<x-button ::class="{ danger: isDeleting }">` will render as `<x-button :class="{ danger: isDeleting }">`. You might want to add properties that are not defined in the component object, such as classes, inside the view. Those properties will be available as an `$attributes` variable inside the view, and can be accessed as such: 
```
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

The `$attributes` variable has a *merge* method, which is used to assign default values to HTML elements. The method accepts an array, where the key is the HTML attribute and the value is the HTML attribute's value.

### Slots

Slots allow you to pass additional data to your component. Echo the `$slot` variable inside your Blade view to render it:
```
<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Content is passed through the slot by injecting content inside the component:
```
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

Slots can be named, in order to display them in specific places inside your views, by using the `<x-slot:example>` notation. The string after the  ':' character has to match the variable echoed in the component. You can assign additional attributes to the slots, such as classes or input types. For very small component, you may register the view directly by inlining it in the component's class *render* method, as such:
```
public function render(): string
{
    return <<<'blade'
        <div class="alert alert-danger">
            {{ $slot }}
        </div>
    blade;
}
```

You may create a component that renders an inline view by using the `--inline` flag when executing the `make:component` command. For anonymous components, you can pass data using the `@props` directive at the top of you blade template.

### Building Layouts

Most websites use the same layout throughout various pages. You can define a `layout.blade.php` component inside your `resources/views/components` directory.

## Session

### Configuration

Because HTTP driven application are stateless, sessions provide a way to persist data about the user across multiple requests. That information is typically placed in a persistent store/backend that can be accessed from subsequent requests. Laravel provide the following *drivers* for session storage:
* _file_: sessions stored in `storage/framework/sessions`
* _cookie_: sessions stored in encrypted cookies
* _database_: sessions stored in a relational database
* _memcached/redis_: sessions stored in a cache based store/database
* _dynamodb_: sessions stored in AWS DynamoDB
* _array_: sessions stored in a PHP array that will not be persisted

Chose your driver in the configuration file stored at `config/session.php`. The array driver is primarily used for testing.
