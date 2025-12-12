<p>Ejercicio 6: Práctica Servidor Web (1º Trimestre)</p>
<p>Guía paso a paso para instalar y configurar un servidor web en Ubuntu. Se montan Apache con WordPress, una aplicación Python usando mod_wsgi, estadísticas con AWStats y un segundo servidor Nginx en el puerto 8080 con PHP y phpMyAdmin. Se incluyen comandos, rutas, capturas y comprobaciones.</p>
<br>

<p>Entorno y requisitos</p>
<p>Sistema operativo: Ubuntu Server o Desktop (recomendado 22.04 o 24.04)<br>
Usuario con permisos sudo<br>
Navegador para probar: Firefox o Chrome<br>
Dominios internos en /etc/hosts: centro.intranet, departamentos.centro.intranet, servidor2.centro.intranet</p>
<br>

<p>Carpetas del repositorio:<br>
docs/ para documentación y apuntes<br>
images/ para capturas de pantalla</p>
<br>

<p>0) Preparar el sistema</p>
<p>Actualiza el sistema y las herramientas básicas:</p>
<p>sudo apt update && sudo apt -y upgrade<br>
sudo apt -y install curl wget unzip git</p>
<br>
<p><img src="images/00-actualizar.png"><br>
<img src="images/01-instalar-curl.png"></p>
<br>

<p>1) Configuración de dominios locales</p>
<p>Edita el archivo /etc/hosts:</p>
<p>sudo nano /etc/hosts</p>
<br>
<p>Añade al final del archivo:<br>
127.0.0.1   centro.intranet<br>
127.0.0.1   departamentos.centro.intranet<br>
127.0.0.1   servidor2.centro.intranet</p>
<br>
<p>Guarda y cierra con CTRL+O y CTRL+X.</p>
<br>
<p>Comprobación:</p>
<p>ping -c 1 centro.intranet<br>
ping -c 1 departamentos.centro.intranet<br>
ping -c 1 servidor2.centro.intranet</p>
<br>
<p><img src="images/02-entrar-hosts.png"><br>
<img src="images/03-modificamos-hosts.png"><br>
<img src="images/04-comprobacion-ping.png"></p>
<br>

<p>2) Instalar Apache y crear la estructura de los sitios</p>
<p>Instalación de Apache y herramientas:</p>
<p>sudo apt -y install apache2 apache2-utils</p>
<br>
<p>Crear carpetas para los sitios:</p>
<p>sudo mkdir -p /var/www/centro.intranet<br>
sudo mkdir -p /var/www/departamentos.centro.intranet</p>
<br>
<p>Permisos:</p>
<p>sudo chown -R www-data:www-data /var/www/centro.intranet /var/www/departamentos.centro.intranet<br>
sudo chmod -R 755 /var/www</p>
<br>
<p>Verificar estado de Apache:</p>
<p>sudo systemctl status apache2 --no-pager</p>
<br>
<p><img src="images/05-instalacion-apache.png"><br>
<img src="images/06-creamos-directorios.png"><br>
<img src="images/07-damos-permisos.png"><br>
<img src="images/08-verificamos.png"></p>
<br>

<p>3) Activar PHP y MySQL</p>
<p>Instalar PHP y MySQL:</p>
<p>sudo apt -y install libapache2-mod-php php php-mysql php-cli php-curl php-xml php-gd<br>
sudo apt -y install mysql-server</p>
<br>
<p>Seguridad básica de MySQL:</p>
<p>sudo mysql_secure_installation</p>
<br>
<p>Reiniciar Apache:</p>
<p>sudo systemctl restart apache2</p>
<br>
<p><img src="images/09-instalamos-php-y-mysql.png"><br>
<img src="images/10-seguridad-basica.png"><br>
<img src="images/11-reiniciamos-servicio-apache2.png"></p>
<br>

<p>4) Configurar VirtualHosts en Apache</p>
<p>Se crearán dos sitios: centro.intranet y departamentos.centro.intranet.</p>
<br>

<p>4.1) VirtualHost de centro.intranet</p>
<p>sudo nano /etc/apache2/sites-available/centro.intranet.conf</p>
<br>
<p>&lt;VirtualHost *:80&gt;<br>
ServerName centro.intranet<br>
DocumentRoot /var/www/centro.intranet<br><br>
&lt;Directory /var/www/centro.intranet&gt;<br>
AllowOverride All<br>
Options Indexes FollowSymLinks<br>
Require all granted<br>
&lt;/Directory&gt;<br><br>
ErrorLog ${APACHE_LOG_DIR}/centro_error.log<br>
CustomLog ${APACHE_LOG_DIR}/centro_access.log combined<br>
&lt;/VirtualHost&gt;</p>
<br>
<p><img src="images/12-configuramos-virtualhost-apache.png"><br>
<img src="images/13-contenido-de-lo-anterior.png"></p>
<br>

<p>4.2) VirtualHost de departamentos.centro.intranet</p>
<p>sudo nano /etc/apache2/sites-available/departamentos.centro.intranet.conf</p>
<br>
<p>&lt;VirtualHost *:80&gt;<br>
ServerName departamentos.centro.intranet<br>
DocumentRoot /var/www/departamentos.centro.intranet<br><br>
WSGIDaemonProcess departamentos user=www-data group=www-data threads=5<br>
WSGIProcessGroup departamentos<br>
WSGIScriptAlias / /var/www/departamentos.centro.intranet/wsgi.py<br><br>
&lt;Directory /var/www/departamentos.centro.intranet&gt;<br>
Require all granted<br>
&lt;/Directory&gt;<br><br>
ErrorLog ${APACHE_LOG_DIR}/departamentos_error.log<br>
CustomLog ${APACHE_LOG_DIR}/departamentos_access.log combined<br>
&lt;/VirtualHost&gt;</p>
<br>
<p>Habilitar sitios y módulos:</p>
<p>sudo a2ensite centro.intranet.conf departamentos.centro.intranet.conf<br>
sudo a2enmod rewrite<br>
sudo systemctl reload apache2</p>
<br>
<p><img src="images/14-ahora-la-otra.png"><br>
<img src="images/15-contenido-de-la-otra.png"><br>
<img src="images/16-habilitamos-modulos-necesarios.png"></p>
<br>

<p>5) Instalación de WordPress en centro.intranet</p>
<p>5.1) Crear base de datos y usuario:</p>
<p>sudo mysql</p>
<br>
<p>CREATE DATABASE wordpress CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;<br>
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'wp_password_seguro';<br>
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';<br>
FLUSH PRIVILEGES;<br>
EXIT;</p>
<br>
<p><img src="images/17-entramos-mysql.png"><br>
<img src="images/18-base-datos-y-usuario.png"></p>
<br>

<p>5.2) Descargar WordPress y preparar DocumentRoot:</p>
<p>cd /tmp<br>
wget https://wordpress.org/latest.zip<br>
unzip latest.zip<br>
sudo rsync -avP wordpress/ /var/www/centro.intranet/</p>
<br>
<p><img src="images/19-descargamos-wordpress.png"></p>
<br>

<p>5.3) Configurar wp-config.php:</p>
<p>cd /var/www/centro.intranet<br>
sudo cp wp-config-sample.php wp-config.php<br>
sudo nano wp-config.php</p>
<br>
<p>define('DB_NAME', 'wordpress');<br>
define('DB_USER', 'wpuser');<br>
define('DB_PASSWORD', 'wp_password_seguro');<br>
define('DB_HOST', 'localhost');</p>
<br>
<p>Permisos:<br>
sudo chown -R www-data:www-data /var/www/centro.intranet<br>
sudo find /var/www/centro.intranet -type d -exec chmod 755 {} \;<br>
sudo find /var/www/centro.intranet -type f -exec chmod 644 {} \;</p>
<br>
<p><img src="images/20-configuramos-wp-config.png"><br>
<img src="images/21-configuracion.png"><br>
<img src="images/22-permisos.png"><br>
<img src="images/23_error.png"><br>
<img src="images/24-wordpress.png"></p>
<br>

<p>6) Activar mod_wsgi y desplegar app Python en departamentos.centro.intranet</p>
<p>sudo apt -y install libapache2-mod-wsgi-py3 python3-venv<br>
sudo a2enmod wsgi<br>
sudo systemctl restart apache2</p>
<br>
<p>Crear aplicación mínima:</p>
<p>sudo bash -c 'cat &gt; /var/www/departamentos.centro.intranet/app.py' &lt;&lt; 'EOF'<br>
def application(environ, start_response):<br>
    status = '200 OK'<br>
    headers = [('Content-Type', 'text/html; charset=utf-8')]<br>
    start_response(status, headers)<br>
    body = [b"&lt;h1&gt;Aplicación Python OK&lt;/h1&gt;",<br>
            b"&lt;p&gt;Ruta: %s&lt;/p&gt;" % environ.get('PATH_INFO', '/').encode('utf-8')]<br>
    return body<br>
EOF</p>
<br>
<p>sudo bash -c 'cat &gt; /var/www/departamentos.centro.intranet/wsgi.py' &lt;&lt; 'EOF'<br>
import sys<br>
sys.path.insert(0, '/var/www/departamentos.centro.intranet')<br>
from app import application<br>
EOF</p>
<br>
<p>sudo chown -R www-data:www-data /var/www/departamentos.centro.intranet</p>
<br>
<p><img src="images/26-Estructura-basica.png"><br>
<img src="images/27-error500.png"><br>
<img src="images/28-funcionando.png"></p>
<br>

<p>6.2) Protección con autenticación HTTP básica</p>
<p>sudo apt -y install apache2-utils<br>
sudo htpasswd -c /etc/apache2/.htpasswd profesor</p>
<br>
<p>Editar VirtualHost para restringir acceso:</p>
<p>&lt;Directory /var/www/departamentos.centro.intranet&gt;<br>
AuthType Basic<br>
AuthName "Acceso restringido"<br>
AuthUserFile /etc/apache2/.htpasswd<br>
Require valid-user<br>
&lt;/Directory&gt;</p>
<br>
<p>sudo systemctl reload apache2</p>
<p><img src="images/29-usuario-profesor.png"><br>
<img src="images/30-ajustamos-Directory.png"><br>
<img src="images/31-verificacion.png"></p>
<br>

<p>7) Instalar y configurar AWStats</p>
<p>sudo apt -y install awstats<br>
sudo a2enmod cgi<br>
sudo a2enconf awstats<br>
sudo systemctl reload apache2</p>
<br>
<p>Crear configuración específica del sitio:</p>
<p>sudo cp /etc/awstats/awstats.conf /etc/awstats/awstats.centro.intranet.conf<br>
sudo nano /etc/awstats/awstats.centro.intranet.conf</p>
<br>
<p>LogFile="/var/log/apache2/centro_access.log"<br>
SiteDomain="centro.intranet"<br>
HostAliases="localhost 127.0.0.1 www.centro.intranet"<br>
LogFormat=1</p>
<br>
<p>Actualizar estadísticas iniciales:<br>
sudo /usr/lib/cgi-bin/awstats.pl -config=centro.intranet -update</p>
<br>
<p><img src="images/32-LogFile.png"><br>
<img src="images/33-SiteDomain.png"><br>
<img src="images/34-HostAliases.png"><br>
<img src="images/35-updatee.png"><br>
<img src="images/36-pagina.png"></p>
<br>

<p>8) Segundo servidor: Nginx en puerto 8080 con PHP y phpMyAdmin</p>
<p>sudo apt -y install nginx php-fpm php-mysql</p>
<br>
<p>Crear DocumentRoot y PHP info:</p>
<p>sudo mkdir -p /var/www/servidor2.centro.intranet<br>
echo "&lt;?php phpinfo(); ?&gt;" | sudo tee /var/www/servidor2.centro.intranet/info.php<br>
sudo chown -R www-data:www-data /var/www/servidor2.centro.intranet</p>
<br>
<p>Configurar Nginx en puerto 8080 (sites-available/servidor2.centro.intranet)</p>
<br>
<p>sudo ln -s /etc/nginx/sites-available/servidor2.centro.intranet /etc/nginx/sites-enabled/<br>
sudo nginx -t<br>
sudo systemctl reload nginx</p>
<br>
<p>Acceso a PHP info:<br>
http://servidor2.centro.intranet:8080/info.php</p>
<br>
<p><img src="images/37-phpinfo.png"></p>
<br>

<p>Instalar phpMyAdmin:</p>
<p>sudo apt -y install phpmyadmin<br>
sudo ln -s /usr/share/phpmyadmin /var/www/servidor2.centro.intranet/phpmyadmin<br>
sudo systemctl reload nginx</p>
<br>
<p>Acceder:<br>
http://servidor2.centro.intranet:8080/phpmyadmin</p>
<br>
<p><img src="images/38-phpmyadmin.png"></p>
<br>

<p>9) Comprobaciones y validaciones</p>
<p>Apache sirve WordPress en http://centro.intranet<br>
Apache sirve app Python en http://departamentos.centro.intranet con autenticación básica<br>
AWStats accesible en http://centro.intranet/awstats/awstats.pl?config=centro.intranet<br>
Nginx en http://servidor2.centro.intranet:8080 con PHP y phpMyAdmin</p>
<br>
<p>Comandos útiles:<br>
sudo systemctl status apache2 nginx mysql<br>
sudo tail -f /var/log/apache2/*.log<br>
sudo tail -f /var/log/nginx/access.log /var/log/nginx/error.log<br>
curl -I http://centro.intranet<br>
curl -I http://departamentos.centro.intranet<br>
curl -I http://servidor2.centro.intranet:8080</p>
<br>
<p><img src="images/39-validacion.png"><br>
<img src="images/40-logs.png"></p>
<br>

<p>10) Entrega en GitHub</p>
<p>Estructura:<br>
README.md<br>
images/<br>
docs/ (opcional)</p>
<br>
<p>Fragmentos de configuración a incluir:<br>
/etc/hosts<br>
/etc/apache2/sites-available/centro.intranet.conf<br>
/etc/apache2/sites-available/departamentos.centro.intranet.conf<br>
/etc/awstats/awstats.centro.intranet.conf<br>
/etc/nginx/sites-available/servidor2.centro.intranet<br>
wsgi.py, app.py, wp-config.php (sin credenciales)</p>
<br>
<p>Sugerencia de mensajes de commit:<br>
docs: añade guía paso a paso y estructura de capturas<br>
feat(apache): vhosts para centro y departamentos<br>
feat(wordpress): instalación y configuración inicial<br>
feat(python): app wsgi mínima y autenticación básica<br>
feat(awstats): configuración sitio centro.intranet<br>
feat(nginx): server 8080 con php y phpmyadmin</p>
<br>

<p>Anexos</p>
<p>Rutas clave:<br>
/etc/hosts — resolución de nombres internos<br>
/var/www/centro.intranet — WordPress<br>
/var/www/departamentos.centro.intranet — App Python + WSGI<br>
/etc/apache2/sites-available/*.conf — VirtualHosts Apache<br>
/var/log/apache2/*.log — Logs Apache<br>
/etc/awstats/awstats.centro.intranet.conf — AWStats<br>
/etc/nginx/sites-available/servidor2.centro.intranet — Server block Nginx<br>
/var/www/servidor2.centro.intranet — Sitio Nginx con PHP</p>
<br>
<p>Problemas frecuentes:<br>
“No carga WordPress”: revisa permisos y wp-config.php<br>
“Python da 500”: revisa wsgi.py, permisos y error.log<br>
“AWStats vacío”: verifica LogFile y ejecuta -update<br>
“Nginx no ejecuta PHP”: confirma socket de php-fpm correcto<br>
“Visualizar las versiones de PHP”</p>
