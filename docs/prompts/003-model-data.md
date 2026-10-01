> Mira, en este caso la migracion funciono y creo la migracion y el modelo, pero lo que necesito es no solo quedarme en las explicaciones y avanzar un oco más rapido en código, yo no soy tan junior y entiendo un lo que voy haciendo, pero gracias por explicar.

> Continuemos, esté proyecto no tiene por que ser demorado.


Perfecto. Entonces cambiamos el ritmo: **explicación corta cuando aporte contexto + implementación directa**. La idea es terminar este MVP rápido, pero aprovechándolo para recuperar Laravel de forma práctica.

Ya tenemos:

```text
Laravel 13
MySQL
users
Post model
create_posts_table migration
```

## Siguiente paso: terminar `Post`

### 1. Migration

En `create_posts_table` déjala así:

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();

    $table->foreignId('user_id')
        ->constrained()
        ->cascadeOnDelete();

    $table->string('title');
    $table->text('content');

    $table->timestamps();
});
```

La parte importante:

```php
$table->foreignId('user_id')
    ->constrained()
    ->cascadeOnDelete();
```

nos crea la FK:

```text
posts.user_id → users.id
```

y si eliminamos un usuario, sus posts también se eliminan.

---

### 2. Modelo `Post`

`app/Models/Post.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Post extends Model
{
    protected $fillable = [
        'title',
        'content',
        'user_id',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

---

### 3. Agregamos la relación en `User`

En `app/Models/User.php`:

```php
use Illuminate\Database\Eloquent\Relations\HasMany;
```

y dentro de la clase:

```php
public function posts(): HasMany
{
    return $this->hasMany(Post::class);
}
```

Con eso tenemos:

```text
User
 └── hasMany → Post

Post
 └── belongsTo → User
```

---

## 4. Ejecutamos la migration

Ahora sí:

```bash
php artisan migrate
```

Comprueba:

```bash
php artisan migrate:status
```

Deberías tener:

```text
create_users_table     Ran
create_cache_table     Ran
create_jobs_table      Ran
create_posts_table     Ran
```

---

# 5. Ahora vamos a hacer CRUD de verdad

No quiero que nos quedemos creando modelos y migrations. Vamos directamente a:

```text
Routes
   ↓
Controller
   ↓
Model
   ↓
Database
```

Creamos el controller:

```bash
php artisan make:controller PostController --resource
```

Esto nos genera los métodos:

```php
index()
create()
store()
show()
edit()
update()
destroy()
```

Y ahora registramos el recurso en `routes/web.php`:

```php
use App\Http\Controllers\PostController;

Route::resource('posts', PostController::class);
```

Laravel automáticamente nos dará:

```text
GET       /posts              index
GET       /posts/create       create
POST      /posts              store
GET       /posts/{post}       show
GET       /posts/{post}/edit  edit
PUT/PATCH /posts/{post}      update
DELETE    /posts/{post}      destroy
```

Puedes comprobarlo con:

```bash
php artisan route:list
```

---

## 6. Primero hagamos el `store`

Como todavía no hemos montado autenticación, para avanzar rápido podemos usar temporalmente un usuario existente.

Pero antes necesito saber algo concreto: **¿tu tabla `users` ya tiene algún usuario creado?**

Si sí, hacemos inmediatamente:

```text
POST /posts
    ↓
validate
    ↓
Post::create()
    ↓
MySQL
```

y después montamos:

* listado
* detalle
* creación
* edición
* eliminación
* Blade
* autenticación
* autorización con Policy
* validación
* relaciones Eloquent
* paginación

Todo el MVP, sin convertirlo en un curso interminable.
