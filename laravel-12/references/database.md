# Laravel 12 Database & Eloquent Reference

## Eloquent Relationships

### Always Use Proper Return Type Hints

```php
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
    
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}

class Post extends Model
{
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
    
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```

### Eager Loading to Prevent N+1

Always eager load relationships when accessing them in loops:

```php
// ❌ BAD - N+1 query problem
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->user->name; // Queries for each post
}

// ✅ GOOD - Single query with eager loading
$posts = Post::with('user')->get();
foreach ($posts as $post) {
    echo $post->user->name;
}

// ✅ GOOD - Multiple relationships
$posts = Post::with(['user', 'comments.user'])->get();
```

### Laravel 12: Limiting Eagerly Loaded Records

Laravel 12 allows limiting eagerly loaded records natively:

```php
$posts = Post::with([
    'comments' => fn($query) => $query->latest()->limit(10),
])->get();

// Or for specific relationships
$users = User::with([
    'posts' => fn($query) => $query->where('published', true)->limit(5),
])->get();
```

## Query Builder Best Practices

### Prefer Eloquent Over DB Facade

```php
// ❌ AVOID - Using DB facade
$users = DB::table('users')->where('active', true)->get();

// ✅ PREFER - Using Eloquent
$users = User::query()->where('active', true)->get();
```

### Using Query Scopes

Define reusable query logic in scopes:

```php
class Post extends Model
{
    public function scopePublished($query)
    {
        return $query->where('published', true);
    }
    
    public function scopeRecent($query, int $days = 7)
    {
        return $query->where('created_at', '>=', now()->subDays($days));
    }
}

// Usage
$posts = Post::published()->recent(14)->get();
```

## Migrations

### Modifying Columns - IMPORTANT

When modifying a column, the migration MUST include ALL attributes previously defined. Otherwise, they will be dropped and lost:

```php
// ❌ WRONG - Will lose 'nullable' attribute
Schema::table('users', function (Blueprint $table) {
    $table->string('email')->unique()->change();
});

// ✅ CORRECT - Preserves all attributes
Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable()->unique()->change();
});
```

### Best Practices for Migrations

```php
// Use descriptive migration names
php artisan make:migration add_published_at_to_posts_table

// Always add down() method
public function down(): void
{
    Schema::table('posts', function (Blueprint $table) {
        $table->dropColumn('published_at');
    });
}

// Use foreign key constraints
$table->foreignId('user_id')->constrained()->onDelete('cascade');

// Add indexes for frequently queried columns
$table->string('email')->unique();
$table->index('status');
```

## Model Casts

### Using casts() Method (Preferred in Laravel 12)

```php
class Post extends Model
{
    protected function casts(): array
    {
        return [
            'published_at' => 'datetime',
            'is_featured' => 'boolean',
            'metadata' => 'array',
            'settings' => 'json',
        ];
    }
}
```

### Custom Casts

```php
use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

class Json implements CastsAttributes
{
    public function get($model, string $key, $value, array $attributes)
    {
        return json_decode($value, true);
    }

    public function set($model, string $key, $value, array $attributes)
    {
        return json_encode($value);
    }
}

// In model
protected function casts(): array
{
    return [
        'options' => Json::class,
    ];
}
```

## Eloquent API Resources

For APIs, use Eloquent API Resources for consistent formatting:

```php
// app/Http/Resources/PostResource.php
namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
            'published_at' => $this->published_at?->toIso8601String(),
            'author' => new UserResource($this->whenLoaded('user')),
            'comments' => CommentResource::collection($this->whenLoaded('comments')),
        ];
    }
}

// In controller
public function show(Post $post): PostResource
{
    return new PostResource($post->load('user', 'comments'));
}
```

## Transactions

Use database transactions for operations that must succeed or fail together:

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () use ($user, $data) {
    $post = $user->posts()->create($data);
    
    $post->tags()->attach($data['tag_ids']);
    
    event(new PostCreated($post));
});

// Or with manual control
DB::beginTransaction();

try {
    // Database operations
    $post->save();
    $comment->save();
    
    DB::commit();
} catch (\Exception $e) {
    DB::rollBack();
    throw $e;
}
```

## Model Events

Use model events for side effects:

```php
class Post extends Model
{
    protected static function booted(): void
    {
        static::creating(function (Post $post) {
            $post->slug = Str::slug($post->title);
        });
        
        static::deleting(function (Post $post) {
            $post->comments()->delete();
        });
    }
}
```

## Best Practices Summary

1. **Always use return type hints** on relationship methods
2. **Prevent N+1 queries** by eager loading relationships
3. **Use `Model::query()`** instead of `DB::table()`
4. **Leverage Laravel 12's native limit** for eager loading
5. **Include ALL column attributes** when modifying migrations
6. **Use `casts()` method** for defining model casts
7. **Use API Resources** for consistent API responses
8. **Use transactions** for multi-step database operations
9. **Define query scopes** for reusable query logic
10. **Use model events** for side effects and hooks
