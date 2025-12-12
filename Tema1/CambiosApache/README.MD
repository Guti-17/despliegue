### Ejercicio 4: Cambiar configuracion de apache

1.  Apache utilizará el puerto 81 además del 80

    Para esto primero primero entramos al archvio de configuracion de los puertos
    <p align="center">
      <img src="./images/01.png" alt="paso1">
    </p>

    Ahora agregamos para que esecuche por el puerto 81 `control + o` y `control + x` para guardar y salir
    <p align="center">
      <img src="./images/02.png" alt="paso2">
    </p>

    Ahora simplemente reiniciamos el servicio con `sudo systemctl restart apache2` y ya estaría escuchando por ese puerto
    <p align="center">
      <img src="./images/03.png" alt="paso3">
    </p>

2.  Añadir el dominio "marisma.intranet" en el fichero "hosts"

    Ejecuta el comando para editar: Escribe `sudo nano /etc/hosts` y presiona Enter.
    <p align="center">
      <img src="./images/04.png" alt="paso4">
    </p>

    Añade la entrada: En una nueva línea, escribe la dirección IP del servidor, un espacio, y luego el dominio: 192.168.1.100 marisma.intranet
    <p align="center">
      <img src="./images/05.png" alt="paso5">
    </p>

3.  Cambia la directiva "ServerTokens" para mostrar el nombre del producto.
    <p align="center">
      <img src="./images/06.png" alt="paso6">
    </p>
    <p align="center">
      <img src="./images/07.png" alt="paso7">
    </p>

4.  Comprueba si se visualiza el pie de página en las páginas generadas por Apache (por ejemplo, en las páginas de error). Cambia el valor de la directiva "ServerSignature" y comprueba que funciona correctamente.
    <p align="center">
      <img src="./images/08.png" alt="paso8">
    </p>
    <p align="center">
      <img src="./images/09.png" alt="paso9">
    </p>
    <p align="center">
      <img src="./images/10.png" alt="paso10">
    </p>

5.  Crea un directorio "prueba" y otro directorio "prueba2". Incluye un par de páginas en cada una de ellas.
    <p align="center">
      <img src="./images/11.png" alt="paso11">
    </p>
    <p align="center">
      <img src="./images/12.png" alt="paso12">
    </p>
    <p align="center">
      <img src="./images/13.png" alt="paso13">
    </p>
6.  Redirecciona el contenido de la carpeta "prueba" hacia "prueba2"
    <p align="center">
      <img src="./images/14.png" alt="paso14">
    </p>
    <p align="center">
      <img src="./images/15.png" alt="paso15">
    </p>
    <p align="center">
      <img src="./images/16.png" alt="paso16">
    </p>

7.  Es posible redireccionar tan solo una página en lugar de toda la carpeta. Pruébalo.
    <p align="center">
      <img src="./images/17.png" alt="paso17">
    </p>

8.  Usa la directiva userdir
    <p align="center">
      <img src="./images/18.png" alt="paso18">
    </p>

9.  Usa la directiva alias para redireccionar a una carpeta dentro del directorio de usuario.
    <p align="center">
      <img src="./images/19.png" alt="paso19">
    </p>
    <p align="center">
      <img src="./images/20.png" alt="paso20">
    </p>
    <p align="center">
      <img src="./images/21.png" alt="paso21">
    </p>
    <p align="center">
      <img src="./images/22.png" alt="paso22">
    </p>

10. ¿Para qué sirve la directiva Options y dónde aparece. Comprueba si apache indexa los directorios. Si es así, ¿cómo lo desactivamos?

- `Options` controla características para un directorio (como listar archivos).
- `Indexes` permite mostrar el contenido de un directorio si no hay archivo índice.
- Para desactivar la indexación, usar:

apache
Options -Indexes

Recargamos apache tras los cambios con `sudo systemctl reload apache2`

# Trabajando con scripts (Debes publicarlos en Github)

### 1.Crea un script que añada un nombre de dominio y una ip al fichero hosts. Debemos comprobar que no existe dicho dominio en el fichero hosts

Primero creamos el reepositorio n la ruta d scripts apach, entramos y vamos a editar el fichero

<p align="center">
  <img src="./images/23.png" alt="paso23">
</p>

Agregamos el codigo para que podamos agregar el puerto al fichero de configuracion de apache, salimos y guardamos con `control + o` y `control + x`

<p align="center">
  <img src="./images/24.png" alt="paso24">
</p>

Ahora probamos el script con `./agregar_puerto_apache.sh 81` y comprobamos que se ha agregado el puerto 81 al fichero de configuracion de apache

<p align="center">
  <img src="./images/25.png" alt="paso25">
</p>

Ahora creamos el fichero añadir_hosts.sh y agregamos el codigo para que podamos agregar el dominio y la ip al fichero hosts

<p align="center">
  <img src="./images/26.png" alt="paso26">
</p>

Agregamos ahora el codigo para ver si el dominio existe en el fichero

<p align="center">
  <img src="./images/27.png" alt="paso27">
</p>

guardamos salimos mostramos de que eel fichero se crio y probamos

<p align="center">
  <img src="./images/28.png" alt="paso28">
</p>

### 2. Crea un script que nos permita crear una página web con un título, una cabecera y un mensaje

Ahora creamos el fichero de crear_web.sh y agregamos el codigo para que podamos crear una pagina web con un titulo, una cabecera y un mensaje

<p align="center">
  <img src="./images/29.png" alt="paso29">
</p>

Ahora mostramos de que hemos creado los 3 ficheros y le damos peermisos con chmod de ejecución y mostramos de nuevo los permisos

<p align="center">
  <img src="./images/32.png" alt="paso32">
</p>

Ahora vamos a probar a crear la web solamentee con el script para eso usamos `./crear_web.sh marisma.intranet "Hola mundo" "Pagina creada con script"` , y mostramos l contnido

<p align="center">
  <img src="./images/33.png" alt="paso33">
</p>

1 Mostramos que ahora todo y vamos a realizar una prueba local

<p align="center">
  <img src="./images/34.png" alt="paso34">
</p>

### 3. Crea un script que añada un nombre de dominio y una ip al fichero hosts. Debemos comprobar que no existe dicho dominio en el fichero hosts

Entramos al etc/hosts para agregar la ip y el dominio prueba.local

<p align="center">
  <img src="./images/35.png" alt="paso35">
</p>

Usamos sed para agregar la ip y el dominio prueba.local al fichero hosts

<p align="center">
  <img src="./images/36.png" alt="paso36">
</p>

Ahora hacemos un sudo ./agregar_puerto_apache.sh 8080 para agregar el puerto 8080 al fichero de configuracion de apache

<p align="center">
  <img src="./images/37.png" alt="paso37">
</p>

Ahora vemos la última linea del fichero hosts para ver si se ha agregado correctamente

<p align="center">
  <img src="./images/38.png" alt="paso38">
</p>

Y por ultimo reiniciamos apache con `sudo systemctl restart apache2`

<p align="center">
  <img src="./images/39.png" alt="paso39">
</p>


