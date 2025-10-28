# Sentry Setup Guide for Laravel 12 in Docker

This guide covers the complete setup of Sentry for error tracking and logging in a Laravel 12 application running in Docker.

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Installation Steps](#installation-steps)
4. [Configuration](#configuration)
5. [Testing Sentry Integration](#testing-sentry-integration)
6. [Usage Examples](#usage-examples)
7. [Performance Monitoring](#performance-monitoring)
8. [Best Practices](#best-practices)
9. [Troubleshooting](#troubleshooting)

---

## Overview

### What is Sentry?

Sentry is an open-source error tracking and performance monitoring platform that helps developers monitor and fix crashes in real-time. It provides:

- **Real-time Error Tracking**: Capture exceptions and errors as they happen
- **Performance Monitoring**: Track application performance and identify bottlenecks
- **Release Tracking**: Associate errors with specific releases
- **User Context**: See which users are affected by errors
- **Breadcrumbs**: Trace the path leading to an error
- **Source Maps**: View original source code for minified JavaScript

### Why Self-Hosted Sentry in Docker?

For this Laravel 12 application, we're using a self-hosted Sentry instance because:

1. **Data Privacy**: All error data stays within your infrastructure
2. **Cost-Effective**: No subscription fees for high-volume applications
3. **Easy Integration**: Runs alongside your Laravel app in the same Docker network
4. **Full Control**: Complete control over data retention and configuration
5. **Development-Friendly**: Perfect for local development and staging environments

---

## Architecture

### Docker Services

The setup includes the following Docker services:

```
┌─────────────────────────────────────────────────────┐
│                  Docker Network                      │
│                  larawell-network                    │
│                                                      │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐     │
│  │ Laravel  │───>│  Sentry  │───>│ Sentry   │     │
│  │   App    │    │  (Web)   │    │ Postgres │     │
│  └──────────┘    └──────────┘    └──────────┘     │
│                       │                              │
│                       │           ┌──────────┐      │
│                       ├──────────>│ Sentry   │      │
│                       │           │  Redis   │      │
│                       │           └──────────┘      │
│                       │                              │
│                  ┌────┴────┐                        │
│            ┌─────┤ Sentry  │                        │
│            │     │ Worker  │                        │
│            │     └─────────┘                        │
│            │                                         │
│       ┌────┴────┐                                   │
│       │ Sentry  │                                   │
│       │  Cron   │                                   │
│       └─────────┘                                   │
└─────────────────────────────────────────────────────┘
```

### Service Breakdown

| Service | Description | Port | Purpose |
|---------|-------------|------|---------|
| `sentry` | Main Sentry web interface | 9000 | Web UI and API endpoint |
| `sentry-postgres` | PostgreSQL database | - | Stores Sentry data |
| `sentry-redis` | Redis cache | - | Caching and task queue |
| `sentry-worker` | Background worker | - | Processes events asynchronously |
| `sentry-cron` | Scheduled tasks | - | Cleanup, digests, and maintenance |

---

## Installation Steps

### Step 1: Docker Compose Configuration

The [docker-compose.yml](docker-compose.yml) has been updated with Sentry services:

```yaml
# Sentry Services
sentry-redis:
  image: redis:alpine
  container_name: sentry-redis
  restart: unless-stopped
  networks:
    - larawell-network

sentry-postgres:
  image: postgres:15
  container_name: sentry-postgres
  restart: unless-stopped
  environment:
    POSTGRES_USER: sentry
    POSTGRES_PASSWORD: sentry
    POSTGRES_DB: sentry
  volumes:
    - sentry-postgres:/var/lib/postgresql/data
  networks:
    - larawell-network

sentry:
  image: sentry:latest
  container_name: sentry
  restart: unless-stopped
  ports:
    - "9000:9000"
  environment:
    SENTRY_SECRET_KEY: 'changeme-please-generate-a-secret-key'
    SENTRY_POSTGRES_HOST: sentry-postgres
    SENTRY_DB_USER: sentry
    SENTRY_DB_PASSWORD: sentry
    SENTRY_REDIS_HOST: sentry-redis
  depends_on:
    - sentry-redis
    - sentry-postgres
  networks:
    - larawell-network
  volumes:
    - sentry-data:/var/lib/sentry/files

sentry-cron:
  image: sentry:latest
  container_name: sentry-cron
  restart: unless-stopped
  command: run cron
  environment:
    SENTRY_SECRET_KEY: 'changeme-please-generate-a-secret-key'
    SENTRY_POSTGRES_HOST: sentry-postgres
    SENTRY_DB_USER: sentry
    SENTRY_DB_PASSWORD: sentry
    SENTRY_REDIS_HOST: sentry-redis
  depends_on:
    - sentry-redis
    - sentry-postgres
  networks:
    - larawell-network
  volumes:
    - sentry-data:/var/lib/sentry/files

sentry-worker:
  image: sentry:latest
  container_name: sentry-worker
  restart: unless-stopped
  command: run worker
  environment:
    SENTRY_SECRET_KEY: 'changeme-please-generate-a-secret-key'
    SENTRY_POSTGRES_HOST: sentry-postgres
    SENTRY_DB_USER: sentry
    SENTRY_DB_PASSWORD: sentry
    SENTRY_REDIS_HOST: sentry-redis
  depends_on:
    - sentry-redis
    - sentry-postgres
  networks:
    - larawell-network
  volumes:
    - sentry-data:/var/lib/sentry/files
```

**Volumes:**
```yaml
volumes:
  sentry-postgres:
    driver: local
  sentry-data:
    driver: local
```

### Step 2: Generate Sentry Secret Key

Before starting the containers, generate a secure secret key:

```bash
# Generate a random secret key
openssl rand -base64 50
```

Replace `'changeme-please-generate-a-secret-key'` in all Sentry service environment variables with your generated key.

### Step 3: Initialize Sentry Database

```bash
# Start only the database services first
docker-compose up -d sentry-postgres sentry-redis

# Wait for PostgreSQL to be ready
sleep 5

# Run Sentry upgrade to initialize the database
docker-compose run --rm sentry upgrade

# This will:
# 1. Create database tables
# 2. Run migrations
# 3. Prompt you to create a superuser account
```

**Important**: When prompted, create a superuser account for accessing the Sentry web interface.

### Step 4: Start All Services

```bash
# Start all services
docker-compose up -d

# Verify all services are running
docker-compose ps
```

You should see all services in "Up" state.

### Step 5: Access Sentry Web Interface

Open your browser and navigate to:

```
http://localhost:9000
```

Log in with the superuser credentials you created in Step 3.

### Step 6: Create a Project in Sentry

1. Click **"Create Project"**
2. Select **"Laravel"** as the platform
3. Enter a project name (e.g., "larawell-app")
4. Click **"Create Project"**
5. Copy the **DSN (Data Source Name)** - it will look like:
   ```
   http://public_key@sentry:9000/1
   ```

### Step 7: Install Laravel Sentry SDK

```bash
# Install the Sentry Laravel package
composer require sentry/sentry-laravel

# Publish the configuration file
php artisan vendor:publish --provider="Sentry\Laravel\ServiceProvider"
```

### Step 8: Configure Laravel Environment

Update your `.env` file:

```bash
# Sentry Configuration
SENTRY_LARAVEL_DSN=http://public_key@sentry:9000/1
SENTRY_TRACES_SAMPLE_RATE=1.0
SENTRY_PROFILES_SAMPLE_RATE=1.0
```

**Configuration Details:**

- `SENTRY_LARAVEL_DSN`: The DSN from your Sentry project (Step 6)
- `SENTRY_TRACES_SAMPLE_RATE`: Percentage of transactions to track (1.0 = 100%)
- `SENTRY_PROFILES_SAMPLE_RATE`: Percentage of profiles to collect (1.0 = 100%)

**Note**: In production, you may want to reduce these rates to avoid performance overhead:
```bash
SENTRY_TRACES_SAMPLE_RATE=0.1  # 10%
SENTRY_PROFILES_SAMPLE_RATE=0.1  # 10%
```

---

## Configuration

### Laravel Integration

The [bootstrap/app.php](bootstrap/app.php#L76-L81) has been configured to integrate Sentry with exception handling:

```php
->withExceptions(function (Exceptions $exceptions) {
    // Don't flash these input fields on validation exceptions
    $exceptions->dontFlash([
        'current_password',
        'password',
        'password_confirmation',
    ]);

    // Integrate Sentry for exception tracking
    $exceptions->reportable(function (Throwable $e) {
        if (app()->bound('sentry')) {
            app('sentry')->captureException($e);
        }
    });
})
```

### Sentry Configuration File

The configuration file is located at [config/sentry.php](config/sentry.php). Key settings:

```php
return [
    'dsn' => env('SENTRY_LARAVEL_DSN'),

    // Capture specific environments
    'environment' => env('APP_ENV', 'production'),

    // Release tracking
    'release' => env('SENTRY_RELEASE'),

    // Breadcrumbs
    'breadcrumbs' => [
        'logs' => true,
        'cache' => true,
        'livewire' => true,
        'sql_queries' => true,
        'sql_bindings' => true,
        'queue_info' => true,
        'command_info' => true,
    ],

    // Tracing
    'traces_sample_rate' => env('SENTRY_TRACES_SAMPLE_RATE', 0.0),
    'profiles_sample_rate' => env('SENTRY_PROFILES_SAMPLE_RATE', 0.0),

    // Send default PII (Personally Identifiable Information)
    'send_default_pii' => false,
];
```

### Environment-Specific Configuration

**Development (.env.local):**
```bash
SENTRY_LARAVEL_DSN=http://public_key@sentry:9000/1
SENTRY_TRACES_SAMPLE_RATE=1.0
SENTRY_PROFILES_SAMPLE_RATE=1.0
APP_ENV=local
```

**Production (.env.production):**
```bash
SENTRY_LARAVEL_DSN=https://public_key@your-production-sentry.com/1
SENTRY_TRACES_SAMPLE_RATE=0.1
SENTRY_PROFILES_SAMPLE_RATE=0.1
SENTRY_RELEASE=v1.0.0
APP_ENV=production
```

---

## Testing Sentry Integration

### Test 1: Manual Exception

Create a test route to verify Sentry is capturing exceptions:

**routes/web.php:**
```php
Route::get('/test-sentry', function () {
    throw new \Exception('Testing Sentry integration!');
});
```

**Test:**
```bash
# Visit the route
curl http://localhost:8000/test-sentry

# Check Sentry dashboard at http://localhost:9000
# You should see the exception appear
```

### Test 2: Artisan Command Test

```bash
php artisan tinker

# In Tinker, throw an exception
throw new Exception('Testing Sentry from Tinker');
```

### Test 3: Manual Capture

```php
// In any controller or service
use Sentry\Laravel\Facade as Sentry;

try {
    // Some risky operation
    $result = riskyOperation();
} catch (\Exception $e) {
    // Manually capture exception
    Sentry::captureException($e);

    // Continue with error handling
    return response()->json(['error' => 'Something went wrong'], 500);
}
```

### Test 4: Capture Custom Messages

```php
use Sentry\Laravel\Facade as Sentry;

// Log an informational message
Sentry::captureMessage('User performed important action', 'info');

// Log a warning
Sentry::captureMessage('Unusual activity detected', 'warning');
```

---

## Usage Examples

### Capturing Exceptions in Controllers

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Sentry\Laravel\Facade as Sentry;

class OrderController extends Controller
{
    public function store(Request $request)
    {
        try {
            $order = Order::create($request->validated());

            // Process payment
            $payment = $this->processPayment($order);

            return response()->json(['order' => $order], 201);

        } catch (\PaymentException $e) {
            // Capture payment-specific exceptions
            Sentry::captureException($e);

            return response()->json([
                'error' => 'Payment failed'
            ], 400);
        }
    }
}
```

### Adding Context to Errors

```php
use Sentry\Laravel\Facade as Sentry;

// Set user context
Sentry::configureScope(function ($scope) use ($user) {
    $scope->setUser([
        'id' => $user->id,
        'email' => $user->email,
        'username' => $user->name,
    ]);
});

// Add breadcrumbs
Sentry::addBreadcrumb([
    'message' => 'User initiated checkout',
    'category' => 'user.action',
    'level' => 'info',
]);

// Set tags
Sentry::configureScope(function ($scope) {
    $scope->setTag('feature', 'checkout');
    $scope->setTag('plan', 'premium');
});

// Set extra context
Sentry::configureScope(function ($scope) use ($order) {
    $scope->setContext('order', [
        'id' => $order->id,
        'total' => $order->total,
        'items' => $order->items_count,
    ]);
});
```

### Middleware for User Context

Create a middleware to automatically set user context:

**app/Http/Middleware/SentryContext.php:**
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Sentry\Laravel\Facade as Sentry;

class SentryContext
{
    public function handle(Request $request, Closure $next)
    {
        if (auth()->check()) {
            Sentry::configureScope(function ($scope) {
                $scope->setUser([
                    'id' => auth()->id(),
                    'email' => auth()->user()->email,
                    'username' => auth()->user()->name,
                ]);
            });
        }

        return $next($request);
    }
}
```

**Register in bootstrap/app.php:**
```php
$middleware->use([
    \App\Http\Middleware\SentryContext::class,
    // ... other middleware
]);
```

### Queue Job Monitoring

```php
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Sentry\Laravel\Facade as Sentry;

class ProcessOrder implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public Order $order
    ) {}

    public function handle()
    {
        // Set job context
        Sentry::configureScope(function ($scope) {
            $scope->setTag('job', 'process_order');
            $scope->setContext('order', [
                'id' => $this->order->id,
            ]);
        });

        try {
            // Process the order
            $this->order->process();
        } catch (\Exception $e) {
            // Exception automatically captured by Laravel
            throw $e;
        }
    }
}
```

---

## Performance Monitoring

### Transaction Tracking

Sentry can track performance of specific operations:

```php
use Sentry\Laravel\Facade as Sentry;

// Start a transaction
$transaction = Sentry::startTransaction([
    'op' => 'http.request',
    'name' => 'Process Order',
]);

// Set the current transaction
Sentry::getCurrentHub()->setSpan($transaction);

try {
    // Your code here
    $order = $this->processOrder($request);

    // Create child spans for specific operations
    $span = $transaction->startChild([
        'op' => 'db.query',
        'description' => 'Fetch user data',
    ]);

    $user = User::find($order->user_id);

    $span->finish();

    // Set transaction status
    $transaction->setStatus('ok');

} catch (\Exception $e) {
    $transaction->setStatus('internal_error');
    throw $e;
} finally {
    $transaction->finish();
}
```

### Database Query Monitoring

Sentry automatically tracks database queries when enabled in config:

```php
// config/sentry.php
'breadcrumbs' => [
    'sql_queries' => true,
    'sql_bindings' => true,
],
```

### HTTP Client Monitoring

Monitor outgoing HTTP requests:

```php
use Illuminate\Support\Facades\Http;
use Sentry\Laravel\Facade as Sentry;

$span = Sentry::getCurrentHub()->getSpan();

if ($span) {
    $childSpan = $span->startChild([
        'op' => 'http.client',
        'description' => 'POST https://api.example.com/endpoint',
    ]);
}

try {
    $response = Http::post('https://api.example.com/endpoint', $data);

    if ($childSpan) {
        $childSpan->setData([
            'status_code' => $response->status(),
            'response_size' => strlen($response->body()),
        ]);
    }
} finally {
    if ($childSpan) {
        $childSpan->finish();
    }
}
```

---

## Best Practices

### 1. Don't Log Sensitive Data

```php
// config/sentry.php
'send_default_pii' => false,

// In your code, scrub sensitive data
Sentry::configureScope(function ($scope) use ($data) {
    // Remove sensitive fields
    unset($data['password'], $data['credit_card']);

    $scope->setContext('request_data', $data);
});
```

### 2. Use Environment-Specific Configuration

```php
// Only enable Sentry in specific environments
if (app()->environment(['production', 'staging'])) {
    // Sentry is active
}

// Or in config/sentry.php
'dsn' => env('APP_ENV') === 'local' ? null : env('SENTRY_LARAVEL_DSN'),
```

### 3. Sample Rates for Production

```bash
# Production .env
SENTRY_TRACES_SAMPLE_RATE=0.1  # Track 10% of transactions
SENTRY_PROFILES_SAMPLE_RATE=0.1  # Profile 10% of transactions
```

### 4. Ignore Expected Exceptions

```php
// config/sentry.php
'ignore_exceptions' => [
    Illuminate\Auth\AuthenticationException::class,
    Illuminate\Validation\ValidationException::class,
],
```

### 5. Release Tracking

Use releases to track which version of your app has errors:

```bash
# .env
SENTRY_RELEASE=v1.2.3

# Or use git commit hash
SENTRY_RELEASE=$(git rev-parse --short HEAD)
```

### 6. Tag Your Errors

```php
Sentry::configureScope(function ($scope) {
    $scope->setTag('feature', 'checkout');
    $scope->setTag('version', config('app.version'));
    $scope->setTag('tenant', auth()->user()->tenant_id);
});
```

### 7. Use Breadcrumbs for Context

```php
// Log user actions leading to an error
Sentry::addBreadcrumb([
    'message' => 'User clicked checkout button',
    'category' => 'user.click',
    'level' => 'info',
]);

Sentry::addBreadcrumb([
    'message' => 'Payment form validated',
    'category' => 'validation',
    'level' => 'info',
]);

// When error occurs, you'll see the full path
```

### 8. Monitor Queue Workers

```bash
# Restart queue workers after Sentry configuration changes
php artisan queue:restart

# Check queue worker logs
docker-compose logs -f app
```

---

## Troubleshooting

### Issue 1: Sentry Not Capturing Exceptions

**Symptoms**: No errors appearing in Sentry dashboard

**Solutions**:

1. **Check DSN Configuration**:
   ```bash
   php artisan tinker
   >>> config('sentry.dsn')
   # Should output your DSN
   ```

2. **Verify Sentry Service is Running**:
   ```bash
   docker-compose ps sentry
   curl http://localhost:9000
   ```

3. **Test Network Connectivity**:
   ```bash
   # From Laravel container
   docker-compose exec app ping sentry
   docker-compose exec app curl http://sentry:9000
   ```

4. **Check Laravel Logs**:
   ```bash
   tail -f storage/logs/laravel.log
   ```

5. **Enable Debug Mode Temporarily**:
   ```bash
   # .env
   APP_DEBUG=true
   ```

### Issue 2: Sentry Service Won't Start

**Symptoms**: `docker-compose up` fails for Sentry

**Solutions**:

1. **Check Secret Key**:
   - Ensure `SENTRY_SECRET_KEY` is properly set and identical across all Sentry services

2. **Verify Database Initialization**:
   ```bash
   # Reinitialize if needed
   docker-compose down -v
   docker-compose up -d sentry-postgres sentry-redis
   docker-compose run --rm sentry upgrade
   docker-compose up -d
   ```

3. **Check Logs**:
   ```bash
   docker-compose logs sentry
   docker-compose logs sentry-postgres
   ```

### Issue 3: High Memory Usage

**Symptoms**: Sentry consuming too much memory

**Solutions**:

1. **Reduce Sample Rates**:
   ```bash
   SENTRY_TRACES_SAMPLE_RATE=0.05  # 5%
   SENTRY_PROFILES_SAMPLE_RATE=0.05
   ```

2. **Limit Data Retention**:
   - Log into Sentry web interface
   - Go to Settings → Data Retention
   - Set shorter retention periods

3. **Increase Docker Memory Limits**:
   ```yaml
   # docker-compose.yml
   sentry:
     # ...
     deploy:
       resources:
         limits:
           memory: 2G
   ```

### Issue 4: Duplicate Exceptions

**Symptoms**: Same exception appearing multiple times

**Solutions**:

1. **Check for Multiple Handlers**:
   - Ensure you're not capturing exceptions in multiple places
   - Remove manual `captureException()` calls if exceptions are auto-captured

2. **Configure Grouping**:
   - Sentry web interface → Project Settings → Issue Grouping
   - Adjust grouping rules

### Issue 5: Missing User Context

**Symptoms**: User information not appearing in error reports

**Solutions**:

1. **Ensure Middleware is Registered**:
   ```php
   // bootstrap/app.php
   $middleware->use([
       \App\Http\Middleware\SentryContext::class,
   ]);
   ```

2. **Check Authentication**:
   ```php
   // Test if auth is working
   Route::get('/test', function () {
       dd(auth()->check(), auth()->user());
   });
   ```

### Issue 6: Performance Impact

**Symptoms**: Application slowing down after Sentry installation

**Solutions**:

1. **Reduce Sample Rates**:
   ```bash
   SENTRY_TRACES_SAMPLE_RATE=0.1
   ```

2. **Disable SQL Query Tracking**:
   ```php
   // config/sentry.php
   'breadcrumbs' => [
       'sql_queries' => false,
       'sql_bindings' => false,
   ],
   ```

3. **Use Async Transport** (for production):
   ```bash
   composer require sentry/sdk
   ```

---

## Additional Resources

### Official Documentation
- [Sentry Laravel Documentation](https://docs.sentry.io/platforms/php/guides/laravel/)
- [Sentry Self-Hosted Guide](https://develop.sentry.dev/self-hosted/)
- [Sentry PHP SDK](https://docs.sentry.io/platforms/php/)

### Useful Commands

```bash
# View Sentry logs
docker-compose logs -f sentry

# Restart Sentry services
docker-compose restart sentry sentry-worker sentry-cron

# Clean Sentry data and restart
docker-compose down
docker volume rm upgrade-app_sentry-data upgrade-app_sentry-postgres
docker-compose up -d

# Test Sentry DSN
php artisan tinker
>>> Sentry\Laravel\Facade::captureMessage('Test message');

# Check Sentry configuration
php artisan config:show sentry

# Clear Laravel caches
php artisan config:clear
php artisan cache:clear
```

### Project Structure

```
.
├── docker-compose.yml        # Sentry services configuration
├── config/
│   └── sentry.php           # Sentry configuration file
├── bootstrap/
│   └── app.php              # Sentry exception handling integration
├── .env.example             # Sentry environment variables template
└── SentrySetup.md           # This documentation file
```

---

## Summary Checklist

Use this checklist to verify your Sentry setup:

- [ ] Generated Sentry secret key
- [ ] Updated docker-compose.yml with secret key
- [ ] Started Sentry database services
- [ ] Ran `sentry upgrade` to initialize database
- [ ] Created Sentry superuser account
- [ ] Started all Docker services
- [ ] Accessed Sentry web interface (http://localhost:9000)
- [ ] Created Laravel project in Sentry
- [ ] Copied project DSN
- [ ] Installed `sentry/sentry-laravel` via Composer
- [ ] Published Sentry configuration
- [ ] Updated .env with SENTRY_LARAVEL_DSN
- [ ] Configured exception handling in bootstrap/app.php
- [ ] Tested exception capture
- [ ] Verified errors appear in Sentry dashboard
- [ ] Configured sample rates appropriately
- [ ] Added user context middleware (optional)
- [ ] Configured breadcrumbs (optional)
- [ ] Set up release tracking (optional)

---

**Congratulations!** Sentry is now fully integrated with your Laravel 12 application running in Docker.

All exceptions will be automatically tracked, and you can access the Sentry dashboard at `http://localhost:9000` to view errors, performance metrics, and user context.
