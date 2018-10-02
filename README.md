# Playing with Laravel and admin-lte

@amlwwalker

## Things to know

-   This is a PoC for local development only. On request I set this up to get someone started.
-   It assumes that database authentication is set to something other than the blindingly obvious
-   Laravel also is set to debug mode meaning if anything breaks, the front end will display all code and errors.
    -   Do not deploy to a production environment with debug mode on. See Laravel docs on how to disable

## You need

-   bower
-   laravel
-   PHP 7.2
-   composer

# Installation of libraries

-   Run `composer global require "laravel/installer=~2.0"`
-   Create a project with laravel: `laravel new PROJECTNAME`
    -   This will create a new directory called PROJECTNAME and will put all the basic files inside it you need
-   `cd PROJECTNAME/public`
-   run `bower install admin-lte`
    -   This will pull down all the admin-lte files as a resource that we can link to
-   Create a new (admin view) inside resources/views called admin.blade.php
    -   Paste into it the following content: https://gist.github.com/amlwwalker/06be920ba977598c8f62c1be6d8abd6d
-   Create a test route to see if it works.
    -   Inside routes/web.php add

```
Route::get('/admin', function () {
    return view('admin');
});
```

Now start the app with php artisan serve and go to `http://127.0.0.1:8000/admin` - you should see the new view that uses admin-lte

## Create and setup user login/registration/authentication

-   Make sure you have mysql installed and can connect with the username root, and no password. Please be aware this is NOT safe for live deployments
-   Connect to the database from terminal with `mysql -u root -p` - Press enter on requested for password as we set none.
-   Create a new database in mysql:
    -   `create database laravel`
    -   Inside the file .env set the database name to laravel, the user name to root and the password to blank

## Configure and setup for authentication and administrators

-   Get artisan to configure basic authentication for the application
    -   `php artisan make:auth`
-   Create a new middleware template for administration `php artisan make:middleware IsAdmin`
-   Create a new controller for administration pages `php artisan make:controller AdminController`
-   Create a new controller for registering new users `php artisan make:controller RegisterController`
-   Inside database/migrations/2014_10_12_000000_create_users_table.php add

```
$table->string('type')->default('default');
```

after

```
$table->timestamps();
```

-   Create a users table in the database `php artisan migrate`
-   In app/Http/Controllers/AdminController.php add

```
public function admin()
    {
        return view('admin');
    }
```

-   Update the isAdmin middleware (app/Http/Middleware/isAdmin.php) add the following to the class

```
    public function handle($request, Closure $next)
    {
        if(auth()->user()->isAdmin()) {
            return $next($request);
        }
        return redirect('home');
    }
```

-   Register the middleware so that it can be called upon in HTTP routing

    -   In app/Http/Kernel.php add the follwoing `'is_admin' => \App\Http\Middleware\IsAdmin::class,` in the section called `protected $routeMiddleware = [`

-   In RegisterController.php add the following to the class

```
protected function create(array $data)    {
    return User::create([
        'name' => $data['name'],
        'email' => $data['email'],
        'password' => bcrypt($data['password']),
        'type' => User::DEFAULT_TYPE,
    ]);
}
```

-   Add the following to the user class in app/User.php

```
const ADMIN_TYPE = 'admin';
const DEFAULT_TYPE = 'default';
public function isAdmin()    {
    return $this->type === self::ADMIN_TYPE;
}
```

-   Update the routes now so that depending on wehther someone is logged in/is an admin, they can experiecen different things. Set the content of app/routes/web.php to

```
<?php
Route::get('/', function () {
    return view('welcome');
});

Route::get('/admin', 'AdminController@admin')
    ->middleware('is_admin')
    ->name('admin');

Auth::routes();

Route::get('/home', 'HomeController@index')->name('home');
```

```
Route::get('/', function () {
    return view('welcome');
});

Route::get('/admin', 'AdminController@admin')
    ->middleware('is_admin')
    ->name('admin');

Auth::routes();

Route::get('/home', 'HomeController@index')->name('home');
```

# Get going!

-   OK you should have everything you need. Compare files to this project to see where there are discrepencies. This one works, so this README might potentially have errors.

Register a new user. They will not be a default user. In a new terminal, navigate to the project and you can upgrade a user with

-   `php artisan tinker`
-   `use App\User;`
-   `User::where('email', 'admin@admin.com')->update(['type' => 'admin']);`

Now when that user is logged in, you can navigate to /admin and they should be allowed to see the admin-lte admin page

## References:

-   https://medium.com/employbl/easily-build-administrator-login-into-a-laravel-5-app-8a942e4fef37
