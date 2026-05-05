# Букинг система переговорочных

Разработать букинг систему используя Laravel через REST API

# Теория

# 1. Request и работа с входными данными

## Request (Illuminate\Http\Request)

Передаётся в каждый контроллер:

```php
public function store(Request $request)
```

### Основные методы

#### validate()

```php
$data = $request->validate([
    'email' => 'required|email'
]);
```

Что делает:

* валидирует входные данные
* если ошибка → сразу 422 и выход из метода
* если ок → возвращает массив `$data`

Важно:

* возвращает **array**, не объект
* дальше работаешь через `$data['field']`

---

#### user()

```php
$user = $request->user();
```

Что делает:

* возвращает текущего авторизованного пользователя
* работает только если есть middleware `auth:sanctum`

---

# 2. Ответы (Response)

## response()->json()

```php
return response()->json($data, 201);
```

Что делает:

* формирует JSON-ответ
* второй параметр — HTTP статус

Часто используемые статусы:

* 200 — OK
* 201 — создано
* 401 — не авторизован
* 403 — доступ запрещён
* 409 — конфликт (например пересечение брони)
* 422 — ошибка валидации

---

# 3. Модель Eloquent (ORM)

## Базовые операции

### create()

```php
Booking::create($data);
```

Требует:

```php
protected $fillable = [...];
```

Что делает:

* создаёт запись в БД
* автоматически делает INSERT

---

### find()

```php
$user = User::find($id);
```

* ищет по primary key
* возвращает модель или null

---

### where()

```php
Booking::where('user_id', 1)
```

* создаёт query builder
* сам по себе запрос не выполняет

---

### first()

```php
$user = User::where('email', $email)->first();
```

* выполняет запрос
* возвращает один объект или null

---

### get()

```php
$bookings = Booking::where(...)->get();
```

* выполняет запрос
* возвращает коллекцию (массив моделей)

---

### all()

```php
Booking::all();
```

* SELECT * FROM bookings
* без условий
* сразу выполняет запрос

---

### exists()

```php
$exists = Booking::where(...)->exists();
```

* проверяет наличие записей
* возвращает true/false
* быстрее, чем get()

---

### update()

```php
$booking->update($data);
```

* обновляет запись
* использует fillable

---

### delete()

```php
$booking->delete();
```

* удаляет запись

---

# 4. Связи (Relationships)

## hasMany (User → Booking)

```php
public function bookings(): HasMany
{
    return $this->hasMany(Booking::class);
}
```

---

## belongsTo (Booking → User)

```php
public function user(): BelongsTo
{
    return $this->belongsTo(User::class);
}
```

---

## Использование

### как метод

```php
$user->bookings()
```

* возвращает query builder
* можно добавлять условия

```php
$user->bookings()->orderBy('starts_at')->get();
```

---

### как свойство

```php
$user->bookings
```

* сразу выполняет запрос
* возвращает коллекцию

---

### создание через связь

```php
$request->user()->bookings()->create($data);
```

Что происходит:

* автоматически подставляется `user_id`
* INSERT в БД

---

# 5. Query Builder (построение запросов)

## where()

```php
->where('field', '=', value)
```

или коротко:

```php
->where('field', value)
```

---

## where с условиями

```php
->where(function ($query) {
    $query->where(...)->where(...);
})
```

Используется для группировки условий (скобки в SQL)

---

## orderBy()

```php
->orderBy('starts_at')
```

* сортировка

---

## exists()

уже описан выше — проверка наличия

---

# 6. Проверка пересечения времени

Используется комбинация where:

```php
Booking::where('room_name', $room)
    ->where('starts_at', '<', $end)
    ->where('ends_at', '>', $start)
    ->exists();
```

Смысл:

* ищем записи, которые пересекаются по времени

---

## Исключение текущей записи (при update)

```php
->where('id', '!=', $booking->id)
```

---

# 7. Авторизация (Sanctum)

## createToken()

```php
$token = $user->createToken('auth_token')->plainTextToken;
```

Что делает:

* создаёт токен в таблице
* возвращает строку токена

---

## currentAccessToken()

```php
$request->user()->currentAccessToken()->delete();
```

Что делает:

* удаляет текущий токен (logout)

---

# 8. Hash (пароли)

## Hash::make()

```php
Hash::make($password)
```

* хеширует пароль

---

## Hash::check()

```php
Hash::check($inputPassword, $user->password)
```

* проверяет пароль

---

# 9. Route Model Binding

```php
public function show(Booking $booking)
```

Что делает:

* Laravel сам делает `Booking::find(id)`
* id берётся из URL

---

# 10. Middleware

## auth:sanctum

```php
Route::middleware('auth:sanctum')->group(...)
```

Что делает:

* проверяет токен
* если валиден → добавляет `$request->user()`
* если нет → 401

---

# 11. Валидация (rules)

Примеры:

```php
'required'
'email'
'string'
'max:255'
'date'
'after:now'
'after:starts_at'
'sometimes'
'nullable'
```

---

## Особенности

### sometimes

```php
'sometimes|string'
```

* поле необязательно
* но если есть → валидируется

---

### after:starts_at

* сравнивает поля внутри запроса

---

# 12. Итоговая схема работы

1. Request приходит
2. Middleware проверяет токен
3. Контроллер получает Request
4. validate() проверяет данные
5. Eloquent делает запросы к БД
6. Response возвращает JSON

# Как сдавать

- Создайте форк репозитория в вашей организации с названием-этого-репозитория-вашафамилия
- Используя ветку wip сделайте задание
- Зафиксируйте изменения в вашем репозитории
- Когда документ будет готов - создайте пул реквест из ветки wip (вашей) на ветку main (тоже вашу) и укажите меня (ktkv419) как reviewer

Не мержите сами коммит, это сделаю я после проверки задания
