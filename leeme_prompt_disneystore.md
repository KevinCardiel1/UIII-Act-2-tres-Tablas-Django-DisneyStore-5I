1) Procedimientos (1 → 12.5) — comandos y pasos en VS Code (Windows)

Asumo Windows (PowerShell). Ajusta si usas otro shell.

Crear carpeta del proyecto

En el Explorador de Windows: crea la carpeta UIII_DisneyStore_0530.

O en PowerShell:

mkdir UIII_DisneyStore_0530
cd UIII_DisneyStore_0530


Abrir VS Code sobre la carpeta

code .


Abrir terminal en VS Code

Menú: Terminal → New Terminal o `Ctrl+``

Crear carpeta entorno virtual .venv desde terminal de VS Code

python -m venv .venv


Activar el entorno virtual

PowerShell:

.\.venv\Scripts\Activate.ps1


Si PowerShell bloquea scripts (execution policy), puedes usar:

.\.venv\Scripts\Activate.bat


(o ejecutar PowerShell como administrador y Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser).

Activar intérprete de Python en VS Code

En VS Code: Ctrl+Shift+P → Python: Select Interpreter → elige el intérprete dentro de UIII_DisneyStore_0530\.venv\Scripts\python.exe.

Instalar Django

pip install django


(opcional: pip install django==4.2 si deseas fijar versión).

Crear proyecto backend_DisneyStore sin duplicar carpeta
Dentro de la carpeta UIII_DisneyStore_0530 ejecuta:

django-admin startproject backend_DisneyStore .


(el punto . evita crear otra subcarpeta y coloca el proyecto dentro de la carpeta actual)

Ejecutar servidor en el puerto 8006

python manage.py runserver 8006


Copiar y pegar el link en el navegador

Abre: http://127.0.0.1:8006/ (o la URL que muestre runserver)

Crear aplicación app_Disney

python manage.py startapp app_Disney


Archivo models.py (ver sección abajo — versión modificada con Categoria y Producto)

12.5 Procedimiento para realizar las migraciones (makemigrations y migrate)

python manage.py makemigrations
python manage.py migrate


(Si agregas la app después: añadir en settings.py antes de makemigrations — ver paso 25)

2) MODELO: CATEGORÍA y PRODUCTO (models.py actualizado)

Trabajamos primero la Categoría según paso 13. Dejé Cliente y Pedido comentados/pendientes (27).

Crea/edita app_Disney/models.py con:

from django.db import models

# ======================
# MODELO CATEGORIA
# ======================
class Categoria(models.Model):
    nombre = models.CharField(max_length=80, unique=True)
    descripcion = models.TextField(blank=True, null=True)
    creada_en = models.DateField(auto_now_add=True)

    class Meta:
        verbose_name = "Categoría"
        verbose_name_plural = "Categorías"
        ordering = ['nombre']

    def __str__(self):
        return self.nombre

# ======================
# MODELO PRODUCTO
# ======================
class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    categoria = models.ForeignKey(Categoria, on_delete=models.PROTECT, related_name='productos')
    descripcion = models.TextField(blank=True)
    precio = models.DecimalField(max_digits=8, decimal_places=2, default=0.00)
    stock = models.PositiveIntegerField(default=0)
    fecha_lanzamiento = models.DateField(blank=True, null=True)
    en_promocion = models.BooleanField(default=False)
    creado_en = models.DateTimeField(auto_now_add=True)
    actualizado_en = models.DateTimeField(auto_now=True)

    class Meta:
        verbose_name = "Producto"
        verbose_name_plural = "Productos"
        ordering = ['-creado_en']

    def __str__(self):
        return f"{self.nombre} - ${self.precio}"


Nota: categoria es ForeignKey a Categoria (paso 13). Usé on_delete=models.PROTECT para evitar borrar categorías si hay productos.

3) admin.py — registrar modelos (paso 27)

app_Disney/admin.py:

from django.contrib import admin
from .models import Categoria, Producto

@admin.register(Categoria)
class CategoriaAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'creada_en')
    search_fields = ('nombre',)

@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'categoria', 'precio', 'stock', 'en_promocion', 'creado_en')
    list_filter = ('categoria', 'en_promocion')
    search_fields = ('nombre', 'descripcion')


Luego:

python manage.py makemigrations app_Disney
python manage.py migrate

4) Views para productos (paso 14) — sin forms.py (uso request.POST)

Crea/edita app_Disney/views.py:

from django.shortcuts import render, redirect, get_object_or_404
from .models import Producto, Categoria
from django.urls import reverse
from django.http import HttpResponse

# Inicio del sistema — inicio_disneystore (paso 14)
def inicio_disneystore(request):
    productos_recientes = Producto.objects.all()[:6]
    categorias = Categoria.objects.all()
    return render(request, 'inicio.html', {
        'productos_recientes': productos_recientes,
        'categorias': categorias,
    })

# Agregar producto
def agregar_productos(request):
    categorias = Categoria.objects.all()
    if request.method == 'POST':
        nombre = request.POST.get('nombre', '').strip()
        categoria_id = request.POST.get('categoria')
        descripcion = request.POST.get('descripcion', '')
        precio = request.POST.get('precio', '0')
        stock = request.POST.get('stock', '0')
        fecha_lanzamiento = request.POST.get('fecha_lanzamiento') or None
        en_promocion = True if request.POST.get('en_promocion') == 'on' else False

        categoria = None
        if categoria_id:
            try:
                categoria = Categoria.objects.get(pk=categoria_id)
            except Categoria.DoesNotExist:
                categoria = None

        producto = Producto.objects.create(
            nombre=nombre,
            categoria=categoria,
            descripcion=descripcion,
            precio=precio,
            stock=stock,
            fecha_lanzamiento=fecha_lanzamiento,
            en_promocion=en_promocion
        )
        return redirect('ver_productos')
    return render(request, 'productos/agregar_productos.html', {'categorias': categorias})

# Ver productos
def ver_productos(request):
    productos = Producto.objects.select_related('categoria').all()
    return render(request, 'productos/ver_productos.html', {'productos': productos})

# Página para mostrar el formulario de actualización
def actualizar_productos(request, producto_id):
    producto = get_object_or_404(Producto, pk=producto_id)
    categorias = Categoria.objects.all()
    return render(request, 'productos/actualizar_productos.html', {'producto': producto, 'categorias': categorias})

# Procesa la actualización (realizar_actualizacion_productos)
def realizar_actualizacion_productos(request, producto_id):
    producto = get_object_or_404(Producto, pk=producto_id)
    if request.method == 'POST':
        producto.nombre = request.POST.get('nombre', producto.nombre)
        categoria_id = request.POST.get('categoria')
        if categoria_id:
            try:
                producto.categoria = Categoria.objects.get(pk=categoria_id)
            except Categoria.DoesNotExist:
                pass
        producto.descripcion = request.POST.get('descripcion', producto.descripcion)
        producto.precio = request.POST.get('precio', producto.precio)
        producto.stock = request.POST.get('stock', producto.stock)
        fecha_lanzamiento = request.POST.get('fecha_lanzamiento')
        producto.fecha_lanzamiento = fecha_lanzamiento if fecha_lanzamiento else producto.fecha_lanzamiento
        producto.en_promocion = True if request.POST.get('en_promocion') == 'on' else False
        producto.save()
        return redirect('ver_productos')
    return redirect('actualizar_productos', producto_id=producto.id)

# Borrar producto (confirmación y borrado)
def borrar_productos(request, producto_id):
    producto = get_object_or_404(Producto, pk=producto_id)
    if request.method == 'POST':
        producto.delete()
        return redirect('ver_productos')
    return render(request, 'productos/borrar_productos.html', {'producto': producto})


Nombres de vistas: inicio_disneystore, agregar_productos, ver_productos, actualizar_productos, realizar_actualizacion_productos, borrar_productos.

No se usa forms.py, conforme al punto 23.

5) URLs de la app (paso 24)

Crea app_Disney/urls.py:

from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_disneystore, name='inicio_disneystore'),
    # Productos CRUD
    path('productos/agregar/', views.agregar_productos, name='agregar_productos'),
    path('productos/', views.ver_productos, name='ver_productos'),
    path('productos/actualizar/<int:producto_id>/', views.actualizar_productos, name='actualizar_productos'),
    path('productos/actualizar/<int:producto_id>/realizar/', views.realizar_actualizacion_productos, name='realizar_actualizacion_productos'),
    path('productos/borrar/<int:producto_id>/', views.borrar_productos, name='borrar_productos'),
]

6) Enlazar app en settings.py (paso 25)

En backend_DisneyStore/settings.py, dentro de INSTALLED_APPS añade:

INSTALLED_APPS = [
    # ...
    'app_Disney',
    # ...
]

7) Configurar backend_DisneyStore/urls.py (paso 26)

Editar backend_DisneyStore/urls.py para apuntar a la app:

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Disney.urls')),  # raíz del sitio a app_Disney
]

8) Estructura de carpetas y archivos (paso 29)

Crea la siguiente estructura dentro de app_Disney:

app_Disney/
  migrations/
  templates/
    base.html
    header.html
    navbar.html
    footer.html
    inicio.html
    productos/
      agregar_productos.html
      ver_productos.html
      actualizar_productos.html
      borrar_productos.html
  static/
    css/
    img/
  models.py
  views.py
  urls.py
  admin.py

9) Plantillas HTML básicas (pasos 15-22, 17, 18, 19, 20)

A continuación incluyo plantillas mínimas que cumplen tus requisitos (Bootstrap incluido en base.html).

templates/base.html (colores suaves, diseño sencillo)

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block titulo %}Sistema DisneyStore{% endblock %}</title>

    <!-- Bootstrap CSS (CDN) -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">

    <style>
      /* Colores suaves y moderno */
      :root{
        --bg-light:#f7fafc;
        --accent:#f7c6c6; /* suave */
        --primary:#f2a3a3;
        --muted:#6b7280;
      }
      body{
        background: linear-gradient(180deg, var(--bg-light), white);
        color: #222;
        padding-bottom: 80px; /* espacio para footer fijo */
      }
      .footer-fixed{
        position: fixed;
        bottom: 0;
        left: 0;
        right: 0;
        background: #ffffffcc;
        border-top: 1px solid #e6e6e6;
        padding: 10px 20px;
      }
      .nav-icon { margin-right: 6px; }
    </style>
    {% block extra_head %}{% endblock %}
</head>
<body>
    {% include 'navbar.html' %}
    <div class="container my-4">
        {% block contenido %}{% endblock %}
    </div>

    {% include 'footer.html' %}

    <!-- Bootstrap JS (CDN) -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    {% block extra_js %}{% endblock %}
</body>
</html>


templates/navbar.html (según paso 18 — menú con iconos en opciones principales, submenús sin iconos)

<nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm">
  <div class="container-fluid">
    <a class="navbar-brand" href="{% url 'inicio_disneystore' %}">
      <strong class="text-primary">Sistema de Administración Cinepolis</strong>
    </a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMenu">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navMenu">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'inicio_disneystore' %}">Inicio</a></li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
            <span class="nav-icon">📦</span> Categoría
          </a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'agregar_productos' %}">Agregar productos</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_productos' %}">Ver productos</a></li>
            <!-- actualizar y borrar se acceden desde la lista -->
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
            <span class="nav-icon">👥</span> Clientes
          </a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar clientes</a></li>
            <li><a class="dropdown-item" href="#">Ver clientes</a></li>
          </ul>
        </li>

        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">
            <span class="nav-icon">🧾</span> Pedidos
          </a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar pedidos</a></li>
            <li><a class="dropdown-item" href="#">Ver pedidos</a></li>
          </ul>
        </li>

      </ul>
    </div>
  </div>
</nav>


templates/footer.html (paso 19 — fijo abajo)

<footer class="footer-fixed text-center">
  <div class="container">
    <small>
      &copy; {{ now.year }} - Creado por Cardiel Sanchez Kevin, Cbtis 128. Todos los derechos reservados.
    </small>
  </div>
</footer>


templates/inicio.html (paso 20 — imagen desde la red sobre cinepolis)

{% extends "base.html" %}
{% block titulo %}Inicio - DisneyStore{% endblock %}

{% block contenido %}
  <div class="row">
    <div class="col-md-8">
      <h2>Bienvenido al sistema de administración</h2>
      <p>Información del sistema y enlaces rápidos.</p>

      <h4>Productos recientes</h4>
      <div class="row">
        {% for p in productos_recientes %}
          <div class="col-md-4 mb-3">
            <div class="card">
              <div class="card-body">
                <h6 class="card-title">{{ p.nombre }}</h6>
                <p class="card-text">{{ p.descripcion|truncatechars:60 }}</p>
                <small>{{ p.categoria }}</small>
              </div>
            </div>
          </div>
        {% empty %}
          <p>No hay productos.</p>
        {% endfor %}
      </div>
    </div>

    <div class="col-md-4">
      <h5>Imagen Cinepolis</h5>
      <img src="https://upload.wikimedia.org/wikipedia/commons/6/6a/Cinepolis_logo.png" alt="Cinepolis" class="img-fluid">
    </div>
  </div>
{% endblock %}


NOTA: la imagen usa una URL pública de ejemplo; puedes sustituirla.

10) Plantillas de productos (paso 22: dentro templates/productos)

templates/productos/agregar_productos.html:

{% extends "base.html" %}
{% block contenido %}
<h3>Agregar Producto</h3>
<form method="post">
  {% csrf_token %}
  <div class="mb-3">
    <label>Nombre</label>
    <input name="nombre" class="form-control" required>
  </div>
  <div class="mb-3">
    <label>Categoría</label>
    <select name="categoria" class="form-select">
      <option value="">-- Seleccionar --</option>
      {% for c in categorias %}
      <option value="{{ c.id }}">{{ c.nombre }}</option>
      {% endfor %}
    </select>
  </div>
  <div class="mb-3">
    <label>Descripción</label>
    <textarea name="descripcion" class="form-control"></textarea>
  </div>
  <div class="mb-3 row">
    <div class="col">
      <label>Precio</label>
      <input name="precio" class="form-control" type="number" step="0.01">
    </div>
    <div class="col">
      <label>Stock</label>
      <input name="stock" class="form-control" type="number">
    </div>
  </div>
  <div class="mb-3">
    <label>Fecha lanzamiento</label>
    <input name="fecha_lanzamiento" class="form-control" type="date">
  </div>
  <div class="form-check mb-3">
    <input name="en_promocion" class="form-check-input" type="checkbox" id="promo">
    <label for="promo" class="form-check-label">En promoción</label>
  </div>
  <button class="btn btn-primary">Agregar</button>
</form>
{% endblock %}


templates/productos/ver_productos.html (tabla con botones ver, editar, borrar):

{% extends "base.html" %}
{% block contenido %}
<h3>Lista de Productos</h3>
<table class="table table-striped">
  <thead>
    <tr>
      <th>ID</th>
      <th>Nombre</th>
      <th>Categoría</th>
      <th>Precio</th>
      <th>Stock</th>
      <th>Acciones</th>
    </tr>
  </thead>
  <tbody>
    {% for p in productos %}
    <tr>
      <td>{{ p.id }}</td>
      <td>{{ p.nombre }}</td>
      <td>{{ p.categoria.nombre }}</td>
      <td>${{ p.precio }}</td>
      <td>{{ p.stock }}</td>
      <td>
        <!-- Ver (puede ser modal o página separada) -->
        <a class="btn btn-sm btn-outline-secondary" href="#">Ver</a>
        <a class="btn btn-sm btn-warning" href="{% url 'actualizar_productos' p.id %}">Editar</a>
        <a class="btn btn-sm btn-danger" href="{% url 'borrar_productos' p.id %}">Borrar</a>
      </td>
    </tr>
    {% empty %}
    <tr><td colspan="6">No hay productos registrados.</td></tr>
    {% endfor %}
  </tbody>
</table>
{% endblock %}


templates/productos/actualizar_productos.html:

{% extends "base.html" %}
{% block contenido %}
<h3>Actualizar Producto: {{ producto.nombre }}</h3>
<form method="post" action="{% url 'realizar_actualizacion_productos' producto.id %}">
  {% csrf_token %}
  <div class="mb-3">
    <label>Nombre</label>
    <input name="nombre" value="{{ producto.nombre }}" class="form-control">
  </div>
  <div class="mb-3">
    <label>Categoría</label>
    <select name="categoria" class="form-select">
      <option value="">-- Seleccionar --</option>
      {% for c in categorias %}
      <option value="{{ c.id }}" {% if producto.categoria and producto.categoria.id == c.id %}selected{% endif %}>{{ c.nombre }}</option>
      {% endfor %}
    </select>
  </div>
  <div class="mb-3">
    <label>Descripción</label>
    <textarea name="descripcion" class="form-control">{{ producto.descripcion }}</textarea>
  </div>
  <div class="mb-3 row">
    <div class="col">
      <label>Precio</label>
      <input name="precio" value="{{ producto.precio }}" class="form-control" type="number" step="0.01">
    </div>
    <div class="col">
      <label>Stock</label>
      <input name="stock" value="{{ producto.stock }}" class="form-control" type="number">
    </div>
  </div>
  <div class="mb-3">
    <label>Fecha lanzamiento</label>
    <input name="fecha_lanzamiento" value="{{ producto.fecha_lanzamiento }}" class="form-control" type="date">
  </div>
  <div class="form-check mb-3">
    <input name="en_promocion" class="form-check-input" type="checkbox" id="promo" {% if producto.en_promocion %}checked{% endif %}>
    <label for="promo" class="form-check-label">En promoción</label>
  </div>
  <button class="btn btn-primary">Guardar cambios</button>
</form>
{% endblock %}


templates/productos/borrar_productos.html:

{% extends "base.html" %}
{% block contenido %}
<h3>Confirmar borrado</h3>
<p>¿Deseas eliminar el producto <strong>{{ producto.nombre }}</strong> ?</p>
<form method="post">
  {% csrf_token %}
  <a href="{% url 'ver_productos' %}" class="btn btn-secondary">Cancelar</a>
  <button class="btn btn-danger" type="submit">Eliminar</button>
</form>
{% endblock %}

11) Sin validaciones de entrada (paso 28 y 23)

Como pediste, no agregué validaciones extra, solo uso de request.POST y csrf_token. Si quieres validaciones en el futuro, lo hacemos con forms o JavaScript.

12) Migraciones y admin (paso 12.5 y 27 resumen)

Pasos para crear migraciones y crear superusuario:

# Después de registrar app_Disney en settings.py
python manage.py makemigrations app_Disney
python manage.py migrate

# Crear superusuario para acceder a /admin/
python manage.py createsuperuser


Luego python manage.py runserver 8006 para probar.

13) Puntos extra / consideraciones (pasos 27, 30, 31)

Trabajamos por ahora sólo con productos; Cliente y Pedido los dejé pendientes: si quieres los reuso later, puedo incorporarlos.

Diseño: plantillas usan colores suaves y bootstrap para una apariencia moderna. Código sencillo y listo para expandir.

Estructura completa creada según punto 29.

Proyecto funcional: al ejecutar runserver 8006 y crear algunas Categoria en admin, podrás agregar productos desde la app.

14) Archivos que ya mostré en la respuesta (resumen para copiar/pegar)

app_Disney/models.py (con Categoria y Producto)

app_Disney/admin.py

app_Disney/views.py

app_Disney/urls.py

backend_DisneyStore/urls.py (modificación)

backend_DisneyStore/settings.py (añadir 'app_Disney' en INSTALLED_APPS)

Plantillas en templates/ y templates/productos/ (código incluido)

Si quieres, lo siguiente que puedo hacer ahora (elige uno — NO esperaré a que confirmes, simplemente te lo genero si lo pides explícitamente):

Generar un archivo zip listo para descargar con toda la estructura y archivos (si quieres el .zip lo puedo preparar).

Añadir Cliente y Pedido ya conectados con Pedido.productos como ManyToMany (si prefieres que los incluya ya).

Añadir validaciones front-end (JS) o mensajes flash tras CRUD.

Reemplazar la imagen de Cinepolis por otra URL que prefieras.
