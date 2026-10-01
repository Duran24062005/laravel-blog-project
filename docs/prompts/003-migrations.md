```cmd
alexi-dg@alexi-dg-HP-Pavilion-Laptop-15-eg0xxx:~/Desktop/proyecto_laravel$ php artisan migrate:status

  Migration name ........................... Batch / Status  
  0001_01_01_000000_create_users_table ............ [1] Ran  
  0001_01_01_000001_create_cache_table ............ [1] Ran  
  0001_01_01_000002_create_jobs_table ............. [1] Ran  

alexi-dg@alexi-dg-HP-Pavilion-Laptop-15-eg0xxx:~/Desktop/proyecto_laravel$ php artisan about

  Environment .............................................  
  Application Name ................................ Laravel  
  Laravel Version ................................. 13.34.0  
  PHP Version ....................................... 8.5.4  
  Composer Version ................................. 2.10.3  
  Environment ....................................... local  
  Debug Mode ...................................... ENABLED  
  URL ........................................... localhost  
  Maintenance Mode .................................... OFF  
  Timezone ............................................ UTC  
  Locale ............................................... en  

  Cache ...................................................  
  Config ....................................... NOT CACHED  
  Events ....................................... NOT CACHED  
  Routes ....................................... NOT CACHED  
  Views ............................................ CACHED  

  Drivers .................................................  
  Broadcasting ........................................ log  
  Cache .......................................... database  
  Database .......................................... mysql  
  Logs ..................................... stack / single  
  Mail ................................................ log  
  Queue .......................................... database  
  Session ........................................ database  

  Storage .................................................  
  public/storage ............................... NOT LINKED  

alexi-dg@alexi-dg-HP-Pavilion-Laptop-15-eg0xxx:~/Desktop/proyecto_laravel$ 
```

Perfecto. Con esto ya confirmamos el estado inicial del proyecto y **no necesitamos modificar nada todavía**.

### 1. ¿Qué nos dice `migrate:status`?

Tus tres migraciones base ya fueron ejecutadas:

```text
create_users_table   [1] Ran
create_cache_table   [1] Ran
create_jobs_table    [1] Ran
```

Eso significa que Laravel ya tiene en MySQL las tablas correspondientes a:

* `users` → usuarios.
* `cache` → caché de Laravel.
* `jobs` → colas/tareas.

Y algo importante para el aprendizaje:

> **Una migration no es la base de datos. Es la definición versionada de cómo Laravel crea o modifica la estructura de la base de datos.**

---

### 2. Tu entorno también está correctamente configurado

Tenemos:

```text
Laravel 13.34.0
PHP 8.5.4
Database: mysql
Environment: local
Debug: ENABLED
```

Para nuestro MVP, esto está bien.

Además:

```text
Session: database
Cache: database
Queue: database
```

Esto explica por qué Laravel creó las tablas `cache` y `jobs`, y posteriormente podremos entender qué función cumple cada una.

---

# Paso 1 — Vamos a crear `posts`

Ahora sí vamos a comenzar con nuestro dominio.

La estructura que queremos inicialmente es:

```text
users
  │
  │ 1
  │
  │ N
posts
```

Es decir:

```text
User
 └── hasMany Posts

Post
 └── belongsTo User
```

¿Por qué empezamos por aquí?

Porque casi todo el MVP depende de esta relación.

Por ejemplo:

```php
$user->posts
```

significará:

> Dame los posts pertenecientes a este usuario.

Y:

```php
$post->user
```

significará:

> Dame el usuario propietario de este post.

Más adelante esa misma relación nos permitirá determinar:

```text
¿El usuario autenticado es propietario de este post?
             │
             ├── Sí → puede editar/eliminar
             │
             └── No → no puede
```

Ahí entraremos posteriormente en **Policies y autorización**.

---

# 3. Antes de escribir la migration

Quiero que pensemos primero qué necesita un `Post`.

Para nuestro MVP mínimo:

| Campo        | Tipo        | Propósito                   |
| ------------ | ----------- | --------------------------- |
| `id`         | bigint      | Identificador               |
| `user_id`    | foreign key | Propietario                 |
| `title`      | string      | Título                      |
| `content`    | text        | Contenido                   |
| `timestamps` | timestamps  | `created_at` / `updated_at` |

Visualmente:

```text
posts
--------------------------------
id
user_id       → users.id
title
content
created_at
updated_at
```

### ¿Por qué `user_id`?

Porque necesitamos almacenar **quién creó el post**.

Por ejemplo:

```text
users

id   name
1    Alexi
2    Juan
```

Y:

```text
posts

id   user_id   title
1    1         Mi primer post
2    1         Aprendiendo Laravel
3    2         Mi experiencia
```

Entonces Laravel puede interpretar:

```text
Alexi
 ├── Mi primer post
 └── Aprendiendo Laravel

Juan
 └── Mi experiencia
```

---

# 4. Ahora crea la migration

Ejecuta:

```bash
php artisan make:model Post -m
```

Aquí hay algo importante para aprender.

No estamos haciendo:

```bash
php artisan make:model Post
```

porque necesitamos **dos cosas relacionadas**:

```text
Post Model
     +
Post Migration
```

La opción:

```text
-m
```

le dice a Artisan que cree también la migration.

Después deberías tener:

```text
app/
└── Models/
    ├── User.php
    └── Post.php

database/
└── migrations/
    ├── ...
    └── xxxx_xx_xx_xxxxxx_create_posts_table.php
```

---

## 5. No ejecutes `migrate` todavía

Después de ejecutar:

```bash
php artisan make:model Post -m
```

**párate ahí.**

Muéstrame el contenido que Laravel generó de:

```text
app/Models/Post.php
```

y:

```text
database/migrations/xxxx_xx_xx_xxxxxx_create_posts_table.php
```

Quiero que primero entendamos **qué generó Laravel automáticamente**, y luego construiremos nosotros la estructura de `posts` línea por línea.

Así evitamos el típico aprendizaje de:

> "Copié el código, ejecuté `migrate` y funcionó."

La idea aquí es que entiendas **por qué cada línea existe y qué problema resuelve**.
