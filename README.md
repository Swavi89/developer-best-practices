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
     * List the current user's projects.
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

* __Security__: This ensures you're not accessing someone else's project by mistake.
* __Clarity__: It's obvious you're only pulling projects that belong to the logged-in user.
* __Reusability__: The relationship is defined once in the model and used throughout your code.

---

### 4. Prefer firstOrFail()

When querying the database in Laravel, it's common to retrieve a single record using Eloquent. While `first()` returns `null` if no result is found, it's **better to use `firstOrFail()`** when the record is required — especially for detail views, editing, or protected resources.

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

> Following Laravel's standard naming conventions makes your application predictable, readable, and easy to navigate for all team members.

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
  

    ->name('projects.index');
```

>Now you can use `projects.index` as a reference throughout your app.

---

### 6. Preferences: Flexible Key-Value Store

>This approach lets you store settings and preferences (like key-value pairs) in a dedicated database table. You can easily save and retrieve these settings using simple get and set methods in your code.

**🔧 Step 1: Migration**

_Create a preferences table with key and value columns:_

```
php artisan make:migration create_preferences_table
```

_Setup that migration file & Run artisan command to migrate the table:_
```php
Schema::create('preferences', function (Blueprint $table) {
    $table->id();
    $table->string('key')->unique();
    $table->longText('value')->nullable();
    $table->timestamps();
});
```

**🧠 Step 2: Model**

_Create the Preference model:_

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Preference extends Model
{
    public static function get($key, $default = null)
    {
        // Retrieves a preference value by its key
    }

    public static function set($key, $value)
    {
        // Set the key value pairs accordingly
    }
}
```

**🧪 Step 3: Usage Example**
```php
// Set a preference
Preference::set('site_name', 'My Awesome Site');

// Get a preference
$name = Preference::get('site_name');
```

**Usage**

_Let's say you have a currency conversion module where you need to store exchange rates (like USD to INR). Instead of creating a new table for each currency pair, you can use this model to store them as key-value pairs. For example:_

```php
// Store exchange rates
Preference::set('usd_to_inr', '83.25');
Preference::set('eur_to_inr', '90.15');

// Retrieve when needed
$usdRate = Preference::get('usd_to_inr'); // Returns 83.25
```

_This way, you can easily update and access these rates from anywhere in your application without creating multiple database tables._

### 7. Telegram Notification Integration

> A simple way to send automated notifications to your Telegram channel or chat using your own bot. This is useful for sending alerts, updates, or form submissions directly to your Telegram.

**🔧 Step 1: Setup**

_First, create a Telegram bot and get your bot token:_

1. Message [@BotFather](https://t.me/botfather) on Telegram
2. Use `/newbot` command to create a new bot
3. Save your bot token in `.env` file:
```
TELEGRAM_BOT_TOKEN=your_bot_token_here
```

**📝 Step 2: Create Model with Notification Methods**

```php
class TelegramNotification extends Model
{
    protected $bot_token;
    protected $forms_chat_id = '8111321280'; // Your channel/chat ID

    public function __construct()
    {
        $this->bot_token = env('TELEGRAM_BOT_TOKEN');
    }

    // Basic method to send any message
    public function sendMessage($message, $chat_id)
    {
        $url = 'https://api.telegram.org/bot' . $this->bot_token . '/sendMessage';
        $data = [
            'chat_id' => $chat_id,
            'text' => $message,
            'parse_mode' => 'markdown',
        ];

        // Send the message using cURL
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_POST, 1);
        curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($data));
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        $result = curl_exec($ch);
        curl_close($ch);
    }

    // Example: Send notification for new demo requests
    public static function notifyNewDemoRequest($name, $email, $phone, $country)
    {
        $message = "📝 *New Demo Request*\n";
        $message .= "Name: $name\n";
        $message .= "Email: $email\n";
        $message .= "Phone: $phone\n";
        $message .= "Country: $country";

        $notification = new TelegramNotification();
        $notification->sendMessage($message, $notification->forms_chat_id);
    }
}
```

**🧪 Step 3: Usage Example**


_Imagine you have a contact form on your website. When someone fills out this form, you want to get instant notifications on your Telegram. Here's how it works:_

```php
// When someone submits a demo request form
TelegramNotification::notifyNewDemoRequest(
    'John Doe',
    'john@example.com',
    '+1234567890',
    'United States'
);
```

_As soon as this code runs, you'll receive a beautifully formatted message in your Telegram channel like this:_

```
📝 New Demo Request
Name: John Doe
Email: john@example.com
Phone: +1234567890
Country: United States
```

_This way, you never miss any important inquiries from your website visitors!_

**💡 Common Use Cases:**
- Form submission notifications
- Error alerts
- Daily reports
- User activity updates
- System status notifications

_This approach lets you easily send formatted messages to your Telegram channel or chat whenever something important happens in your application. The messages can include text, emojis, and markdown formatting for better readability._

