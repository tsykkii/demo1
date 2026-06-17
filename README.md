
# Демо-экзамен КОД 09.02.07-5-2026

## Содержание
- [Установка Laravel](#установка-laravel)
- [База данных](#база-данных)
- [Миграции](#миграции)
- [Модели](#модели)
- [Контроллеры](#контроллеры)
- [Маршруты](#маршруты)
- [Views (Blade)](#views-blade)
- [ 6 модуль ](#6-модуль)

---

## Установка Laravel

```bash
composer create-project laravel/laravel --prefer-dist .
```

---

## МОДУЛЬ 1
### Таблицы в phpMyAdmin

```sql
CREATE TABLE customers (
    customer_id VARCHAR(9) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    inn VARCHAR(20),
    address VARCHAR(255),
    phone VARCHAR(20),
    salesman BOOLEAN,
    buyer BOOLEAN
);

CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    unit VARCHAR(20),
    price DECIMAL(10,2)
);

CREATE TABLE materials (
    material_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    unit VARCHAR(20),
    price DECIMAL(10,2)
);

CREATE TABLE specifications (
    specification_id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    material_id INT NOT NULL,
    quantity DECIMAL(10,3),

    FOREIGN KEY (product_id)
        REFERENCES products(product_id),

    FOREIGN KEY (material_id)
        REFERENCES materials(material_id)
);

CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    order_date DATE,
    customer_id VARCHAR(9),

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    item_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT,

    FOREIGN KEY (order_id)
        REFERENCES orders(order_id),

    FOREIGN KEY (product_id)
        REFERENCES products(product_id)
);
```

---


## МОДУЛЬ 2
### Заполнение этих таблиц

```
CUSTOMERS. Импортировать данные из файла "Заказчики.json":
```

```sql
INSERT INTO customers VALUES
('000000001','ООО "Поставка"','','г.Пятигорск','+79198634592',1,1),

('000000002','ООО "Кинотеатр Квант"','26320045123','г. Железноводск, ул. Мира, 123','+79884581555',1,0),

('000000008','ООО "Новый JDTO"','26320045111','г. Железноводск','+79884581555',1,0),

('000000003','ООО "Ромашка"','4140784214','г. Омск, ул. Строителей, 294','+79882584546',0,1),

('000000009','ООО "Ипподром"','5874045632','г. Уфа, ул. Набережная, 37','+79627486389',1,1),

('000000010','ООО "Ассоль"','2629011278','г. Калуга, ул. Пушкина, 94','+79184572398',0,1);
```

```
Добавить продукцию (products). Импортировать данные из файла "Цены.xlsx":
```

```sql
INSERT INTO products(name,unit,price)
VALUES
('Кефир 2,5% 900г.','шт',80),
('Кефир 3,2% 900г.','шт',82),
('Молоко 2,5% 900г.','шт',70),
('Молоко 3,2% 900г.','шт',76),
('Сметана классическая 15% 540г.','шт',89),
('Сметана классическая 20% 540г.','шт',92);
```

```
Добавить материалы (materials). Импортировать данные из файла "Расчет стоимости продукции.xlsx":
```

```sql
INSERT INTO materials(name,unit,price)
VALUES
('Молоко нормализованное','кг',34),
('Закваска сметанная','кг',45);
```

```
Заполнить спецификацию (specifications)-(Сметана 15% - молоко 0,9кг и закваска 0,07кг):
```

```sql
INSERT INTO specifications(product_id,material_id,quantity)
VALUES
(5,1,0.9),
(5,2,0.07);
```

```
Создать заказ (orders) Из "Заказ покупателя":
```

```sql
INSERT INTO orders(order_date,customer_id)
VALUES ('2025-06-06','000000010');
```

```
Заполнить состав заказа (order_items):
```

```sql
INSERT INTO order_items(order_id,product_id,quantity)
VALUES
(1,1,12),
(1,2,9),
(1,3,10);
```


## МОДУЛЬ 3
### Полная себестоимость заказа

```
order_items → specifications → materials
```

```sql
SELECT *
FROM order_items oi
JOIN specifications s
ON oi.product_id = s.product_id;
```

---


## МОДУЛЬ 5
### Документация сайта


---

```
1. Функциональное назначение

Назначение системы

Система предназначена для авторизации зарегистрированных пользователей и разграничения прав доступа в зависимости от роли пользователя.

Предусмотрены две роли:

Администратор;
Пользователь.

Администратор обладает дополнительными возможностями по управлению учетными записями пользователей, включая создание новых пользователей, изменение данных существующих пользователей и снятие блокировки.

Для защиты от автоматического подбора паролей реализована интерактивная капча в виде пазла. При трех неудачных попытках авторизации или неправильной сборке изображения учетная запись пользователя блокируется.
```


### 2. Описание таблицы Users

| Поле | Описание |
|---|---|
| user_id | Идентификатор пользователя |
| login | Логин |
| password | Пароль |
| role | Роль пользователя |
| blocked | Признак блокировки |



---



## Миграции

### create_users_table

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->boolean('need_to_change_password')->default(true);
            $table->unsignedTinyInteger('wrong_counter')->default(0);
            $table->boolean('is_admin')->default(false);
            $table->timestamps();
        });

        Schema::create('password_reset_tokens', function (Blueprint $table) {
            $table->string('email')->primary();
            $table->string('token');
            $table->timestamp('created_at')->nullable();
        });

        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary();
            $table->foreignId('user_id')->nullable()->index();
            $table->string('ip_address', 45)->nullable();
            $table->text('user_agent')->nullable();
            $table->longText('payload');
            $table->integer('last_activity')->index();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
        Schema::dropIfExists('password_reset_tokens');
        Schema::dropIfExists('sessions');
    }
};
```

---

## Модели

### User.php

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
        'need_to_change_password',
        'wrong_counter',
        'is_admin',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at'        => 'datetime',
            'password'                 => 'hashed',
            'need_to_change_password'  => 'boolean',
        ];
    }
}
```

---

## Контроллеры

### Создание контроллеров

```bash
php artisan make:controller ActionsController
php artisan make:controller ViewsController
```

### ActionsController.php

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class ActionsController extends Controller
{
    public function login(Request $request)
    {
        $request->validate([
            'email'    => 'required|email',
            'password' => 'required|string|min:6',
        ]);

        if (Auth::attempt($request->only(['email', 'password']))) {
            $user = Auth::user();

            if ($user->wrong_counter === 3) {
                Auth::logout();
                return redirect()
                    ->route('login')
                    ->withErrors(['email' => 'Your account is locked. Please contact support.']);
            }

            if ($user->need_to_change_password) {
                return redirect()
                    ->route('change_password')
                    ->with('status', 'You need to change your password before proceeding.');
            }

            return redirect()->route('dashboard');
        }

        if ($user = User::firstWhere('email', $request->input('email'))) {
            if ($user->wrong_counter != 3) {
                $user->wrong_counter++;
                $user->save();
            }

            if ($user->wrong_counter === 3) {
                return redirect()
                    ->route('login')
                    ->withErrors(['email' => 'Your account is locked. Please contact support.']);
            }
        }

        return redirect()
            ->route('login')
            ->withErrors(['email' => 'Invalid credentials. Please try again.'])
            ->withInput($request->only('email'));
    }

    public function changePassword(Request $request)
    {
        $request->validate([
            'current_password' => 'required|string|min:6|current_password',
            'new_password'     => 'required|string|min:6|confirmed:new_password_repeat',
        ]);

        $user = Auth::user();
        $user->password = $request->input('new_password');
        $user->need_to_change_password = false;
        $user->save();

        return redirect()
            ->route('dashboard')
            ->with('status', 'Password changed successfully.');
    }

    public function editUser(User $user, Request $request)
    {
        $request->validate([
            'name'     => 'required|string|max:255',
            'email'    => "required|email|unique:users,email,{$user->id}",
            'password' => 'nullable|string|min:6|confirmed',
        ]);

        $user->name  = $request->input('name');
        $user->email = $request->input('email');

        if ($request->filled('password')) {
            $user->password = $request->input('password');
        }

        $user->save();

        return redirect()
            ->route('dashboard')
            ->with('status', 'User updated successfully.');
    }

    public function createUser(Request $request)
    {
        $request->validate([
            'name'     => 'required|string|max:255',
            'email'    => 'required|email|unique:users,email',
            'password' => 'required|string|min:6|confirmed',
        ]);

        $user = new User();
        $user->name     = $request->input('name');
        $user->email    = $request->input('email');
        $user->password = $request->input('password');
        $user->save();

        return redirect()
            ->route('dashboard')
            ->with('status', 'User created successfully.');
    }

    public function deleteUser(User $user)
    {
        if ($user->id === Auth::id()) {
            return redirect()
                ->route('dashboard')
                ->withErrors(['user' => 'You cannot delete your own account.']);
        }

        $user->delete();

        return redirect()
            ->route('dashboard')
            ->with('status', 'User deleted successfully.');
    }

    public function logout(Request $request)
    {
        Auth::logout();
        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return redirect()->route('login');
    }

    public function unlockUser(User $user)
    {
        if ($user->wrong_counter !== 3) {
            $user->wrong_counter = 3;
            $user->save();

            return redirect()
                ->route('dashboard')
                ->with('status', 'User locked successfully.');
        }

        $user->wrong_counter = 0;
        $user->save();

        return redirect()
            ->route('dashboard')
            ->with('status', 'User unlocked successfully.');
    }
}
```

### ViewsController.php

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class ViewsController extends Controller
{
    public function index()
    {
        return view('index', [
            'users' => User::all(),
        ]);
    }

    public function login()
    {
        return view('login');
    }

    public function changePassword()
    {
        return view('change_password');
    }

    public function editUser(?User $user = null)
    {
        return view('user_form', [
            'user' => $user,
        ]);
    }
}
```

---

## Маршруты

### web.php

```php
<?php

use Illuminate\Support\Facades\Route;

Route::get('/', [App\Http\Controllers\ViewsController::class, 'index'])
    ->name('dashboard')
    ->middleware(['auth']);

Route::get('/login', [App\Http\Controllers\ViewsController::class, 'login'])
    ->name('login')
    ->middleware(['guest']);

Route::post('/login', [App\Http\Controllers\ActionsController::class, 'login'])
    ->middleware(['guest']);

Route::post('/logout', [App\Http\Controllers\ActionsController::class, 'logout'])
    ->name('logout')
    ->middleware(['auth']);

Route::get('/login/change-password', [App\Http\Controllers\ViewsController::class, 'changePassword'])
    ->name('change_password')
    ->middleware(['auth']);

Route::post('/login/change-password', [App\Http\Controllers\ActionsController::class, 'changePassword'])
    ->name('login.change-password')
    ->middleware(['auth']);

Route::get('/user', [App\Http\Controllers\ViewsController::class, 'editUser'])
    ->name('user.create')
    ->middleware(['auth']);

Route::get('/user/{user}', [App\Http\Controllers\ViewsController::class, 'editUser'])
    ->name('user')
    ->middleware(['auth']);

Route::delete('/user/{user}', [App\Http\Controllers\ActionsController::class, 'deleteUser'])
    ->name('user.delete')
    ->middleware(['auth']);

Route::patch('/user/{user}', [App\Http\Controllers\ActionsController::class, 'unlockUser'])
    ->name('user.unlock')
    ->middleware(['auth']);

Route::post('/user', [App\Http\Controllers\ActionsController::class, 'createUser'])
    ->name('user.save')
    ->middleware(['auth']);

Route::put('/user/{user}', [App\Http\Controllers\ActionsController::class, 'editUser'])
    ->name('user.update')
    ->middleware(['auth']);
```

---

## Views (Blade)

### login.blade.php

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login</title>
</head>
<body>
    <form action="{{ route('login') }}" method="POST">
        @csrf
        <div>
            <label for="email">Email:</label>
            <input type="email" id="email" name="email" value="{{ old('email') }}" required>
            @error('email')
                <div class="error">{{ $message }}</div>
            @enderror
        </div>
        <div>
            <label for="password">Password:</label>
            <input type="password" id="password" name="password" required>
            @error('password')
                <div class="error">{{ $message }}</div>
            @enderror
        </div>
        <button type="submit">Login</button>
    </form>
</body>
</html>
```

### change_password.blade.php

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Change Password</title>
</head>
<body>
    <form action="{{ route('login.change-password') }}" method="POST">
        @csrf
        <div>
            <label for="current_password">Current Password:</label>
            <input type="password" id="current_password" name="current_password" required>
            @error('current_password')
                <div class="error">{{ $message }}</div>
            @enderror
        </div>
        <div>
            <label for="new_password">New Password:</label>
            <input type="password" id="new_password" name="new_password" required>
            @error('new_password')
                <div class="error">{{ $message }}</div>
            @enderror
        </div>
        <div>
            <label for="new_password_repeat">Repeat Password:</label>
            <input type="password" id="new_password_repeat" name="new_password_repeat" required>
        </div>
        <button type="submit">Change Password</button>
    </form>
</body>
</html>
```

### index.blade.php

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard</title>
</head>
<body>
    <h1>Welcome to Dashboard</h1>

    <form action="{{ route('logout') }}" method="POST" style="display:inline; float:right;">
        @csrf
        <button type="submit">Logout</button>
    </form>

    <h2>List of Users</h2>
    <table border="1">
        <thead>
            <tr>
                <th>Name</th>
                <th>Email</th>
                <th>Need to change password</th>
                <th>User is blocked</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($users as $user)
                <tr>
                    <td>{{ $user->name }}</td>
                    <td>{{ $user->email }}</td>
                    <td>{{ $user->need_to_change_password ? 'Yes' : 'No' }}</td>
                    <td>{{ $user->wrong_counter === 3 ? 'Yes' : 'No' }}</td>
                    <td>
                        <a href="{{ route('user', $user->id) }}">Edit</a>

                        <form action="{{ route('user.delete', $user->id) }}" method="POST" style="display:inline;">
                            @csrf
                            @method('DELETE')
                            <button type="submit">Delete</button>
                        </form>

                        <form action="{{ route('user.unlock', $user->id) }}" method="POST" style="display:inline;">
                            @csrf
                            @method('PATCH')
                            <button type="submit">
                                {{ $user->wrong_counter === 3 ? 'Unblock' : 'Block' }}
                            </button>
                        </form>
                    </td>
                </tr>
            @endforeach
        </tbody>
        <tfoot>
            <tr>
                <td colspan="5">
                    <a href="{{ route('user.create') }}">Create new user</a>
                </td>
            </tr>
        </tfoot>
    </table>

    @if (session('status'))
        <div>
            <strong>{{ session('status') }}</strong>
        </div>
    @endif
</body>
</html>
```

### user_form.blade.php

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $user ? 'Edit User' : 'Create User' }}</title>
</head>
<body>
    <h1>{{ $user ? 'Edit User' : 'Create User' }}</h1>

    <form action="{{ $user ? route('user.update', $user->id) : route('user.save') }}" method="POST">
        @csrf
        @if ($user)
            @method('PUT')
        @endif

        <div>
            <label for="name">Name:</label>
            <input type="text" id="name" name="name" value="{{ $user->name ?? '' }}" required>
        </div>
        <div>
            <label for="email">Email:</label>
            <input type="email" id="email" name="email" value="{{ $user->email ?? '' }}" required>
        </div>
        <div>
            <label for="password">Password:</label>
            <input type="password" id="password" name="password" {{ $user ? '' : 'required' }}>
        </div>
        <div>
            <label for="password_confirmation">Confirm Password:</label>
            <input type="password" id="password_confirmation" name="password_confirmation" {{ $user ? '' : 'required' }}>
        </div>

        <button type="submit">{{ $user ? 'Update' : 'Create' }}</button>
        <a href="{{ route('dashboard') }}">Cancel</a>
    </form>

    @if ($errors->any())
        <div>
            <h2>Errors:</h2>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif
</body>
</html>
```

---

```bash
php artisan migrate:refresh
```

---

### php artisan tinker

```bash
App\Models\User::create(['email' => 'ivanov.ivan@gmail.com' , 'password' => '12345678' , 'need_to_change_password' => true , 'name' => 'Ivanov Ivan']);
```

---

### 4 МОДУЛЬ за 3 команды

```bash
composer create-project laravel/laravel demo
```
```bash
composer require laravel/ui
```
```bash
php artisan ui bootstrap --auth
```
Открой файл: resources/views/layouts/app.blade.php и Удалить

```bash
@vite(['resources/sass/app.scss', 'resources/js/app.js'])
```

---

## 6 модуль

```
composer create-project laravel/laravel --prefer-dist .
```
### Контроллеры
```
php artisan make:controller TestCaseController
Удалить welcome.blade.php, создать index.blade.php
```
### TestCaseController.php:
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TestCaseController extends Controller
{
    public function show(Request $request)
    {
        return view('index');
    }

    public function getData()
    {
        $data = file_get_contents('http://localhost:8080/api/fullName');

        if ($data) {
            $data = json_decode($data)->value;
        }

        return redirect()->route('test-case')->with('value', $data);
    }

    public function checkData(Request $request)
    {
        $value = $request->input('value');

        if (preg_match('/^[А-Яа-яЁё]+ [А-Яа-яЁё]+ [А-Яа-яЁё]+$/u', $value)) {
            return redirect()->route('test-case')
                ->with('value', $value)
                ->with('message', 'ФИО корректно');
        } else {
            return redirect()->route('test-case')
                ->with('value', $value)
                ->with('message', 'ФИО содержит некорректные символы');
        }
    }
}
```

### index.blade.php:
```
<p>
    <form action="{{ route('test-case.get') }}">
        <button type="submit">Получить данные</button>
        <span>{{ session('value') }}</span>
    </form>
</p>

<p>
    <form method="POST" action="{{ route('test-case.check') }}">
        @csrf
        <input type="hidden" name="value" value="{{ session('value') }}">
        <button type="submit">Отправить результаты теста</button>
        <span>{{ session('message') }}</span>
    </form>
</p>
```
### web.php:
```
<?php

use App\Http\Controllers\TestCaseController;
use Illuminate\Support\Facades\Route;

Route::get('/', [TestCaseController::class, 'show'])
    ->name('test-case');

Route::get('/get', [TestCaseController::class, 'getData'])
    ->name('test-case.get');

Route::post('/check', [TestCaseController::class, 'checkData'])
    ->name('test-case.check');
```
