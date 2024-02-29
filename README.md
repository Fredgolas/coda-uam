# Instalación y configuración del servidor
Para facilitar la instalación del entorno, el repositorio incluye archivos de configuración de contenedores de docker. A continuación se presentan dos métodos para inicializar y configurar esos contenedores (docker compose y docker manual). Asegurate de utilizar solamente **uno** de los dos métodos.
### Requisitos

- Docker (Docker compose) y Docker Desktop 


## Docker compose
Usando el plugin compose (integrado en docker), podemos configurar el entorno de manera sencilla y utilizando menos comandos que con la configuración manual. 

### Instalación y configuración del entorno

1. Cambia los valores en `enviroment` dentro de `services` `web` para que se ajusten a las necesidades de tu entorno. Especialmente las variable de entorno `DJANGO_SECRET_KEY`,  `EMAIL_HOST_PASSWORD` y `EMAIL_DOMAIN`.

2. Para el despliegue en producción es necesario que cambies las variables de entorno el los servicios `db` y `web`, además de la ubicación en tu equipo donde se almacenaran la base de datos y el código del servidor `volumes`
```yaml
services:
  db:
    image: postgres
    volumes:
      - # Cambiar ubicación en disco de la base de datos
    environment:
      - # Cambiar variables de entorno
    ###  COSAS  ###
  web:
    volumes:
     - # Cambiar ubicación del código en disco, eliminar sección en producción
    ###  COSAS  ###
    environment:
      - # Cambiar variables de entorno
    depends_on:
      - db
``` 
3. Finalmente, desde la carpeta raíz (coda-uam), ejecuta el comando `docker compose up` para iniciar y correr los contenedores correspondientes al entorno.

4. Para detener la ejecución del entorno, utiliza el comando `docker compose down`

### Inicialización de base de datos
Para poder acceder al admin de django, es necesario crear un superusuario. Para esto, nos conectamos al contenedor de nuestro servidor llamado `web` 

1. Desde terminal ejecuta el comando `docker ps` para ver tus contenedores activos.
2. Busca el contenedor llamado `coda-uam-web` y copia el `CONTAINER ID` que aparece.
3. Ejecuta el comando `docker exec -it <CONTAINER_ID> python manage.py createsuperuser` colocando el ID que copiaste en lugar de `<CONTAINER_ID>`
4. Llena los datos para crear un superusuario


## Docker manual
Antes de empezar, descomenta la línea 17 del archivo `coda-src/ssocial/settings.py` como se muestra a continuación:
```python
# TODO mochar esto en prod
from .env_vars import *
```

### Creación de archivo de variables de entorno

1. En el directorio `coda-src/ssocial` Crea un archivo llamado `env_vars.py`.
2. Copia y pega el siguiente código en el archivo:

```python
import os

# Valores de base de datos, dejar como estan o ajustar al entorno segun se requiera
os.environ.setdefault('RDS_DB_NAME', 'coda')
os.environ.setdefault('RDS_USERNAME', 'postgres')
os.environ.setdefault('RDS_PASSWORD', '<contraseña>')
os.environ.setdefault('RDS_HOSTNAME', 'margarito-contenedor')
os.environ.setdefault('RDS_PORT', '5432')
os.environ.setdefault('DJANGO_DEBUG', 'True')
os.environ.setdefault('TUTORIAS_DOMINIO', '<dominio>')
os.environ.setdefault('IP_COMPUTADORA', 'localhost')
os.environ.setdefault('EMAIL_HOST_PASSWORD', '<email>')
```
3. Reemplaza los valores entre `*** CAMBIAR VALORES ***` con la información correspondiente al entorno de produccion.


### Creación de la base de datos

1. Cambia al directorio `DB`.
2. Modifica las líneas 5 y 10 del archivo `Dockerfile` para que coincidan con tu configuración.
3. Ejecuta los siguientes comandos:

```
docker network create <nombre-red>
docker build -t margarito .
docker run -d --network <nombre-red> -p 5432:5432 --name margarito-contenedor margarito
docker exec -it margarito-contenedor psql -U postgres -c 'CREATE DATABASE coda;'
```

### Creación del servidor Django

1. Cambia al directorio `coda-src`.
2. Ejecuta el siguiente comando:

```
docker build -t djangarito .
```

3. Inicia el servidor Django:

```
docker run -d --network <nombre-red> -p 8000:8000 --name djangarito_contenedor djangarito
```

### Implementación del servidor

1. En Docker Desktop, selecciona la pestaña "Contenedores".
2. Busca y selecciona el contenedor `djangarito_contenedor`.
3. Selecciona la pestaña "Exec".
4. Ejecuta los siguientes comandos:

```
python3 manage.py migrate
python3 manage.py createsuperuser
```

## Notas adicionales

* Asegúrate de reemplazar los valores predeterminados en el archivo `env_vars.py` con la información de tu proyecto.
* Puedes cambiar el nombre de la red y los nombres de los contenedores según tus preferencias.
* Para obtener más información sobre cómo usar Docker y Django, consulta la documentación oficial:
    * [docs.docker.com](https://docs.docker.com/)
    * [docs.djangoproject.com](https://docs.djangoproject.com/en/4.2/)
* En linux, los comandos `docker` en terminal requieren como permiso de acceso ser parte del grupo `docker` que se crea automaticamente con la instalación de Docker. En caso de que no se quiera agregar el usuario al grupo, debera usarse `sudo` antes de cada comando.