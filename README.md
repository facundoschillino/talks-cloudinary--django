# Django Cloudinary Storage

**Django Cloudinary Storage** es un paquete de Django que facilita la integración de [Cloudinary](http://cloudinary.com/) en nuestro proyecto.  
Con muy poca configuración, podremos comenzar a utilizar Cloudinary para el manejo de archivos **MEDIA**.

---

## Requisitos previos
- Tener un proyecto Django funcionando.  
- Tener una cuenta en Cloudinary.

---

## Instalación

Para instalar el paquete usando **uv**:

```bash
uv add django-cloudinary-storage
```

En caso de necesitar manejar videos también instalamos:

```bash
uv add django-cloudinary-storage[video]
```

Para manejar imágenes en nuestro modelo mediante `ImageField`, debemos instalar **Pillow**:

```bash
uv add Pillow
```

Guardamos nuestras nuevas dependencias en nuestro `requirements.txt`:

```bash
uv pip freeze > requirements.txt
```

---

## Configuración en `settings.py`

Una vez instalado, agregamos `cloudinary` y `cloudinary_storage` a nuestra lista `INSTALLED_APPS`.  
Si solo usaremos Cloudinary para el manejo de archivos media, debemos asegurarnos que `django-cloudinary-storage` esté luego de `'django.contrib.staticfiles'`, ya que de lo contrario se sobreescribe el comando `collectstatic` de Django.

```python
INSTALLED_APPS = [
    # ...
    'django.contrib.staticfiles',
    'cloudinary_storage',
    'cloudinary',
    # ...
]
```

Si estás usando **Django 5.1** o posteriores, necesitaremos definir un diccionario `STORAGES` en `settings.py`:

```python
STORAGES = {
  'default': {
    'BACKEND': 'cloudinary_storage.storage.MediaCloudinaryStorage' 
  },
  'staticfiles': {                                                 
    'BACKEND': 'django.core.files.storage.FileSystemStorage'       # Default storage que usa Django
  },
}
```

Y agregamos `MEDIA_URL` a nuestro `settings.py`:

```python
MEDIA_URL = '/media/'  
```

Con esto ya definimos lo necesario para manejar archivos de imagen, video o "raw".

---

## Credenciales de Cloudinary

Luego, vamos a definir nuestras credenciales de Cloudinary en `settings.py`:

```python
CLOUDINARY_STORAGE = {
    'CLOUD_NAME': os.environ.get("CLOUD_NAME"),
    'API_KEY': os.environ.get("API_KEY"),
    'API_SECRET': os.environ.get("API_SECRET")
}
```

Vamos a buscar los valores en Cloudinary:

![FOTOTUTO](https://github.com/user-attachments/assets/90075610-349d-4675-b221-526c95fb8f3c)

Ir a la pestaña **API keys**:

<img width="1522" height="947" alt="aplikeys_djangotut" src="https://github.com/user-attachments/assets/91f7df06-01e7-4406-b871-debd502c68dc" />

Y luego definir las variables en Render:

<img width="1277" height="168" alt="image" src="https://github.com/user-attachments/assets/0eaf8fe2-70a3-4cf7-af67-1ea6af6942a2" />

---

## Uso de archivos media

### Imágenes

```python
class Noticia(models.Model):
    titulo = models.CharField(max_length=100)
    portada = models.ImageField(upload_to='images/', blank=True)
```

Y listo! Con esto, todos los modelos con `ImageField` estarán conectados con Cloudinary.

Para mostrar las imágenes en nuestro template podemos usar:

```django
<img src="{{ noticia.portada.url }}" alt="{{ noticia.portada.name }}">
```

Si queremos transformar la imagen directamente desde Cloudinary:

```django
{% load cloudinary %}
{% cloudinary test_model_instance.image.name width=100 height=100 %}
```

---

### Archivos RAW

Tendremos que aclararle al modelo que queremos emplear el storage `RawMediaCloudinaryStorage`:

```python
from django.db import models
from cloudinary_storage.storage import RawMediaCloudinaryStorage

class Noticia(models.Model):
    titulo = models.CharField(max_length=100)
    pruebas_en_pdf = models.FileField(upload_to='raw/', blank=True, storage=RawMediaCloudinaryStorage()) 
    portada = models.ImageField(upload_to='images/', blank=True)  
```


Adicionalmente, si necesitamos manejar archivos PDF o ZIP debemos habilitar la opción de CLoudinary que permite esto en la pestaña [Seguiridad](https://console.cloudinary.com/settings/security)

---

### Videos

Para videos necesitaremos utilizar el validador `validate_video` para validar las entradas de este campo:

```python
from django.db import models
from cloudinary_storage.storage import VideoMediaCloudinaryStorage
from cloudinary_storage.validators import validate_video

class Noticia(models.Model):
    name = models.CharField(max_length=100)
    video = models.FileField(
        upload_to='videos/',
        blank=True,
        storage=VideoMediaCloudinaryStorage(),
        validators=[validate_video]
    )
    portada = models.ImageField(upload_to='images/', blank=True)
```
