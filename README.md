# clase14-control-escolar-django-principiante
¡Excelente! Crear un sistema de administración escolar es un proyecto clásico y muy completo para aprender Django. Los modelos que has proporcionado, con relaciones `ForeignKey` y `ManyToManyField`, nos permitirán explorar funcionalidades más avanzadas.

Aquí tienes la guía completa, paso a paso, para construir tu proyecto.

---

### **Proyecto Django: Sistema de Administración Escolar (CRUD)**

Construiremos una aplicación web para gestionar Profesores, Alumnos y Materias, incluyendo la carga de imágenes para los perfiles.

#### **Estructura del Proyecto Final**

La estructura de carpetas y archivos más relevante será:

```
schoolsystem/
├── manage.py
├── schoolsystem/
│   ├── settings.py
│   ├── urls.py
│   └── ...
├── gestion_escolar/
│   ├── migrations/
│   ├── admin.py
│   ├── apps.py
│   ├── models.py  <-- Tus modelos irán aquí
│   ├── urls.py    <-- Lo crearemos
│   └── views.py
├── templates/
│   ├── base.html
│   ├── footer.html
│   ├── navbar.html
│   └── gestion_escolar/
│       ├── home.html
│       ├── profesor/
│       │   ├── profesor_list.html
│       │   ├── profesor_detail.html
│       │   ├── profesor_form.html
│       │   └── profesor_confirm_delete.html
│       ├── alumno/
│       │   ├── (archivos similares)
│       └── materia/
│           ├── (archivos similares)
└── media/
    ├── fotos_alumnos/
    └── fotos_profesores/
```

---

### **Paso 0: Prerrequisitos e Instalación**

1.  **Instala Python y pip.**
2.  **Crea y activa un entorno virtual:**
    ```bash
    python -m venv venv
    # En Windows: .\venv\Scripts\activate
    # En macOS/Linux: source venv/bin/activate
    ```
3.  **Instala Django y Pillow:** Pillow es una librería necesaria para manejar campos de imagen (`ImageField`).
    ```bash
    pip install django pillow
    ```

---

### **Paso 1: Creación del Proyecto y la Aplicación**

1.  **Crea el proyecto Django.** Lo llamaremos `schoolsystem`.
    ```bash
    django-admin startproject schoolsystem
    ```
2.  **Entra al directorio del proyecto.**
    ```bash
    cd schoolsystem
    ```
3.  **Crea la aplicación.** La llamaremos `gestion_escolar`.
    ```bash
    python manage.py startapp gestion_escolar
    ```

---

### **Paso 2: Configuración del Proyecto**

Esta parte es crucial, especialmente por los campos de imagen.

1.  **Registra la aplicación:**
    Abre `schoolsystem/settings.py` y añade `'gestion_escolar'` a `INSTALLED_APPS`.

    ```python
    # schoolsystem/settings.py
    INSTALLED_APPS = [
        # ...
        'django.contrib.staticfiles',
        'gestion_escolar', # <-- AÑADIR ESTA LÍNEA
    ]
    ```

2.  **Configura el directorio de plantillas (templates):**
    En el mismo archivo `settings.py`, modifica `TEMPLATES`.

    ```python
    # schoolsystem/settings.py
    import os
    TEMPLATES = [
        {
            # ...
            'DIRS': [os.path.join(BASE_DIR, 'templates')], # <-- AÑADIR ESTA LÍNEA
            # ...
        },
    ]
    ```

3.  **Configura los archivos multimedia (MEDIA):**
    Para que las fotos de los alumnos y profesores se guarden y se muestren correctamente, añade estas dos líneas al final de `schoolsystem/settings.py`.

    ```python
    # schoolsystem/settings.py

    # Al final del archivo
    MEDIA_URL = '/media/'
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
    ```
    *   `MEDIA_URL`: La URL base desde la que se servirán los archivos multimedia.
    *   `MEDIA_ROOT`: La carpeta en el sistema de archivos donde se guardarán los archivos subidos.

---

### **Paso 3: Definir Modelos y Migrar la Base de Datos**

1.  **Pega tus modelos en `gestion_escolar/models.py`:**
    Abre este archivo y reemplaza su contenido con tu código.

    ```python
    # gestion_escolar/models.py
    from django.db import models

    class Profesor(models.Model):
        nombre = models.CharField(max_length=100)
        apellido = models.CharField(max_length=100)
        email = models.EmailField(unique=True)
        foto_profesor = models.ImageField(upload_to='fotos_profesores/', blank=True, null=True)

        def __str__(self):
            return f"{self.nombre} {self.apellido}"

    class Alumno(models.Model):
        nombre = models.CharField(max_length=100)
        apellido = models.CharField(max_length=100)
        matricula = models.CharField(max_length=20, unique=True)
        foto_alumno = models.ImageField(upload_to='fotos_alumnos/', blank=True, null=True)

        def __str__(self):
            return f"{self.nombre} {self.apellido}"

    class Materia(models.Model):
        nombre = models.CharField(max_length=100)
        descripcion = models.TextField()
        profesor = models.ForeignKey(Profesor, on_delete=models.SET_NULL, null=True, related_name='materias')
        alumnos = models.ManyToManyField(Alumno, related_name='materias', blank=True)

        def __str__(self):
            return self.nombre
    ```
    **Nota:** He cambiado `on_delete=models.CASCADE` a `on_delete=models.SET_NULL` con `null=True` en el `ForeignKey` de `Materia`. Esto es una mejor práctica: si eliminas un profesor, las materias que impartía no se eliminarán, simplemente se quedarán sin profesor asignado. También he añadido `blank=True` en el `ManyToManyField` para permitir crear materias sin alumnos inicialmente.

2.  **Crea y aplica las migraciones:**
    ```bash
    python manage.py makemigrations gestion_escolar
    python manage.py migrate
    ```

---

### **Paso 4: Panel de Administración y Superusuario**

1.  **Registra los modelos en `gestion_escolar/admin.py`:**
    ```python
    # gestion_escolar/admin.py
    from django.contrib import admin
    from .models import Profesor, Alumno, Materia

    admin.site.register(Profesor)
    admin.site.register(Alumno)
    admin.site.register(Materia)
    ```
2.  **Crea un superusuario:**
    ```bash
    python manage.py createsuperuser
    ```
    Sigue las instrucciones. Ahora puedes ir a `/admin` y gestionar todo desde allí.

---

### **Paso 5: Creación de las Plantillas (Templates)**

Crearemos plantillas reutilizables y organizadas.

1.  **Crea los directorios y archivos base:**
    *   `templates/base.html`
    *   `templates/navbar.html`
    *   `templates/footer.html`
    *   `templates/gestion_escolar/home.html`

2.  **Edita `templates/base.html`:**
    ```html
    <!doctype html>
    <html lang="es">
    <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <title>{% block title %}Sistema Escolar{% endblock %}</title>
      <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    </head>
    <body>
      {% include 'navbar.html' %}
      <main class="container mt-4">
          {% block content %}{% endblock %}
      </main>
      {% include 'footer.html' %}
      <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
    </body>
    </html>
    ```

3.  **Edita `templates/navbar.html`:**
    ```html
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
      <div class="container">
        <a class="navbar-brand" href="{% url 'home' %}">SchoolSystem</a>
        <div class="collapse navbar-collapse">
          <ul class="navbar-nav me-auto mb-2 mb-lg-0">
            <li class="nav-item"><a class="nav-link" href="{% url 'profesor_list' %}">Profesores</a></li>
            <li class="nav-item"><a class="nav-link" href="{% url 'alumno_list' %}">Alumnos</a></li>
            <li class="nav-item"><a class="nav-link" href="{% url 'materia_list' %}">Materias</a></li>
          </ul>
        </div>
      </div>
    </nav>
    ```

4.  **Edita `templates/footer.html`:** (Sencillo, como en el ejemplo anterior)
    ```html
    <footer class="py-3 my-4 bg-light"><p class="text-center text-muted">© 2023 SchoolSystem</p></footer>
    ```

5.  **Edita `templates/gestion_escolar/home.html`:**
    ```html
    {% extends 'base.html' %}
    {% block title %}Inicio{% endblock %}
    {% block content %}
      <div class="p-5 mb-4 bg-light rounded-3">
        <div class="container-fluid py-5">
          <h1 class="display-5 fw-bold">Bienvenido al Sistema de Gestión Escolar</h1>
          <p class="col-md-8 fs-4">Utilice la barra de navegación para gestionar profesores, alumnos y materias.</p>
        </div>
      </div>
    {% endblock %}
    ```

6.  **Crea las plantillas CRUD (Ejemplo con Profesor):**
    Crea la carpeta `templates/gestion_escolar/profesor/`. Los archivos para `Alumno` y `Materia` serán muy similares.

    *   **`profesor_list.html`**
        ```html
        {% extends 'base.html' %}
        {% block title %}Lista de Profesores{% endblock %}
        {% block content %}
          <div class="d-flex justify-content-between align-items-center mb-3">
            <h1>Profesores</h1>
            <a href="{% url 'profesor_create' %}" class="btn btn-primary">Añadir Profesor</a>
          </div>
          <div class="list-group">
            {% for profesor in object_list %}
              <a href="{% url 'profesor_detail' profesor.pk %}" class="list-group-item list-group-item-action">
                {{ profesor.nombre }} {{ profesor.apellido }}
              </a>
            {% empty %}
              <p class="list-group-item">No hay profesores registrados.</p>
            {% endfor %}
          </div>
        {% endblock %}
        ```

    *   **`profesor_detail.html`**
        ```html
        {% extends 'base.html' %}
        {% block title %}{{ object.nombre }} {{ object.apellido }}{% endblock %}
        {% block content %}
          <div class="card">
            <div class="card-header">
              <h2>{{ object.nombre }} {{ object.apellido }}</h2>
            </div>
            <div class="card-body">
              {% if object.foto_profesor %}
                <img src="{{ object.foto_profesor.url }}" alt="Foto de {{ object.nombre }}" class="img-fluid rounded mb-3" style="max-width: 200px;">
              {% endif %}
              <p><strong>Email:</strong> {{ object.email }}</p>
              
              <h5 class="mt-4">Materias que imparte:</h5>
              <ul>
                {% for materia in object.materias.all %}
                  <li>{{ materia.nombre }}</li>
                {% empty %}
                  <li>No imparte ninguna materia actualmente.</li>
                {% endfor %}
              </ul>
            </div>
            <div class="card-footer">
              <a href="{% url 'profesor_update' object.pk %}" class="btn btn-warning">Editar</a>
              <a href="{% url 'profesor_delete' object.pk %}" class="btn btn-danger">Eliminar</a>
              <a href="{% url 'profesor_list' %}" class="btn btn-secondary">Volver a la lista</a>
            </div>
          </div>
        {% endblock %}
        ```
    
    *   **`profesor_form.html`** (Este formulario genérico servirá para todos los modelos)
        ```html
        {% extends 'base.html' %}
        {% block title %}Formulario{% endblock %}
        {% block content %}
          <h1>{% if form.instance.pk %}Editar{% else %}Añadir{% endif %} {{ model_name }}</h1>
          <form method="post" enctype="multipart/form-data">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit" class="btn btn-success">Guardar</button>
          </form>
        {% endblock %}
        ```
        **Importante:** `enctype="multipart/form-data"` es **obligatorio** para que la subida de archivos funcione.

    *   **`profesor_confirm_delete.html`** (Genérico también)
        ```html
        {% extends 'base.html' %}
        {% block title %}Confirmar Eliminación{% endblock %}
        {% block content %}
          <h1>¿Seguro que quieres eliminar a "{{ object }}"?</h1>
          <form method="post">
            {% csrf_token %}
            <button type="submit" class="btn btn-danger">Sí, eliminar</button>
            <a href="{{ object.get_absolute_url }}" class="btn btn-secondary">Cancelar</a>
          </form>
        {% endblock %}
        ```
    
    **Repite el proceso:** Crea carpetas `alumno` y `materia` dentro de `templates/gestion_escolar/` y crea archivos `_list`, `_detail`, `_form` y `_confirm_delete` para cada uno, adaptando los campos y los títulos. Por ejemplo, en `materia_detail.html` mostrarás el profesor y una lista de alumnos inscritos.

---

### **Paso 6: Creación de las Vistas (Views)**

Usaremos Vistas Basadas en Clases (CBVs) para cada modelo.

**Edita `gestion_escolar/views.py`:**

```python
# gestion_escolar/views.py
from django.urls import reverse_lazy
from django.views.generic import TemplateView, ListView, DetailView, CreateView, UpdateView, DeleteView
from .models import Profesor, Alumno, Materia

# --- Vistas Generales ---
class HomePageView(TemplateView):
    template_name = 'gestion_escolar/home.html'

# --- Vistas para Profesor ---
class ProfesorListView(ListView):
    model = Profesor
    template_name = 'gestion_escolar/profesor/profesor_list.html'

class ProfesorDetailView(DetailView):
    model = Profesor
    template_name = 'gestion_escolar/profesor/profesor_detail.html'

class ProfesorCreateView(CreateView):
    model = Profesor
    fields = ['nombre', 'apellido', 'email', 'foto_profesor']
    template_name = 'gestion_escolar/profesor/profesor_form.html'
    success_url = reverse_lazy('profesor_list')
    extra_context = {'model_name': 'Profesor'}

class ProfesorUpdateView(UpdateView):
    model = Profesor
    fields = ['nombre', 'apellido', 'email', 'foto_profesor']
    template_name = 'gestion_escolar/profesor/profesor_form.html'
    success_url = reverse_lazy('profesor_list')
    extra_context = {'model_name': 'Profesor'}

class ProfesorDeleteView(DeleteView):
    model = Profesor
    template_name = 'gestion_escolar/profesor/profesor_confirm_delete.html'
    success_url = reverse_lazy('profesor_list')

# --- Vistas para Alumno (siguen el mismo patrón) ---
class AlumnoListView(ListView):
    model = Alumno
    template_name = 'gestion_escolar/alumno/alumno_list.html'

class AlumnoDetailView(DetailView):
    model = Alumno
    template_name = 'gestion_escolar/alumno/alumno_detail.html'
    
class AlumnoCreateView(CreateView):
    model = Alumno
    fields = ['nombre', 'apellido', 'matricula', 'foto_alumno']
    template_name = 'gestion_escolar/alumno/alumno_form.html'
    success_url = reverse_lazy('alumno_list')
    extra_context = {'model_name': 'Alumno'}

class AlumnoUpdateView(UpdateView):
    model = Alumno
    fields = ['nombre', 'apellido', 'matricula', 'foto_alumno']
    template_name = 'gestion_escolar/alumno/alumno_form.html'
    success_url = reverse_lazy('alumno_list')
    extra_context = {'model_name': 'Alumno'}

class AlumnoDeleteView(DeleteView):
    model = Alumno
    template_name = 'gestion_escolar/alumno/alumno_confirm_delete.html'
    success_url = reverse_lazy('alumno_list')

# --- Vistas para Materia (el formulario manejará ForeignKey y ManyToMany automáticamente) ---
class MateriaListView(ListView):
    model = Materia
    template_name = 'gestion_escolar/materia/materia_list.html'

class MateriaDetailView(DetailView):
    model = Materia
    template_name = 'gestion_escolar/materia/materia_detail.html'

class MateriaCreateView(CreateView):
    model = Materia
    fields = ['nombre', 'descripcion', 'profesor', 'alumnos']
    template_name = 'gestion_escolar/materia/materia_form.html'
    success_url = reverse_lazy('materia_list')
    extra_context = {'model_name': 'Materia'}

class MateriaUpdateView(UpdateView):
    model = Materia
    fields = ['nombre', 'descripcion', 'profesor', 'alumnos']
    template_name = 'gestion_escolar/materia/materia_form.html'
    success_url = reverse_lazy('materia_list')
    extra_context = {'model_name': 'Materia'}

class MateriaDeleteView(DeleteView):
    model = Materia
    template_name = 'gestion_escolar/materia/materia_confirm_delete.html'
    success_url = reverse_lazy('materia_list')
```

---

### **Paso 7: Configuración de las URLs**

1.  **Crea `gestion_escolar/urls.py`:**
    ```python
    # gestion_escolar/urls.py
    from django.urls import path
    from . import views

    urlpatterns = [
        path('', views.HomePageView.as_view(), name='home'),
        
        # URLs para Profesor
        path('profesores/', views.ProfesorListView.as_view(), name='profesor_list'),
        path('profesores/<int:pk>/', views.ProfesorDetailView.as_view(), name='profesor_detail'),
        path('profesores/nuevo/', views.ProfesorCreateView.as_view(), name='profesor_create'),
        path('profesores/<int:pk>/editar/', views.ProfesorUpdateView.as_view(), name='profesor_update'),
        path('profesores/<int:pk>/eliminar/', views.ProfesorDeleteView.as_view(), name='profesor_delete'),

        # URLs para Alumno
        path('alumnos/', views.AlumnoListView.as_view(), name='alumno_list'),
        path('alumnos/<int:pk>/', views.AlumnoDetailView.as_view(), name='alumno_detail'),
        path('alumnos/nuevo/', views.AlumnoCreateView.as_view(), name='alumno_create'),
        path('alumnos/<int:pk>/editar/', views.AlumnoUpdateView.as_view(), name='alumno_update'),
        path('alumnos/<int:pk>/eliminar/', views.AlumnoDeleteView.as_view(), name='alumno_delete'),

        # URLs para Materia
        path('materias/', views.MateriaListView.as_view(), name='materia_list'),
        path('materias/<int:pk>/', views.MateriaDetailView.as_view(), name='materia_detail'),
        path('materias/nueva/', views.MateriaCreateView.as_view(), name='materia_create'),
        path('materias/<int:pk>/editar/', views.MateriaUpdateView.as_view(), name='materia_update'),
        path('materias/<int:pk>/eliminar/', views.MateriaDeleteView.as_view(), name='materia_delete'),
    ]
    ```

2.  **Incluye estas URLs en el proyecto principal (`schoolsystem/urls.py`):**
    Añade también la configuración para servir los archivos multimedia en modo de desarrollo.

    ```python
    # schoolsystem/urls.py
    from django.contrib import admin
    from django.urls import path, include
    from django.conf import settings # Importar settings
    from django.conf.urls.static import static # Importar static

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('', include('gestion_escolar.urls')), # <-- AÑADIR ESTA LÍNEA
    ]

    # AÑADIR ESTO AL FINAL: Sirve archivos multimedia en desarrollo
    if settings.DEBUG:
        urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    ```

---

### **Paso 8: ¡Ejecutar y Probar!**

1.  **Inicia el servidor:**
    ```bash
    python manage.py runserver
    ```
2.  **Navega por tu aplicación:**
    *   **Página de inicio:** `http://127.0.0.1:8000/`
    *   **Profesores:** `http://127.0.0.1:8000/profesores/`
    *   **Alumnos:** `http://127.0.0.1:8000/alumnos/`
    *   **Materias:** `http://127.0.0.1:8000/materias/`

Prueba a crear profesores y alumnos primero. Luego, al crear una materia, verás que puedes seleccionarlos desde listas desplegables y de selección múltiple. ¡Todo el CRUD debería funcionar!

---

### **Resumen Final del Proyecto**

1.  **Instalación y Configuración:** Creamos un proyecto `schoolsystem` y una app `gestion_escolar`. Instalamos `Pillow` para las imágenes. Configuramos `settings.py` para incluir la app, el directorio de plantillas y, fundamentalmente, `MEDIA_ROOT` y `MEDIA_URL` para gestionar los archivos subidos.
2.  **Modelos y Base de Datos:** Implementamos tus tres modelos (`Profesor`, `Alumno`, `Materia`), ajustando las relaciones (`ForeignKey`, `ManyToManyField`) para mayor robustez. Creamos y aplicamos las migraciones.
3.  **URLs y Multimedia:** Configuramos las URLs del proyecto para incluir las de nuestra app y añadimos una regla especial para que el servidor de desarrollo pueda mostrar las imágenes subidas (`fotos_profesores`, `fotos_alumnos`).
4.  **Vistas CRUD:** Utilizamos las Vistas Basadas en Clases genéricas de Django (`ListView`, `DetailView`, `CreateView`, etc.) para implementar de forma rápida y limpia toda la funcionalidad CRUD para cada uno de los tres modelos.
5.  **Plantillas (Templates):** Creamos una estructura de plantillas organizada y reutilizable. El formulario (`_form.html`) es genérico y usa `enctype="multipart/form-data"` para permitir la subida de fotos. Las plantillas de detalle muestran la información relacionada (ej. materias de un profesor, alumnos de una materia).

Has construido una base sólida para un sistema de gestión escolar, manejando relaciones complejas y subida de archivos, dos de los aspectos más importantes en el desarrollo web con Django.
