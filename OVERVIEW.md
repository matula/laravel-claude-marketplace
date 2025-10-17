# Laravel Development Bundle - Overview

## 📦 Bundle Contents

```
laravel-marketplace/
│
├── 📄 README.md                    # Full documentation
├── 📄 QUICKSTART.md               # Get started in minutes
├── 📄 LICENSE                     # MIT License
├── 📄 marketplace.json            # Marketplace metadata
│
├── 🔧 laravel-12/                 # Laravel Backend Development
│   ├── SKILL.md                   # Core Laravel 12 principles
│   └── references/
│       ├── authentication.md      # Auth & authorization
│       ├── database.md            # Eloquent & migrations
│       ├── structure.md           # Laravel 12 architecture
│       ├── validation.md          # Form validation
│       └── queues.md              # Background jobs
│
├── 🎨 tailwind-v4/                # UI Styling
│   ├── SKILL.md                   # Core Tailwind v4 principles
│   └── references/
│       ├── utilities.md           # All utility classes
│       ├── dark-mode.md           # Dark mode implementation
│       └── patterns.md            # UI component patterns
│
└── 🧪 pest-testing/               # Quality Assurance
    ├── SKILL.md                   # Core testing principles
    └── references/
        ├── core.md                # Pest syntax & API
        ├── laravel.md             # Laravel testing patterns
        └── browser.md             # Browser testing (Pest v4)
```

## 🎯 What Each Skill Does

### 🔧 Laravel 12 Skill
**Your backend development expert**

Helps with:
- ✅ Creating models, controllers, migrations
- ✅ Eloquent relationships and queries
- ✅ Authentication and authorization
- ✅ Form validation with Form Requests
- ✅ Queue and job management
- ✅ API development
- ✅ Following Laravel 12 best practices

**Example guidance:**
```php
// Skill ensures you use best practices
public function index(Request $request): Response
{
    $user = $request->user(); // ✅ Correct way
    
    $posts = Post::with('user', 'comments')->get(); // ✅ Prevents N+1
    
    return view('posts.index', compact('posts'));
}
```

---

### 🎨 Tailwind v4 Skill
**Your styling expert**

Helps with:
- ✅ Modern Tailwind v4 syntax
- ✅ Dark mode implementation
- ✅ Responsive design patterns
- ✅ Common UI components
- ✅ Proper spacing with gap utilities
- ✅ Color system with opacity

**Example guidance:**
```html
<!-- Skill ensures you use v4 syntax -->
<div class="bg-white dark:bg-gray-900 rounded-lg shadow-lg p-6">
  <!-- ✅ Correct v4 opacity syntax -->
  <h2 class="text-2xl font-bold text-gray-900/90 dark:text-white/90">
    Title
  </h2>
  
  <!-- ✅ Proper gap usage -->
  <div class="flex gap-4">
    <button class="bg-blue-600/100 hover:bg-blue-700 text-white px-4 py-2 rounded">
      Action
    </button>
  </div>
</div>
```

---

### 🧪 Pest Testing Skill
**Your quality assurance expert**

Helps with:
- ✅ Writing feature and unit tests
- ✅ Browser testing (Pest v4 new feature!)
- ✅ HTTP and API testing
- ✅ Using model factories
- ✅ Testing with datasets
- ✅ Mocking and faking
- ✅ Testing best practices

**Example guidance:**
```php
// Skill ensures you follow testing best practices
it('creates a post', function () {
    $user = User::factory()->create(); // ✅ Use factories
    
    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'My Post',
        'body' => 'Content',
    ]);
    
    $response->assertCreated(); // ✅ Specific assertion
    
    $this->assertDatabaseHas('posts', [
        'title' => 'My Post',
    ]);
});

// Browser testing (NEW in Pest v4!)
it('displays the form', function () {
    $page = visit('/posts/create');
    
    $page->fill('title', 'My Post')
        ->click('Create')
        ->assertNoJavascriptErrors(); // ✅ Check for errors
});
```

## 🔄 How They Work Together

### Example: Building a Blog Post Feature

**1. Backend (Laravel 12 skill)**
```bash
php artisan make:model Post -mfs
php artisan make:controller PostController --resource
php artisan make:request StorePostRequest
```

**2. Styling (Tailwind v4 skill)**
```html
<form class="bg-white dark:bg-gray-900 p-6 rounded-lg">
  <input class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 rounded">
  <button class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded">
    Create Post
  </button>
</form>
```

**3. Testing (Pest Testing skill)**
```php
it('creates a post', function () {
    $user = User::factory()->create();
    
    $response = $this->actingAs($user)->post('/posts', [
        'title' => 'My Post',
        'body' => 'Content',
    ]);
    
    $response->assertCreated();
});
```

## 📊 Skill Statistics

### Laravel 12
- **SKILL.md**: ~350 lines
- **References**: 5 detailed files
- **Topics covered**: 50+
- **Code examples**: 100+

### Tailwind v4
- **SKILL.md**: ~400 lines
- **References**: 3 detailed files
- **Topics covered**: 40+
- **Code examples**: 150+

### Pest Testing
- **SKILL.md**: ~450 lines
- **References**: 3 detailed files
- **Topics covered**: 60+
- **Code examples**: 200+

**Total**: ~1,200 lines of core guidance + ~2,500 lines of detailed references

## 🚀 Installation Options

### Option 1: Individual Skills
Install each skill separately for granular control:
- Upload `laravel-12.zip`
- Upload `tailwind-v4.zip`
- Upload `pest-testing.zip`

### Option 2: Complete Bundle
Install everything at once:
- Upload `laravel-marketplace.zip`

Both options give you the same skills - choose what works best for your workflow!

## 💡 Key Benefits

### 1. **Comprehensive Coverage**
Every aspect of Laravel development is covered from backend to testing.

### 2. **Best Practices Built-In**
Follow Laravel, Tailwind, and Pest conventions automatically.

### 3. **Progressive Learning**
- Quick reference in SKILL.md
- Deep dives in references/
- Load what you need, when you need it

### 4. **Time Saving**
- No more searching documentation
- Context-aware guidance
- Copy-paste ready examples

### 5. **Modern Stack**
- Laravel 12 (latest)
- Tailwind CSS v4 (latest)
- Pest v4 (latest with browser testing)

## 📈 What's Next?

1. **Install the skills**
2. **Read QUICKSTART.md** (5 min)
3. **Start coding** with expert guidance
4. **Build amazing Laravel apps** 🚀

---

**Questions?** Each skill has comprehensive documentation in SKILL.md and references/

**Ready to start?** Check out QUICKSTART.md for a guided walkthrough!
