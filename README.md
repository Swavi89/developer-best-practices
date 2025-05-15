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

_In a Fat Model approach, business logic such as generating an API key should reside in the model, <s>not the controller</s>. This ensures that the logic is __reusable__, __testable__, and cleanly separated from the __request/response cycle__._

**For example**, _instead of generating an API key in a controller method, we encapsulate the logic within the model itself. Suppose we want every API key to start with specific letters (e.g., API-) followed by a random string of characters. This rule is part of our business logic, so it belongs in the model where the concept of an "API key" is defined._

#### app/Models/User.php

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
---

### 2. Skinny Controllers

> A "Skinny Controller" ensures your controllers handle as little logic as possible — just enough to delegate work to models or services.

**Benefits:**

- Cleaner code
- Easier debugging
- Promotes modularity

**📌 Example: Skinny Controller Using Fat Model**

_In the Skinny Controller pattern, the controller acts only as a mediator between the request and the model. It does not contain business logic; instead, it __delegates__ responsibility to the model (or service class). This makes controllers __short__, __focused__, and __easy to maintain__._

**For example**, _when generating an API key, the controller should not handle string manipulation or save operations. Instead, it should call a method like __generateApiKey()__ from the model and return the result. This ensures a clean separation of concerns, making the codebase easier to test and scale._

#### app/Http/Controllers/UserController.php

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
---
