# Step 1: Setup Laravel Project

## 1️⃣ Install Laravel & Sanctum

Run the following commands:

```bash
composer create-project --prefer-dist laravel/laravel TaskManager
cd TaskManager
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

## 2️⃣ Configure Sanctum

In `app/Http/Kernel.php`, add middleware:

```php
use Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful;

protected $middlewareGroups = [
    'api' => [
        EnsureFrontendRequestsAreStateful::class,
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
```

In `config/cors.php`, update:

```php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
```

---

# Step 2: Create Models, Migrations & Controllers

## 1️⃣ Generate Models and Migrations

Run the following commands:

```bash
php artisan make:model Task -mcr
php artisan make:model Project -mcr
```
*The `-mcr` option creates the model, migration, and controller.*

---

# Step 3: Define Database Schema

## 1️⃣ Edit `database/migrations/xxxx_xx_xx_create_tasks_table.php`

```php
public function up()
{
    Schema::create('tasks', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('description')->nullable();
        $table->foreignId('project_id')->constrained()->onDelete('cascade');
        $table->boolean('is_completed')->default(false);
        $table->timestamps();
    });
}
```

## 2️⃣ Edit `database/migrations/xxxx_xx_xx_create_projects_table.php`

```php
public function up()
{
    Schema::create('projects', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->timestamps();
    });
}
```

Run migrations:

```bash
php artisan migrate
```

---

# Step 4: Define Model Relationships

## 1️⃣ In `app/Models/Project.php`

```php
class Project extends Model
{
    use HasFactory;
    protected $fillable = ['name'];

    public function tasks()
    {
        return $this->hasMany(Task::class);
    }
}
```

## 2️⃣ In `app/Models/Task.php`

```php
class Task extends Model
{
    use HasFactory;
    protected $fillable = ['title', 'description', 'project_id', 'is_completed'];

    public function project()
    {
        return $this->belongsTo(Project::class);
    }
}
```

---

# Step 5: Implement API Endpoints

## 1️⃣ Define Routes in `routes/api.php`

```php
use App\Http\Controllers\TaskController;
use App\Http\Controllers\ProjectController;
use App\Http\Controllers\AuthController;
use Illuminate\Support\Facades\Route;

Route::middleware(['auth:sanctum'])->group(function () {
    Route::apiResource('projects', ProjectController::class);
    Route::apiResource('tasks', TaskController::class);
});

Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);
```

---

# Step 6: Implement Controllers

## 1️⃣ Create `AuthController` for User Authentication

Run:

```bash
php artisan make:controller AuthController
```

### `AuthController.php`

```php
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Auth;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:8|confirmed',
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        return response()->json($user, 201);
    }

    public function login(Request $request)
    {
        $request->validate([
            'email' => 'required|string|email',
            'password' => 'required|string',
        ]);

        if (!Auth::attempt($request->only('email', 'password'))) {
            return response()->json(['message' => 'Unauthorized'], 401);
        }

        return response()->json(['token' => Auth::user()->createToken('API Token')->plainTextToken]);
    }
}
```

---

# Step 7: Implement Task and Project Controllers

## 1️⃣ `TaskController.php`

```php
namespace App\Http\Controllers;

use App\Models\Task;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    public function index()
    {
        return Task::all();
    }

    public function store(Request $request)
    {
        $request->validate(['title' => 'required|string']);
        return Task::create($request->all());
    }

    public function show(Task $task)
    {
        return $task;
    }

    public function update(Request $request, Task $task)
    {
        $task->update($request->all());
        return $task;
    }

    public function destroy(Task $task)
    {
        $task->delete();
        return response()->json(null, 204);
    }
}
```

## 2️⃣ `ProjectController.php`

```php
namespace App\Http\Controllers;

use App\Models\Project;
use Illuminate\Http\Request;

class ProjectController extends Controller
{
    public function index()
    {
        return Project::with('tasks')->get();
    }

    public function store(Request $request)
    {
        $request->validate(['name' => 'required|string']);
        return Project::create($request->all());
    }

    public function show(Project $project)
    {
        return $project->load('tasks');
    }

    public function update(Request $request, Project $project)
    {
        $project->update($request->all());
        return $project;
    }

    public function destroy(Project $project)
    {
        $project->delete();
        return response()->json(null, 204);
    }
}
```

---

# Step 8: Testing the API

Use Postman or any API testing tool to test the endpoints:

- **Register:** `POST /api/register`
- **Login:** `POST /api/login`
- **Projects:** 
  - `GET /api/projects`
  - `POST /api/projects`
  - `GET /api/projects/{id}`
  - `PUT /api/projects/{id}`
  - `DELETE /api/projects/{id}`
- **Tasks:**
  - `GET /api/tasks`
  - `POST /api/tasks`
  - `GET /api/tasks/{id}`
  - `PUT /api/tasks/{id}`
  - `DELETE /api/tasks/{id}`
