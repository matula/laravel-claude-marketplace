# Pest Core Testing Reference

## Basic Test Structure

Pest tests use a simple, expressive syntax:

```php
<?php

use App\Models\User;

it('has a name', function () {
    $user = User::factory()->create(['name' => 'John Doe']);
    
    expect($user->name)->toBe('John Doe');
});

test('users can be created', function () {
    $user = User::factory()->create();
    
    expect($user)->toBeInstanceOf(User::class);
});
```

## Test Functions

### it() - BDD Style

```php
it('calculates the sum correctly', function () {
    $result = 2 + 2;
    
    expect($result)->toBe(4);
});

it('has users', function () {
    $users = User::all();
    
    expect($users)->not->toBeEmpty();
});
```

### test() - Traditional Style

```php
test('user can login', function () {
    $user = User::factory()->create();
    
    $response = $this->post('/login', [
        'email' => $user->email,
        'password' => 'password',
    ]);
    
    $response->assertRedirect('/dashboard');
});
```

## Expectations API

### Equality

```php
expect($value)->toBe(10);              // Strict equality (===)
expect($value)->toEqual(10);           // Loose equality (==)
expect($value)->not->toBe(5);          // Negation
```

### Type Checks

```php
expect($value)->toBeInt();
expect($value)->toBeFloat();
expect($value)->toBeString();
expect($value)->toBeBool();
expect($value)->toBeArray();
expect($value)->toBeObject();
expect($value)->toBeResource();
expect($value)->toBeScalar();
expect($value)->toBeCallable();
expect($value)->toBeIterable();
expect($value)->toBeNumeric();
```

### Instance Checks

```php
expect($user)->toBeInstanceOf(User::class);
expect($model)->toBeModel();
expect($collection)->toBeCollection();
```

### Null and Empty

```php
expect($value)->toBeNull();
expect($value)->not->toBeNull();
expect($value)->toBeEmpty();
expect($value)->not->toBeEmpty();
```

### Boolean

```php
expect($value)->toBeTrue();
expect($value)->toBeFalse();
expect($value)->toBeTruthy();    // Any truthy value
expect($value)->toBeFalsy();     // Any falsy value
```

### Numeric Comparisons

```php
expect($value)->toBeGreaterThan(5);
expect($value)->toBeGreaterThanOrEqual(10);
expect($value)->toBeLessThan(20);
expect($value)->toBeLessThanOrEqual(15);
expect($value)->toBeBetween(1, 100);
```

### String Assertions

```php
expect($string)->toContain('substring');
expect($string)->toStartWith('Hello');
expect($string)->toEndWith('World');
expect($string)->toMatch('/pattern/');
expect($string)->toBeJson();
expect($string)->toBeUuid();
```

### Array and Collection

```php
expect($array)->toHaveCount(5);
expect($array)->toHaveKey('name');
expect($array)->toHaveKeys(['name', 'email']);
expect($array)->toContain('value');
expect($array)->each->toBeString();  // All items are strings
expect($array)->sequence(
    fn($item) => $item->toBeString(),
    fn($item) => $item->toBeInt(),
);
```

### File System

```php
expect('path/to/file.txt')->toBeFile();
expect('path/to/directory')->toBeDirectory();
expect('path/to/file.txt')->toBeReadable();
expect('path/to/file.txt')->toBeWritable();
```

## Laravel-Specific Assertions

### HTTP Response Assertions

```php
$response = $this->get('/users');

$response->assertOk();                    // 200
$response->assertCreated();               // 201
$response->assertAccepted();              // 202
$response->assertNoContent();             // 204
$response->assertNotFound();              // 404
$response->assertForbidden();             // 403
$response->assertUnauthorized();          // 401
$response->assertUnprocessable();         // 422
$response->assertServerError();           // 500
$response->assertSuccessful();            // 2xx
$response->assertRedirect('/dashboard');
$response->assertRedirectToRoute('home');

// Content assertions
$response->assertSee('Welcome');
$response->assertSeeText('Welcome');
$response->assertDontSee('Error');
$response->assertJson(['name' => 'John']);
$response->assertJsonStructure(['data' => ['id', 'name']]);
$response->assertJsonPath('data.name', 'John');
$response->assertJsonCount(5, 'data');
```

### Database Assertions

```php
$this->assertDatabaseHas('users', [
    'email' => 'john@example.com',
]);

$this->assertDatabaseMissing('users', [
    'email' => 'deleted@example.com',
]);

$this->assertDatabaseCount('users', 10);

$this->assertDatabaseEmpty('posts');

$this->assertSoftDeleted('users', [
    'id' => $user->id,
]);
```

### Authentication Assertions

```php
$this->assertAuthenticated();
$this->assertGuest();
$this->assertAuthenticatedAs($user);
```

### Session Assertions

```php
$response->assertSessionHas('status');
$response->assertSessionHas('status', 'success');
$response->assertSessionMissing('error');
$response->assertSessionHasErrors(['email', 'password']);
$response->assertSessionHasNoErrors();
```

## Higher Order Expectations

Chain multiple expectations:

```php
expect($users)
    ->toBeCollection()
    ->toHaveCount(3)
    ->each->toBeInstanceOf(User::class);

expect($post)
    ->toBeInstanceOf(Post::class)
    ->title->toBe('My Post')
    ->published->toBeTrue();
```

## Using `and()` for Multiple Assertions

```php
expect($user->name)->toBe('John Doe')
    ->and($user->email)->toBe('john@example.com')
    ->and($user->age)->toBeGreaterThan(18);
```

## Test Lifecycle Hooks

### beforeEach / afterEach

```php
beforeEach(function () {
    $this->user = User::factory()->create();
});

afterEach(function () {
    // Cleanup
});

it('can access user', function () {
    expect($this->user)->toBeInstanceOf(User::class);
});
```

### beforeAll / afterAll

```php
beforeAll(function () {
    // Runs once before all tests in the file
});

afterAll(function () {
    // Runs once after all tests in the file
});
```

## Datasets

Use datasets to run the same test with different inputs:

```php
it('validates email addresses', function (string $email, bool $valid) {
    $validator = validator(['email' => $email], ['email' => 'email']);
    
    expect($validator->passes())->toBe($valid);
})->with([
    ['john@example.com', true],
    ['invalid-email', false],
    ['test@test.co', true],
    ['@example.com', false],
]);
```

### Named Datasets

```php
it('has valid emails', function (string $email) {
    expect($email)->toMatch('/^.+@.+\..+$/');
})->with([
    'gmail' => 'user@gmail.com',
    'yahoo' => 'user@yahoo.com',
    'outlook' => 'user@outlook.com',
]);
```

### Dataset Functions

```php
dataset('emails', function () {
    return [
        'john@example.com',
        'jane@example.com',
        'admin@example.com',
    ];
});

it('has valid format', function (string $email) {
    expect($email)->toContain('@');
})->with('emails');
```

### Combining Datasets

```php
it('validates inputs', function (string $email, string $password) {
    // Test with all combinations
})->with('emails', 'passwords');
```

## Skipping and Focusing Tests

### Skip Tests

```php
it('is pending', function () {
    //
})->skip();

it('skips on condition', function () {
    //
})->skip(! app()->isProduction(), 'Only in production');

it('skips on windows', function () {
    //
})->skipOnWindows();

it('skips on mac', function () {
    //
})->skipOnMac();

it('skips on linux', function () {
    //
})->skipOnLinux();
```

### Focus Tests

Run only specific tests (all other tests are skipped):

```php
it('runs only this test', function () {
    //
})->only();

// Or
fit('runs only this test', function () {
    //
});
```

## Test Organization

### Grouping Tests

```php
describe('User Management', function () {
    it('can create users', function () {
        //
    });
    
    it('can update users', function () {
        //
    });
    
    it('can delete users', function () {
        //
    });
});
```

### Test Tags

```php
it('is an integration test', function () {
    //
})->group('integration');

it('is slow', function () {
    //
})->group('slow', 'integration');

// Run with: php artisan test --group=integration
```

## Mocking

### Using Pest's mock Function

```php
use function Pest\Laravel\mock;

it('mocks a service', function () {
    $mock = mock(PaymentService::class);
    
    $mock->shouldReceive('charge')
        ->once()
        ->with(100)
        ->andReturn(true);
    
    $result = app(PaymentService::class)->charge(100);
    
    expect($result)->toBeTrue();
});
```

### Partial Mocking

```php
it('partially mocks a class', function () {
    $mock = mock(User::class)->makePartial();
    
    $mock->shouldReceive('getName')
        ->once()
        ->andReturn('Mocked Name');
    
    expect($mock->getName())->toBe('Mocked Name');
});
```

## Best Practices

1. **Use descriptive test names** - Tests should read like specifications
2. **One assertion per test** - Keep tests focused (though multiple expects are okay)
3. **Use factories** - Always use model factories instead of manual model creation
4. **Use datasets** - Avoid duplicate test code with datasets
5. **Test all paths** - Happy path, failure path, and edge cases
6. **Keep tests isolated** - Each test should be independent
7. **Use expect over assertions** - Prefer `expect()` for better readability
8. **Import mock function** - Always `use function Pest\Laravel\mock;` before using it
9. **Use higher order expectations** - Chain assertions for cleaner tests
10. **Organize with describe** - Group related tests together
