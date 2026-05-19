# 🐳 Docker — Práctica 1: Instalación de Docker Engine en Ubuntu

**Arturo Gutierrez Sanchez**

---

## ¿Qué se hace en esta práctica?

El objetivo es dejar Docker completamente operativo sobre Ubuntu 24.04 LTS: desde la actualización del sistema hasta ejecutar el primer contenedor real. Al terminar, el entorno quedará listo para trabajar con contenedores sin necesidad de permisos de superusuario.

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

## Preparación del sistema

Antes de instalar nada, actualizamos el índice de paquetes y el software instalado para evitar conflictos de dependencias:

```bash
sudo apt update
sudo apt upgrade -y
```

A continuación instalamos las herramientas que Docker necesita para trabajar con repositorios HTTPS y verificar firmas GPG:

```bash
sudo apt install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```

---

## Añadir el repositorio oficial de Docker

Docker no está en los repositorios por defecto de Ubuntu, así que hay que registrar el suyo propio. Primero importamos su clave criptográfica para que el sistema confíe en los paquetes que descargue de él:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
```

Después registramos el repositorio estable de Docker Community Edition:

```bash
echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## Instalación de Docker Engine

Con el repositorio ya disponible, actualizamos el índice de paquetes e instalamos Docker junto con sus componentes principales:

```bash
sudo apt update

sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-compose-plugin
```

Verificamos que la instalación se completó correctamente consultando la versión:

```bash
docker --version
```

```text
Docker version 29.x.x
```

---

## Configurar Docker para usarlo sin sudo

Por defecto, los comandos Docker requieren privilegios de administrador. Para evitarlo, añadimos nuestro usuario al grupo `docker`:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

A partir de aquí todos los comandos `docker` funcionan sin anteponer `sudo`.

---

## Primera prueba: contenedor hello-world

Ejecutamos el contenedor oficial de prueba para confirmar que el daemon responde correctamente:

```bash
docker run hello-world
```

Docker descarga la imagen automáticamente si no la tiene en local y lanza el contenedor. Si aparece el mensaje de bienvenida de Docker, la instalación es correcta.

---

## Revisión del estado del sistema

Con Docker ya en marcha, conviene conocer los comandos básicos de diagnóstico:

```bash
# Estado del servicio en systemd
sudo systemctl status docker

# Información completa del daemon y el entorno
docker info

# Contenedores existentes (activos y parados)
docker ps -a

# Imágenes descargadas localmente
docker images
```

---

## Arranque automático con el sistema

Para que Docker se inicie solo cada vez que arranque la máquina:

```bash
sudo systemctl enable docker
```

---

## Resumen de verificaciones

| Comprobación | Estado |
|---|---|
| Docker Engine instalado | ✅ |
| Daemon activo y operativo | ✅ |
| Contenedor hello-world ejecutado | ✅ |
| Usuario configurado sin sudo | ✅ |
| Docker Compose disponible | ✅ |
| Arranque automático habilitado | ✅ |

---

## Medidas de seguridad aplicadas

- Repositorio registrado con firma GPG verificada, no descargado de fuentes no oficiales
- Permisos gestionados a través del grupo `docker` en lugar de ejecutar como root
- Servicio administrado por systemd, con arranque controlado y trazabilidad de estado

---

## Conclusión

Tras completar esta práctica el sistema dispone de Docker CE completamente funcional: el daemon corre como servicio del sistema, el usuario puede operar sin privilegios elevados y el entorno está preparado para continuar con las siguientes prácticas.
