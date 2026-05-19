# 🐳 Práctica Docker #1 — Despliegue y configuración de Docker en Ubuntu

## 👨‍💻 Autor

**Arturo Gutierrez Sanchez**

---

# 📌 Objetivo de la práctica

Instalar Docker Engine sobre Ubuntu 24.04 LTS, preparar el entorno de trabajo y comprobar el correcto funcionamiento del sistema de contenedores mediante pruebas elementales.

---

# 🖥️ Entorno de trabajo

| Componente            | Detalle          |
| --------------------- | ---------------- |
| Sistema operativo     | Ubuntu 24.04 LTS |
| Arquitectura          | x86_64           |
| Plataforma            | VirtualBox       |
| Motor de contenedores | Docker CE        |
| Usuario del sistema   | arturo           |

---

# ⚙️ 1. Puesta al día del sistema

Antes de proceder con la instalación de Docker, se actualiza la lista de paquetes y el software del sistema.

## 💻 Comandos utilizados

```bash
sudo apt update
sudo apt upgrade -y
```

## ✅ Resultado obtenido

* Sistema operativo actualizado sin errores.
* Repositorios refrescados correctamente.
* Dependencias del sistema preparadas.

---

# 📦 2. Instalación de dependencias necesarias

Docker requiere una serie de herramientas previas para gestionar repositorios seguros mediante HTTPS y claves GPG.

## 💻 Instalación

```bash
sudo apt install -y \
apt-transport-https \
ca-certificates \
curl \
gnupg \
lsb-release
```

## ✅ Resultado obtenido

Todos los paquetes requeridos se instalaron sin incidencias ni conflictos de dependencias.

---

# 🔑 3. Importación de la clave GPG oficial de Docker

Se descarga e importa la clave criptográfica de Docker para garantizar la integridad y autenticidad del repositorio.

## 💻 Comando ejecutado

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
```

## ✅ Resultado obtenido

La clave GPG quedó guardada correctamente en el almacén del sistema.

---

# 🌐 4. Registro del repositorio oficial de Docker

Se añade el repositorio estable de Docker Community Edition a las fuentes del sistema.

## 💻 Configuración aplicada

```bash
echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## ✅ Resultado obtenido

El repositorio fue añadido satisfactoriamente al sistema de paquetes.

---

# 🐳 5. Instalación del motor Docker

Se instala Docker Engine junto con sus componentes esenciales.

## 💻 Instalación

```bash
sudo apt update

sudo apt install -y \
docker-ce \
docker-ce-cli \
containerd.io \
docker-compose-plugin
```

## ✅ Resultado obtenido

Docker CE instalado con éxito, incluyendo el complemento de Docker Compose.

---

# 🔎 6. Comprobación de la instalación

Se verifica que Docker está correctamente instalado consultando su versión.

## 💻 Verificación

```bash
docker --version
```

## ✅ Salida esperada

```bash
Docker version 29.x.x
```

---

# 🔐 7. Configuración de ejecución sin privilegios de superusuario

Para poder utilizar Docker sin necesidad de anteponer `sudo`, se agrega el usuario al grupo `docker`.

## 💻 Pasos realizados

```bash
sudo groupadd docker

sudo usermod -aG docker $USER

newgrp docker
```

## ✅ Resultado obtenido

El usuario fue incorporado al grupo Docker de forma correcta.

---

# 🚀 8. Ejecución del primer contenedor: hello-world

Se lanza el contenedor de prueba oficial para confirmar que el daemon Docker responde correctamente.

## 💻 Comando

```bash
docker run hello-world
```

## ✅ Resultado obtenido

Docker descargó la imagen automáticamente y ejecutó el contenedor sin ningún problema.

---

# 📊 9. Revisión del estado del sistema Docker

## Consultar estado del servicio

```bash
sudo systemctl status docker
```

## Ver información completa del entorno

```bash
docker info
```

## Listar todos los contenedores

```bash
docker ps -a
```

## Ver imágenes almacenadas localmente

```bash
docker images
```

## ✅ Resultado obtenido

* Servicio Docker activo y operativo.
* Daemon en funcionamiento.
* Imagen `hello-world` disponible localmente.

---

# ⚡ 10. Activar Docker en el arranque del sistema

## 💻 Comando

```bash
sudo systemctl enable docker
```

## ✅ Resultado obtenido

Docker se inicia automáticamente cada vez que el sistema arranca.

---

# 🧪 Resumen de verificaciones

| Comprobación              | Estado |
| ------------------------- | ------ |
| Docker instalado          | ✅      |
| Daemon activo             | ✅      |
| Contenedor ejecutado      | ✅      |
| Usuario sin sudo          | ✅      |
| Docker Compose disponible | ✅      |
| Servicio persistente      | ✅      |

---

# 📚 Referencia de comandos empleados

```bash
docker --version
docker info
docker ps -a
docker images
docker run hello-world
sudo systemctl status docker
sudo systemctl enable docker
```

---

# 🔒 Medidas de seguridad aplicadas

* Uso exclusivo del repositorio oficial de Docker
* Verificación de integridad mediante clave GPG
* Gestión de permisos a través del grupo `docker`
* Administración del servicio con systemd

---

# 📈 Conclusión

La instalación y puesta en marcha de Docker sobre Ubuntu 24.04 se completó sin contratiempos, dejando el entorno preparado para:

* Lanzar y gestionar contenedores
* Administrar imágenes Docker locales
* Controlar el daemon del sistema
* Operar sin necesidad de permisos de root
* Continuar con actividades y prácticas futuras

---
