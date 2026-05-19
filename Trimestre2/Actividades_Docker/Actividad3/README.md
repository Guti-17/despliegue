# 🐳 Docker — Práctica 3: Imágenes, contenedores y su administración

**Arturo Gutierrez Sanchez**

---

## ¿Qué aprenderemos?

En esta práctica nos centramos en dos pilares fundamentales de Docker: las **imágenes** y los **contenedores**. Veremos cómo obtener imágenes desde Docker Hub, instanciarlas como contenedores con nombres propios y recorrer todo su ciclo de vida hasta eliminarlos limpiamente.

---

## Conceptos previos

Antes de comenzar conviene tener claros estos términos:

**Imagen Docker** — Plantilla inmutable que empaqueta una aplicación con todo lo que necesita para funcionar. No se modifica al usarla.

**Contenedor Docker** — Proceso en ejecución generado a partir de una imagen. Puede arrancarse, pausarse y eliminarse de forma independiente.

**Docker Hub** — Registro público en la nube desde el que Docker descarga imágenes oficiales y de la comunidad.

---

## Entorno de la práctica

| Campo | Valor |
|---|---|
| Sistema operativo | Ubuntu 24.04 LTS |
| Arquitectura | x86_64 |
| Plataforma de virtualización | VirtualBox |
| Motor de contenedores | Docker CE |
| Usuario del sistema | arturo |

---

## Desarrollo de la práctica

### Obtener las imágenes necesarias

Descargamos tres imágenes desde Docker Hub:

```bash
docker pull ubuntu
docker pull hello-world
docker pull nginx
```

Cada una quedará almacenada en el repositorio local de la máquina. Podemos verificarlo con:

```bash
docker images
```

La salida mostrará algo similar a esto:

```text
REPOSITORY    TAG       IMAGE ID       SIZE
ubuntu        latest    xxxxxxxx       77MB
hello-world   latest    xxxxxxxx       13kB
nginx         latest    xxxxxxxx       180MB
```

---

### Crear contenedores con nombre propio

Lanzamos tres contenedores a partir de `hello-world`, asignando un identificador distinto a cada uno mediante `--name`:

```bash
docker run --name prueba1 hello-world
docker run --name prueba2 hello-world
docker run --name prueba3 hello-world
```

Usar nombres explícitos evita depender de los identificadores aleatorios que Docker asigna por defecto, facilitando la gestión posterior.

Para comprobar que los tres existen:

```bash
docker ps -a
```

```text
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     NAMES
abc123...      hello-world   "/hello"   1 minute ago    Exited (0) 1 minute ago   prueba1
def456...      hello-world   "/hello"   1 minute ago    Exited (0) 1 minute ago   prueba2
ghi789...      hello-world   "/hello"   1 minute ago    Exited (0) 1 minute ago   prueba3
```

---

### Eliminar contenedores paso a paso

Primero detenemos `prueba1` (aunque ya haya terminado, es buena práctica):

```bash
docker stop prueba1
```

A continuación lo eliminamos:

```bash
docker rm prueba1
```

Verificamos que solo quedan los otros dos:

```bash
docker ps -a
```

Por último, eliminamos los contenedores restantes en una sola instrucción:

```bash
docker rm prueba2 prueba3
```

Confirmamos que el listado ha quedado vacío:

```bash
docker ps -a
```

```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS   NAMES
```

---

## Errores frecuentes y cómo resolverlos

**"Conflict. The container name is already in use"**
Significa que ya existe un contenedor con ese nombre, aunque esté parado. Hay que eliminarlo antes de reutilizar el nombre:

```bash
docker rm nombre_contenedor
```

**"No such image"**
La imagen no existe localmente. Hay que descargarla primero:

```bash
docker pull nombre_imagen
```

**"Cannot remove a running container"**
No se puede borrar un contenedor que está activo. Primero hay que detenerlo:

```bash
docker stop nombre_contenedor
docker rm nombre_contenedor
```

---

## Buenas prácticas

Nombra siempre los contenedores con algo descriptivo, no dejes que Docker asigne nombres aleatorios:

```bash
# ❌ Sin nombre — difícil de identificar después
docker run nginx

# ✅ Con nombre — claro y manejable
docker run --name servidor-web nginx
```

Para eliminar de golpe todos los contenedores que ya no están en uso:

```bash
docker container prune -f
```

---

## Referencia de comandos de la práctica

| Comando | Para qué sirve | Ejemplo de uso |
|---|---|---|
| `docker pull` | Descargar una imagen de Docker Hub | `docker pull ubuntu` |
| `docker images` | Listar imágenes almacenadas localmente | `docker images` |
| `docker run --name` | Crear un contenedor con nombre definido | `docker run --name web nginx` |
| `docker ps` | Ver únicamente los contenedores en marcha | `docker ps` |
| `docker ps -a` | Ver todos los contenedores, incluso parados | `docker ps -a` |
| `docker stop` | Detener un contenedor activo | `docker stop prueba1` |
| `docker rm` | Borrar un contenedor detenido | `docker rm prueba1` |
| `docker container prune` | Borrar todos los contenedores parados | `docker container prune -f` |

---

## Resumen de lo trabajado

- Descarga de imágenes desde Docker Hub con `docker pull`
- Consulta del repositorio local de imágenes con `docker images`
- Creación de contenedores identificados por nombre con `--name`
- Diferencia entre `docker ps` y `docker ps -a`
- Eliminación individual y múltiple de contenedores con `docker rm`
- Resolución de los errores más habituales al gestionar contenedores
