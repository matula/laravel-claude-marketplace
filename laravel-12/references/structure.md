# Laravel 12 Structure & Conventions Reference

## Laravel 12 Streamlined File Structure

Laravel 12 (building on Laravel 11's changes) uses a simplified, streamlined structure:

### Key Structural Changes

1. **No `app/Http/Middleware/` directory** - Middleware is registered in `bootstrap/app.php`
2. **No `app/Console/Kernel.php`** - Console configuration in `bootstrap/app.php` or `routes/console.php`
3. **No `app/Http/Kernel.php`** - Middleware registration in `bootstrap/app.php`
4. **Commands auto-register** - Files in `app/Console/Commands/` automatically available

### Important Files

#### `bootstrap/app.php`

This is the central configuration file for middleware, routing, and exception handling:

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->web(append: [
            \App\Http\Middleware\HandleInertiaRequests::class,
        ]);
        
        $middleware->alias([
            'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

#### `bootstrap/providers.php`

Contains application-specific service providers:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
];
```

#### `routes/console.php`

Define console commands and schedules:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->daily();

Artisan::command('inspire', function () {
    $this->comment(Inspiring::quote());
})->purpose('Display an inspiring quote');
```

## Directory Structure

```
app/
├── Console/
│   └── Commands/          # Custom Artisan commands (auto-registered)
├── Exceptions/            # Custom exception classes
├── Http/
│   ├── Controllers/       # HTTP controllers
│   └── Requests/          # Form request validation classes
├── Models/                # Eloquent models
├── Policies/              # Authorization policies
└── Providers/             # Service providers
    └── AppServiceProvider.php

bootstrap/
├── app.php               # Main application configuration
├── cache/                # Framework cache files
└── providers.php         # Service provider registration

config/                   # Configuration files
database/
├── factories/           # Model factories
├── migrations/          # Database migrations
└── seeders/             # Database seeders

resources/
├── css/                 # Stylesheets
├── js/                  # JavaScript/TypeScript
└── views/               # Blade templates

routes/
├── web.php              # Web routes
├── api.php              # API routes
├── console.php          # Console commands and schedule
└── channels.php         # Broadcasting channels

tests/
├── Feature/             # Feature tests
└── Unit/                # Unit tests
```

## Artisan Commands

### Creating Files

Always use `php artisan make:` commands with proper options:

```bash
# Models with migrations, factories, and seeders
php artisan make:model Post -mfs

# Controllers with resource methods
php artisan make:controller PostController --resource

# Form request validation
php artisan make:request StorePostRequest

# Policies
php artisan make:policy PostPolicy --model=Post

# Custom Artisan command
php artisan make:command SendEmails

# Generic PHP class
php artisan make:class Services/PaymentService

# Jobs (queued by default)
php artisan make:job ProcessPodcast
```

### Common Options

- `-m` or `--migration` - Create migration
- `-f` or `--factory` - Create factory
- `-s` or `--seed` - Create seeder
- `-c` or `--controller` - Create controller
- `-p` or `--policy` - Create policy
- `-r` or `--resource` - Resource controller
- `--api` - API resource controller
- `--test` - Create test
- `--pest` - Create Pest test

### Always Use --no-interaction

When running Artisan commands programmatically, always pass `--no-interaction`:

```bash
php artisan make:model Post --no-interaction -mfs
```

## Console Commands Auto-Registration

Commands in `app/Console/Commands/` are automatically discovered and registered. No manual registration needed:

```php
// app/Console/Commands/SendEmails.php
namespace App\Console\Commands;

use Illuminate\Console\Command;

class SendEmails extends Command
{
    protected $signature = 'emails:send {user}';
    
    protected $description = 'Send emails to a user';
    
    public function handle(): void
    {
        $userId = $this->argument('user');
        
        $this->info("Sending emails to user {$userId}");
    }
}

// Command is automatically available:
// php artisan emails:send 123
```

## Service Providers

Keep service providers minimal in Laravel 12:

```php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Register bindings
    }

    public function boot(): void
    {
        // Bootstrap services
    }
}
```

## Configuration Best Practices

### Never Use env() Outside Config Files

```php
// ❌ WRONG - Using env() in application code
$apiKey = env('API_KEY');

// ✅ CORRECT - Use config() instead
$apiKey = config('services.api.key');
```

### Config Files

Always access configuration through the `config()` helper:

```php
// config/services.php
return [
    'api' => [
        'key' => env('API_KEY'),
        'secret' => env('API_SECRET'),
    ],
];

// In application code
$apiKey = config('services.api.key');
$apiSecret = config('services.api.secret');
```

## URL Generation

### Use Named Routes

Always prefer named routes and the `route()` function:

```php
// Define named route
Route::get('/posts/{post}', [PostController::class, 'show'])->name('posts.show');

// Generate URL
$url = route('posts.show', ['post' => $post->id]);

// In Blade
<a href="{{ route('posts.show', $post) }}">View Post</a>

// Redirect to route
return redirect()->route('posts.index');
```

## Middleware Registration

Register middleware in `bootstrap/app.php`:

```php
->withMiddleware(function (Middleware $middleware) {
    // Append to web middleware group
    $middleware->web(append: [
        \App\Http\Middleware\CustomMiddleware::class,
    ]);
    
    // Register middleware aliases
    $middleware->alias([
        'admin' => \App\Http\Middleware\AdminMiddleware::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ]);
    
    // Global middleware
    $middleware->append(\App\Http\Middleware\LogRequests::class);
})
```

## Best Practices Summary

1. **Use `bootstrap/app.php`** for middleware, routing, and exception configuration
2. **Commands auto-register** from `app/Console/Commands/`
3. **Always use `php artisan make:`** commands to create files
4. **Pass `--no-interaction`** when running Artisan programmatically
5. **Never use `env()`** outside configuration files
6. **Use named routes** with `route()` helper
7. **Keep service providers minimal** - only essential bindings and bootstrapping
8. **Follow the streamlined structure** - don't create old Laravel directories
