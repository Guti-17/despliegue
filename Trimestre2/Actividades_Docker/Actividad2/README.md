# 🐳 Docker — Práctica 2: Ciclo de vida y administración de contenedores

**Arturo Gutierrez Sanchez**

---

## ¿Qué se trabaja en esta práctica?

Docker encapsula aplicaciones en entornos completamente aislados denominados **contenedores**, cada uno con sus propias dependencias y configuración. En esta práctica se explora cómo interactuar con ellos desde la línea de comandos: desde crearlos y arrancarlos hasta inspeccionarlos, monitorizarlos y detenerlos.

Áreas cubiertas:

- Sesiones interactivas dentro de contenedores
- Servicios corriendo en segundo plano
- Seguimiento de registros (logs)
- Exposición de puertos al sistema anfitrión
- Inspección interna y ejecución remota de comandos
- Monitorización de recursos

---

## Entorno de trabajo

| Componente | Valor |
|---|---|
| SO | Ubuntu 24.04 LTS |
| Arquitectura | x86_64 |
| Hipervisor | VirtualBox |
| Motor Docker | Docker CE |
| Usuario | arturo |

---

## Paso 1 — Sesión interactiva en un contenedor Ubuntu

Arrancamos un contenedor basado en Ubuntu y accedemos directamente a su shell:

```bash
docker run -it ubuntu bash
```

Una vez dentro, comprobamos el sistema:

```bash
cat /etc/os-release
```

El flag `-it` combina modo interactivo (`-i`) con asignación de terminal (`-t`), lo que permite trabajar como si estuviéramos dentro de una máquina real. Para abandonar el contenedor:

```bash
exit
```

---

## Paso 2 — Servidor web Nginx en background

Desplegamos Nginx como servicio en segundo plano, mapeando el puerto 80 del contenedor al 8080 del host:

```bash
docker run -d \
  --name nginx-server \
  -p 8080:80 \
  nginx
```

| Flag | Significado |
|---|---|
| `-d` | Modo desacoplado (background) |
| `--name` | Identificador personalizado del contenedor |
| `-p 8080:80` | Puerto del host → puerto del contenedor |

Nginx queda accesible en `http://localhost:8080` sin bloquear la terminal.

---

## Paso 3 — Lectura de logs

Logs estáticos (histórico):

```bash
docker logs nginx-server
```

Logs en vivo (streaming):

```bash
docker logs -f nginx-server
```

La opción `-f` (follow) mantiene abierta la salida del registro y muestra las nuevas entradas en tiempo real, útil para depurar.

---

## Paso 4 — Verificar que Nginx responde

Desde la terminal del host:

```bash
curl http://localhost:8080
```

O directamente desde el navegador en `http://localhost:8080`. En ambos casos se recibe la página de bienvenida de Nginx, confirmando que el mapeo de puertos funciona correctamente.

---

## Paso 5 — Varios contenedores simultáneos

Lanzamos tres instancias del contenedor de prueba `hello-world` con nombres distintos:

```bash
docker run --name test1 hello-world
docker run --name test2 hello-world
docker run --name test3 hello-world
```

Esto demuestra que Docker puede gestionar múltiples contenedores independientes a la vez, cada uno con su propio ciclo de vida.

---

## Paso 6 — Inventario de contenedores

Para ver todos los contenedores del sistema (activos y parados):

```bash
docker ps -a
```

Sin el flag `-a` solo aparecen los que están en ejecución en ese momento. Con él, se listan también los detenidos, mostrando su nombre, imagen base, estado y puertos.

---

## Paso 7 — Parar y reanudar un contenedor

Detener Nginx:

```bash
docker stop nginx-server
```

Volver a arrancarlo sin recrearlo:

```bash
docker start nginx-server
```

A diferencia de `docker run`, el comando `start` recupera el contenedor ya existente con toda su configuración previa, sin descargar ni crear nada nuevo.

---

## Paso 8 — Inspección interna detallada

```bash
docker inspect nginx-server
```

Devuelve un JSON completo con toda la información del contenedor:

- Identificador único (ID largo)
- Configuración de red e IP interna
- Variables de entorno definidas
- Imagen y capa base utilizada
- Estado actual y fechas de arranque
- Volúmenes y montajes configurados

---

## Paso 9 — Ejecutar comandos dentro del contenedor en marcha

Sin entrar al shell, ejecutamos un comando puntual:

```bash
docker exec nginx-server ls -la /usr/share/nginx/html
```

Para una sesión interactiva completa dentro del contenedor activo:

```bash
docker exec -it nginx-server bash
```

Desde ahí podemos navegar el sistema de archivos:

```bash
pwd
ls
cat /usr/share/nginx/html/index.html
exit
```

`exec` se diferencia de `run` en que actúa sobre un contenedor **ya en marcha**, sin crear uno nuevo.

---

## Paso 10 — Monitorización de recursos en tiempo real

```bash
docker stats nginx-server
```

Muestra un panel actualizado continuamente con:

| Métrica | Descripción |
|---|---|
| CPU % | Consumo de procesador |
| MEM USAGE | Memoria RAM utilizada |
| NET I/O | Tráfico de red entrante/saliente |
| BLOCK I/O | Lecturas y escrituras en disco |

Para salir de la vista de estadísticas: `Ctrl + C`.

---

## Ciclo de vida de un contenedor Docker

```
docker run        docker stop       docker rm
   ↓                   ↓                ↓
CREATED  ──►  RUNNING  ──►  STOPPED  ──►  REMOVED
                 ▲               │
                 └───────────────┘
                   docker start
```

Un contenedor detenido **no se pierde**: conserva su estado y configuración hasta que se elimina explícitamente con `docker rm`.

---

## Referencia rápida de comandos

| Comando | Descripción |
|---|---|
| `docker run -it <imagen> bash` | Contenedor interactivo |
| `docker run -d -p <host>:<cont> <imagen>` | Contenedor en background con puerto |
| `docker ps` | Contenedores en ejecución |
| `docker ps -a` | Todos los contenedores |
| `docker logs <nombre>` | Ver logs |
| `docker logs -f <nombre>` | Logs en tiempo real |
| `docker stop <nombre>` | Detener contenedor |
| `docker start <nombre>` | Reanudar contenedor detenido |
| `docker exec <nombre> <cmd>` | Ejecutar comando en contenedor activo |
| `docker exec -it <nombre> bash` | Shell interactivo en contenedor activo |
| `docker inspect <nombre>` | Configuración completa en JSON |
| `docker stats <nombre>` | Monitorización de recursos |

---

## Conclusiones

Con esta práctica se han asentado los fundamentos del trabajo cotidiano con Docker:

- Entender qué ocurre en cada fase del ciclo de vida de un contenedor
- Distinguir cuándo usar `run`, `start` y `exec`
- Consultar logs e inspeccionar el estado interno sin detener los servicios
- Vigilar el consumo de recursos de forma no invasiva
- Gestionar varios contenedores de manera ordenada y con nombres significativos
