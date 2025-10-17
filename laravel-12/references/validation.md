# Laravel 12 Validation & Forms Reference

## Form Request Validation

### Always Create Form Request Classes

Never use inline validation in controllers. Always create Form Request classes:

```bash
php artisan make:request StorePostRequest
php artisan make:request UpdatePostRequest
```

### Form Request Structure

```php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // Or add authorization logic
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'body' => ['required', 'string'],
            'published_at' => ['nullable', 'date', 'after:now'],
            'tags' => ['array'],
            'tags.*' => ['integer', 'exists:tags,id'],
        ];
    }
    
    public function messages(): array
    {
        return [
            'title.required' => 'A post title is required.',
            'title.max' => 'The post title cannot exceed 255 characters.',
            'body.required' => 'The post body is required.',
        ];
    }
    
    public function attributes(): array
    {
        return [
            'published_at' => 'publication date',
        ];
    }
}
```

### Using Form Requests in Controllers

```php
public function store(StorePostRequest $request): RedirectResponse
{
    $post = Post::create($request->validated());
    
    return redirect()->route('posts.show', $post);
}
```

## Validation Rules

### Array-Based vs String-Based Rules

Check your project's conventions. Both formats are valid:

```php
// Array-based (recommended)
public function rules(): array
{
    return [
        'email' => ['required', 'email', 'unique:users'],
        'password' => ['required', 'min:8', 'confirmed'],
    ];
}

// String-based
public function rules(): array
{
    return [
        'email' => 'required|email|unique:users',
        'password' => 'required|min:8|confirmed',
    ];
}
```

### Common Validation Rules

```php
// Required
'field' => ['required']

// String with length
'name' => ['required', 'string', 'min:3', 'max:255']

// Email
'email' => ['required', 'email']

// Unique (except current record on update)
'email' => ['required', 'email', 'unique:users,email,'.$this->user->id]

// Exists in database
'user_id' => ['required', 'exists:users,id']

// Confirmed (password_confirmation field must match)
'password' => ['required', 'min:8', 'confirmed']

// Arrays
'tags' => ['array', 'min:1', 'max:5']
'tags.*' => ['integer', 'exists:tags,id']

// Conditional validation
'reason' => ['required_if:status,rejected']
'other_reason' => ['required_unless:reason,null']

// Dates
'start_date' => ['required', 'date', 'after:today']
'end_date' => ['required', 'date', 'after:start_date']

// Files
'avatar' => ['nullable', 'image', 'max:2048'] // 2MB
'document' => ['required', 'file', 'mimes:pdf,doc,docx', 'max:10240'] // 10MB

// URLs
'website' => ['nullable', 'url']

// Boolean
'is_active' => ['boolean']

// Numeric
'age' => ['integer', 'min:18', 'max:100']
'price' => ['numeric', 'min:0']

// Regex
'phone' => ['required', 'regex:/^[0-9]{10}$/']
```

### Custom Validation Rules

```php
use Illuminate\Validation\Rule;

public function rules(): array
{
    return [
        // Enum validation (PHP 8.1+)
        'status' => ['required', Rule::enum(PostStatus::class)],
        
        // Custom unique rule
        'email' => [
            'required',
            'email',
            Rule::unique('users')->ignore($this->user),
        ],
        
        // In validation
        'role' => [
            'required',
            Rule::in(['admin', 'editor', 'viewer']),
        ],
    ];
}
```

### Complex Validation

```php
public function rules(): array
{
    return [
        'items' => ['required', 'array', 'min:1'],
        'items.*.name' => ['required', 'string', 'max:255'],
        'items.*.quantity' => ['required', 'integer', 'min:1'],
        'items.*.price' => ['required', 'numeric', 'min:0'],
    ];
}

// Validates:
// [
//     'items' => [
//         ['name' => 'Item 1', 'quantity' => 2, 'price' => 10.99],
//         ['name' => 'Item 2', 'quantity' => 1, 'price' => 5.99],
//     ]
// ]
```

## Authorization in Form Requests

Combine validation with authorization:

```php
public function authorize(): bool
{
    $post = $this->route('post');
    
    return $post && $this->user()->can('update', $post);
}

// Or use Gate
public function authorize(): bool
{
    return Gate::allows('admin');
}
```

## Custom Validation Messages

### Method 1: In messages() Method

```php
public function messages(): array
{
    return [
        'title.required' => 'Please provide a title for your post.',
        'title.max' => 'The title is too long.',
        'tags.*.exists' => 'One or more tags are invalid.',
    ];
}
```

### Method 2: In Language Files

```php
// resources/lang/en/validation.php
return [
    'custom' => [
        'email' => [
            'required' => 'We need your email address.',
            'unique' => 'This email is already registered.',
        ],
    ],
];
```

## Validated Data

Use `validated()` method to get only validated data:

```php
public function store(StorePostRequest $request): RedirectResponse
{
    // ✅ GOOD - Only validated data
    $validated = $request->validated();
    $post = Post::create($validated);
    
    // ❌ AVOID - Gets all request data
    $post = Post::create($request->all());
}
```

## Safe Data Handling

Use `safe()` for additional security:

```php
public function store(StorePostRequest $request): RedirectResponse
{
    $post = Post::create(
        $request->safe()->only(['title', 'body'])
    );
    
    // Or merge with additional data
    $post = Post::create(
        $request->safe()->merge(['user_id' => $request->user()->id])
    );
}
```

## Conditional Validation

```php
public function rules(): array
{
    return [
        'status' => ['required', Rule::in(['draft', 'published'])],
        'published_at' => [
            'required_if:status,published',
            'date',
            'after:now',
        ],
    ];
}

// Or with closure
public function withValidator($validator): void
{
    $validator->after(function ($validator) {
        if ($this->status === 'published' && !$this->published_at) {
            $validator->errors()->add(
                'published_at',
                'Publication date is required for published posts.'
            );
        }
    });
}
```

## Nested Array Validation

```php
public function rules(): array
{
    return [
        'address' => ['required', 'array'],
        'address.street' => ['required', 'string'],
        'address.city' => ['required', 'string'],
        'address.state' => ['required', 'string', 'size:2'],
        'address.zip' => ['required', 'string', 'regex:/^[0-9]{5}$/'],
    ];
}
```

## Best Practices Summary

1. **Always use Form Request classes** - never inline validation
2. **Include custom error messages** for better UX
3. **Use `validated()` method** to get only validated data
4. **Check project conventions** for array vs string-based rules
5. **Add authorization logic** in `authorize()` method when needed
6. **Use Rule class** for complex validation rules
7. **Validate nested arrays** with dot notation
8. **Use `safe()` method** for additional security
9. **Add conditional validation** when rules depend on other fields
10. **Use descriptive attribute names** in `attributes()` method
