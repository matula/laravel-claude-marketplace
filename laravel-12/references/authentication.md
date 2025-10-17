# Laravel 12 Authentication & Authorization Reference

## Authentication Best Practices

### Retrieving the Authenticated User

**CRITICAL**: Always use `$request->user()` to retrieve the authenticated user in controllers, NOT `auth()->user()` or `Auth::user()`.

```php
// ✅ CORRECT - Use $request->user()
public function index(Request $request): Response
{
    $user = $request->user();
    
    return view('dashboard', [
        'user' => $user,
    ]);
}

// ❌ WRONG - Do not use auth() or Auth::user()
public function index(): Response
{
    $user = auth()->user(); // Don't do this
    $user = Auth::user();   // Don't do this
}
```

### Type-Hinting Controllers

Always type-hint `Request $request` in controller methods to access the authenticated user:

```php
use Illuminate\Http\Request;
use Illuminate\Http\Response;

public function store(Request $request): Response
{
    $user = $request->user();
    
    // Use the authenticated user
    $post = $user->posts()->create([
        'title' => $request->input('title'),
        'body' => $request->input('body'),
    ]);
    
    return response()->json($post);
}
```

## Authorization with Gates and Policies

### Using Policies

Define authorization logic in policy classes:

```php
// app/Policies/PostPolicy.php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
    
    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

### Authorizing in Controllers

```php
public function update(Request $request, Post $post): Response
{
    $this->authorize('update', $post);
    
    $post->update($request->validated());
    
    return response()->json($post);
}
```

### Using Gates

For simple authorization checks:

```php
// In a service provider
Gate::define('edit-settings', function (User $user) {
    return $user->is_admin;
});

// In a controller
public function edit(Request $request): Response
{
    if (! $request->user()->can('edit-settings')) {
        abort(403);
    }
    
    // Continue...
}
```

## Laravel Sanctum for APIs

### Setting Up Sanctum

Sanctum provides token-based authentication for SPAs and mobile apps:

```php
// Create a token
$token = $user->createToken('token-name')->plainTextToken;

// Revoke tokens
$user->tokens()->delete();

// Revoke specific token
$user->currentAccessToken()->delete();
```

### Protecting API Routes

```php
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user', function (Request $request) {
        return $request->user();
    });
    
    Route::apiResource('posts', PostController::class);
});
```

## Password Reset & Email Verification

### Email Verification

```php
// In routes (typically routes/auth.php)
Route::get('/email/verify/{id}/{hash}', [VerificationController::class, 'verify'])
    ->middleware(['signed'])
    ->name('verification.verify');

// Check if user has verified email
if (! $request->user()->hasVerifiedEmail()) {
    // Redirect to verification notice
}
```

### Password Reset

Laravel provides built-in password reset functionality via Laravel Fortify or Breeze.

## Two-Factor Authentication

When using Laravel Fortify or Jetstream, 2FA is available:

```php
// Enable 2FA
$user->fortifyTwoFactorAuthenticationEnabled = true;
$user->save();

// Get recovery codes
$recoveryCodes = json_decode(decrypt($user->two_factor_recovery_codes), true);
```

## Session Management

### Regenerating Session

Always regenerate session on login to prevent fixation:

```php
$request->session()->regenerate();
```

### Invalidating Other Sessions

```php
Auth::logoutOtherDevices($password);
```

## Best Practices Summary

1. **Always use `$request->user()`** instead of `auth()->user()` or `Auth::user()`
2. **Type-hint Request** in all controller methods that need authentication
3. **Use Policies** for model-based authorization logic
4. **Use Gates** for simple, model-independent authorization
5. **Leverage Sanctum** for API authentication
6. **Regenerate sessions** on login to prevent fixation attacks
7. **Use Form Requests** for validation that includes authorization checks
