> @GitHub comencemos

Listo. Ya revisé el estado real de `Duran24062005/laravel-blog-project`.

### Estado actual

El repositorio está prácticamente en estado **Laravel recién creado**:

* Laravel **13**
* PHP `^8.3`
* Eloquent configurado
* Migraciones base de `users`, `cache` y `jobs`
* Modelo `User` ya creado
* Una única ruta `/` que muestra `welcome`
* No existe todavía `Post`
* No existe autenticación implementada
* No existen controladores del blog
* No existen relaciones entre usuarios y publicaciones
* No existe autorización/policies
* No existe CRUD

Esto es bueno para aprender porque podemos construir el sistema desde la base sin tener que deshacer decisiones previas.

### Primera decisión importante

**No vamos a comenzar creando el CRUD.**

Primero necesitamos definir el modelo de datos:

```text
User
 │
 │ 1
 │
 │ N
 ▼
Post
```

Es decir:

> Un usuario puede tener muchas publicaciones, y cada publicación pertenece a un usuario.

De ahí salen posteriormente:

```text
users
 └── id
      │
      └──── posts.user_id
                 │
                 ├── title
                 ├── content
                 ├── created_at
                 └── updated_at
```

Y esta relación será fundamental para poder implementar después:

> “Alexi puede editar su publicación, pero no la de otro usuario.”

No queremos solucionar eso simplemente ocultando botones. **La autorización debe existir en el backend.**

### Nuestro Paso 1

Antes de tocar código, quiero que entiendas una pieza fundamental de Laravel:

**Migration → Database → Model → Relationship**

La migración define **cómo debe existir la información en la BD**.

El modelo `Post` representará esa información en Laravel.

La relación permitirá que Laravel entienda:

```php
$user->posts
```

y:

```php
$post->user
```

Después podremos construir el CRUD encima de esa estructura.

---

## Primera tarea

En tu proyecto local, ejecuta:

```bash
php artisan migrate:status
```

y luego:

```bash
php artisan about
```

**No modifiques nada todavía.**

Pásame la salida de ambos comandos.

A partir de eso hacemos nuestro **Paso 1: entender la base de datos actual y crear nuestra primera migration**, explicando línea por línea qué estamos haciendo y por qué.
