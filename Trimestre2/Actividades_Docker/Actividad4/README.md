# 🐳 Docker — Práctica 4: Almacenamiento persistente y comunicación entre contenedores

**Arturo Gutierrez Sanchez**

---

## Introducción

Por defecto, todo lo que ocurre dentro de un contenedor desaparece cuando este se elimina. Esta práctica aborda las dos herramientas que Docker ofrece para superar esa limitación — **volúmenes** y **bind mounts** — y también cómo hacer que varios contenedores hablen entre sí mediante **redes personalizadas**.

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

## Glosario rápido

**Volumen Docker** — Unidad de almacenamiento gestionada íntegramente por Docker. Los datos sobreviven al ciclo de vida del contenedor y pueden compartirse entre varios.

**Bind Mount** — Enlace directo entre una carpeta del sistema anfitrión y un directorio interno del contenedor. Ideal durante el desarrollo.

**Red personalizada** — Segmento de red virtual creado por Docker que permite a los contenedores que la comparten encontrarse por nombre, sin necesidad de conocer sus IPs.

---

## Bloque 1 — Volúmenes Docker

### Crear un volumen y montarlo en un contenedor

Primero creamos el volumen:

```bash
docker volume create volumen_practica
```

Comprobamos que aparece en el listado:

```bash
docker volume ls
```

```text
DRIVER    VOLUME NAME
local     volumen_practica
```

Arrancamos un contenedor Ubuntu que mantenga el volumen montado en `/datos`:

```bash
docker run -d --name contenedor_datos -v volumen_practica:/datos ubuntu sleep 1000
```

El flag `-v volumen_practica:/datos` indica a Docker que conecte el volumen al directorio `/datos` dentro del contenedor. `sleep 1000` simplemente mantiene el contenedor activo los segundos que necesitamos.

---

### Escribir datos en el volumen y comprobar que persisten

Creamos un fichero dentro del volumen desde el propio contenedor:

```bash
docker exec contenedor_datos bash -c "echo 'Docker persistente' > /datos/info.txt"
```

Verificamos su contenido:

```bash
docker exec contenedor_datos cat /datos/info.txt
```

```text
Docker persistente
```

Si queremos ver los metadatos del volumen (ruta en el host, fecha de creación, etc.):

```bash
docker volume inspect volumen_practica
```

Devuelve un JSON con el punto de montaje real en el sistema de ficheros del host.

---

### La prueba de persistencia

Eliminamos el contenedor por completo:

```bash
docker rm -f contenedor_datos
```

Creamos uno nuevo que use exactamente el mismo volumen:

```bash
docker run -d --name contenedor_nuevo -v volumen_practica:/datos ubuntu sleep 1000
```

Leemos el fichero de antes:

```bash
docker exec contenedor_nuevo cat /datos/info.txt
```

```text
Docker persistente
```

El archivo sigue ahí. El contenedor desapareció, pero los datos permanecen en el volumen.

---

## Bloque 2 — Bind Mounts

### Servir contenido propio con Nginx

Creamos una carpeta en el host y colocamos dentro un fichero HTML sencillo:

```bash
mkdir -p ~/web_compartida
echo "<h1>Servidor Docker</h1>" > ~/web_compartida/index.html
```

Arrancamos Nginx enlazando esa carpeta al directorio que el servidor usa para servir contenido, en modo solo lectura (`:ro`):

```bash
docker run -d --name nginx_bind -p 8090:80 -v ~/web_compartida:/usr/share/nginx/html:ro nginx
```

Verificamos que responde:

```bash
curl http://localhost:8090
```

```html
<h1>Servidor Docker</h1>
```

---

### Edición en caliente desde el host

Modificamos el HTML directamente desde el sistema anfitrión, sin tocar el contenedor:

```bash
echo "<h1>Contenido actualizado</h1>" > ~/web_compartida/index.html
```

Volvemos a consultar el servidor:

```bash
curl http://localhost:8090
```

```html
<h1>Contenido actualizado</h1>
```

El cambio es inmediato porque el bind mount apunta directamente al fichero del host. No hace falta reiniciar ni reconstruir nada. Esta es la principal ventaja del bind mount en entornos de desarrollo.

---

## Bloque 3 — Redes Docker

### Crear una red y conectar dos contenedores

Creamos la red personalizada de tipo bridge:

```bash
docker network create red_practica
```

Confirmamos que existe:

```bash
docker network ls
```

```text
NETWORK ID     NAME            DRIVER    SCOPE
xxxxxx         red_practica    bridge    local
```

Arrancamos un servidor Nginx conectado a esa red:

```bash
docker run -d --name servidor_web --network red_practica nginx
```

Ahora lanzamos un segundo contenedor Ubuntu en la misma red, con sesión interactiva:

```bash
docker run -it --name cliente --network red_practica ubuntu bash
```

Dentro del contenedor `cliente`, instalamos curl y hacemos una petición al servidor usando **su nombre**:

```bash
apt update && apt install -y curl
curl http://servidor_web
```

La respuesta es la página de bienvenida de Nginx. Los contenedores se han encontrado por nombre (`servidor_web`), sin especificar ninguna IP, gracias a la red personalizada.

---

### Inspeccionar la red

```bash
docker network inspect red_practica
```

El JSON resultante muestra los contenedores conectados, sus IPs internas y la configuración del driver bridge.

---

## Limpieza final

Eliminamos todos los recursos creados durante la práctica en un solo bloque:

```bash
docker rm -f nginx_bind servidor_web cliente contenedor_nuevo
docker network rm red_practica
docker volume rm volumen_practica
```

---

## Errores habituales

**"Volume already exists"** — El volumen ya fue creado en una ejecución anterior. Borrarlo y recrearlo:

```bash
docker volume rm volumen_practica
docker volume create volumen_practica
```

**"Network already exists"** — Mismo caso pero con la red:

```bash
docker network rm red_practica
docker network create red_practica
```

**Los contenedores no se ven entre sí** — Probablemente no están en la misma red. Comprobarlo con `docker network inspect red_practica` y verificar que ambos aparecen en la lista de contenedores conectados.

**Puerto ya en uso** — Cambiar el puerto del host por uno libre:

```bash
docker run -d -p 8091:80 nginx
```

---

## Referencia de comandos

| Comando | Descripción |
|---|---|
| `docker volume create <nombre>` | Crear un volumen gestionado por Docker |
| `docker volume ls` | Listar volúmenes disponibles |
| `docker volume inspect <nombre>` | Ver detalles técnicos de un volumen |
| `docker volume rm <nombre>` | Eliminar un volumen |
| `docker network create <nombre>` | Crear una red personalizada |
| `docker network ls` | Listar redes disponibles |
| `docker network inspect <nombre>` | Ver contenedores y configuración de la red |
| `docker network rm <nombre>` | Eliminar una red |
| `docker run -v <vol>:<ruta>` | Montar un volumen en el contenedor |
| `docker run -v <ruta_host>:<ruta_cont>:ro` | Bind mount en modo solo lectura |
| `docker run --network <nombre>` | Conectar un contenedor a una red concreta |
| `docker rm -f <nombre>` | Forzar eliminación de un contenedor activo |

---

## Conclusiones

Esta práctica demuestra tres capacidades clave que hacen que Docker sea útil más allá de pruebas puntuales:

- Los **volúmenes** desvinculan los datos del ciclo de vida del contenedor, haciendo que bases de datos y ficheros importantes sobrevivan a reinicios y eliminaciones.
- Los **bind mounts** acortan el bucle de desarrollo al reflejar cambios del host en tiempo real dentro del contenedor.
- Las **redes personalizadas** permiten arquitecturas multi-contenedor donde cada servicio se ubica por nombre, sin gestión manual de IPs.
