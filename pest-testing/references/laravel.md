# Pest Laravel Testing Reference

## Feature vs Unit Tests

### Feature Tests
- Test complete features or user workflows
- Interact with HTTP layer (controllers, routes)
- May touch database
- Located in `tests/Feature/`

### Unit Tests
- Test individual classes or methods
- Isolated from framework components
- Fast and focused
- Located in `tests/Unit/`

## Creating Tests

```bash
# Feature test
php artisan make:test UserTest --pest

# Unit test
php artisan make:test CalculatorTest --pest --unit
```

## HTTP Testing

### GET Requests

```php
it('displays the homepage', function () {
    $response = $this->get('/');
    
    $response->assertOk()
        ->assertSee('Welcome');
});

it('shows user profile', function () {
    $user = User::factory()->create();
    
    $response = $this->get("/users/{$user->id}");
    
    $response->assertOk()
        ->assertSee($user->name);
});
```

### POST Requests

```php
it('creates a post', function () {
    $user = User::factory()->create();
    
    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'My Post',
        'body' => 'Post content',
    ]);
    
    $response->assertCreated()
        ->assertJson(['title' => 'My Post']);
    
    $this->assertDatabaseHas('posts', [
        'title' => 'My Post',
        'user_id' => $user->id,
    ]);
});
```

### PUT/PATCH Requests

```php
it('updates a post', function () {
    $user = User::factory()->create();
    $post = Post::factory()->create(['user_id' => $user->id]);
    
    $response = $this->actingAs($user)->put("/posts/{$post->id}", [
        'title' => 'Updated Title',
    ]);
    
    $response->assertOk();
    
    expect($post->fresh()->title)->toBe('Updated Title');
});
```

### DELETE Requests

```php
it('deletes a post', function () {
    $user = User::factory()->create();
    $post = Post::factory()->create(['user_id' => $user->id]);
    
    $response = $this->actingAs($user)->delete("/posts/{$post->id}");
    
    $response->assertNoContent();
    
    $this->assertDatabaseMissing('posts', [
        'id' => $post->id,
    ]);
});
```

## Authentication Testing

### Acting as User

```php
it('requires authentication', function () {
    $response = $this->get('/dashboard');
    
    $response->assertRedirect('/login');
});

it('allows authenticated access', function () {
    $user = User::factory()->create();
    
    $response = $this->actingAs($user)->get('/dashboard');
    
    $response->assertOk();
});
```

### Login/Logout

```php
it('can login', function () {
    $user = User::factory()->create([
        'password' => bcrypt('password'),
    ]);
    
    $response = $this->post('/login', [
        'email' => $user->email,
        'password' => 'password',
    ]);
    
    $response->assertRedirect('/dashboard');
    $this->assertAuthenticated();
    $this->assertAuthenticatedAs($user);
});

it('can logout', function () {
    $user = User::factory()->create();
    
    $response = $this->actingAs($user)->post('/logout');
    
    $response->assertRedirect('/');
    $this->assertGuest();
});
```

### Registration

```php
it('can register', function () {
    $response = $this->post('/register', [
        'name' => 'John Doe',
        'email' => 'john@example.com',
        'password' => 'password',
        'password_confirmation' => 'password',
    ]);
    
    $response->assertRedirect('/dashboard');
    
    $this->assertDatabaseHas('users', [
        'email' => 'john@example.com',
    ]);
    
    $this->assertAuthenticated();
});
```

## Authorization Testing

```php
it('prevents unauthorized access', function () {
    $user = User::factory()->create();
    $post = Post::factory()->create(); // Different author
    
    $response = $this->actingAs($user)->delete("/posts/{$post->id}");
    
    $response->assertForbidden();
});

it('allows authorized access', function () {
    $user = User::factory()->create();
    $post = Post::factory()->create(['user_id' => $user->id]);
    
    $response = $this->actingAs($user)->delete("/posts/{$post->id}");
    
    $response->assertNoContent();
});
```

## Validation Testing

```php
it('validates required fields', function () {
    $user = User::factory()->create();
    
    $response = $this->actingAs($user)->post('/posts', []);
    
    $response->assertUnprocessable()
        ->assertJsonValidationErrors(['title', 'body']);
});

it('validates email format', function () {
    $response = $this->post('/register', [
        'name' => 'John Doe',
        'email' => 'invalid-email',
        'password' => 'password',
    ]);
    
    $response->assertUnprocessable()
        ->assertJsonValidationErrors(['email']);
});

// Using datasets for validation
it('validates email addresses', function (string $email, bool $shouldPass) {
    $data = [
        'name' => 'John Doe',
        'email' => $email,
        'password' => 'password',
        'password_confirmation' => 'password',
    ];
    
    $response = $this->post('/register', $data);
    
    if ($shouldPass) {
        $response->assertRedirect();
    } else {
        $response->assertUnprocessable()
            ->assertJsonValidationErrors(['email']);
    }
})->with([
    ['valid@example.com', true],
    ['invalid', false],
    ['@example.com', false],
    ['user@domain.co', true],
]);
```

## JSON API Testing

```php
it('returns JSON list', function () {
    $users = User::factory()->count(3)->create();
    
    $response = $this->getJson('/api/users');
    
    $response->assertOk()
        ->assertJsonCount(3, 'data')
        ->assertJsonStructure([
            'data' => [
                '*' => ['id', 'name', 'email'],
            ],
        ]);
});

it('returns single resource', function () {
    $user = User::factory()->create([
        'name' => 'John Doe',
    ]);
    
    $response = $this->getJson("/api/users/{$user->id}");
    
    $response->assertOk()
        ->assertJson([
            'data' => [
                'id' => $user->id,
                'name' => 'John Doe',
            ],
        ])
        ->assertJsonPath('data.name', 'John Doe');
});
```

## Database Testing

### Using RefreshDatabase

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

it('cleans database between tests', function () {
    User::factory()->create();
    
    expect(User::count())->toBe(1);
});
```

### Using DatabaseTransactions

```php
use Illuminate\Foundation\Testing\DatabaseTransactions;

uses(DatabaseTransactions::class);

it('rolls back after test', function () {
    User::factory()->create();
    
    // Rolled back after test
});
```

### Database Assertions

```php
it('creates record', function () {
    $user = User::factory()->create([
        'email' => 'john@example.com',
    ]);
    
    $this->assertDatabaseHas('users', [
        'email' => 'john@example.com',
    ]);
});

it('deletes record', function () {
    $user = User::factory()->create();
    
    $user->delete();
    
    $this->assertDatabaseMissing('users', [
        'id' => $user->id,
    ]);
});

it('counts records', function () {
    User::factory()->count(5)->create();
    
    $this->assertDatabaseCount('users', 5);
});

it('soft deletes', function () {
    $user = User::factory()->create();
    
    $user->delete();
    
    $this->assertSoftDeleted('users', [
        'id' => $user->id,
    ]);
});
```

## Model Testing

```php
it('has fillable attributes', function () {
    $user = new User([
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ]);
    
    expect($user->name)->toBe('John Doe')
        ->and($user->email)->toBe('john@example.com');
});

it('hides password', function () {
    $user = User::factory()->create();
    
    expect($user->toArray())->not->toHaveKey('password');
});

it('casts attributes', function () {
    $post = Post::factory()->create([
        'published_at' => '2024-01-01',
    ]);
    
    expect($post->published_at)->toBeInstanceOf(Carbon::class);
});

it('has relationships', function () {
    $user = User::factory()
        ->has(Post::factory()->count(3))
        ->create();
    
    expect($user->posts)->toHaveCount(3)
        ->each->toBeInstanceOf(Post::class);
});
```

## Event Testing

```php
use Illuminate\Support\Facades\Event;

it('dispatches event', function () {
    Event::fake();
    
    $user = User::factory()->create();
    
    Event::assertDispatched(UserCreated::class, function ($event) use ($user) {
        return $event->user->id === $user->id;
    });
});

it('does not dispatch event', function () {
    Event::fake();
    
    // Some action
    
    Event::assertNotDispatched(UserDeleted::class);
});
```

## Job Testing

```php
use Illuminate\Support\Facades\Queue;

it('dispatches job', function () {
    Queue::fake();
    
    dispatch(new ProcessPodcast($podcast));
    
    Queue::assertPushed(ProcessPodcast::class, function ($job) use ($podcast) {
        return $job->podcast->id === $podcast->id;
    });
});

it('dispatches job to queue', function () {
    Queue::fake();
    
    dispatch(new ProcessPodcast($podcast));
    
    Queue::assertPushedOn('podcasts', ProcessPodcast::class);
});
```

## Mail Testing

```php
use Illuminate\Support\Facades\Mail;

it('sends email', function () {
    Mail::fake();
    
    $user = User::factory()->create();
    
    Mail::to($user)->send(new WelcomeEmail($user));
    
    Mail::assertSent(WelcomeEmail::class, function ($mail) use ($user) {
        return $mail->hasTo($user->email);
    });
});

it('queues email', function () {
    Mail::fake();
    
    $user = User::factory()->create();
    
    Mail::to($user)->queue(new WelcomeEmail($user));
    
    Mail::assertQueued(WelcomeEmail::class);
});
```

## Notification Testing

```php
use Illuminate\Support\Facades\Notification;

it('sends notification', function () {
    Notification::fake();
    
    $user = User::factory()->create();
    
    $user->notify(new AccountVerified());
    
    Notification::assertSentTo($user, AccountVerified::class);
});

it('sends to multiple users', function () {
    Notification::fake();
    
    $users = User::factory()->count(3)->create();
    
    Notification::send($users, new NewFeature());
    
    Notification::assertSentTo($users, NewFeature::class);
});
```

## Storage Testing

```php
use Illuminate\Support\Facades\Storage;

it('stores file', function () {
    Storage::fake('public');
    
    $file = UploadedFile::fake()->image('avatar.jpg');
    
    $response = $this->post('/avatar', [
        'avatar' => $file,
    ]);
    
    Storage::disk('public')->assertExists('avatars/' . $file->hashName());
});

it('deletes file', function () {
    Storage::fake('public');
    
    Storage::disk('public')->put('test.txt', 'content');
    
    Storage::disk('public')->delete('test.txt');
    
    Storage::disk('public')->assertMissing('test.txt');
});
```

## Session Testing

```php
it('stores data in session', function () {
    $response = $this->withSession(['key' => 'value'])
        ->get('/dashboard');
    
    $response->assertSessionHas('key', 'value');
});

it('flashes data to session', function () {
    $response = $this->post('/posts', [
        'title' => 'My Post',
        'body' => 'Content',
    ]);
    
    $response->assertSessionHas('status', 'Post created!');
});
```

## Testing Commands

```php
it('runs artisan command', function () {
    $this->artisan('email:send')
        ->expectsOutput('Emails sent!')
        ->assertExitCode(0);
});

it('accepts command arguments', function () {
    $this->artisan('user:create', ['email' => 'john@example.com'])
        ->assertExitCode(0);
    
    $this->assertDatabaseHas('users', [
        'email' => 'john@example.com',
    ]);
});
```

## Best Practices

1. **Most tests should be feature tests** - Test the full workflow
2. **Use factories** - Always use model factories for test data
3. **One concept per test** - Keep tests focused on single behavior
4. **Test all paths** - Happy path, error path, edge cases
5. **Use descriptive test names** - Should read like specifications
6. **Use datasets** - Avoid duplicate test code
7. **Fake external services** - Use Laravel's fakes (Mail, Queue, etc.)
8. **Use RefreshDatabase** - Keep database clean between tests
9. **Test validation** - Verify both valid and invalid inputs
10. **Test authorization** - Verify permissions work correctly
11. **Keep tests isolated** - Each test should be independent
12. **Use proper assertions** - Prefer specific assertions over generic ones
13. **Test edge cases** - Empty collections, null values, boundaries
14. **Group related tests** - Use describe() for organization
15. **Run minimal tests** - Use filters when developing to run only relevant tests
