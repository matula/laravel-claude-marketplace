# Laravel Development Bundle

A comprehensive collection of Claude Code skills for modern Laravel development, covering backend architecture, UI styling, and testing best practices.

## 📦 Included Skills

### 1. **Laravel 12** - Backend Development
Expert guidance for building Laravel 12 applications with:
- Modern Laravel 12 streamlined architecture
- Eloquent ORM patterns and relationships
- Authentication and authorization best practices
- Form validation with Form Request classes
- Queue and job management
- Database migrations and API development
- Configuration and environment management

**Use when**: Creating models, controllers, APIs, implementing authentication, writing database queries, or any Laravel backend task.

### 2. **Tailwind CSS v4** - UI Styling
Expert guidance for building beautiful interfaces with:
- Modern v4 utility classes and syntax
- Dark mode implementation patterns
- Responsive design with mobile-first approach
- Common component patterns (cards, buttons, forms, navigation)
- Color system and opacity variants
- Interactive states and transitions

**Use when**: Styling HTML, creating UI components, implementing dark mode, or working with Tailwind CSS v4.

### 3. **Pest Testing** - Quality Assurance
Comprehensive testing guidance with Pest v4:
- Feature and unit testing patterns
- Browser testing with Pest v4 (new!)
- HTTP testing and authentication
- Database testing and factories
- Mocking and faking Laravel services
- Datasets for efficient test organization
- Testing best practices

**Use when**: Writing tests, implementing TDD, testing APIs, creating browser automation, or ensuring code quality.

## 🚀 Quick Start

1. **Install the skills** in Claude Code. Install the marketplace with `\plugin`, and use 
2. **Start coding** - The skills automatically activate when working on relevant tasks
3. **Get expert guidance** - Claude provides context-aware help based on what you're doing

## 💡 Key Features

### Integrated Knowledge
All three skills work together seamlessly:
- Build Laravel backend with **Laravel 12** skill
- Style your UI with **Tailwind v4** skill  
- Test everything with **Pest Testing** skill

### Best Practices Built-In
- ✅ Laravel 12 streamlined structure
- ✅ Proper authentication patterns (`$request->user()`)
- ✅ Eloquent best practices (N+1 prevention, type hints)
- ✅ Modern Tailwind v4 syntax (opacity variants, gap spacing)
- ✅ Comprehensive testing patterns (feature, unit, browser)

### Progressive Disclosure
- **SKILL.md** - Core principles and quick reference
- **references/** - Detailed guides for deep dives
- Load only what you need, when you need it

## 📚 What's Covered

### Laravel 12 Coverage
- ✅ Artisan commands and file creation
- ✅ Streamlined Laravel 12 architecture
- ✅ Authentication and authorization
- ✅ Eloquent models and relationships
- ✅ Form validation with Form Requests
- ✅ Queue and job management
- ✅ API development with resources
- ✅ Configuration management

### Tailwind v4 Coverage
- ✅ v4 syntax updates (import, deprecated utilities)
- ✅ Utility classes (layout, spacing, typography)
- ✅ Dark mode implementation
- ✅ Responsive design patterns
- ✅ Common UI components
- ✅ Interactive states
- ✅ Color system with opacity

### Pest Testing Coverage
- ✅ Core Pest syntax and expectations
- ✅ Feature and unit tests
- ✅ Browser testing (Pest v4)
- ✅ HTTP and API testing
- ✅ Authentication and authorization tests
- ✅ Database testing
- ✅ Mocking and faking
- ✅ Datasets and test organization

## 🎯 Use Cases

### Building a New Feature
1. **Laravel 12 skill** helps create models, controllers, routes
2. **Tailwind v4 skill** helps style the UI components
3. **Pest Testing skill** helps write comprehensive tests

### Implementing Authentication
1. **Laravel 12 skill** guides proper authentication patterns
2. **Tailwind v4 skill** styles login/register forms
3. **Pest Testing skill** tests authentication flows

### Creating an API
1. **Laravel 12 skill** guides API resources and routes
2. **Pest Testing skill** tests all API endpoints

### Adding Dark Mode
1. **Tailwind v4 skill** provides dark mode patterns for all components
2. **Pest Testing skill** helps test both color schemes with browser tests

## 📖 Documentation Structure

Each skill includes:

```
skill-name/
├── SKILL.md                    # Core principles and quick reference
└── references/                 # Detailed guides
    ├── topic1.md              # In-depth documentation
    ├── topic2.md              # More detailed info
    └── topic3.md              # Additional references
```

### Laravel 12 References
- `structure.md` - Laravel 12 architecture and conventions
- `authentication.md` - Auth patterns and best practices
- `database.md` - Eloquent, migrations, queries
- `validation.md` - Form Requests and validation
- `queues.md` - Jobs, queues, and workers

### Tailwind v4 References
- `utilities.md` - Complete utility class reference
- `dark-mode.md` - Dark mode implementation
- `patterns.md` - Common UI component patterns

### Pest Testing References
- `core.md` - Pest syntax and expectations API
- `laravel.md` - Laravel-specific testing patterns
- `browser.md` - Browser testing with Pest v4

## 🔧 Requirements

- PHP 8.4+
- Laravel 12+
- Tailwind CSS 4+
- Pest 4+

## 📝 Best Practices Highlights

### Laravel 12
```php
// ✅ Always use $request->user()
public function index(Request $request): Response
{
    $user = $request->user();
}

// ✅ Use Eloquent over DB facade
$users = User::query()->where('active', true)->get();

// ✅ Always create Form Request classes
php artisan make:request StorePostRequest
```

### Tailwind v4
```html
<!-- ✅ Use v4 opacity syntax -->
<div class="bg-blue-500/50 text-gray-900/80">

<!-- ✅ Use gap for spacing -->
<div class="flex gap-4">

<!-- ✅ Always support dark mode -->
<div class="bg-white dark:bg-gray-900">
```

### Pest Testing
```php
// ✅ Use factories
$user = User::factory()->create();

// ✅ Use datasets
it('validates emails', function ($email, $valid) {
    // ...
})->with([...]);

// ✅ Use specific assertions
$response->assertOk(); // Not assertStatus(200)
```

## 📄 License

MIT License - feel free to use and modify these skills for your projects.

## 🙏 Acknowledgments

These skills are designed to work with Claude Code and follow Laravel, Tailwind CSS, and Pest best practices and official documentation.
