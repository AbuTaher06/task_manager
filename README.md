# Step 1: Setup Laravel Project

## 1️⃣ Install Laravel & Sanctum

Run the following commands:

```bash
composer create-project --prefer-dist laravel/laravel TaskManager
cd TaskManager
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate

##2️⃣ Configure Sanctum
In app/Http/Kernel.php, add middleware:

Copy
use Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful;

protected $middlewareGroups = [
    'api' => [
        EnsureFrontendRequestsAreStateful::class,
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
In config/cors.php, update:

Copy
'paths' => ['api/*', 'sanctum/csrf-cookie'],
