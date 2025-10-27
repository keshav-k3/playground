# Laravel 8 to 12 Upgrade Guide

This guide provides a comprehensive walkthrough for upgrading a Laravel 8.75 application to Laravel 12.35.0, covering all breaking changes, configuration updates, and structural migrations.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Overview of Changes](#overview-of-changes)
3. [Step-by-Step Upgrade Process](#step-by-step-upgrade-process)
4. [Breaking Changes by Version](#breaking-changes-by-version)
5. [Configuration File Updates](#configuration-file-updates)
6. [Structure Migration (Laravel 11+)](#structure-migration-laravel-11)
7. [Frontend Build Tool Migration](#frontend-build-tool-migration)
8. [Testing and Verification](#testing-and-verification)
9. [Post-Upgrade Tasks](#post-upgrade-tasks)
10. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### System Requirements

- **PHP**: 8.2 or higher (recommended: 8.4+)
- **Composer**: 2.0 or higher
- **Node.js**: 18.0 or higher (for Vite)
- **Database**: Check compatibility with your database version

### Before You Start

1. **Backup your application** (database, files, .env)
2. **Review your codebase** for deprecated features
3. **Update your local environment** to meet system requirements
4. **Read the official upgrade guides**:
   - [Laravel 9 Upgrade Guide](https://laravel.com/docs/9.x/upgrade)
   - [Laravel 10 Upgrade Guide](https://laravel.com/docs/10.x/upgrade)
   - [Laravel 11 Upgrade Guide](https://laravel.com/docs/11.x/upgrade)
   - [Laravel 12 Upgrade Guide](https://laravel.com/docs/12.x/upgrade)

---

## Overview of Changes

### Major Version Transitions

```
Laravel 8.75 (PHP 7.3|8.0) → Laravel 9.x (PHP 8.0+)
→ Laravel 10.x (PHP 8.1+) → Laravel 11.x (PHP 8.2+)
→ Laravel 12.x (PHP 8.2+)
```

### Key Dependency Updates

| Package | Laravel 8 | Laravel 12 |
|---------|-----------|------------|
| PHP | ^7.3\|^8.0 | ^8.2 |
| PHPUnit | ^9.0 | ^11.0 |
| Symfony | ^5.0 | ^7.0 |
| Monolog | ^2.0 | ^3.0 |
| Carbon | ^2.0 | ^3.0 |
| Flysystem | ^1.0 | ^3.0 |
| Laravel Sanctum | ^2.0 | ^4.0 |

### Architectural Changes

- **Laravel 9**: SwiftMailer → Symfony Mailer
- **Laravel 10**: Policy auto-registration
- **Laravel 11**: Streamlined application structure (removal of Kernel files)
- **Laravel 12**: Updated filesystem defaults, enhanced type safety

---

## Step-by-Step Upgrade Process

### Phase 1: Update composer.json

Replace your `composer.json` dependencies with:

```json
{
    "require": {
        "php": "^8.2",
        "guzzlehttp/guzzle": "^7.8",
        "laravel/framework": "^12.0",
        "laravel/sanctum": "^4.0",
        "laravel/tinker": "^2.10"
    },
    "require-dev": {
        "fakerphp/faker": "^1.24",
        "laravel/pint": "^1.13",
        "laravel/sail": "^1.32",
        "mockery/mockery": "^1.6",
        "nunomaduro/collision": "^8.0",
        "phpunit/phpunit": "^11.0",
        "spatie/laravel-ignition": "^2.4",
        "pestphp/pest": "^3.0",
        "pestphp/pest-plugin-laravel": "^3.0"
    },
    "config": {
        "minimum-stability": "stable",
        "allow-plugins": {
            "pestphp/pest-plugin": true,
            "php-http/discovery": true
        }
    }
}
```

**Key Changes:**
- PHP requirement: `^7.3|^8.0` → `^8.2`
- Laravel Framework: `^8.75` → `^12.0`
- Removed packages: `fruitcake/laravel-cors`, `facade/ignition`, `laravel/mix`
- Added packages: `laravel/pint`, `pestphp/pest`, `spatie/laravel-ignition`

### Phase 2: Install Dependencies

```bash
# Allow Pest plugin
composer config allow-plugins.pestphp/pest-plugin true

# Update all dependencies
composer update

# Clear caches
php artisan config:clear
php artisan cache:clear
php artisan view:clear
```

**Expected Issues:**
- If you encounter dependency conflicts, remove `composer.lock` and try again
- Some packages may require manual version updates

---

## Breaking Changes by Version

### Laravel 9 Breaking Changes

#### 1. Symfony Mailer (Replaces SwiftMailer)

**Impact**: All email functionality

**config/mail.php changes:**

```php
// REMOVE deprecated auth_mode
'smtp' => [
    'transport' => 'smtp',
    'host' => env('MAIL_HOST', 'smtp.mailgun.org'),
    'port' => env('MAIL_PORT', 587),
    'encryption' => env('MAIL_ENCRYPTION', 'tls'),
    'username' => env('MAIL_USERNAME'),
    'password' => env('MAIL_PASSWORD'),
    'timeout' => null,
    // 'auth_mode' => null, // ← REMOVE THIS LINE
],
```

#### 2. Flysystem 3.x

**Impact**: File storage operations

**config/filesystems.php changes:**

```php
'disks' => [
    'local' => [
        'driver' => 'local',
        'root' => storage_path('app/private'), // Changed from storage_path('app')
        'serve' => true, // NEW: Enable local serving
        'throw' => false, // NEW: Flysystem 3 compatibility
    ],
],
```

#### 3. Environment Variable Rename

**Impact**: All filesystem disk references

**.env changes:**

```bash
# Before
FILESYSTEM_DRIVER=local

# After
FILESYSTEM_DISK=local
```

#### 4. PostgreSQL Configuration

**Impact**: PostgreSQL users only

**config/database.php changes:**

```php
'pgsql' => [
    'driver' => 'pgsql',
    // ...
    'search_path' => 'public', // Changed from 'schema'
],
```

### Laravel 10 Breaking Changes

#### 1. Policy Auto-Registration

**Impact**: `app/Providers/AuthServiceProvider.php`

```php
// Before
public function boot()
{
    $this->registerPolicies();
}

// After (registerPolicies is auto-invoked)
public function boot(): void
{
    // Just remove the call, it's automatic now
}
```

#### 2. Minimum PHP Version

**Requirement**: PHP 8.1+ (8.2+ recommended)

### Laravel 11 Breaking Changes

#### 1. Streamlined Application Structure (Kernel Files Removed)

**Impact**: Major architectural change - complete restructuring of application bootstrap

**What's Changed:**

Laravel 11 eliminates the traditional Kernel and Handler pattern, consolidating all middleware, exception handling, and console configuration into a single, streamlined `bootstrap/app.php` file using a fluent builder pattern.

**Files to REMOVE:**
- `app/Http/Kernel.php` - Middleware configuration moved to `bootstrap/app.php`
- `app/Console/Kernel.php` - Console scheduling moved to `routes/console.php`
- `app/Exceptions/Handler.php` - Exception handling moved to `bootstrap/app.php`

**Files to CREATE/UPDATE:**
- `bootstrap/app.php` - Complete rewrite with builder pattern
- `routes/console.php` - Now handles scheduled tasks using `Schedule` facade

**Why This Change:**

1. **Simplicity**: Reduces boilerplate code from ~500 lines across 3 files to ~80 lines in 1 file
2. **Clarity**: All application bootstrapping in one place
3. **Modern PHP**: Uses fluent interface and method chaining
4. **Easier Testing**: Simpler to mock and test individual configurations

**Migration Impact:**

- **Middleware**: All global, group, and route middleware must be registered in `bootstrap/app.php`
- **Scheduled Tasks**: Move `$schedule->...` calls from `app/Console/Kernel.php` to `routes/console.php`
- **Exception Handling**: Custom `report()` and `render()` methods move to `bootstrap/app.php`
- **Route Configuration**: Routing setup moves to `withRouting()` method
- **Middleware Priority**: Priority middleware configured via `$middleware->priority([])`

See [Structure Migration](#structure-migration-laravel-11) section for complete implementation details.

#### 2. Console Schedule Location (No More Console Kernel)

**Impact**: All scheduled tasks must be moved

**What Changed:**

The `app/Console/Kernel.php` file is completely removed. The `schedule()` method that contained all your scheduled tasks is gone. Instead, scheduled tasks are now defined directly in `routes/console.php` using the `Schedule` facade.

**Before (Laravel 8-10):**

```php
// app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    $schedule->command('inspire')->hourly();
    $schedule->command('emails:send')->daily();
    $schedule->job(new ProcessPodcast)->everyFifteenMinutes();
}
```

**After (Laravel 11+):**

```php
// routes/console.php
<?php

use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\Schedule;

// Define scheduled tasks directly
Schedule::command('inspire')->hourly();
Schedule::command('emails:send')->daily();
Schedule::job(new ProcessPodcast)->everyFifteenMinutes();

// Artisan commands can still be defined here
Artisan::command('inspire', function () {
    $this->comment(Inspiring::quote());
})->purpose('Display an inspiring quote');
```

**Migration Steps:**

1. Copy all `$schedule->...` calls from `app/Console/Kernel.php`
2. Paste into `routes/console.php`
3. Add `use Illuminate\Support\Facades\Schedule;` at the top
4. Change `$schedule->` to `Schedule::` (uppercase, static calls)
5. Remove `app/Console/Kernel.php`

#### 3. Exception Handler Removed (No More Handler.php)

**Impact**: All exception handling logic must be moved

**What Changed:**

The `app/Exceptions/Handler.php` file is completely removed. All exception reporting, rendering, and configuration is now done through the `withExceptions()` method in `bootstrap/app.php`.

**Before (Laravel 8-10):**

```php
// app/Exceptions/Handler.php
namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Throwable;

class Handler extends ExceptionHandler
{
    protected $dontReport = [
        //
    ];

    protected $dontFlash = [
        'current_password',
        'password',
        'password_confirmation',
    ];

    public function register()
    {
        $this->reportable(function (Throwable $e) {
            // Custom reporting logic
        });

        $this->renderable(function (CustomException $e, $request) {
            return response()->json(['error' => $e->getMessage()], 500);
        });
    }
}
```

**After (Laravel 11+):**

```php
// bootstrap/app.php
use Illuminate\Foundation\Configuration\Exceptions;

return Application::configure(basePath: dirname(__DIR__))
    // ... other configuration ...
    ->withExceptions(function (Exceptions $exceptions) {
        // Fields to not flash on validation errors
        $exceptions->dontFlash([
            'current_password',
            'password',
            'password_confirmation',
        ]);

        // Custom exception reporting
        $exceptions->report(function (CustomException $e) {
            // Report to external service
            Log::error('Custom exception', ['exception' => $e]);
        });

        // Custom exception rendering
        $exceptions->render(function (CustomException $e, $request) {
            return response()->json([
                'error' => $e->getMessage()
            ], 500);
        });

        // Don't report specific exceptions
        $exceptions->dontReport([
            SomeSpecificException::class,
        ]);

        // Throttle reporting
        $exceptions->throttle(function (Throwable $e) {
            return Limit::perMinute(5)->by($e->getCode());
        });
    })
    ->create();
```

**Available Exception Methods:**

- `dontFlash()` - Specify input fields to not flash on validation errors
- `report()` - Custom exception reporting logic
- `render()` - Custom exception rendering (JSON, views, etc.)
- `dontReport()` - Exceptions that should not be reported
- `throttle()` - Throttle exception reporting to prevent spam
- `level()` - Set custom log levels for specific exceptions

**Migration Steps:**

1. Open your `app/Exceptions/Handler.php`
2. Copy the contents of `$dontFlash` array to `$exceptions->dontFlash([])`
3. Copy any `reportable()` callbacks to `$exceptions->report()`
4. Copy any `renderable()` callbacks to `$exceptions->render()`
5. Move `$dontReport` array to `$exceptions->dontReport([])`
6. Remove `app/Exceptions/Handler.php`

### Laravel 12 Breaking Changes

#### 1. Filesystem Local Disk Root Change

**Impact**: All local file storage

**Default path changed:**
- Before: `storage/app`
- After: `storage/app/private`

**Migration steps:**
1. Update `config/filesystems.php` (see above)
2. Move existing files if needed:
   ```bash
   mkdir -p storage/app/private
   # Review and move files as appropriate
   ```

#### 2. Enhanced Type Hints

**Impact**: Custom providers, middleware, and service classes

**Example - RouteServiceProvider.php:**

```php
// Before
public function boot()
{
    //...
}

// After
public function boot(): void
{
    //...
}
```

---

## Configuration File Updates

### config/filesystems.php

Complete file with all Laravel 12 changes:

```php
<?php

return [
    'default' => env('FILESYSTEM_DISK', 'local'),

    'disks' => [
        'local' => [
            'driver' => 'local',
            'root' => storage_path('app/private'),
            'serve' => true,
            'throw' => false,
        ],

        'public' => [
            'driver' => 'local',
            'root' => storage_path('app/public'),
            'url' => env('APP_URL').'/storage',
            'visibility' => 'public',
            'throw' => false,
        ],

        's3' => [
            'driver' => 's3',
            'key' => env('AWS_ACCESS_KEY_ID'),
            'secret' => env('AWS_SECRET_ACCESS_KEY'),
            'region' => env('AWS_DEFAULT_REGION'),
            'bucket' => env('AWS_BUCKET'),
            'url' => env('AWS_URL'),
            'endpoint' => env('AWS_ENDPOINT'),
            'use_path_style_endpoint' => env('AWS_USE_PATH_STYLE_ENDPOINT', false),
            'throw' => false,
        ],
    ],

    'links' => [
        public_path('storage') => storage_path('app/public'),
    ],
];
```

### config/database.php

```php
// PostgreSQL users - update this:
'pgsql' => [
    'driver' => 'pgsql',
    'url' => env('DATABASE_URL'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '5432'),
    'database' => env('DB_DATABASE', 'forge'),
    'username' => env('DB_USERNAME', 'forge'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8',
    'prefix' => '',
    'prefix_indexes' => true,
    'search_path' => 'public', // Changed from 'schema'
    'sslmode' => 'prefer',
],
```

### config/mail.php

```php
'mailers' => [
    'smtp' => [
        'transport' => 'smtp',
        'host' => env('MAIL_HOST', 'smtp.mailgun.org'),
        'port' => env('MAIL_PORT', 587),
        'encryption' => env('MAIL_ENCRYPTION', 'tls'),
        'username' => env('MAIL_USERNAME'),
        'password' => env('MAIL_PASSWORD'),
        'timeout' => null,
        // REMOVED: 'auth_mode' => null,
    ],
],
```

### .env.example

Update your environment variable names:

```bash
# Filesystem
FILESYSTEM_DISK=local  # Changed from FILESYSTEM_DRIVER

# Vite (if migrating from Mix)
VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

---

## Structure Migration (Laravel 11+)

### Overview

Laravel 11 introduced a streamlined application structure that consolidates middleware, exception handling, and console command registration into a single `bootstrap/app.php` file.

### Step 1: Backup Old Files

```bash
mkdir -p backup
cp app/Http/Kernel.php backup/
cp app/Console/Kernel.php backup/
cp app/Exceptions/Handler.php backup/
```

### Step 2: Create New bootstrap/app.php

**Create/replace** `bootstrap/app.php`:

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
        // Global Middleware Stack
        $middleware->use([
            \App\Http\Middleware\TrustProxies::class,
            \Illuminate\Http\Middleware\HandleCors::class,
            \App\Http\Middleware\PreventRequestsDuringMaintenance::class,
            \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
            \App\Http\Middleware\TrimStrings::class,
            \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
        ]);

        // Web Middleware Group
        $middleware->group('web', [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ]);

        // API Middleware Group
        $middleware->group('api', [
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ]);

        // Route Middleware Aliases
        $middleware->alias([
            'auth' => \App\Http\Middleware\Authenticate::class,
            'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
            'auth.session' => \Illuminate\Session\Middleware\AuthenticateSession::class,
            'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
            'can' => \Illuminate\Auth\Middleware\Authorize::class,
            'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
            'password.confirm' => \Illuminate\Auth\Middleware\RequirePassword::class,
            'precognition' => \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
            'signed' => \App\Http\Middleware\ValidateSignature::class,
            'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
            'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        // Exception handling configuration
        $exceptions->dontFlash([
            'current_password',
            'password',
            'password_confirmation',
        ]);
    })
    ->create();
```

### Step 3: Update routes/console.php

```php
<?php

use Illuminate\Foundation\Inspiring;
use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\Schedule;

Artisan::command('inspire', function () {
    $this->comment(Inspiring::quote());
})->purpose('Display an inspiring quote')->hourly();

// Schedule tasks here (moved from app/Console/Kernel.php)
Schedule::command('inspire')->hourly();
```

### Step 4: Remove Old Files

```bash
rm app/Http/Kernel.php
rm app/Console/Kernel.php
rm app/Exceptions/Handler.php
```

### Step 5: Migrate Custom Middleware

If you had custom middleware in your old Kernel files:

1. **Middleware registration**: Add to `bootstrap/app.php` in appropriate section
2. **Middleware groups**: Add to the `$middleware->group()` calls
3. **Route middleware**: Add to the `$middleware->alias()` array

### Step 6: Migrate Custom Exception Handling

If you had custom exception handling in `Handler.php`:

```php
// In bootstrap/app.php, enhance the withExceptions() callback:
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->dontFlash([
        'current_password',
        'password',
        'password_confirmation',
    ]);

    // Add custom exception reporting
    $exceptions->report(function (CustomException $e) {
        // Custom reporting logic
    });

    // Add custom exception rendering
    $exceptions->render(function (CustomException $e, $request) {
        // Custom rendering logic
    });
})
```

---

## Frontend Build Tool Migration

### Option 1: Keep Laravel Mix (Not Recommended)

If you want to keep using Laravel Mix temporarily:

```bash
npm install --save-dev laravel-mix@^6.0
```

However, Laravel Mix is no longer actively maintained for Laravel 12+.

### Option 2: Migrate to Vite (Recommended)

#### Step 1: Update package.json

```json
{
    "private": true,
    "scripts": {
        "dev": "vite",
        "build": "vite build"
    },
    "devDependencies": {
        "axios": "^1.7.0",
        "laravel-vite-plugin": "^2.0.1",
        "vite": "^7.1.12"
    }
}
```

#### Step 2: Create vite.config.js

```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
            ],
            refresh: true,
        }),
    ],
});
```

#### Step 3: Remove webpack.mix.js

```bash
rm webpack.mix.js
```

#### Step 4: Convert JavaScript to ES Modules

**resources/js/app.js:**

```javascript
// Before (CommonJS)
require('./bootstrap');

// After (ES Modules)
import './bootstrap';
```

**resources/js/bootstrap.js:**

```javascript
// Before (CommonJS)
window._ = require('lodash');
try {
    window.Popper = require('popper.js').default;
    window.$ = window.jQuery = require('jquery');
    require('bootstrap');
} catch (e) {}
window.axios = require('axios');

// After (ES Modules)
import axios from 'axios';
window.axios = axios;
window.axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';

// For Echo/Pusher (when needed):
// import Echo from 'laravel-echo';
// import Pusher from 'pusher-js';
// window.Pusher = Pusher;
// window.Echo = new Echo({...});
```

#### Step 5: Update Blade Templates

```blade
{{-- Before (Mix) --}}
<link href="{{ mix('css/app.css') }}" rel="stylesheet">
<script src="{{ mix('js/app.js') }}" defer></script>

{{-- After (Vite) --}}
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

#### Step 6: Update Environment Variables

```bash
# Before (Mix)
MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

# After (Vite)
VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

**In JavaScript:**

```javascript
// Before (Mix)
process.env.MIX_PUSHER_APP_KEY

// After (Vite)
import.meta.env.VITE_PUSHER_APP_KEY
```

#### Step 7: Update .gitignore

```gitignore
/public/hot
/public/storage
/public/build  # Add this for Vite output
```

#### Step 8: Install and Build

```bash
npm install
npm run build  # Production build
npm run dev    # Development server
```

**Benefits of Vite:**
- 30-50x faster dev server startup
- Hot Module Replacement (HMR)
- 93% reduction in npm packages
- Native ES modules support
- Better development experience

---

## Testing and Verification

### Step 1: Clear All Caches

```bash
php artisan config:clear
php artisan cache:clear
php artisan view:clear
php artisan route:clear
php artisan event:clear
```

### Step 2: Run Composer Diagnostics

```bash
composer diagnose
composer validate
```

### Step 3: Test Application

```bash
# Start development server
php artisan serve

# In another terminal, test frontend build
npm run dev
```

### Step 4: Run Tests

```bash
# PHPUnit
php artisan test

# Or Pest (if installed)
./vendor/bin/pest
```

### Step 5: Check Routes

```bash
php artisan route:list
```

### Step 6: Verify Middleware

```bash
php artisan route:list --columns=Method,URI,Name,Middleware
```

### Step 7: Check Database Connection

```bash
php artisan migrate:status
```

### Step 8: Test Core Features

Manual testing checklist:
- [ ] Homepage loads correctly
- [ ] Authentication works (login/register)
- [ ] Database queries execute
- [ ] File uploads work
- [ ] Email sending works
- [ ] API endpoints respond
- [ ] Scheduled tasks can be listed: `php artisan schedule:list`

---

## Post-Upgrade Tasks

### 1. Update CI/CD Pipelines

Update your CI/CD configuration to use PHP 8.2+:

```yaml
# Example: GitHub Actions
- name: Setup PHP
  uses: shivammathur/setup-php@v2
  with:
    php-version: '8.2'
```

### 2. Update Documentation

- Update README.md with new PHP requirements
- Document any breaking changes affecting your team
- Update deployment documentation

### 3. Review Deprecated Features

```bash
# Check for deprecated code
php artisan about
```

### 4. Optimize Performance

```bash
# Optimize composer autoloader
composer dump-autoload --optimize

# Cache configuration (production only)
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### 5. Update Production Environment

1. Update PHP version on production server
2. Update environment variables
3. Run migrations if needed
4. Clear and rebuild caches
5. Restart queue workers: `php artisan queue:restart`

### 6. Monitor Error Logs

```bash
tail -f storage/logs/laravel.log
```

### 7. Optional: Install Laravel Pint

Laravel Pint is now the official code style fixer:

```bash
composer require laravel/pint --dev
./vendor/bin/pint
```

### 8. Optional: Install Pest for Testing

```bash
composer require pestphp/pest --dev --with-all-dependencies
php artisan pest:install
```

---

## Troubleshooting

### Issue 1: Composer Dependency Conflicts

**Symptom**: "Your requirements could not be resolved to an installable set of packages"

**Solution**:
```bash
rm composer.lock
rm -rf vendor/
composer clear-cache
composer install
```

### Issue 2: Undefined Method Errors

**Symptom**: "Call to undefined method" errors after upgrade

**Cause**: Breaking changes in facades or helper functions

**Solution**: Check the specific version upgrade guide for method changes

### Issue 3: Storage Permission Errors

**Symptom**: "The stream or file could not be opened"

**Solution**:
```bash
chmod -R 775 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache
```

### Issue 4: Vite Build Errors

**Symptom**: "Failed to resolve entry" or module errors

**Solution**:
```bash
rm -rf node_modules package-lock.json
npm install
npm run build
```

### Issue 5: Database Migration Errors

**Symptom**: Migrations fail with unknown columns

**Solution**: Check migration order and foreign key constraints

### Issue 6: Middleware Not Working

**Symptom**: Middleware not executing after structure migration

**Solution**: Verify middleware is properly registered in `bootstrap/app.php`:
- Check global middleware in `$middleware->use([])`
- Check group middleware in `$middleware->group('web', [])`
- Check route middleware aliases in `$middleware->alias([])`

### Issue 7: Schedule Not Running

**Symptom**: Scheduled tasks not executing

**Solution**:
```bash
# Verify schedule is set up
php artisan schedule:list

# Test schedule manually
php artisan schedule:run

# Update crontab (production)
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

### Issue 8: Queue Workers Not Processing

**Symptom**: Jobs stuck in queue

**Solution**:
```bash
# Restart queue workers
php artisan queue:restart

# Check queue connection
php artisan queue:work --once
```

---

## Additional Resources

### Official Documentation
- [Laravel 12 Documentation](https://laravel.com/docs/12.x)
- [Laravel Upgrade Guide](https://laravel.com/docs/12.x/upgrade)
- [Vite Documentation](https://vitejs.dev/)

### Community Resources
- [Laravel News](https://laravel-news.com/)
- [Laracasts](https://laracasts.com/)
- [Laravel.io Forum](https://laravel.io/forum)

### Recommended Packages
- **laravel/pint**: Code style fixer
- **laravel/telescope**: Debug assistant
- **barryvdh/laravel-debugbar**: Development toolbar
- **spatie/laravel-permission**: Role/permission management
- **laravel/horizon**: Queue monitoring

---

## Version History

| Date | Version | Changes |
|------|---------|---------|
| 2025-10-27 | 1.0.0 | Initial guide - Laravel 8.75 to 12.35.0 upgrade |

---

## Summary Checklist

Use this checklist to ensure you've completed all upgrade steps:

- [ ] Updated `composer.json` with Laravel 12 dependencies
- [ ] Ran `composer update` successfully
- [ ] Updated `config/filesystems.php` (FILESYSTEM_DISK, local root path)
- [ ] Updated `config/database.php` (PostgreSQL search_path)
- [ ] Updated `config/mail.php` (removed auth_mode)
- [ ] Updated `.env` and `.env.example` files
- [ ] Created new `bootstrap/app.php` with builder pattern
- [ ] Updated `routes/console.php` with Schedule facade
- [ ] Removed old Kernel files and Handler
- [ ] Migrated to Vite (or updated Mix)
- [ ] Updated all Blade templates to use `@vite()`
- [ ] Converted JavaScript to ES modules
- [ ] Updated environment variables (MIX_ to VITE_)
- [ ] Cleared all caches
- [ ] Ran test suite
- [ ] Tested application manually
- [ ] Updated CI/CD pipelines
- [ ] Updated documentation
- [ ] Deployed to production
- [ ] Monitored error logs

---

**Congratulations!** Your Laravel application is now running on Laravel 12.

If you encounter issues not covered in this guide, refer to the official Laravel documentation or seek help in the Laravel community forums.
