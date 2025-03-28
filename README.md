Step 1: Setup Laravel Project
1️⃣ Install Laravel & Sanctum
Run the following commands:

bash
Copy
Edit
composer create-project --prefer-dist laravel/laravel TaskManager
cd TaskManager
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
2️⃣ Configure Sanctum
In app/Http/Kernel.php, add middleware:

php
Copy
Edit
use Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful;

protected $middlewareGroups = [
    'api' => [
        EnsureFrontendRequestsAreStateful::class,
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
In config/cors.php, update:

php
Copy
Edit
'paths' => ['api/*', 'sanctum/csrf-cookie'],
Step 2: Create Models, Migrations & Controllers
1️⃣ Generate Models and Migrations
bash
Copy
Edit
php artisan make:model Task -mcr
php artisan make:model Project -mcr
-mcr creates the model, migration, and controller.

Step 3: Define Database Schema
1️⃣ Edit database/migrations/xxxx_xx_xx_create_tasks_table.php
php
Copy
Edit
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
2️⃣ Edit database/migrations/xxxx_xx_xx_create_projects_table.php
php
Copy
Edit
public function up()
{
    Schema::create('projects', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->timestamps();
    });
}
Run migrations:

bash
Copy
Edit
php artisan migrate
Step 4: Define Model Relationships
1️⃣ In app/Models/Project.php
php
Copy
Edit
class Project extends Model
{
    use HasFactory;
    protected $fillable = ['name'];

    public function tasks()
    {
        return $this->hasMany(Task::class);
    }
}
2️⃣ In app/Models/Task.php
php
Copy
Edit
class Task extends Model
{
    use HasFactory;
    protected $fillable = ['title', 'description', 'project_id', 'is_completed'];

    public function project()
    {
        return $this->belongsTo(Project::class);
    }
}
Step 5: Implement API Endpoints
1️⃣ Define Routes in routes/api.php
php
Copy
Edit
use App\Http\Controllers\TaskController;
use App\Http\Controllers\ProjectController;
use Illuminate\Support\Facades\Route;

Route::middleware(['auth:sanctum'])->group(function () {
    Route::apiResource('projects', ProjectController::class);
    Route::apiResource('tasks', TaskController::class);
});

Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);
Step 6: Implement Controllers
1️⃣ Create AuthController for User Authentication
bash
Copy
Edit
php artisan make:controller AuthController
php
Copy
Edit
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required|string',
            'email' => 'required|string|email|unique:users',
            'password' => 'required|string|min:6'
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        return response()->json(['token' => $user->createToken('auth_token')->plainTextToken]);
    }

    public function login(Request $request)
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required'
        ]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages(['email' => 'Invalid credentials']);
        }

        return response()->json(['token' => $user->createToken('auth_token')->plainTextToken]);
    }
}
2️⃣ Create ProjectController
php
Copy
Edit
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
        $request->validate(['name' => 'required']);
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
        return response()->json(['message' => 'Project deleted']);
    }
}
3️⃣ Create TaskController
php
Copy
Edit
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
        $request->validate([
            'title' => 'required',
            'project_id' => 'required|exists:projects,id'
        ]);
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
        return response()->json(['message' => 'Task deleted']);
    }
}
Step 7: Test API with Postman
1️⃣ Register User
POST /api/register

json
Copy
Edit
{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "password"
}
2️⃣ Login User
POST /api/login

json
Copy
Edit
{
    "email": "john@example.com",
    "password": "password"
}
Response:

json
Copy
Edit
{
    "token": "your-auth-token"
}
Use this token for authentication in the next requests.

3️⃣ Create a Project
POST /api/projects

json
Copy
Edit
{
    "name": "Laravel API Development"
}
4️⃣ Create a Task
POST /api/tasks

json
Copy
Edit
{
    "title": "Complete Laravel API",
    "description": "Finish the task manager API",
    "project_id": 1
}
🎯 Congratulations! You’ve built a full Task Management API using Laravel! 🚀
