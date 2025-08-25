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

Para manejar imágenes en nuestro modelo mediante `ImageField`, debemos instalar **Pillow**:

```bash
uv add Pillow
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

Si estás usando **Django 5.1** o posteriores, necesitaremos definir un diccionario `STORAGES` en las settings de producción de nuestro `settings.py`:

```python
STORAGES = {
  'default': {
    'BACKEND': 'cloudinary_storage.storage.MediaCloudinaryStorage' 
  },
  'staticfiles': {                                                 
    'BACKEND': 'whitenoise.storage.CompressedManifestStaticFilesStorage'    
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

Luego, vamos a definir nuestras credenciales de Cloudinary en  `settings.py`:

```python
CLOUDINARY_STORAGE = {
    'CLOUD_NAME': os.environ.get("CLOUDINARY_CLOUD_NAME"),
    'API_KEY': os.environ.get("CLOUDINARY_API_KEY"),
    'API_SECRET': os.environ.get("CLOUDINARY_API_SECRET")
}
```

Vamos a buscar los valores en Cloudinary:

![FOTOTUTO](https://github.com/user-attachments/assets/90075610-349d-4675-b221-526c95fb8f3c)

Ir a la pestaña **API keys**:

<img width="1522" height="947" alt="aplikeys_djangotut" src="https://github.com/user-attachments/assets/91f7df06-01e7-4406-b871-debd502c68dc" />

Y luego definir las variables en Render:

<img width="1279" height="219" alt="image" src="https://github.com/user-attachments/assets/7dc09857-2838-4ffb-902e-e4a374721223" />

---

Finalmente, nuestra configuración de produccion deberia verse como:

```python
if 'RENDER' in os.environ:
    print("USING RENDER.COM SETTINGS!")
    DEBUG = False
    ALLOWED_HOSTS = [os.environ.get('RENDER_EXTERNAL_HOSTNAME')]
    DATABASES = {'default': dj_database_url.config(conn_max_age=600)}
    MIDDLEWARE.insert(MIDDLEWARE.index('django.middleware.security.SecurityMiddleware') + 1,
                      'whitenoise.middleware.WhiteNoiseMiddleware')
    MEDIA_URL= "/media/"
    STORAGES = {
        "default":
                {"BACKEND": "cloudinary_storage.storage.MediaCloudinaryStorage"},

        "staticfiles":
                {"BACKEND": "whitenoise.storage.CompressedManifestStaticFilesStorage"},
    }
    STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

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
    pruebas_en_pdf = models.FileField(upload_to='raw/', blank=True) 
    portada = models.ImageField(upload_to='images/', blank=True)  
```


Adicionalmente, si necesitamos manejar archivos PDF o ZIP debemos habilitar la opción de CLoudinary que permite esto en la pestaña [Seguiridad](https://console.cloudinary.com/settings/security)

---

### Videos



```python
from django.db import models
class Noticia(models.Model):
    titulo = models.CharField(max_length=200)
    portada = models.ImageField(upload_to="fotos/noticias/", blank=True)
    pruebas_en_pdf = models.FileField(upload_to='raw/noticias', blank=True)
    video = models.FileField(upload_to='videos/noticias',blank=True)
    def __str__(self):
        return self.titulo
```
