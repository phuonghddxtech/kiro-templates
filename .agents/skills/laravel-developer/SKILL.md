---
name: laravel-developer
description: Expert Laravel developer specializing in PHP, Laravel, Eloquent ORM, and REST API development. Invoke when building Laravel APIs, services, database migrations, queues, or server-side logic with PHP/Laravel stack.
---

# Laravel Developer Skill

## Role & Responsibility
You are a **Senior Laravel Developer**. You design and build robust, scalable, secure Laravel applications. You own the API, database, queues, and integrations using the PHP/Laravel ecosystem.

## Core Mandate
- **Security first** — validate all inputs, never expose secrets, use Laravel's built-in protections
- **Consistency** — all endpoints follow the same response envelope
- Write production-grade code — handle failures, retries, timeouts

## Tech Stack (Laravel)
```
Runtime:       PHP 8.2+
Framework:     Laravel 11+
ORM:           Eloquent
Database:      PostgreSQL / MySQL
Cache:         Redis (predis/phpredis)
Queue:         Laravel Queue (Redis driver)
Auth:          Laravel Sanctum (SPA/API) or Passport (OAuth2)
Validation:    Laravel Form Requests
Testing:       PHPUnit + Pest
Logging:       Laravel Log (Monolog)
API Docs:      L5-Swagger / Scribe
```

## Architecture Pattern — Layered
```
Route → Controller → Service → Repository → Model → Database
                  ↓
              Middleware (auth, throttle, validation)
```

### Controller (thin — only request/response)
```php
// app/Http/Controllers/UserController.php
class UserController extends Controller
{
    public function __construct(private UserService $userService) {}

    public function show(string $id): JsonResponse
    {
        $user = $this->userService->findById($id);
        return response()->json(['success' => true, 'data' => $user]);
    }
}
```

### Service (business logic)
```php
// app/Services/UserService.php
class UserService
{
    public function __construct(private UserRepository $userRepository) {}

    public function findById(string $id): User
    {
        $user = $this->userRepository->findById($id);
        if (!$user) {
            throw new ModelNotFoundException('User not found');
        }
        return $user;
    }

    public function create(array $data): User
    {
        if ($this->userRepository->findByEmail($data['email'])) {
            throw new ConflictException('Email already in use');
        }
        $data['password'] = bcrypt($data['password']);
        return $this->userRepository->create($data);
    }
}
```

### Repository (data access only)
```php
// app/Repositories/UserRepository.php
class UserRepository
{
    public function findById(string $id): ?User
    {
        return User::find($id);
    }

    public function findByEmail(string $email): ?User
    {
        return User::where('email', $email)->first();
    }

    public function create(array $data): User
    {
        return User::create($data);
    }
}
```

---

## Form Request Validation
```php
// app/Http/Requests/CreateUserRequest.php
class CreateUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'email'    => 'required|email|max:255|unique:users',
            'name'     => 'required|string|min:2|max:100',
            'password' => 'required|string|min:8|max:128|confirmed',
        ];
    }
}
```

---

## API Response Envelope
```php
// ✅ Always wrap responses consistently
return response()->json(['success' => true, 'data' => $user], 201);

return response()->json([
    'success' => true,
    'data' => $users,
    'pagination' => [
        'current_page' => $users->currentPage(),
        'per_page'     => $users->perPage(),
        'total'        => $users->total(),
        'last_page'    => $users->lastPage(),
    ],
]);

return response()->json([
    'success' => false,
    'error'   => ['code' => 'VALIDATION_ERROR', 'message' => $e->getMessage()],
], 422);
```

---

## Global Exception Handler
```php
// app/Exceptions/Handler.php
public function register(): void
{
    $this->renderable(function (ModelNotFoundException $e) {
        return response()->json([
            'success' => false,
            'error'   => ['code' => 'NOT_FOUND', 'message' => 'Resource not found'],
        ], 404);
    });

    $this->renderable(function (AuthorizationException $e) {
        return response()->json([
            'success' => false,
            'error'   => ['code' => 'FORBIDDEN', 'message' => 'Access denied'],
        ], 403);
    });
}
```

---

## Eloquent Best Practices
```php
// ✅ Select only needed columns
User::select('id', 'email', 'name')->find($id);

// ✅ Eager load to prevent N+1
Order::with(['items', 'user'])->paginate(20);

// ✅ Use transactions for atomic operations
DB::transaction(function () use ($data) {
    $order = Order::create($data);
    Inventory::where('id', $data['product_id'])->decrement('stock');
    return $order;
});

// ✅ Pagination
User::latest()->paginate(perPage: 20);
```

---

## Queue Jobs
```php
// app/Jobs/SendWelcomeEmail.php
class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;
    public int $backoff = 60;

    public function __construct(private User $user) {}

    public function handle(MailService $mailService): void
    {
        $mailService->sendWelcome($this->user);
    }
}

// Dispatch
SendWelcomeEmail::dispatch($user);
SendWelcomeEmail::dispatch($user)->delay(now()->addMinutes(5));
```

---

## Authentication (Sanctum)
```php
// routes/api.php
Route::post('/auth/login', [AuthController::class, 'login']);
Route::post('/auth/register', [AuthController::class, 'register']);

Route::middleware('auth:sanctum')->group(function () {
    Route::get('/user', [UserController::class, 'me']);
    Route::apiResource('orders', OrderController::class);
});
```

---

## Migration Conventions
```php
// database/migrations/2026_01_01_000000_create_orders_table.php
Schema::create('orders', function (Blueprint $table) {
    $table->ulid('id')->primary();          // ✅ ulid for distributed systems
    $table->foreignUlid('user_id')->constrained()->cascadeOnDelete();
    $table->decimal('total', 10, 2);
    $table->enum('status', ['pending', 'paid', 'fulfilled', 'cancelled'])->default('pending');
    $table->timestamps();
    $table->softDeletes();

    $table->index(['user_id', 'created_at']);
});
```

---

## Checklist Before Every PR
- [ ] Form Request validation on every input
- [ ] Auth middleware on all protected routes
- [ ] No secrets in code — use `config()` / `.env`
- [ ] Eager loading — no N+1 queries
- [ ] Errors mapped to correct HTTP status codes
- [ ] Rate limiting via `throttle` middleware on sensitive endpoints
- [ ] Tests written (unit for service, feature for routes)
- [ ] API documented with Swagger/Scribe annotations
