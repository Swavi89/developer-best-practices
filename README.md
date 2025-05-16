# Essential Coding Concepts

Welcome 👋 to **Essential Coding Concepts**, a curated collection of best practices and architectural patterns aimed at helping developers write clean, maintainable, and scalable code.

This repository serves as a **developer knowledge hub**, consolidating best practices, coding standards, habits, and foundational concepts every developer should know. Whether you're a beginner looking to build strong coding habits or a seasoned programmer aiming to refine your skills, you'll find valuable guidance here.

## 📚 What's Inside?

This guide is structured across several focused documents:

### 1. Thick Model / Fat Model

> A "Fat Model" or "Thick Model" architecture emphasizes placing business logic in the model layer instead of the controller or view. This keeps your code organized and makes testing and maintenance easier.

**Benefits:**

- Centralized business logic
- Easier to test and reuse
- Better separation of concerns

**📌 Example: API Key Generation**

_In a Fat Model approach, business logic such as generating an API key should reside in the model, <s>not the controller</s>. This ensures that the logic is **reusable**, **testable**, and cleanly separated from the **request/response cycle**._

**For example**, _instead of generating an API key in a controller method, we encapsulate the logic within the model itself. Suppose we want every API key to start with specific letters (e.g., API-) followed by a random string of characters. This rule is part of our business logic, so it belongs in the model where the concept of an "API key" is defined._

#### app/Models/User.php

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Str;

class User extends Model
{
    /**
     * Generate and assign a new API key to the user.
     *
     * @return void
     */

    protected $fillable = ['name', 'email', 'password', 'api_key'];

    public function generateApiKey()
    {
        // Example format: API-<random 10-character string>
        $this->api_key = 'API-' . Str::random(10);
        $this->save();
    }
}
?>
```
---

### 2. Skinny Controllers

> A "Skinny Controller" ensures your controllers handle as little logic as possible — just enough to delegate work to models or services.

**Benefits:**

- Cleaner code
- Easier debugging
- Promotes modularity

**📌 Example: Skinny Controller Using Fat Model**

_In the Skinny Controller pattern, the controller acts only as a mediator between the request and the model. It does not contain business logic; instead, it **delegates** responsibility to the model (or service class). This makes controllers **short**, **focused**, and **easy to maintain**._

**For example**, _when generating an API key, the controller should not handle string manipulation or save operations. Instead, it should call a method like **generateApiKey()** from the model and return the result. This ensures a clean separation of concerns, making the codebase easier to test and scale._

#### app/Http/Controllers/UserController.php

```php
<?php

    namespace App\Http\Controllers;

    use App\Models\User;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {

      /**
       * Create a new API key for the given user.
       */

      public function createApiKey(Request $request)
        {
            $validator = Validator::make($request->all(), [
            'name'     => 'required|string|max:255',
            'email'    => 'required|email|unique:users,email',
            'password' => 'required|min:6',
            ]);

            $data = $request->only(['name', 'email', 'password']);
            $data['name'] = trim($data['name']);
            $data['email'] = trim($data['email']);
            $data['password'] = Hash::make($data['password']);

            $user = User::create($data);

            $user->generateApiKey(); // Delegates logic to the model

            return response()->json([
              'message' => 'User created successfully.',
              'user' => $user,
            ]);
        }
    }
?>
```

---

### 3. Accessing Related Records via Auth::user()

> Always query a resource through the currently authenticated user's relationship, so that authorisation and ownership are enforced in a single line.

> In Laravel, when a user is authenticated, you can access their information using Auth::user(). This gives you the currently logged-in user — and with Eloquent relationships defined properly, you can easily access related data like their projects, posts, tasks, or anything else they own.

**✅ Why Use Auth::user()->relationshipName()?**

When you access related records through Auth::user(), you're:

- _Making sure you only fetch data that belongs to the logged-in user_.
- _Preventing unauthorized data access (security best practice)_.
- _Keeping your code clean and aligned with Laravel's relationship-driven structure_.

**📌 Example: User Has Many Projects**

#### app/Models/User.php

```php
class User extends Model
{
    public function projects()
    {
        return $this->hasMany(Project::class);
    }
}
```

#### app/Models/Project.php

```php
class Project extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

__Inside Controller, instead of writing a raw query like:__

#### app/Http/Controllers/ProjectController.php

```php
use Illuminate\Support\Facades\Auth;

class ProjectController extends Controller
{
    /**
     * List the current user’s projects.
     */
    public function index()
    {
      $projects = Auth::user()->projects()
                  ->latest()        // example additional scope
                  ->get();
      return response()->json($projects);
    }
}
```

**🧠 How This Helps:**

* __Security__: This ensures you're not accessing someone else’s project by mistake.
* __Clarity__: It’s obvious you're only pulling projects that belong to the logged-in user.
* __Reusability__: The relationship is defined once in the model and used throughout your code.

---

### 4. Prefer firstOrFail()

When querying the database in Laravel, it’s common to retrieve a single record using Eloquent. While `first()` returns `null` if no result is found, it's **better to use `firstOrFail()`** when the record is required — especially for detail views, editing, or protected resources.

Using `firstOrFail()` makes your code more **secure**, **robust**, and **expressive** by:

- Automatically throwing a `404 Not Found` if the record doesn't exist
- Reducing the need for manual `if (!$model)` checks
- Keeping controller methods clean and focused

**📌 Example:  Recommended: `firstOrFail()` for Safer, cleaner approach**

#### app/Http/Controllers/ProjectController.php

```php
class ProjectController extends Controller
{
    public function editProject($id)
    {
        $project = Auth::user()->projects()
                    ->where('id', $id)
                    ->firstOrFail();
        return view ('projects.index', [
            'project' => $project,
        ]);
    }
}
```

**🔒 Why This is Safe and Clean**

| ✅ Feature                  | 💡 Benefit                                                       |
| -------------------------- | ---------------------------------------------------------------- |
| `Auth::user()->projects()` | Prevents access to projects not owned by the user                |
| `->firstOrFail()`          | Auto 404 if not found or not authorized                          |
| No manual checks           | Cleaner, more readable code                                      |
| Logical separation         | User-scoped logic lives in the model (`projects()` relationship) |

---

### 5. Laravel Naming Conventions

> Following Laravel’s standard naming conventions makes your application predictable, readable, and easy to navigate for all team members.

***🏗️ Models***

- Singular, PascalCase (also called StudlyCase)
- One model = one database table (Laravel handles pluralization)

```
/ ✅ Correct
User, Project, BlogPost, ProductImage

// ❌ Avoid
Users, project_model, blog_post_model
```

📝 File Path: app/Models/User.php || 📌 Migration: create_users_table

***🎮 Controllers***

- Plural, PascalCase, ends in Controller

```
// ✅ Correct
UserController, ProjectController, BlogPostController

// ❌ Avoid
UsersCtrl, projectcontroller, blogpostcontrol
```

📝 File Path: app/Http/Controllers/ProjectController.php

***📄 Blade Views (Templates)***

- kebab-case, lowercase file names
- Grouped by resource folders if needed

```
# ✅ Correct
resources/views/projects/index.blade.php
resources/views/projects/project-form.blade.php

# ❌ Avoid
resources/views/Projects/Index.blade.php
resources/views/projectForm.blade.php
```

>Tip: Keep view names consistent with your controller methods like `index()`, `create()`, `edit()`, etc.

***🌐 Routes***

- Use kebab-case URIs
- Controller methods follow standard RESTful naming

#### routes/web.php
```php
Route::get('/user-dashboard', [UserController::class, 'dashboard']);
```

While using `Named routes` allow you to assign a short, readable alias to any route. This makes your app easier to maintain — you can reference routes by name instead of hardcoding URLs.

#### routes/web.php (for named routes)

```php
Route::get('/projects', [ProjectController::class, 'index'])

### 6. Preferences: Flexible Key-Value Store

> Sometimes you need a simple place to stash feature flags, UI settings, or company meta-data without adding dozens of columns to an existing table.  

    ->name('projects.index');
```

>Now you can use `projects.index` as a reference throughout your app.
