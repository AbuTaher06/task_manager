# Step 1: Setup Laravel Project

## 1️⃣ Install Laravel & Sanctum

Run the following commands:

```bash
composer create-project --prefer-dist laravel/laravel TaskManager
cd TaskManager
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
