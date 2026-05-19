# 🐳 Docker — Práctica 5: Aplicaciones multi-servicio con Docker Compose

**Arturo Gutierrez Sanchez**

---

## ¿Qué es Docker Compose y por qué usarlo?

Cuando una aplicación real necesita varios servicios trabajando juntos — un servidor web, una base de datos, una caché — gestionar cada contenedor por separado se vuelve tedioso. **Docker Compose** resuelve esto: describe toda la arquitectura en un único fichero `docker-compose.yml` y levanta o derriba el conjunto con un solo comando.

Conceptos clave que se trabajan en esta práctica:

- **Servicio** — Cada contenedor declarado dentro del fichero Compose.
- **Volumen persistente** — Almacenamiento que sobrevive al ciclo de vida del contenedor.
- **Variables de entorno** — Parámetros externos que configuran los servicios sin tocar el código.
- **Red automática** — Compose crea una red compartida para que los servicios se localicen por nombre.

---

## Entorno de trabajo

| Campo | Valor |
|---|---|
| Sistema operativo | Ubuntu 24.04 LTS |
| Arquitectura | x86_64 |
| Plataforma de virtualización | VirtualBox |
| Motor de contenedores | Docker CE |
| Usuario del sistema | arturo |

---

## Ejercicio 1 — Servidor Nginx con fichero Compose

### Estructura del proyecto

Creamos la carpeta de trabajo y nos situamos en ella:

```bash
mkdir -p ~/compose-web && cd ~/compose-web
```

Dentro creamos el subdirectorio que contendrá el HTML servido:

```bash
mkdir sitio
echo "<h1>Servidor Nginx usando Docker Compose</h1>" > sitio/index.html
```

### Fichero docker-compose.yml

```yaml
services:
  servidor_web:
    image: nginx:stable
    container_name: nginx_compose
    ports:
      - "8090:80"
    volumes:
      - ./sitio:/usr/share/nginx/html:ro
    restart: unless-stopped
```

`restart: unless-stopped` hace que el contenedor se recupere automáticamente si se cae, salvo que lo detengamos nosotros a propósito.

### Arranque y verificación

```bash
docker compose up -d
docker compose ps
```

Comprobamos que el servidor responde:

```bash
curl http://localhost:8090
```

```html
<h1>Servidor Nginx usando Docker Compose</h1>
```

Para consultar los registros del servicio:

```bash
docker compose logs
```

Al terminar, derribamos todo:

```bash
docker compose down
```

---

## Ejercicio 2 — Stack PHP + MariaDB

Este ejercicio monta dos servicios interconectados: un servidor Apache con PHP y una base de datos MariaDB con un volumen persistente.

### Estructura del proyecto

```bash
mkdir -p ~/compose-lamp && cd ~/compose-lamp
mkdir app
```

Fichero PHP de prueba en `app/index.php`:

```php
<?php
echo "<h1>Aplicación PHP funcionando correctamente</h1>";
echo "<p>Servidor ejecutándose con Docker Compose</p>";
echo "<p>Versión PHP: " . phpversion() . "</p>";
?>
```

### Fichero docker-compose.yml

```yaml
services:
  web:
    image: php:8.2-apache
    container_name: php_web
    ports:
      - "8085:80"
    volumes:
      - ./app:/var/www/html
    depends_on:
      - database

  database:
    image: mariadb:11
    container_name: mariadb_server
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: ejemplo_db
    volumes:
      - datos_mysql:/var/lib/mysql

volumes:
  datos_mysql:
```

`depends_on` le indica a Compose que el servicio `web` no debe arrancar antes de que `database` esté en marcha. El volumen `datos_mysql` se declara al final para que Compose lo gestione automáticamente.

### Arranque y comprobación

```bash
docker compose up -d
docker compose ps
curl http://localhost:8085
```

Para ver los últimos registros de ambos servicios a la vez:

```bash
docker compose logs --tail=30
```

Verificamos que el volumen de la base de datos se creó:

```bash
docker volume ls
```

Al terminar:

```bash
docker compose down
```

---

## Ejercicio 3 — Variables de entorno con fichero .env

Docker Compose permite externalizar valores sensibles o que cambian según el entorno (puertos, nombres, credenciales) en un fichero `.env` que se lee automáticamente.

### Estructura del proyecto

```bash
mkdir -p ~/compose-env && cd ~/compose-env
```

Fichero `.env`:

```env
APP_PORT=3001
APP_NAME=MiAplicacion
ENTORNO=desarrollo
```

### Fichero docker-compose.yml

```yaml
services:
  app:
    image: nginx:latest
    container_name: nginx_env
    ports:
      - "${APP_PORT}:80"
    environment:
      - APP_NAME=${APP_NAME}
      - ENTORNO=${ENTORNO}
```

Las llaves `${}` son sustituidas por Compose con los valores del `.env` antes de crear los contenedores. Para ver la configuración resultante antes de arrancar:

```bash
docker compose config
```

Esto es útil para depurar: muestra el YAML final con todas las variables ya resueltas.

### Arranque y parada

```bash
docker compose up -d
docker compose ps
docker compose down
```

---

## Problemas habituales

**El puerto ya está en uso** — Algún proceso del host ocupa el mismo puerto. Comprobamos cuál es con `docker ps` o `ss -tlnp` y cambiamos el puerto del host en el `docker-compose.yml`.

**Un servicio no arranca** — Lo primero es revisar los logs de ese servicio concreto:

```bash
docker compose logs nombre_servicio
```

**Queremos borrar también los volúmenes al limpiar** — El flag `-v` los elimina junto con los contenedores:

```bash
docker compose down -v
```

---

## Buenas prácticas

Fijar versiones exactas de las imágenes en lugar de usar `latest` evita sorpresas cuando el registro actualiza la imagen:

```yaml
image: nginx:1.27      # ✅ versión fija
image: nginx:latest    # ❌ puede cambiar sin avisar
```

Externalizar toda configuración variable al `.env` y no escribirla directamente en el YAML:

```yaml
# ✅ El puerto viene del .env, fácil de cambiar por entorno
ports:
  - "${APP_PORT}:80"
```

Separar responsabilidades: cada servicio corre en su propio contenedor. No mezclar base de datos y servidor web en el mismo.

Usar volúmenes nombrados para los datos importantes (bases de datos, uploads) en lugar de bind mounts, ya que Docker los gestiona de forma más segura.

---

## Referencia de comandos Docker Compose

| Comando | Descripción |
|---|---|
| `docker compose up -d` | Levantar todos los servicios en segundo plano |
| `docker compose down` | Detener y eliminar contenedores y redes |
| `docker compose down -v` | Lo anterior, eliminando también los volúmenes |
| `docker compose ps` | Estado de los servicios del proyecto |
| `docker compose logs` | Registros de todos los servicios |
| `docker compose logs <servicio>` | Registros de un servicio concreto |
| `docker compose restart` | Reiniciar todos los servicios |
| `docker compose pull` | Descargar las versiones más recientes de las imágenes |
| `docker compose build` | Construir imágenes definidas con `build:` |
| `docker compose exec <servicio> <cmd>` | Ejecutar un comando en un servicio activo |
| `docker compose config` | Mostrar la configuración final con variables resueltas |

---

## Conclusiones

Docker Compose transforma la gestión de stacks multi-servicio: en lugar de encadenar comandos `docker run` con flags largos, toda la arquitectura queda documentada en un fichero versionable. Los tres ejercicios de esta práctica cubren los casos más frecuentes del día a día:

- Un servicio único con contenido estático (Nginx)
- Un stack clásico web + base de datos con persistencia real (PHP + MariaDB)
- Configuración parametrizada para múltiples entornos mediante `.env`
