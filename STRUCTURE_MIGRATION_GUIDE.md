# Laravel 11+ Application Structure Migration Guide

## Current Status

Your application is running **Laravel 12** but uses the **Laravel 8 structure**. This is fully supported and functional, but Laravel 11+ offers a more streamlined structure.

## Should You Migrate?

### ✅ Reasons to Migrate:
- Cleaner, more modern codebase
- Easier to understand for new developers
- Aligns with current Laravel best practices
- Simpler configuration in one place
- Reduced boilerplate code

### ⚠️ Reasons to Wait:
- Current structure works perfectly fine
- Takes time to migrate and test
- Team needs to be familiar with new structure
- Custom kernel methods need careful migration

**Our Recommendation:** Migrate when you have time for thorough testing, not during critical periods.

---

## What Will Change

### Files to Remove After Migration

```
❌ app/Console/Kernel.php
❌ app/Exceptions/Handler.php
❌ app/Http/Kernel.php
❌ app/Console/ (if empty)
❌ app/Exceptions/ (if empty)
```

### Files to Create/Update

```
✅ bootstrap/app.php (complete rewrite)
✅ routes/console.php (move scheduling here)
```

---

## Step-by-Step Migration

### Step 1: Backup Current Files

```bash
# Create backups
cp app/Http/Kernel.php app/Http/Kernel.php.backup
cp app/Console/Kernel.php app/Console/Kernel.php.backup
cp app/Exceptions/Handler.php app/Exceptions/Handler.php.backup
cp bootstrap/app.php bootstrap/app.php.backup
```

### Step 2: Create New `bootstrap/app.php`

Replace your current `bootstrap/app.php` with this:

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
        // Global Middleware
        // (Equivalent to $middleware in app/Http/Kernel.php)
        //
        $middleware->use([
            // \App\Http\Middleware\TrustHosts::class,
            \App\Http\Middleware\TrustProxies::class,
            \Illuminate\Http\Middleware\HandleCors::class,
            \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
            \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
            \App\Http\Middleware\TrimStrings::class,
            \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
        ]);

        //
        // Middleware Groups
        // (Equivalent to $middlewareGroups in app/Http/Kernel.php)
        //
        $middleware->group('web', [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            // \Illuminate\Session\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ]);

        $middleware->group('api', [
            // \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ])->throttle('api');

        //
        // Route Middleware Aliases
        // (Equivalent to $routeMiddleware in app/Http/Kernel.php)
        //
        $middleware->alias([
            'auth' => \App\Http\Middleware\Authenticate::class,
            'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
            'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
            'can' => \Illuminate\Auth\Middleware\Authorize::class,
            'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
            'password.confirm' => \Illuminate\Auth\Middleware\RequirePassword::class,
            'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
            'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
            'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
        ]);

        //
        // Middleware Priority (optional)
        // Define the order in which middleware should be executed
        //
        // $middleware->priority([
        //     \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        //     \Illuminate\Cookie\Middleware\EncryptCookies::class,
        //     \Illuminate\Session\Middleware\StartSession::class,
        //     \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        //     \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        //     \Illuminate\Routing\Middleware\ThrottleRequests::class,
        //     \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        //     \Illuminate\Contracts\Session\Middleware\AuthenticatesSessions::class,
        //     \Illuminate\Routing\Middleware\SubstituteBindings::class,
        //     \Illuminate\Auth\Middleware\Authorize::class,
        // ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
        // Exception Handling
        // (Equivalent to app/Exceptions/Handler.php)
        //

        // Reportable exceptions
        // $exceptions->report(function (InvalidOrderException $e) {
        //     // Custom reporting logic
        // });

        // Renderable exceptions
        // $exceptions->render(function (InvalidOrderException $e, Request $request) {
        //     return response()->view('errors.invalid-order', [], 500);
        // });

        // Don't report these exception types
        // $exceptions->dontReport([
        //     CustomException::class,
        // ]);

        // Don't flash these input fields
        // $exceptions->dontFlash([
        //     'current_password',
        //     'password',
        //     'password_confirmation',
        // ]);

        // Throttle reporting
        // $exceptions->throttle(function (Throwable $e) {
        //     return Limit::perMinute(5)->by($e->getMessage());
        // });
    })
    ->create();
```

### Step 3: Check for Custom Kernel Methods

Review your current Kernel files for any custom methods:

#### In `app/Http/Kernel.php`:
```bash
# Check for custom methods
grep -A 5 "public function" app/Http/Kernel.php
```

If you have custom methods like:
- `boot()` method with custom logic
- Custom middleware methods
- Middleware priority definitions

You'll need to migrate these. Common examples:

```php
// If you have custom middleware priority in app/Http/Kernel.php
protected $middlewarePriority = [
    // Your custom priority
];

// Add this to the new bootstrap/app.php in withMiddleware():
$middleware->priority([
    // Your custom priority array
]);
```

#### In `app/Console/Kernel.php`:
```bash
# Check your scheduled tasks
cat app/Console/Kernel.php
```

### Step 4: Migrate Scheduled Tasks

Move your scheduled tasks from `app/Console/Kernel.php` to `routes/console.php`:

**Old Way (app/Console/Kernel.php):**
```php
protected function schedule(Schedule $schedule)
{
    $schedule->command('inspire')->hourly();
    $schedule->command('backup:run')->daily();
}
```

**New Way (routes/console.php):**
```php
<?php

use Illuminate\Support\Facades\Schedule;

Schedule::command('inspire')->hourly();
Schedule::command('backup:run')->daily();
```

### Step 5: Migrate Custom Exception Handling

If you have custom exception handling in `app/Exceptions/Handler.php`:

**Old Way (app/Exceptions/Handler.php):**
```php
public function register()
{
    $this->reportable(function (Throwable $e) {
        // Custom reporting
    });
}
```

**New Way (bootstrap/app.php in withExceptions):**
```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (Throwable $e) {
        // Custom reporting
    });
})
```

### Step 6: Test Thoroughly

```bash
# Clear all caches
php artisan optimize:clear

# Test application boots
php artisan about

# Test middleware is working
php artisan route:list

# Test scheduled commands
php artisan schedule:list

# Run your test suite
php artisan test
```

### Step 7: Remove Old Files (Only After Testing!)

Once everything works:

```bash
# Remove old kernel files
rm app/Http/Kernel.php
rm app/Console/Kernel.php
rm app/Exceptions/Handler.php

# Remove directories if empty
rmdir app/Console 2>/dev/null || echo "app/Console has other files"
rmdir app/Exceptions 2>/dev/null || echo "app/Exceptions has other files"

# Remove backup files after you're confident
rm app/Http/Kernel.php.backup
rm app/Console/Kernel.php.backup
rm app/Exceptions/Handler.php.backup
rm bootstrap/app.php.backup
```

---

## Common Migration Scenarios

### Scenario 1: Custom Trusted Proxies

**Old (app/Http/Kernel.php):**
```php
protected $middleware = [
    \App\Http\Middleware\TrustProxies::class,
];
```

**New (bootstrap/app.php):**
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->use([
        \App\Http\Middleware\TrustProxies::class,
    ]);
})
```

### Scenario 2: CORS Configuration

**Old (app/Http/Kernel.php):**
```php
protected $middleware = [
    \Fruitcake\Cors\HandleCors::class, // Laravel 8
];
```

**New (bootstrap/app.php):**
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->use([
        \Illuminate\Http\Middleware\HandleCors::class, // Laravel 11+
    ]);
})
```

### Scenario 3: API Rate Limiting

Rate limiting configuration stays in `app/Providers/RouteServiceProvider.php` - no changes needed!

### Scenario 4: Sanctum Middleware

**Old (app/Http/Kernel.php):**
```php
'api' => [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    'throttle:api',
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```

**New (bootstrap/app.php):**
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->group('api', [
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ])->throttle('api');
})
```

---

## Troubleshooting

### Issue: "Class not found" errors

**Solution:** Clear and rebuild caches:
```bash
composer dump-autoload
php artisan optimize:clear
```

### Issue: Middleware not executing

**Solution:** Check middleware order and ensure all middleware are registered:
```bash
php artisan route:list --verbose
```

### Issue: Scheduled tasks not running

**Solution:** Check `routes/console.php` exists and contains your schedule:
```bash
php artisan schedule:list
```

### Issue: Custom exception handling not working

**Solution:** Verify `withExceptions()` configuration in `bootstrap/app.php`

---

## Comparison: Old vs New

### Old Structure (Laravel 8-10)
```
✗ Scattered configuration across multiple files
✗ More boilerplate code
✗ Multiple Kernel files to maintain
✗ Less discoverable for new developers
```

### New Structure (Laravel 11+)
```
✓ Centralized configuration in bootstrap/app.php
✓ Minimal boilerplate
✓ Single entry point for app configuration
✓ Fluent, readable API
✓ Easier to understand and maintain
```

---

## Example: Complete Migration Diff

### Before (55 lines in bootstrap/app.php + 3 Kernel files)

**bootstrap/app.php:**
```php
<?php
$app = new Illuminate\Foundation\Application(
    $_ENV['APP_BASE_PATH'] ?? dirname(__DIR__)
);

$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);

return $app;
```

Plus:
- `app/Http/Kernel.php` (67 lines)
- `app/Console/Kernel.php` (32 lines)
- `app/Exceptions/Handler.php` (41 lines)

**Total:** ~195 lines across 4 files

### After (Single file with fluent API)

**bootstrap/app.php:** (~80 lines with comments)
```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(/* ... */)
    ->withMiddleware(/* ... */)
    ->withExceptions(/* ... */)
    ->create();
```

**Total:** ~80 lines in 1 file

**Reduction:** ~60% fewer lines, 75% fewer files

---

## Next Steps

1. **Review this guide completely**
2. **Check for custom Kernel methods** in your current files
3. **Plan migration during low-traffic period**
4. **Create a feature branch** for the migration
5. **Follow steps 1-7 above**
6. **Test thoroughly** before merging
7. **Deploy to staging first**
8. **Monitor for issues** after deployment

---

## Questions?

- Review [Laravel 11 Release Notes](https://laravel.com/docs/11.x/releases)
- Check [Laravel 11 Upgrade Guide](https://laravel.com/docs/11.x/upgrade)
- Read [Application Structure Documentation](https://laravel.com/docs/11.x/structure)

**Remember:** This migration is **optional**. Your app works perfectly with the current structure!
