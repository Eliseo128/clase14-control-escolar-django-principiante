Claro, aquí tienes la pregunta y la respuesta convertidas a formato Markdown:

---

### 🟡 Pregunta

> **¿Puedes crear un `models.py` con las clases `Profesor` (cuatro campos), `Alumno` (cuatro campos) y `Materia`, y la relación entre ellos? En la clase `Profesor` y `Alumno` incluir el campo `foto_profesor` y `foto_alumno`.**

---

### 🟢 Respuesta

Aquí tienes un ejemplo completo del archivo `models.py` en Django:

```python
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
    profesor = models.ForeignKey(Profesor, on_delete=models.CASCADE, related_name='materias')
    alumnos = models.ManyToManyField(Alumno, related_name='materias')

    def __str__(self):
        return self.nombre
```

---

### ✅ Explicación:

* **`Profesor`**:

  * Campos: `nombre`, `apellido`, `email`, `foto_profesor`.

* **`Alumno`**:

  * Campos: `nombre`, `apellido`, `matricula`, `foto_alumno`.

* **`Materia`**:

  * Campos: `nombre`, `descripcion`.
  * Relaciones:

    * `ForeignKey` a `Profesor`: una materia es impartida por un profesor.
    * `ManyToManyField` con `Alumno`: una materia tiene varios alumnos, y un alumno puede estar inscrito en varias materias.

---

### ⚙️ Recomendaciones:

1. Asegúrate de configurar en `settings.py`:

   ```python
   MEDIA_URL = '/media/'
   MEDIA_ROOT = BASE_DIR / 'media'
   ```

2. Agrega al `urls.py` principal:

   ```python
   from django.conf import settings
   from django.conf.urls.static import static

   urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
   ```

3. Instala Pillow si no lo tienes:

   ```bash
   pip install Pillow
   ```

¿Deseas que lo integre también en `admin.py`, `forms.py` o los templates?


