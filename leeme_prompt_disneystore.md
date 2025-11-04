¡Claro! Aquí tienes la primera parte del proyecto DisneyStore con todos los pasos y el código que solicitaste.

---

**1. Procedimiento para crear carpeta del Proyecto:**

Abre tu terminal o símbolo del sistema y ejecuta:

```bash
mkdir UIII_DisneyStore_0530
```

**2. Procedimiento para abrir VS Code sobre la carpeta `UIII_DisneyStore_0530`:**

```bash
cd UIII_DisneyStore_0530
code .
```

**3. Procedimiento para abrir terminal en VS Code:**

Dentro de VS Code, ve a `Terminal > New Terminal` o usa el atajo `Ctrl + Shift + ` (Windows/Linux) o `Cmd + Shift + ` (macOS).

**4. Procedimiento para crear carpeta entorno virtual “.venv” desde terminal de VS Code:**

```bash
python -m venv .venv
```

**5. Procedimiento para activar el entorno virtual:**

*   **En Windows:**
    ```bash
    .venv\Scripts\activate
    ```
*   **En macOS/Linux:**
    ```bash
    source .venv/bin/activate
    ```

**6. Procedimiento para activar intérprete de Python:**

Una vez activado el entorno virtual, VS Code debería detectarlo automáticamente. Si no, presiona `Ctrl + Shift + P` (o `Cmd + Shift + P`) y busca "Python: Select Interpreter", luego selecciona la opción que apunta a `.venv/Scripts/python.exe` (Windows) o `.venv/bin/python` (macOS/Linux).

**7. Procedimiento para instalar Django:**

```bash
pip install Django
```

**8. Procedimiento para crear proyecto `backend_DisneyStore` sin duplicar carpeta:**

Asegúrate de estar en la carpeta `UIII_DisneyStore_0530` con el entorno virtual activado.

```bash
django-admin startproject backend_DisneyStore .
```

**9. Procedimiento para ejecutar servidor en el puerto 8006:**

```bash
python manage.py runserver 8006
```

**10. Procedimiento para copiar y pegar el link en el navegador:**

Una vez que el servidor se esté ejecutando, verás un mensaje como: `Starting development server at http://127.0.0.1:8006/`. Copia esa URL y pégala en tu navegador.

**11. Procedimiento para crear aplicación `app_Disney`:**

Asegúrate de que el servidor no esté corriendo (`Ctrl + C` si lo está) y ejecuta:

```bash
python manage.py startapp app_Disney
```

**12. Aquí el modelo `models.py` (dentro de `app_Disney/models.py`):**

```python
# app_Disney/models.py

from django.db import models

# ======================
# MODELO PRODUCTO
# ======================
class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    categoria = models.CharField(max_length=50)
    descripcion = models.TextField()
    precio = models.DecimalField(max_digits=8, decimal_places=2)
    stock = models.PositiveIntegerField()
    fecha_lanzamiento = models.DateField()
    en_promocion = models.BooleanField(default=False)
    imagen = models.ImageField(upload_to='productos/', null=True, blank=True) # Campo para imagen

    def __str__(self):
        return f"{self.nombre} - ${self.precio}"

# ======================
# MODELO CLIENTE
# ======================
class Cliente(models.Model):
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    correo = models.EmailField(unique=True)
    telefono = models.CharField(max_length=15)
    direccion = models.CharField(max_length=200)
    fecha_registro = models.DateField(auto_now_add=True)
    puntos_disney = models.PositiveIntegerField(default=0)

    def __str__(self):
        return f"{self.nombre} {self.apellido}"

# ======================
# MODELO PEDIDO
# ======================
class Pedido(models.Model):
    cliente = models.ForeignKey(
        Cliente, on_delete=models.CASCADE, related_name='pedidos'
    )  # Relación 1-N (un cliente puede tener muchos pedidos)

    productos = models.ManyToManyField(
        Producto, related_name='pedidos'
    )  # Relación N-N (un pedido puede tener varios productos)

    fecha_pedido = models.DateTimeField(auto_now_add=True)
    estado = models.CharField(max_length=50, choices=[
        ('Pendiente', 'Pendiente'),
        ('Enviado', 'Enviado'),
        ('Entregado', 'Entregado'),
        ('Cancelado', 'Cancelado'),
    ])
    total = models.DecimalField(max_digits=10, decimal_places=2)
    metodo_pago = models.CharField(max_length=50)
    direccion_envio = models.CharField(max_length=200)

    def __str__(self):
        return f"Pedido #{self.id} - {self.cliente.nombre} ({self.estado})"

```

**12.5 Procedimiento para realizar las migraciones (`makemigrations` y `migrate`):**

```bash
python manage.py makemigrations
python manage.py migrate
```

**13. Primero trabajamos con el MODELO: `PRODUCTO`** (El paso 13 lo mencionas pero luego te refieres a Categoría, me enfocaré en Producto como lo pides más adelante para el CRUD).

**14. En `views.py` de `app_Disney` crear las funciones con sus códigos correspondientes:**

```python
# app_Disney/views.py

from django.shortcuts import render, redirect, get_object_or_404
from .models import Producto
from datetime import date # Importa date para el campo fecha_lanzamiento

def inicio_disneystore(request):
    return render(request, 'app_Disney/inicio.html')

def agregar_productos(request):
    if request.method == 'POST':
        nombre = request.POST['nombre']
        categoria = request.POST['categoria']
        descripcion = request.POST['descripcion']
        precio = request.POST['precio']
        stock = request.POST['stock']
        # Usamos date.today() como valor por defecto si no se proporciona, o asumimos un campo de entrada de fecha
        fecha_lanzamiento = request.POST.get('fecha_lanzamiento', date.today())
        en_promocion = request.POST.get('en_promocion') == 'on' # Checkbox
        imagen = request.FILES.get('imagen') # Para manejar la carga de imágenes

        Producto.objects.create(
            nombre=nombre,
            categoria=categoria,
            descripcion=descripcion,
            precio=precio,
            stock=stock,
            fecha_lanzamiento=fecha_lanzamiento,
            en_promocion=en_promocion,
            imagen=imagen
        )
        return redirect('ver_productos')
    return render(request, 'app_Disney/productos/agregar_productos.html')

def ver_productos(request):
    productos = Producto.objects.all()
    return render(request, 'app_Disney/productos/ver_productos.html', {'productos': productos})

def actualizar_productos(request, pk):
    producto = get_object_or_404(Producto, pk=pk)
    if request.method == 'POST':
        producto.nombre = request.POST['nombre']
        producto.categoria = request.POST['categoria']
        producto.descripcion = request.POST['descripcion']
        producto.precio = request.POST['precio']
        producto.stock = request.POST['stock']
        producto.fecha_lanzamiento = request.POST.get('fecha_lanzamiento', producto.fecha_lanzamiento)
        producto.en_promocion = request.POST.get('en_promocion') == 'on'
        
        # Manejar la actualización de la imagen
        if 'imagen' in request.FILES:
            producto.imagen = request.FILES['imagen']

        producto.save()
        return redirect('ver_productos')
    return render(request, 'app_Disney/productos/actualizar_productos.html', {'producto': producto})

# En Django, la función de actualización se suele usar para "mostrar el formulario de edición"
# y "procesar el envío del formulario". El nombre `realizar_actualizacion_productos`
# podría ser redundante si `actualizar_productos` ya maneja ambos.
# Por simplicidad, unificaremos en `actualizar_productos` el mostrar el formulario y procesar la actualización.
# Si prefieres una función separada para procesar, podríamos hacer esto:
# def realizar_actualizacion_productos(request, pk):
#     producto = get_object_or_404(Producto, pk=pk)
#     if request.method == 'POST':
#         # Lógica de actualización
#         producto.save()
#         return redirect('ver_productos')
#     return redirect('actualizar_productos', pk=pk) # Redirige de vuelta al formulario si no es POST válido o para mostrarlo

def borrar_productos(request, pk):
    producto = get_object_or_404(Producto, pk=pk)
    if request.method == 'POST': # Confirmación de borrado
        producto.delete()
        return redirect('ver_productos')
    return render(request, 'app_Disney/productos/borrar_productos.html', {'producto': producto})

```

**15. Crear la carpeta “templates” dentro de “app_Disney”:**

```
UIII_DisneyStore_0530/app_Disney/templates/
```

**16. En la carpeta `templates` crear los archivos html:**

```
UIII_DisneyStore_0530/app_Disney/templates/base.html
UIII_DisneyStore_0530/app_Disney/templates/header.html
UIII_DisneyStore_0530/app_Disney/templates/navbar.html
UIII_DisneyStore_0530/app_Disney/templates/footer.html
UIII_DisneyStore_0530/app_Disney/templates/inicio.html
```

**17. En el archivo `base.html` agregar Bootstrap para CSS y JS:**

```html
<!-- app_Disney/templates/base.html -->
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}DisneyStore{% endblock %}</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
    <!-- Font Awesome para iconos -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css" integrity="sha512-SnH5WK+bZxgPHs44uWIX+LLJAJ9/2PkPKZ5QiAj6Ta86w+fsb2TkcmfRyVX3pBnMFcV7oQPJkl9QevSCWr3W6A==" crossorigin="anonymous" referrerpolicy="no-referrer" />
    <style>
        body {
            display: flex;
            flex-direction: column;
            min-height: 100vh;
        }
        .content {
            flex: 1;
            padding-bottom: 70px; /* Espacio para el footer fijo */
        }
        .footer {
            position: fixed;
            bottom: 0;
            width: 100%;
            height: 60px; /* Altura del footer */
            line-height: 60px; /* Centrar verticalmente el texto */
            background-color: #f5f5f5;
            text-align: center;
        }
        /* Colores Disney sugeridos */
        .navbar-disney {
            background-color: #007bff; /* Azul primario de Disney */
            color: white;
        }
        .navbar-disney .nav-link, .navbar-disney .navbar-brand {
            color: white !important;
        }
        .navbar-disney .dropdown-menu {
            background-color: #0056b3; /* Azul más oscuro para dropdown */
        }
        .navbar-disney .dropdown-item {
            color: white;
        }
        .navbar-disney .dropdown-item:hover {
            background-color: #003b7f;
        }
    </style>
</head>
<body>
    {% include 'header.html' %}
    {% include 'navbar.html' %}

    <div class="container mt-4 content">
        {% block content %}
        {% endblock %}
    </div>

    {% include 'footer.html' %}

    <!-- Bootstrap JS y Popper.js -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz" crossorigin="anonymous"></script>
</body>
</html>
```

**`header.html` (Puedes dejarlo simple o añadir algo):**

```html
<!-- app_Disney/templates/header.html -->
<header class="bg-primary text-white text-center py-3">
    <h1>Administración DisneyStore</h1>
</header>
```

**18. En el archivo `navbar.html` incluir las opciones:**

```html
<!-- app_Disney/templates/navbar.html -->
<nav class="navbar navbar-expand-lg navbar-disney">
    <div class="container-fluid">
        <a class="navbar-brand" href="{% url 'inicio_disneystore' %}">
            <i class="fas fa-magic me-2"></i>Sistema de Administración DisneyStore
        </a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNavDropdown" aria-controls="navbarNavDropdown" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNavDropdown">
            <ul class="navbar-nav ms-auto">
                <li class="nav-item">
                    <a class="nav-link active" aria-current="page" href="{% url 'inicio_disneystore' %}"><i class="fas fa-home me-1"></i>Inicio</a>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                        <i class="fas fa-box me-1"></i>Productos
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="{% url 'agregar_productos' %}">Agregar Productos</a></li>
                        <li><a class="dropdown-item" href="{% url 'ver_productos' %}">Ver Productos</a></li>
                        <!-- La actualización y el borrado se manejarán desde "Ver Productos" -->
                    </ul>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                        <i class="fas fa-users me-1"></i>Clientes
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="#">Agregar Clientes</a></li>
                        <li><a class="dropdown-item" href="#">Ver Clientes</a></li>
                        <li><a class="dropdown-item" href="#">Actualizar Clientes</a></li>
                        <li><a class="dropdown-item" href="#">Borrar Clientes</a></li>
                    </ul>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                        <i class="fas fa-shopping-cart me-1"></i>Pedidos
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="#">Agregar Pedidos</a></li>
                        <li><a class="dropdown-item" href="#">Ver Pedidos</a></li>
                        <li><a class="dropdown-item" href="#">Actualizar Pedidos</a></li>
                        <li><a class="dropdown-item" href="#">Borrar Pedidos</a></li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</nav>
```

**19. En el archivo `footer.html` incluir derechos de autor, fecha del sistema y “Creado por Cardiel Sanchez Kevin, Cbtis 128” y mantenerla fija al final de la página.**

```html
<!-- app_Disney/templates/footer.html -->
<footer class="footer bg-dark text-white py-3">
    <div class="container">
        <span class="text-muted">
            &copy; {% now "Y" %} DisneyStore. Todos los derechos reservados. | 
            Fecha del sistema: {% now "d/m/Y H:i" %} | 
            Creado por Cardiel Sanchez Kevin, Cbtis 128
        </span>
    </div>
</footer>
```

**20. En el archivo `inicio.html` se usa para colocar información del sistema más una imagen tomada desde la red sobre Disney:**

```html
<!-- app_Disney/templates/inicio.html -->
{% extends 'base.html' %}

{% block title %}Inicio - DisneyStore{% endblock %}

{% block content %}
<div class="container text-center py-5">
    <h2 class="mb-4">Bienvenido al Sistema de Administración DisneyStore</h2>
    <p class="lead">Gestiona fácilmente tus productos, clientes y pedidos de la tienda Disney.</p>
    <div class="row justify-content-center mt-5">
        <div class="col-md-8">
            <img src="https://wallpapers.com/images/hd/disney-castle-hd-desktop-2560-x-1440-wallpaper-h5791z7s1330l5y6.jpg" alt="Castillo Disney" class="img-fluid rounded shadow-lg">
            <p class="mt-3 text-muted">Explora el mágico mundo de Disney.</p>
        </div>
    </div>
</div>
{% endblock %}
```
`
**21. Crear la subcarpeta `productos` dentro de `app_Disney/templates`:**

```
UIII_DisneyStore_0530/app_Disney/templates/productos/
```

**22. Crear los archivos html con su código correspondiente de `agregar_productos.html`, `ver_productos.html` (mostrar en tabla con los botones ver, editar y borrar), `actualizar_productos.html`, `borrar_productos.html` dentro de `app_Disney/templates/productos`.**

**`agregar_productos.html`:**

```html
<!-- app_Disney/templates/productos/agregar_productos.html -->
{% extends 'base.html' %}

{% block title %}Agregar Productos - DisneyStore{% endblock %}

{% block content %}
<div class="container">
    <h2 class="mb-4 text-center">Agregar Nuevo Producto</h2>
    <div class="card shadow-sm mx-auto" style="max-width: 600px;">
        <div class="card-body">
            <form method="post" enctype="multipart/form-data">
                {% csrf_token %}
                <div class="mb-3">
                    <label for="nombre" class="form-label">Nombre del Producto:</label>
                    <input type="text" class="form-control" id="nombre" name="nombre" required>
                </div>
                <div class="mb-3">
                    <label for="categoria" class="form-label">Categoría:</label>
                    <input type="text" class="form-control" id="categoria" name="categoria" required>
                </div>
                <div class="mb-3">
                    <label for="descripcion" class="form-label">Descripción:</label>
                    <textarea class="form-control" id="descripcion" name="descripcion" rows="3" required></textarea>
                </div>
                <div class="row">
                    <div class="col-md-6 mb-3">
                        <label for="precio" class="form-label">Precio:</label>
                        <input type="number" step="0.01" class="form-control" id="precio" name="precio" required>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label for="stock" class="form-label">Stock:</label>
                        <input type="number" class="form-control" id="stock" name="stock" required>
                    </div>
                </div>
                <div class="mb-3">
                    <label for="fecha_lanzamiento" class="form-label">Fecha de Lanzamiento:</label>
                    <input type="date" class="form-control" id="fecha_lanzamiento" name="fecha_lanzamiento" value="{{ current_date|date:'Y-m-d' }}">
                </div>
                <div class="mb-3 form-check">
                    <input type="checkbox" class="form-check-input" id="en_promocion" name="en_promocion">
                    <label class="form-check-label" for="en_promocion">En Promoción</label>
                </div>
                <div class="mb-3">
                    <label for="imagen" class="form-label">Imagen del Producto:</label>
                    <input type="file" class="form-control" id="imagen" name="imagen" accept="image/*">
                </div>
                <button type="submit" class="btn btn-primary w-100">Guardar Producto</button>
                <a href="{% url 'ver_productos' %}" class="btn btn-secondary w-100 mt-2">Cancelar</a>
            </form>
        </div>
    </div>
</div>
{% endblock %}
```

**`ver_productos.html`:**

```html
<!-- app_Disney/templates/productos/ver_productos.html -->
{% extends 'base.html' %}
{% load static %}

{% block title %}Ver Productos - DisneyStore{% endblock %}

{% block content %}
<div class="container">
    <h2 class="mb-4 text-center">Lista de Productos</h2>
    <div class="text-end mb-3">
        <a href="{% url 'agregar_productos' %}" class="btn btn-success"><i class="fas fa-plus-circle me-1"></i>Agregar Nuevo Producto</a>
    </div>
    {% if productos %}
    <div class="table-responsive">
        <table class="table table-striped table-hover shadow-sm">
            <thead class="bg-primary text-white">
                <tr>
                    <th>ID</th>
                    <th>Imagen</th>
                    <th>Nombre</th>
                    <th>Categoría</th>
                    <th>Precio</th>
                    <th>Stock</th>
                    <th>Promoción</th>
                    <th>Acciones</th>
                </tr>
            </thead>
            <tbody>
                {% for producto in productos %}
                <tr>
                    <td>{{ producto.id }}</td>
                    <td>
                        {% if producto.imagen %}
                            <img src="{{ producto.imagen.url }}" alt="{{ producto.nombre }}" class="img-thumbnail" style="width: 50px; height: 50px; object-fit: cover;">
                        {% else %}
                            <img src="{% static 'img/default_product.png' %}" alt="Sin imagen" class="img-thumbnail" style="width: 50px; height: 50px; object-fit: cover;">
                        {% endif %}
                    </td>
                    <td>{{ producto.nombre }}</td>
                    <td>{{ producto.categoria }}</td>
                    <td>${{ producto.precio|floatformat:2 }}</td>
                    <td>{{ producto.stock }}</td>
                    <td>
                        {% if producto.en_promocion %}
                            <span class="badge bg-success">Sí</span>
                        {% else %}
                            <span class="badge bg-danger">No</span>
                        {% endif %}
                    </td>
                    <td>
                        <a href="#" class="btn btn-info btn-sm me-1" title="Ver Detalles"><i class="fas fa-eye"></i></a>
                        <a href="{% url 'actualizar_productos' producto.pk %}" class="btn btn-warning btn-sm me-1" title="Editar"><i class="fas fa-edit"></i></a>
                        <a href="{% url 'borrar_productos' producto.pk %}" class="btn btn-danger btn-sm" title="Eliminar"><i class="fas fa-trash-alt"></i></a>
                    </td>
                </tr>
                {% endfor %}
            </tbody>
        </table>
    </div>
    {% else %}
    <div class="alert alert-info text-center" role="alert">
        No hay productos registrados. <a href="{% url 'agregar_productos' %}">¡Agrega uno ahora!</a>
    </div>
    {% endif %}
</div>
{% endblock %}
```

**`actualizar_productos.html`:**

```html
<!-- app_Disney/templates/productos/actualizar_productos.html -->
{% extends 'base.html' %}

{% block title %}Actualizar Producto - DisneyStore{% endblock %}

{% block content %}
<div class="container">
    <h2 class="mb-4 text-center">Actualizar Producto: {{ producto.nombre }}</h2>
    <div class="card shadow-sm mx-auto" style="max-width: 600px;">
        <div class="card-body">
            <form method="post" enctype="multipart/form-data">
                {% csrf_token %}
                <div class="mb-3">
                    <label for="nombre" class="form-label">Nombre del Producto:</label>
                    <input type="text" class="form-control" id="nombre" name="nombre" value="{{ producto.nombre }}" required>
                </div>
                <div class="mb-3">
                    <label for="categoria" class="form-label">Categoría:</label>
                    <input type="text" class="form-control" id="categoria" name="categoria" value="{{ producto.categoria }}" required>
                </div>
                <div class="mb-3">
                    <label for="descripcion" class="form-label">Descripción:</label>
                    <textarea class="form-control" id="descripcion" name="descripcion" rows="3" required>{{ producto.descripcion }}</textarea>
                </div>
                <div class="row">
                    <div class="col-md-6 mb-3">
                        <label for="precio" class="form-label">Precio:</label>
                        <input type="number" step="0.01" class="form-control" id="precio" name="precio" value="{{ producto.precio }}" required>
                    </div>
                    <div class="col-md-6 mb-3">
                        <label for="stock" class="form-label">Stock:</label>
                        <input type="number" class="form-control" id="stock" name="stock" value="{{ producto.stock }}" required>
                    </div>
                </div>
                <div class="mb-3">
                    <label for="fecha_lanzamiento" class="form-label">Fecha de Lanzamiento:</label>
                    <input type="date" class="form-control" id="fecha_lanzamiento" name="fecha_lanzamiento" value="{{ producto.fecha_lanzamiento|date:'Y-m-d' }}">
                </div>
                <div class="mb-3 form-check">
                    <input type="checkbox" class="form-check-input" id="en_promocion" name="en_promocion" {% if producto.en_promocion %}checked{% endif %}>
                    <label class="form-check-label" for="en_promocion">En Promoción</label>
                </div>
                <div class="mb-3">
                    <label for="imagen" class="form-label">Imagen Actual:</label>
                    {% if producto.imagen %}
                        <img src="{{ producto.imagen.url }}" alt="{{ producto.nombre }}" class="img-thumbnail mb-2" style="max-width: 150px;">
                    {% else %}
                        <p>No hay imagen actual.</p>
                    {% endif %}
                    <label for="imagen" class="form-label">Cambiar Imagen:</label>
                    <input type="file" class="form-control" id="imagen" name="imagen" accept="image/*">
                </div>
                <button type="submit" class="btn btn-primary w-100">Guardar Cambios</button>
                <a href="{% url 'ver_productos' %}" class="btn btn-secondary w-100 mt-2">Cancelar</a>
            </form>
        </div>
    </div>
</div>
{% endblock %}
```

**`borrar_productos.html`:**

```html
<!-- app_Disney/templates/productos/borrar_productos.html -->
{% extends 'base.html' %}

{% block title %}Borrar Producto - DisneyStore{% endblock %}

{% block content %}
<div class="container">
    <h2 class="mb-4 text-center">Confirmar Borrado de Producto</h2>
    <div class="card border-danger shadow-sm mx-auto" style="max-width: 500px;">
        <div class="card-header bg-danger text-white">
            Advertencia
        </div>
        <div class="card-body">
            <p class="card-text">¿Estás seguro de que deseas eliminar el producto **"{{ producto.nombre }}"**?</p>
            <p class="card-text text-danger"><i class="fas fa-exclamation-triangle me-2"></i>Esta acción no se puede deshacer.</p>
            <form method="post">
                {% csrf_token %}
                <button type="submit" class="btn btn-danger w-100 mb-2">Sí, Eliminar</button>
                <a href="{% url 'ver_productos' %}" class="btn btn-secondary w-100">Cancelar</a>
            </form>
        </div>
    </div>
</div>
{% endblock %}
```

**23. No utilizar `forms.py`.** (Ya se ha manejado la lógica de formularios directamente en las vistas y templates).

**24. Procedimiento para crear el archivo `urls.py` en `app_Disney` con el código correspondiente para acceder a las funciones de `views.py` para operaciones de crud en productos.**

Crea el archivo: `UIII_DisneyStore_0530/app_Disney/urls.py`

```python
# app_Disney/urls.py

from django.urls import path
from . import views
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('', views.inicio_disneystore, name='inicio_disneystore'),
    path('productos/agregar/', views.agregar_productos, name='agregar_productos'),
    path('productos/', views.ver_productos, name='ver_productos'),
    path('productos/actualizar/<int:pk>/', views.actualizar_productos, name='actualizar_productos'),
    path('productos/borrar/<int:pk>/', views.borrar_productos, name='borrar_productos'),
]

# Configuración para servir archivos de medios (imágenes) durante el desarrollo
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

**25. Procedimiento para agregar `app_Disney` en `settings.py` de `backend_DisneyStore`:**

Abre `UIII_DisneyStore_0530/backend_DisneyStore/settings.py` y busca `INSTALLED_APPS`. Agrega `'app_Disney'` a la lista:

```python
# backend_DisneyStore/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_Disney', # <--- Agrega tu aplicación aquí
]

# ...

# Configuración para archivos de medios (imágenes)
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

**26. Realizar las configuraciones correspondiente a `urls.py` de `backend_DisneyStore` para enlazar con `app_Disney`:**

Abre `UIII_DisneyStore_0530/backend_DisneyStore/urls.py`:

```python
# backend_DisneyStore/urls.py

from django.contrib import admin
from django.urls import path, include # Importa include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Disney.urls')), # <--- Enlaza las URLs de app_Disney aquí
]

# Para servir archivos de medios (imágenes) en desarrollo
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

    ¡Tienes toda la razón! Mil disculpas. En mi respuesta anterior, se cortaron algunos pasos y el registro de los modelos. Aquí te completo lo que faltaba y retomo desde donde nos quedamos, asegurándome de cubrir todo lo que pediste.

---

**Retomando desde el paso 27:**

**27. Procedimiento para registrar los modelos en `admin.py` y volver a realizar las migraciones:**

Abre `UIII_DisneyStore_0530/app_Disney/admin.py` y registra tus modelos:

```python
# app_Disney/admin.py

from django.contrib import admin
from .models import Producto, Cliente, Pedido

@admin.register(Producto)
class ProductoAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'categoria', 'precio', 'stock', 'en_promocion', 'fecha_lanzamiento')
    list_filter = ('categoria', 'en_promocion', 'fecha_lanzamiento')
    search_fields = ('nombre', 'descripcion', 'categoria')
    ordering = ('nombre',) # Ordenar por nombre por defecto

@admin.register(Cliente)
class ClienteAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'apellido', 'correo', 'telefono', 'fecha_registro', 'puntos_disney')
    list_filter = ('fecha_registro',)
    search_fields = ('nombre', 'apellido', 'correo', 'telefono')

@admin.register(Pedido)
class PedidoAdmin(admin.ModelAdmin):
    list_display = ('id', 'cliente', 'fecha_pedido', 'estado', 'total', 'metodo_pago')
    list_filter = ('estado', 'metodo_pago', 'fecha_pedido')
    search_fields = ('cliente__nombre', 'cliente__apellido', 'estado')
    date_hierarchy = 'fecha_pedido'
    raw_id_fields = ('cliente',) # Permite buscar cliente por ID
    filter_horizontal = ('productos',) # Mejora la interfaz para ManyToMany
```

**Ahora, después de registrar en `admin.py`, y dado que hemos añadido un campo `imagen` al modelo `Producto`, necesitamos nuevas migraciones:**

```bash
python manage.py makemigrations
python manage.py migrate
```

**Además, para poder subir imágenes, Django necesita la librería `Pillow`. Instálala:**

```bash
pip install Pillow
```

**27 (Continuación). Por lo pronto solo trabajar con “productos” dejar pendiente `# MODELO: CLIENTE` y `# MODELO: PEDIDO`**

(Esto ya lo hemos implementado en las vistas y URLs, enfocándonos solo en el CRUD de `Producto`.)

**28. Utilizar colores suaves, atractivos y modernos, el código de las páginas web sencillas.**

(Esto se ha abordado con la inclusión de Bootstrap 5 y los estilos CSS personalizados en `base.html` y `navbar.html` con la clase `navbar-disney` para un look moderno y suave. La imagen del castillo de Disney en `inicio.html` también contribuye a la estética.)

**28 (Continuación). No validar entrada de datos.**

(Las vistas actuales no incluyen validación explícita más allá de lo que Django maneja por defecto en el modelo, como tipos de datos.)

**29. Al inicio crear la estructura completa de carpetas y archivos.**

Vamos a asegurarnos de que la estructura de carpetas que necesitas para todo el proyecto esté clara.

Tu estructura de proyecto debería verse así después de seguir todos los pasos:

```
UIII_DisneyStore_0530/
├── .venv/                      # Entorno virtual
├── backend_DisneyStore/        # Proyecto principal de Django
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py             # Configuración del proyecto
│   ├── urls.py                 # URLs principales del proyecto
│   └── wsgi.py
├── app_Disney/                 # Aplicación Django
│   ├── migrations/
│   │   ├── __init__.py
│   │   └── 0001_initial.py     # Migración inicial
│   │   └── 0002_producto_imagen.py # Migración para el campo imagen
│   ├── static/                 # Para archivos estáticos específicos de la app (si los necesitas)
│   │   └── img/
│   │       └── default_product.png # (Crear esta imagen por defecto si quieres)
│   ├── templates/
│   │   └── app_Disney/
│   │       ├── base.html
│   │       ├── header.html
│   │       ├── navbar.html
│   │       ├── footer.html
│   │       └── inicio.html
│   │       └── productos/
│   │           ├── agregar_productos.html
│   │           ├── ver_productos.html
│   │           ├── actualizar_productos.html
│   │           └── borrar_productos.html
│   ├── __init__.py
│   ├── admin.py                # Registro de modelos en el panel de admin
│   ├── apps.py
│   ├── models.py               # Definición de modelos
│   ├── tests.py
│   └── views.py                # Lógica de las vistas
│   └── urls.py                 # URLs de la aplicación
├── media/                      # Carpeta para archivos subidos (imágenes de productos)
│   └── productos/              # Django creará subcarpetas aquí
├── manage.py                   # Script para administrar el proyecto
└── db.sqlite3                  # Base de datos SQLite (se crea con las migraciones)
```

**30. Proyecto totalmente funcional.**

Con todos los pasos anteriores, incluyendo la configuración de URLs, vistas, modelos, templates y el entorno, tu proyecto debería ser funcional para las operaciones CRUD de productos.

**31. Finalmente ejecutar servidor en el puerto 8006.**

Asegúrate de estar en la carpeta raíz `UIII_DisneyStore_0530` y con el entorno virtual activado.

```bash
python manage.py runserver 8006
```

Luego, abre tu navegador y ve a `http://127.0.0.1:8006/` para ver la página de inicio. Desde allí, podrás navegar al CRUD de productos.

---

**Resumen de Archivos a Crear/Modificar y su Contenido (para claridad):**

**1. `UIII_DisneyStore_0530/backend_DisneyStore/settings.py`**
   - Añadir `'app_Disney'` a `INSTALLED_APPS`.
   - Añadir `MEDIA_URL` y `MEDIA_ROOT`.

**2. `UIII_DisneyStore_0530/backend_DisneyStore/urls.py`**
   - Añadir `path('', include('app_Disney.urls'))`.
   - Añadir configuración para `MEDIA_URL` en `DEBUG` mode.

**3. `UIII_DisneyStore_0530/app_Disney/models.py`**
   - Contiene los modelos `Producto`, `Cliente`, `Pedido`. (Asegúrate de tener el campo `imagen` en `Producto`.)

**4. `UIII_DisneyStore_0530/app_Disney/views.py`**
   - Funciones `inicio_disneystore`, `agregar_productos`, `ver_productos`, `actualizar_productos`, `borrar_productos`.

**5. `UIII_DisneyStore_0530/app_Disney/urls.py`**
   - Define las rutas para las vistas de `app_Disney`, incluyendo el CRUD de productos y la configuración de `MEDIA_URL`.

**6. `UIII_DisneyStore_0530/app_Disney/admin.py`**
   - Registra `Producto`, `Cliente`, `Pedido` con configuraciones `ModelAdmin`.

**7. `UIII_DisneyStore_0530/app_Disney/templates/base.html`**
   - Estructura HTML base, Bootstrap CSS/JS, Font Awesome, estilos generales y Disney.

**8. `UIII_DisneyStore_0530/app_Disney/templates/header.html`**
   - Encabezado del sistema.

**9. `UIII_DisneyStore_0530/app_Disney/templates/navbar.html`**
   - Barra de navegación con opciones y submenús.

**10. `UIII_DisneyStore_0530/app_Disney/templates/footer.html`**
    - Pie de página fijo con derechos y autor.

**11. `UIII_DisneyStore_0530/app_Disney/templates/inicio.html`**
    - Página de inicio con texto e imagen del castillo.

**12. `UIII_DisneyStore_0530/app_Disney/templates/productos/agregar_productos.html`**
    - Formulario para añadir productos.

**13. `UIII_DisneyStore_0530/app_Disney/templates/productos/ver_productos.html`**
    - Tabla para mostrar productos con botones de acción.

**14. `UIII_DisneyStore_0530/app_Disney/templates/productos/actualizar_productos.html`**
    - Formulario para editar productos existentes.

**15. `UIII_DisneyStore_0530/app_Disney/templates/productos/borrar_productos.html`**
    - Página de confirmación para borrar un producto.

**16. (Opcional) `UIII_DisneyStore_0530/app_Disney/static/img/default_product.png`**
    - Una imagen de reemplazo si un producto no tiene imagen cargada.

---

**Pasos de Terminal Finales:**

1.  **Asegúrate de estar en `UIII_DisneyStore_0530` y que tu entorno virtual esté activado.**
    *   Windows: `.\.venv\Scripts\activate`
    *   Linux/macOS: `source ./.venv/bin/activate`

2.  **Instala Pillow (si no lo has hecho):**
    ```bash
    pip install Pillow
    ```

3.  **Realiza las migraciones finales (para el campo `imagen` y cualquier ajuste en `admin.py`):**
    ```bash
    python manage.py makemigrations
    python manage.py migrate
    ```

4.  **Crea un superusuario para acceder al panel de administración de Django:**
    ```bash
    python manage.py createsuperuser
    ```
    Sigue las indicaciones para crear un nombre de usuario, correo electrónico y contraseña.

5.  **Ejecuta el servidor:**
    ```bash
    python manage.py runserver 8006
    ```

Ahora, tu proyecto está completo según tus especificaciones y debería ser totalmente funcional. Puedes acceder a la administración en `http://127.0.0.1:8006/admin` con el superusuario que creaste, y a la tienda en `http://127.0.0.1:8006/`.

Si hay alguna otra cosa, ¡no dudes en preguntar!
