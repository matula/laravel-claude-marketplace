# Pest v4 Browser Testing Reference

## Overview

Pest v4 introduces powerful browser testing capabilities that allow you to test your application in real browsers (Chrome, Firefox, Safari) with full JavaScript support.

## Basic Browser Test

```php
<?php

use App\Models\User;
use function Pest\Laravel\visit;

it('displays the homepage', function () {
    $page = visit('/');
    
    $page->assertSee('Welcome');
});
```

## Starting Browser Tests

Browser tests should be in the `tests/Browser/` directory:

```bash
php artisan test tests/Browser/
```

## Navigation

### Visiting Pages

```php
it('can navigate', function () {
    $page = visit('/home');
    
    $page->assertSee('Home Page')
        ->click('About')
        ->assertSee('About Us');
});
```

### Direct URL Navigation

```php
it('navigates to profile', function () {
    $user = User::factory()->create();
    
    $page = visit("/users/{$user->id}");
    
    $page->assertSee($user->name);
});
```

## Interacting with Elements

### Clicking

```php
it('clicks buttons and links', function () {
    $page = visit('/dashboard');
    
    $page->click('Settings')
        ->click('button[type="submit"]')
        ->click('#save-button');
});
```

### Typing Text

```php
it('fills in form', function () {
    $page = visit('/register');
    
    $page->fill('name', 'John Doe')
        ->fill('email', 'john@example.com')
        ->fill('password', 'secret123')
        ->click('Register');
});
```

### Selecting Options

```php
it('selects from dropdown', function () {
    $page = visit('/settings');
    
    $page->select('country', 'United States')
        ->select('timezone', 'America/New_York');
});
```

### Checkboxes and Radio Buttons

```php
it('checks options', function () {
    $page = visit('/survey');
    
    $page->check('terms')
        ->check('newsletter')
        ->radio('gender', 'male');
});
```

### File Uploads

```php
it('uploads a file', function () {
    $page = visit('/upload');
    
    $page->attach('avatar', storage_path('test-files/avatar.jpg'))
        ->click('Upload');
    
    $page->assertSee('File uploaded successfully');
});
```

### Drag and Drop

```php
it('drags and drops', function () {
    $page = visit('/kanban');
    
    $page->drag('.task-item', '.completed-column')
        ->assertSee('Task moved');
});
```

## Assertions

### Content Assertions

```php
it('asserts content', function () {
    $page = visit('/');
    
    $page->assertSee('Welcome')
        ->assertDontSee('Error')
        ->assertSeeIn('header', 'Logo')
        ->assertSeeLink('About Us');
});
```

### Element Assertions

```php
it('asserts elements exist', function () {
    $page = visit('/profile');
    
    $page->assertPresent('.user-avatar')
        ->assertMissing('.error-message')
        ->assertVisible('#welcome-message')
        ->assertHidden('#debug-panel');
});
```

### URL Assertions

```php
it('asserts URL', function () {
    $page = visit('/login');
    
    $page->fill('email', 'user@example.com')
        ->fill('password', 'password')
        ->click('Login')
        ->assertPath('/dashboard')
        ->assertUrl(url('/dashboard'))
        ->assertRouteIs('dashboard');
});
```

### Input Value Assertions

```php
it('asserts input values', function () {
    $page = visit('/profile/edit');
    
    $page->assertInputValue('name', 'John Doe')
        ->assertInputValue('email', 'john@example.com')
        ->assertChecked('notifications')
        ->assertNotChecked('marketing');
});
```

### JavaScript Error Assertions

```php
it('has no JavaScript errors', function () {
    $page = visit('/');
    
    $page->assertNoJavascriptErrors();
});
```

### Console Log Assertions

```php
it('has no console logs', function () {
    $page = visit('/');
    
    $page->assertNoConsoleLogs();
});
```

## Waiting

### Wait for Elements

```php
it('waits for dynamic content', function () {
    $page = visit('/dashboard');
    
    $page->waitFor('.loading-indicator')
        ->waitUntilMissing('.loading-indicator')
        ->assertSee('Data loaded');
});
```

### Wait for Text

```php
it('waits for text', function () {
    $page = visit('/results');
    
    $page->waitForText('Results loaded')
        ->assertSee('100 items');
});
```

### Custom Wait Conditions

```php
it('waits for condition', function () {
    $page = visit('/live-data');
    
    $page->waitUsing(5, 100, function ($page) {
        return count($page->elements('.data-row')) > 0;
    });
});
```

## Testing with Authentication

```php
use function Pest\Laravel\actingAs;

it('accesses authenticated page', function () {
    $user = User::factory()->create();
    
    actingAs($user);
    
    $page = visit('/dashboard');
    
    $page->assertSee('Welcome, ' . $user->name);
});
```

## Laravel Features in Browser Tests

### Using Model Factories

```php
it('displays user profile', function () {
    $user = User::factory()->create([
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ]);
    
    $page = visit("/users/{$user->id}");
    
    $page->assertSee('John Doe')
        ->assertSee('john@example.com');
});
```

### Database Assertions

```php
it('creates a post', function () {
    $user = User::factory()->create();
    
    actingAs($user);
    
    $page = visit('/posts/create');
    
    $page->fill('title', 'My Post')
        ->fill('body', 'Post content')
        ->click('Publish');
    
    $this->assertDatabaseHas('posts', [
        'title' => 'My Post',
        'user_id' => $user->id,
    ]);
});
```

### Event Faking

```php
use Illuminate\Support\Facades\Event;

it('dispatches event on action', function () {
    Event::fake();
    
    $page = visit('/dashboard');
    
    $page->click('Delete Account');
    
    Event::assertDispatched(AccountDeleted::class);
});
```

### Notification Faking

```php
use Illuminate\Support\Facades\Notification;

it('sends notification', function () {
    Notification::fake();
    
    $user = User::factory()->create();
    
    $page = visit('/notifications/test');
    
    $page->click('Send Welcome Email');
    
    Notification::assertSentTo($user, WelcomeNotification::class);
});
```

## Multiple Browsers

Test with different browsers:

```php
it('works on Chrome', function () {
    $page = visit('/', browser: 'chrome');
    
    $page->assertSee('Welcome');
});

it('works on Firefox', function () {
    $page = visit('/', browser: 'firefox');
    
    $page->assertSee('Welcome');
});

it('works on Safari', function () {
    $page = visit('/', browser: 'safari');
    
    $page->assertSee('Welcome');
});
```

## Viewport and Device Testing

### Custom Viewport

```php
it('works on tablet', function () {
    $page = visit('/', viewport: [768, 1024]);
    
    $page->assertSee('Welcome');
});
```

### Device Emulation

```php
it('works on iPhone', function () {
    $page = visit('/', device: 'iPhone 14 Pro');
    
    $page->assertSee('Welcome');
});

it('works on iPad', function () {
    $page = visit('/', device: 'iPad Pro');
    
    $page->assertSee('Welcome');
});
```

## Color Schemes

Test light and dark modes:

```php
it('works in light mode', function () {
    $page = visit('/', colorScheme: 'light');
    
    $page->assertSee('Welcome');
});

it('works in dark mode', function () {
    $page = visit('/', colorScheme: 'dark');
    
    $page->assertSee('Welcome')
        ->assertNoJavascriptErrors();
});
```

## Screenshots

### Taking Screenshots

```php
it('takes screenshot', function () {
    $page = visit('/');
    
    $page->screenshot('homepage');
    
    // Screenshot saved to tests/Browser/screenshots/homepage.png
});
```

### Screenshots on Failure

```php
it('takes screenshot on failure', function () {
    $page = visit('/');
    
    $page->assertSee('This will fail'); // Screenshot auto-saved on failure
});
```

## Pausing Execution

Pause for debugging:

```php
it('pauses for debugging', function () {
    $page = visit('/');
    
    $page->pause(); // Browser stays open, execution pauses
    
    $page->click('Continue'); // Resume after clicking
});
```

## Smoke Testing

Test multiple pages quickly:

```php
it('has no errors on key pages', function () {
    $pages = visit(['/', '/about', '/contact', '/pricing']);
    
    $pages->assertNoJavascriptErrors()
        ->assertNoConsoleLogs();
});
```

## Touch Gestures

```php
it('handles touch gestures', function () {
    $page = visit('/gallery', device: 'iPhone 14 Pro');
    
    $page->tap('.image-1')
        ->swipeLeft('.gallery')
        ->swipeRight('.gallery')
        ->pinch('.image', scale: 2.0);
});
```

## Scrolling

```php
it('scrolls the page', function () {
    $page = visit('/long-page');
    
    $page->scrollTo('.footer')
        ->assertSee('Footer content');
});

it('scrolls element into view', function () {
    $page = visit('/dashboard');
    
    $page->scrollToElement('.hidden-section')
        ->assertVisible('.hidden-section');
});
```

## Complex Interactions

### Multi-step Forms

```php
it('completes multi-step form', function () {
    $page = visit('/register');
    
    // Step 1
    $page->fill('name', 'John Doe')
        ->fill('email', 'john@example.com')
        ->click('Next');
    
    // Step 2
    $page->waitForText('Profile Information')
        ->fill('bio', 'Developer')
        ->select('country', 'United States')
        ->click('Next');
    
    // Step 3
    $page->waitForText('Confirmation')
        ->check('terms')
        ->click('Complete Registration');
    
    $page->assertRouteIs('dashboard')
        ->assertSee('Welcome, John Doe');
});
```

### Testing Modal Dialogs

```php
it('interacts with modal', function () {
    $page = visit('/dashboard');
    
    $page->click('Delete Account')
        ->waitFor('.modal')
        ->assertSee('Are you sure?')
        ->click('.modal button.confirm');
    
    $page->waitUntilMissing('.modal')
        ->assertSee('Account deleted');
});
```

## Using RefreshDatabase

Clean database state between tests:

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

it('creates a user', function () {
    $page = visit('/register');
    
    $page->fill('name', 'John Doe')
        ->fill('email', 'john@example.com')
        ->fill('password', 'password')
        ->fill('password_confirmation', 'password')
        ->click('Register');
    
    $this->assertDatabaseHas('users', [
        'email' => 'john@example.com',
    ]);
});
```

## Best Practices

1. **Organize browser tests** - Keep them in `tests/Browser/` directory
2. **Use descriptive names** - Test names should explain what's being tested
3. **Check for JavaScript errors** - Always use `assertNoJavascriptErrors()`
4. **Test key user flows** - Focus on critical paths through the application
5. **Use factories** - Leverage model factories for test data
6. **Test different browsers** - Verify cross-browser compatibility
7. **Test responsive design** - Use device emulation for mobile testing
8. **Test both color schemes** - Verify light and dark mode work correctly
9. **Use waits appropriately** - Wait for dynamic content to load
10. **Take screenshots** - Helpful for debugging failures
11. **Keep tests isolated** - Each test should be independent
12. **Use smoke testing** - Quickly verify multiple pages don't have errors
13. **Clean database state** - Use `RefreshDatabase` when needed
14. **Test with real data** - Use factories to create realistic test scenarios
15. **Pause for debugging** - Use `pause()` when developing tests
