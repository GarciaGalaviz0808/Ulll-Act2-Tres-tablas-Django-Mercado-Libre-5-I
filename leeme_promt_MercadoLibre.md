Perfecto — te doy **todo** lo que pediste: pasos exactos (terminal / VS Code), comandos, y el código completo para que tengas un **proyecto Django funcional** llamado `UIII_MercadoLibre_0583` con la app `app_MercadoLibre`, el modelo **Vendedor** (solo Vendedor por ahora), vistas CRUD, plantillas Bootstrap, registro en admin y pruebas en el puerto **8015**. No usamos `forms.py` (manejo manual con `request.POST`). Usa copiar/pegar directamente en tus archivos.

---

# 1 — Crear la carpeta del proyecto

Abre un terminal (o el Explorador) y crea la carpeta raíz:

```bash
# en la ruta donde quieras crear el proyecto
mkdir UIII_MercadoLibre_0583
cd UIII_MercadoLibre_0583
```

# 2 — Abrir VS Code sobre la carpeta

Desde terminal:

```bash
code .
```

(O abre VS Code y `File → Open Folder…` → selecciona `UIII_MercadoLibre_0583`).

# 3 — Abrir terminal en VS Code

En VS Code: `Ctrl + \`` (tecla de tilde, `View → Terminal`) — se abrirá la terminal integrada.

# 4 — Crear carpeta entorno virtual `.venv`

En la terminal integrada (Linux / macOS / Windows PowerShell funcionan):

```bash
# crea el virtualenv oculto en la carpeta del proyecto
python -m venv .venv
```

# 5 — Activar el entorno virtual

* Windows (PowerShell):

```powershell
# si PowerShell
.\.venv\Scripts\Activate.ps1
```

Si da error por política de ejecución, usa:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
.\.venv\Scripts\Activate.ps1
```

* Windows (cmd):

```cmd
.\.venv\Scripts\activate
```

* macOS / Linux:

```bash
source .venv/bin/activate
```

Al activarlo deberías ver `(.venv)` al inicio de tu prompt.

# 6 — Activar intérprete de Python en VS Code

En VS Code: `Ctrl+Shift+P` → escribe `Python: Select Interpreter` → selecciona la ruta que apunta a `.../UIII_MercadoLibre_0583/.venv/bin/python` (o `Scripts\python.exe` en Windows). Esto hace que los ejecutables y depuración usen ese entorno.

# 7 — Instalar Django

Con el virtualenv activado:

```bash
pip install --upgrade pip
pip install django
```

# 8 — Crear proyecto `backend_MercadoLibre` sin duplicar carpeta

Para **evitar** crear una carpeta extra llamada `backend_MercadoLibre` dentro de otra, ejecuta desde la raíz:

```bash
django-admin startproject backend_MercadoLibre .
```

**Nota:** el punto (`.`) crea el proyecto en la carpeta actual (no duplica).

Estructura ahora (lo principal):

```
UIII_MercadoLibre_0583/
├─ backend_MercadoLibre/
│  ├─ __init__.py
│  ├─ settings.py
│  ├─ urls.py
│  └─ wsgi.py
├─ manage.py
└─ .venv/
```

# 9 — Ejecutar servidor en el puerto 8015

Primero crea la app (siguiente paso), luego migraciones y luego:

```bash
python manage.py runserver 8015
```

O si quieres aceptar conexiones externas (no necesario localmente):

```bash
python manage.py runserver 0.0.0.0:8015
```

# 10 — Copiar y pegar el link en el navegador

Abre:

```
http://127.0.0.1:8015/
```

(ó `http://localhost:8015/`)

# 11 — Crear aplicación `app_MercadoLibre`

Desde la raíz (con venv activado):

```bash
python manage.py startapp app_MercadoLibre
```

Estructura:

```
app_MercadoLibre/
  ├─ migrations/
  ├─ templates/
  ├─ static/
  ├─ views.py
  ├─ models.py
  ├─ urls.py   # lo crearás
  └─ admin.py
```

---

# 12 — models.py (usa el que ya diste, lo pongo listo)

Archivo: `app_MercadoLibre/models.py`

```python
from django.db import models

# ==========================================
# MODELO: VENDEDOR
# ==========================================
class Vendedor(models.Model):
    id_vendedor = models.AutoField(primary_key=True, unique=True)
    nombre = models.CharField(max_length=200)
    apellido = models.CharField(max_length=200)
    email = models.EmailField(max_length=300, unique=True)
    telefono = models.CharField(max_length=15)
    direccion = models.CharField(max_length=300)
    ciudad = models.CharField(max_length=100)
    fecha_registro = models.DateField(auto_now_add=True)

    def __str__(self):
        return f"{self.nombre} {self.apellido}"


# ==========================================
# MODELO: CATEGORIA (pendiente)
# ==========================================
class Categoria(models.Model):
    id_categoria = models.AutoField(primary_key=True, unique=True)
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField()
    tipo = models.CharField(max_length=50)
    fecha_creacion = models.DateField(auto_now_add=True)
    activa = models.BooleanField(default=True)
    img_url = models.URLField(max_length=254, blank=True, null=True)

    def __str__(self):
        return self.nombre

# ==========================================
# MODELO: PRODUCTO (pendiente)
# ==========================================
class Producto(models.Model):
    id_producto = models.AutoField(primary_key=True, unique=True)

    vendedor = models.ForeignKey(
        Vendedor, on_delete=models.CASCADE, related_name="productos"
    )

    categorias = models.ManyToManyField(
        Categoria, related_name="productos"
    )

    nombre = models.CharField(max_length=100)
    precio = models.IntegerField()
    stock = models.IntegerField()
    fecha_publicacion = models.DateField(auto_now_add=True)
    descripcion = models.CharField(max_length=200)
    disponible = models.BooleanField(default=True)

    def __str__(self):
        return self.nombre
```

> Por ahora trabajarás con **Vendedor**; deja Categoria y Producto como pendientes (ya presentes pero no usadas aún).

---

# 12.5 — Procedimiento para realizar migraciones (makemigrations y migrate)

1. Añade la app en `settings.py` (punto 25 abajo).
2. Genera migraciones y aplícalas:

```bash
python manage.py makemigrations app_MercadoLibre
python manage.py migrate
```

---

# 13 — Vamos a trabajar primero con el MODELO: VENDEDOR (sí)

# 14 — Vistas (views.py) de `app_MercadoLibre` (CRUD vendedor)

Crea/edita `app_MercadoLibre/views.py` con este contenido:

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Vendedor

# Página de inicio del sistema
def inicio_mercadolibre(request):
    contexto = {}
    return render(request, "inicio.html", contexto)

# Agregar vendedor (form simple POST)
def agregar_vendedor(request):
    if request.method == "POST":
        nombre = request.POST.get("nombre", "")
        apellido = request.POST.get("apellido", "")
        email = request.POST.get("email", "")
        telefono = request.POST.get("telefono", "")
        direccion = request.POST.get("direccion", "")
        ciudad = request.POST.get("ciudad", "")
        # No validamos (según tu pedido)
        Vendedor.objects.create(
            nombre=nombre,
            apellido=apellido,
            email=email,
            telefono=telefono,
            direccion=direccion,
            ciudad=ciudad
        )
        return redirect("ver_vendedor")
    return render(request, "vendedor/agregar_vendedor.html")

# Ver vendedores (lista en tabla)
def ver_vendedor(request):
    vendedores = Vendedor.objects.all().order_by("-fecha_registro")
    return render(request, "vendedor/ver_vendedor.html", {"vendedores": vendedores})

# Actualizar — muestra formulario con datos
def actualizar_vendedor(request, id_vendedor):
    vendedor = get_object_or_404(Vendedor, id_vendedor=id_vendedor)
    return render(request, "vendedor/actualizar_vendedor.html", {"vendedor": vendedor})

# Realizar actualización (POST)
def realizar_actualizacion_vendedor(request, id_vendedor):
    vendedor = get_object_or_404(Vendedor, id_vendedor=id_vendedor)
    if request.method == "POST":
        vendedor.nombre = request.POST.get("nombre", vendedor.nombre)
        vendedor.apellido = request.POST.get("apellido", vendedor.apellido)
        vendedor.email = request.POST.get("email", vendedor.email)
        vendedor.telefono = request.POST.get("telefono", vendedor.telefono)
        vendedor.direccion = request.POST.get("direccion", vendedor.direccion)
        vendedor.ciudad = request.POST.get("ciudad", vendedor.ciudad)
        vendedor.save()
        return redirect("ver_vendedor")
    return redirect("actualizar_vendedor", id_vendedor=id_vendedor)

# Borrar — confirmar y eliminar
def borrar_vendedor(request, id_vendedor):
    vendedor = get_object_or_404(Vendedor, id_vendedor=id_vendedor)
    if request.method == "POST":
        vendedor.delete()
        return redirect("ver_vendedor")
    return render(request, "vendedor/borrar_vendedor.html", {"vendedor": vendedor})
```

# 15 — Crear carpeta `templates` dentro de `app_MercadoLibre`

Estructura de plantillas:

```
app_MercadoLibre/
  templates/
    base.html
    header.html
    navbar.html
    footer.html
    inicio.html
    vendedor/
      agregar_vendedor.html
      ver_vendedor.html
      actualizar_vendedor.html
      borrar_vendedor.html
```

# 16 —/17/18/19/20 — Código de las plantillas

## `app_MercadoLibre/templates/base.html`

```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>UIII Mercado Libre - Sistema</title>

    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Bootstrap Icons -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" rel="stylesheet">

    <style>
      :root{
        --primary-soft: #f5f7fb;
        --accent: #7aa2d8;
        --muted:#6c7a89;
      }
      body { background: var(--primary-soft); color:#24313a; padding-bottom:80px; }
      .site-footer{
        position: fixed;
        left: 0;
        bottom: 0;
        width: 100%;
        background: white;
        border-top: 1px solid #e6e9ef;
        padding: 10px 20px;
      }
      .card-soft { border-radius:12px; box-shadow: 0 8px 18px rgba(24,39,60,0.05); }
    </style>

  </head>
  <body>
    {% include "navbar.html" %}
    <main class="container py-4">
      {% block content %}{% endblock %}
    </main>

    {% include "footer.html" %}

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
  </body>
</html>
```

## `app_MercadoLibre/templates/navbar.html`

```html
<nav class="navbar navbar-expand-lg navbar-light bg-white border-bottom">
  <div class="container">
    <a class="navbar-brand d-flex align-items-center" href="{% url 'inicio' %}">
      <i class="bi bi-cart3 fs-3 me-2" style="color:#3b82c4"></i>
      <span class="fw-bold">Sistema de Administración Mercado Libre</span>
    </a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMenu">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse" id="navMenu">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'inicio' %}">Inicio</a></li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
            <i class="bi bi-person-badge me-1"></i> Vendedor
          </a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'agregar_vendedor' %}">Agregar vendedor</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_vendedor' %}">Ver vendedor</a></li>
            <li><a class="dropdown-item" href="#">Actualizar vendedor</a></li>
            <li><a class="dropdown-item" href="#">Borrar vendedor</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
            <i class="bi bi-tags-fill me-1"></i> Categorías
          </a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar categoria</a></li>
            <li><a class="dropdown-item" href="#">Ver categoria</a></li>
            <li><a class="dropdown-item" href="#">Actualizar categoria</a></li>
            <li><a class="dropdown-item" href="#">Borrar categoria</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
            <i class="bi bi-box-seam me-1"></i> Productos
          </a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar producto</a></li>
            <li><a class="dropdown-item" href="#">Ver productos</a></li>
            <li><a class="dropdown-item" href="#">Actualizar productos</a></li>
            <li><a class="dropdown-item" href="#">Borrar productos</a></li>
          </ul>
        </li>

      </ul>
    </div>
  </div>
</nav>
```

## `app_MercadoLibre/templates/footer.html`

```html
<footer class="site-footer">
  <div class="container d-flex justify-content-between">
    <div>© Derechos reservados</div>
    <div id="fecha-sistema"></div>
    <div>Creado por Melanie Garcia, Cbtis 128</div>
  </div>
  <script>
    document.getElementById('fecha-sistema').innerText = new Date().toLocaleDateString();
  </script>
</footer>
```

## `app_MercadoLibre/templates/inicio.html`

```html
{% extends "base.html" %}
{% block content %}
<div class="row">
  <div class="col-md-8">
    <div class="card card-soft p-4">
      <h3>Bienvenido al Sistema Mercado Libre</h3>
      <p>Aquí puedes administrar vendedores, categorías y productos.</p>
      <p>Utiliza el menú para navegar por el CRUD de vendedores.</p>
    </div>
  </div>
  <div class="col-md-4">
    <div class="card card-soft p-2 text-center">
      <img src="https://upload.wikimedia.org/wikipedia/commons/6/6a/Cinepolis_logo.png" alt="Cinépolis" class="img-fluid" style="max-height:120px;">
      <p class="small mt-2">Imagen tomada desde la red sobre Cinépolis</p>
    </div>
  </div>
</div>
{% endblock %}
```

> **Nota**: imagen pública tomada de la red (ejemplo). Si quieres otra, reemplázala.

---

# 21 & 22 — Carpeta `vendedor` y plantillas CRUD

Crea `app_MercadoLibre/templates/vendedor/` y archivos:

## `agregar_vendedor.html`

```html
{% extends "base.html" %}
{% block content %}
<div class="card card-soft p-4">
  <h4>Agregar Vendedor</h4>
  <form method="post">
    {% csrf_token %}
    <div class="mb-2">
      <label class="form-label">Nombre</label>
      <input type="text" name="nombre" class="form-control" required>
    </div>
    <div class="mb-2">
      <label class="form-label">Apellido</label>
      <input type="text" name="apellido" class="form-control" required>
    </div>
    <div class="mb-2">
      <label class="form-label">Email</label>
      <input type="email" name="email" class="form-control" required>
    </div>
    <div class="mb-2">
      <label class="form-label">Teléfono</label>
      <input type="text" name="telefono" class="form-control">
    </div>
    <div class="mb-2">
      <label class="form-label">Dirección</label>
      <input type="text" name="direccion" class="form-control">
    </div>
    <div class="mb-2">
      <label class="form-label">Ciudad</label>
      <input type="text" name="ciudad" class="form-control">
    </div>
    <button class="btn btn-primary mt-2">Agregar</button>
    <a href="{% url 'ver_vendedor' %}" class="btn btn-secondary mt-2">Cancelar</a>
  </form>
</div>
{% endblock %}
```

## `ver_vendedor.html`

```html
{% extends "base.html" %}
{% block content %}
<div class="card card-soft p-3">
  <div class="d-flex justify-content-between mb-3">
    <h4>Vendedores</h4>
    <a class="btn btn-success" href="{% url 'agregar_vendedor' %}"><i class="bi bi-plus-lg"></i> Nuevo</a>
  </div>

  <div class="table-responsive">
    <table class="table table-hover">
      <thead class="table-light">
        <tr>
          <th>ID</th>
          <th>Nombre</th>
          <th>Email</th>
          <th>Teléfono</th>
          <th>Ciudad</th>
          <th>Fecha Registro</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        {% for v in vendedores %}
        <tr>
          <td>{{ v.id_vendedor }}</td>
          <td>{{ v.nombre }} {{ v.apellido }}</td>
          <td>{{ v.email }}</td>
          <td>{{ v.telefono }}</td>
          <td>{{ v.ciudad }}</td>
          <td>{{ v.fecha_registro }}</td>
          <td>
            <a class="btn btn-sm btn-info" href="{% url 'actualizar_vendedor' v.id_vendedor %}">Editar</a>
            <a class="btn btn-sm btn-danger" href="{% url 'borrar_vendedor' v.id_vendedor %}">Borrar</a>
          </td>
        </tr>
        {% empty %}
        <tr><td colspan="7" class="text-center">No hay vendedores aún.</td></tr>
        {% endfor %}
      </tbody>
    </table>
  </div>
</div>
{% endblock %}
```

## `actualizar_vendedor.html`

```html
{% extends "base.html" %}
{% block content %}
<div class="card card-soft p-4">
  <h4>Actualizar Vendedor</h4>
  <form method="post" action="{% url 'realizar_actualizacion_vendedor' vendedor.id_vendedor %}">
    {% csrf_token %}
    <div class="mb-2">
      <label class="form-label">Nombre</label>
      <input type="text" name="nombre" class="form-control" value="{{ vendedor.nombre }}">
    </div>
    <div class="mb-2">
      <label class="form-label">Apellido</label>
      <input type="text" name="apellido" class="form-control" value="{{ vendedor.apellido }}">
    </div>
    <div class="mb-2">
      <label class="form-label">Email</label>
      <input type="email" name="email" class="form-control" value="{{ vendedor.email }}">
    </div>
    <div class="mb-2">
      <label class="form-label">Teléfono</label>
      <input type="text" name="telefono" class="form-control" value="{{ vendedor.telefono }}">
    </div>
    <div class="mb-2">
      <label class="form-label">Dirección</label>
      <input type="text" name="direccion" class="form-control" value="{{ vendedor.direccion }}">
    </div>
    <div class="mb-2">
      <label class="form-label">Ciudad</label>
      <input type="text" name="ciudad" class="form-control" value="{{ vendedor.ciudad }}">
    </div>
    <button class="btn btn-primary mt-2">Guardar</button>
    <a href="{% url 'ver_vendedor' %}" class="btn btn-secondary mt-2">Cancelar</a>
  </form>
</div>
{% endblock %}
```

## `borrar_vendedor.html`

```html
{% extends "base.html" %}
{% block content %}
<div class="card card-soft p-4">
  <h4>¿Eliminar vendedor?</h4>
  <p>Estás por eliminar a: <strong>{{ vendedor.nombre }} {{ vendedor.apellido }}</strong></p>
  <form method="post">
    {% csrf_token %}
    <button type="submit" class="btn btn-danger">Sí, eliminar</button>
    <a href="{% url 'ver_vendedor' %}" class="btn btn-secondary">Cancelar</a>
  </form>
</div>
{% endblock %}
```

---

# 23 — No utilizar `forms.py`

El manejo de formularios es con `request.POST` en las vistas mostradas (sin validación).

# 24 — `urls.py` en `app_MercadoLibre`

Crea `app_MercadoLibre/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.inicio_mercadolibre, name="inicio"),
    path("vendedor/agregar/", views.agregar_vendedor, name="agregar_vendedor"),
    path("vendedor/ver/", views.ver_vendedor, name="ver_vendedor"),
    path("vendedor/actualizar/<int:id_vendedor>/", views.actualizar_vendedor, name="actualizar_vendedor"),
    path("vendedor/realizar_actualizacion/<int:id_vendedor>/", views.realizar_actualizacion_vendedor, name="realizar_actualizacion_vendedor"),
    path("vendedor/borrar/<int:id_vendedor>/", views.borrar_vendedor, name="borrar_vendedor"),
]
```

# 25 — Agregar `app_MercadoLibre` en `settings.py` de `backend_MercadoLibre`

En `backend_MercadoLibre/settings.py` -> `INSTALLED_APPS` añade:

```python
INSTALLED_APPS = [
    # apps por defecto...
    'app_MercadoLibre',
]
```

Además en `TEMPLATES` asegúrate que `DIRS` incluye nada especial (usaremos templates dentro de la app). El setting por defecto funciona.

Si quieres archivos estáticos locales:

```python
# al final del archivo settings.py (opcional)
STATIC_URL = '/static/'
STATICFILES_DIRS = [ BASE_DIR / "static", ]
```

# 26 — Configurar `backend_MercadoLibre/urls.py` para enlazar con `app_MercadoLibre`

Edita `backend_MercadoLibre/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_MercadoLibre.urls')),  # ruta raíz al app
]
```

# 27 — Registrar modelos en `admin.py` y volver a migrar

Edita `app_MercadoLibre/admin.py`:

```python
from django.contrib import admin
from .models import Vendedor, Categoria, Producto

@admin.register(Vendedor)
class VendedorAdmin(admin.ModelAdmin):
    list_display = ("id_vendedor", "nombre", "apellido", "email", "telefono", "ciudad", "fecha_registro")
    search_fields = ("nombre", "apellido", "email", "ciudad")

@admin.register(Categoria)
class CategoriaAdmin(admin.ModelAdmin):
    list_display = ("id_categoria", "nombre", "tipo", "activa")

@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = ("id_producto", "nombre", "vendedor", "precio", "stock", "disponible")
```

Luego:

```bash
python manage.py makemigrations
python manage.py migrate
```

# 27 (bis) — Crear superusuario para acceder al admin (opcional)

```bash
python manage.py createsuperuser
# sigue instrucciones (email, contraseña)
```

# 28 — Estética: colores suaves y código sencillo

Las plantillas usan colores suaves (variables CSS). Puedes ajustar `--accent` y `--primary-soft` en `base.html` a tu gusto.

# 29 — Al inicio crear la estructura completa de carpetas y archivos

Resumen de la estructura minimal para iniciar:

```
UIII_MercadoLibre_0583/
├─ .venv/
├─ backend_MercadoLibre/
│  ├─ settings.py
│  └─ urls.py
├─ app_MercadoLibre/
│  ├─ migrations/
│  ├─ templates/
│  │  ├─ base.html
│  │  ├─ navbar.html
│  │  ├─ footer.html
│  │  ├─ inicio.html
│  │  └─ vendedor/
│  │     ├─ agregar_vendedor.html
│  │     ├─ ver_vendedor.html
│  │     ├─ actualizar_vendedor.html
│  │     └─ borrar_vendedor.html
│  ├─ models.py
│  ├─ views.py
│  ├─ urls.py
│  └─ admin.py
├─ manage.py
```

# 30 — Proyecto totalmente funcional (pasos finales — resumen)

1. Activar `.venv`.
2. `pip install django`
3. `python manage.py makemigrations app_MercadoLibre`
4. `python manage.py migrate`
5. `python manage.py createsuperuser` (opcional)
6. `python manage.py runserver 8015`
7. Abrir `http://127.0.0.1:8015/`

# 31 — Ejecutar servidor en el puerto 8015 (ya indicado arriba)

```bash
python manage.py runserver 8015
```

---

## Consejos rápidos y solución de problemas comunes

* Si `django-admin` no se reconoce, asegúrate de activar `.venv` y que `pip install django` haya finalizado sin errores.
* Si al crear `backend_MercadoLibre` ya existe la carpeta con el mismo nombre, borra el contenido redundante o crea en una carpeta limpia.
* Si te sale error de "unique email" al crear vendedores manualmente con el mismo email: elimina el duplicado en la BD o usa otro email (no validamos entradas).
* Para editar estilos, ajusta `base.html`.
* Si no ves cambios en template, asegúrate de no tener `DEBUG = False` y limpia caché del navegador.

---

Si quieres, ahora puedo:

* Generar los archivos listos para descargar (zip) con todo el contenido que puse, o
* Enseñarte cómo poblar la base de datos con datos de ejemplo (fixtures), o
* Crear las vistas para `Categoria` y `Producto` igual que `Vendedor`.

Dime cuál de estas tres acciones quieres y lo hago ahora mismo. 😉

