# Portal Andino

[![Build Status](https://travis-ci.org/datosgobar/portal-andino.svg?branch=master)](https://travis-ci.org/datosgobar/portal-andino)

[![Docs Status](https://readthedocs.org/projects/portal-andino/badge/?version=master)](http://portal-andino.readthedocs.io/es/master/)

Implementación de CKAN en Docker, desarrollada en el contexto del portal distribuible Andino.

También podés [ver el repositorio del tema visual](https://github.com/datosgobar/portal-andino-theme).

## Instalación

Teniendo en cuenta la dificultad de implementacion e incluso la cantidad de pasos para lograr un deploy existoso, existen dos formas de instalar esta distribución de **CKAN**. 

- Si no tenés muchos conocimientos de CKAN, Docker o de administracion de servidores en general, es recomendable usar la instalación **[simplificada  de Andino](#instalacion-simplificada-de-andino)**. Está pensada para que en la menor cantidad de pasos y de manera sencilla, tengas un portal de datos funcionando. 
- Si ya conocés la plataforma, tenés experiencia con Docker o simplemente, querés entender cómo funciona esta implementación, te sugiero que revises la **[instalacion avanzada de Andino](#instalacion-avanzada-de-andino)**

### Dependencias

+ DOCKER: [Guía de instalación](https://docs.docker.com/engine/installation).
+ Docker Compose: [Guía de instalación](https://docs.docker.com/compose/install/)

### Instalación simplificada

La idea detrás de esta implementación de CKAN es **que sólo te encargues de tus datos**, nada más. Por eso, si "copiás y pegás" el comando de consola, en sólo unos momentos, tendrás un Andino listo para usar.
Esta clase de instalación no requiere que clones el repositorio, ya que usamos contenedores alojados en [DockerHub](https://hub.docker.com/r/datosgobar)

+ Ubuntu|Debian|RHEL|CentOS:

+ Instalación:

Para esta instalación ciertos parametros deben ser pasados a la aplicacion:
    + Email donde se mandarán los errores. `EMAIL=admin@example.com`
    + Dominio o IP de la aplicación: `HOST=datos.gob.ar`
    + Usuario de la base de datos: `DB_USER=<my db user>`
    + Password de la base de datos: `DB_PASS=<my db pass>`
    + Usuario del datastore: `STORE_USER=<my datastore user>`
    + Password del datastore: `STORE_PASS=<my datastore password>`

```bash
wget https://raw.github.com/datosgobar/portal-andino/development/deploy/install.py
python ./install.py --error_email "$EMAIL" --site_host="$HOST" \
    --database_user="$DB_USER" --database_password="$DB_PASS" \
    --datastore_user="$STORE_USER" --datastore_password="$STORE_PASS"
```

### Instalación avanzada

La instalación avanzada está pensada para usarios que quieren ver cómo funciona internamente `Andino`

Para instalar y ejecutar Andino, seguimos estos pasos:

+ Paso 1: Clonar repositorio.

		$ sudo mkdir /etc/andino
		$ cd /etc/andino
		$ git clone https://github.com/datosgobar/portal-andino.git andino
		
+ Paso 2: Setear las variables de entorno para el contenedor de postgresql

        $ DB_USER=<my user>
        $ DB_PASSWORD=<my pass>
        $ echo "POSTGRES_USER=$DB_USER" > .env
        $ echo "POSTGRES_PASWORD=$DB_PASS" >> .env
        $ echo "CKAN_HOST=andino" >> .env # Esta configuración es para nginx
        

+ Paso 3: _construir y lanzar los contenedor de servicios usando el archivo **latest.yml**_

        $ docker-compose -f latest.yml up -d db postfix redis solr        

+ Paso 4: _construir y lanzar el contenedor de **andino** usando el archivo **latest.yml**_

		$ docker-compose -f latest.yml up -d andino
		
+ Paso 5: Inicializar la base de datos y la configuración de la aplicación:


```bash
EMAIL=admin@example.com
HOST=datos.gob.ar
DB_USER=<my db user>
DB_PASS=<my db pass>
STORE_USER=<my datastore user>
STORE_PASS=<my datastore password>
docker exec andino /etc/ckan_init.d/init.sh -e "$EMAIL" -h "$HOST" \
        -p "$DB_USER" -P "$DB_PASS" \
        -d "$STORE_USER" -D "$STORE_PASS"

```

+ Paso 8: _construir el contenedor de **nginx** usando el archivo **latest.yml**_

		$ docker-compose -f latest.yml up -d nginx

## Uso

Una vez finalizada la instalación, bajo cualquiera de los métodos, deberíamos:

### Crear usuario administrador
	
```bash		
# Agregar un admin
# Asumo que el contenedor de ckan es llamado "andino"
ADMIN_USER=<my_admin>        
docker exec -it andino /etc/ckan_init.d/add_admin.sh "$ADMIN_USER"
```

### Configurar url de Andino
```bash
# Asumo que el contenedor de ckan es llamado "andino"
URL_NUEVA=<my url>
DATAPUSHER_URL=<my datapusher url>
docker exec -it andino /etc/ckan_init.d/change_site_url.sh "$URL_NUEVA"
docker exec -it andino /etc/ckan_init.d/change_datapusher_url.sh "$DATAPUHSER_URL"
docker restart andino
```
