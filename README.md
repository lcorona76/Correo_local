<h1>MANUAL DE INSTALACIÓN Y CONFIGURACIÓN: LABORATORIO DE CORREO LOCAL</h1>
<p><strong>Versión:</strong> 1.0<br>
<strong>Uso:</strong> Laboratorio de Ciberseguridad, Phishing y Pruebas de Red
<strong>Sistema Operativo: Ubuntu</strong> 24.04<br></p>

<hr>

<h2>1. INTRODUCCIÓN</h2>
<p>
Este manual describe el procedimiento técnico para el despliegue de un sistema de mensajería unificado en entorno local,
diseñado para laboratorios de ciberseguridad, pruebas de penetración y simulaciones de ingeniería social.
</p>

<ul>
  <li><strong>Simulación de Vectores de Ataque:</strong> Envío de phishing controlado</li>
  <li><strong>Validación de Protocolos:</strong> Análisis de SMTP y SASL</li>
  <li><strong>Gestión de Usuarios:</strong> Múltiples buzones locales</li>
</ul>

<hr>

<h2>2. ARQUITECTURA Y FLUJO DE DATOS</h2>
<ol>
  <li><b>Emisión:</b> Gophish/Swaks → Postfix (SMTP 25)</li>
  <li><b>Recepción:</b> Postfix valida IP → Dovecot</li>
  <li><b>Almacenamiento:</b> Formato Maildir</li>
  <li><b>Visualización:</b> Roundcube vía IMAP (143)</li>
</ol>

<hr>

<h2>4. INSTALACIÓN DEL SERVIDOR (Postfix)</h2>

<h3>Instalación básica</h3>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo apt update && sudo apt install postfix mailutils</code></pre>
</div>

<p>Durante la instalación, selecciona <em>"Local only"</em> (Solo local) si solo quieres que el correo circule dentro de la misma máquina, o "Internet Site" si planeas enviar correos a dominios externos en el futuro.</p>

<h3>Archivo principal</h3>
<p>El archivo de configuración principal se encuentra en /etc/postfix/main.cf. Para un entorno local lab.local es mi dominio de pruebas, asegúrate de ajustar estos parámetros: </p>

<p>El archivo de configuración principal se encuentra en /etc/postfix/main.cf. Para un entorno local, asegúrate de ajustar estos parámetros:<p/>
<ol>  
  <li><b>myhostname:</b> El nombre de tu máquina (ej. servidor.local).</li>
  <li><b>mydestination:</b> Lista de dominios que Postfix aceptará como locales (ej. $myhostname, localhost.localdomain, localhost).</li>
  <li><b>inet_interfaces:</b> Define en qué interfaces escucha el servidor. Para uso estrictamente local, usa loopback-only.</li>
</ol>

<p><code>sudo nano /etc/postfix/main.cf</code></p>

<ul>
  <li><code>myhostname = servidor.lab.local</code></li>
  <li><code>mydestination = $myhostname, localhost, lab.local</code></li>
  <li><code>mydomain = lab.local</code></li>
  <li><code>inet_interfaces = all</code></li>
</ul>

<h3>Complementos esenciales para un servidor completo. </h3>
<p>Postfix por sí solo solo envía/recibe; para leer los correos desde un cliente o webmail, necesitarás:</p> 
<ul>
  <li><b>Dovecot:</b> Funciona como el agente de entrega (MDA) que permite acceder a los buzones mediante protocolos IMAP o POP3.</li>
  <li><b>Webmail:</b> Herramientas como Roundcube permiten visualizar los correos desde un navegador.</li>
  <li><b>Mailx:</b> Una utilidad de línea de comandos muy útil para realizar pruebas rápidas de envío: echo "Contenido" | mail -s "Asunto" usuario@localhost. </li>
</ul>

<h3>Puertos comunes</h3>

<p>Si configuras clientes externos para conectar con tu Postfix local, ten en cuenta los puertos estándar:</p>
<ul>
  <li>SMTP: Puerto 25 (sin cifrado) o 587 (con STARTTLS).</li>
  <li>SMTPS: Puerto 465 (cifrado SSL/TLS).</li>
</ul>

<hr>

<h2>5. CONFIGURACIÓN DE USUARIOS</h2>

<p>Para crear un entorno de pruebas con múltiples usuarios en un servidor Postfix local, el enfoque cambia de "solo enviar" a "gestionar buzones". Aquí tienes los pasos clave para que tus usuarios puedan enviarse correos entre sí:</p>

<h3>Crear los usuarios en el sistema</h3>

<p>Postfix, por defecto, utiliza los usuarios reales de Linux. Si quieres probar con "user1" y "user2", debes crearlos en tu terminal:</p>

<h3>Crear usuarios en Linux</h3>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo adduser user1
sudo adduser user2</code></pre>
</div>

<p>Es recomendable usar el formato Maildir (un archivo por mensaje) en lugar de mbox (un solo archivo gigante), ya que es más moderno y compatible con lectores de correo:</p>

<p>Edita el archivo: sudo nano /etc/postfix/main.cf</p>

<h3>Habilitar Maildir</h3>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo nano /etc/postfix/main.cf
home_mailbox = Maildir/
sudo systemctl restart postfix</code></pre>
</div>

<h3>Instalar un servidor IMAP (Dovecot)</h3>

<p>Postfix entrega el correo, pero para que tus usuarios "vean" su bandeja de entrada desde un cliente (como Thunderbird o Outlook), necesitas Dovecot:</p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo apt install dovecot-imapd</code></pre>
</div>
    
<p>Configuración básica: En /etc/dovecot/conf.d/10-mail.conf, asegúrate de que coincida con Postfix:</p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>mail_location = maildir:~/Maildir
Reinicia: sudo systemctl restart dovecot</code></pre>
</div>

<h3>Prueba de envío</h3>
<p>Puedes usar la herramienta mail para verificar que la comunicación interna funciona:
# Logueado como user1, envía a user2</p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>echo "Hola usuario 2" | mail -s "Prueba Local" user2@localhost</code></pre>
</div>

<h3>Configuración del Cliente de Correo</h3>
<p>Para conectar un cliente de escritorio a tu entorno de pruebas:</p>
<ul>
  <li><b>Servidor SMTP/IMAP:</b> localhost o la IP local de tu servidor.</li>
  <li><b>Usuario/Password:</b> Los mismos que creaste anteriormente.</li>
  <li><b>Seguridad:</b> Selecciona "Ninguna" o "STARTTLS" (si configuraste certificados) y permite contraseñas normales/planas para pruebas locales.</li>
</ul>

<hr>

<h2>6. CONFIGURACIÓN DE WEBMAIL (Roundcube)</h2>

<p>Para un entorno de pruebas local, Roundcube es la opción más sólida y profesional. Es ligero, tiene una interfaz moderna y funciona perfectamente sobre Apache o Nginx.</p>
<p>Aquí tienes los pasos para configurarlo en un servidor basado en Ubuntu/Debian:</p>

<h3>Instalar dependencias</h3>

<p>Roundcube necesita un servidor web, PHP y una base de datos (MariaDB o MySQL). Instálalos con:</p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo apt install apache2 mariadb-server php php-mysql libapache2-mod-php \
php-xml php-mbstring php-intl php-zip php-curl php-gd php-imagick -y</code></pre>
</div>

<h3>Crear base de datos</h3>
<p>Entra a MariaDB y crea el espacio para los datos de la interfaz:</p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo mysql -u root
CREATE DATABASE roundcubemail;
CREATE USER 'roundcubeuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON roundcubemail.* TO 'roundcubeuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;</code></pre>
</div>

<h3>Descargar e instalar Roundcube</h3>

<p>
Lo ideal es descargar la versión estable directamente desde el sitio oficial de Roundcube. <em>https://github.com/roundcube/roundcubemail/releases</em>
</p>

<ol>
  <li>Ve a la carpeta web:</li>
</ol>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>cd /var/www/html</code></pre>
</div>

<ol start="2">
  <li>Descarga el paquete (verifica la última versión en su web oficial):</li>
</ol>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo wget https://github.com/roundcube/roundcubemail/releases/download/1.6.15/roundcubemail-1.6.15-complete.tar.gz</code></pre>
</div>

<ol start="3">
  <li>Descomprime el archivo descargado:</li>
</ol>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo tar -xvzf roundcubemail-*.tar.gz</code></pre>
</div>

<ol start="4">
  <li>Cambia el nombre de la carpeta a uno más simple:</li>
</ol>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo mv roundcubemail-1.6.6 roundcube</code></pre>
</div>

<ol start="5">
  <li>Asigna permisos al servidor web:</li>
</ol>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo chown -R www-data:www-data /var/www/html/roundcube</code></pre>
</div>

<hr>

<h3>4. Configuración vía Navegador</h3>

<p>
Accede desde tu red a la siguiente URL:
</p>

<p>
<strong>http://tu_ip_o_localhost/roundcube/installer</strong>
</p>

<p>
Sigue estos pasos dentro del asistente de instalación:
</p>

<ul>
  <li><strong>Database Setup:</strong> Ingresa los datos creados previamente
    (<code>roundcubemail</code>, <code>roundcubeuser</code>, <code>tu_contraseña_segura</code>).</li>
  <li><strong>IMAP Settings:</strong> Host <code>localhost</code> y puerto <code>143</code> (Dovecot debe estar activo).</li>
  <li><strong>SMTP Settings:</strong> Host <code>localhost</code> y puerto <code>25</code>.</li>
  <li><strong>Create Config:</strong> Al finalizar, genera el archivo de configuración.</li>
</ul>

<h3>5. Toque Final: Importar la base de datos</h3>

<p>
Para que Roundcube funcione correctamente, es necesario crear sus tablas internas.
Ejecuta el siguiente comando (ajusta la ruta si es necesario):
</p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo mysql -u roundcubeuser -p roundcubemail < /var/www/html/roundcube/SQL/mysql.initial.sql</code></pre>
</div>

<p>
Los errores más comunes durante el instalador de Roundcube suelen dividirse en tres categorías:
</p>

<ul>
  <li>Extensiones de PHP faltantes</li>
  <li>Permisos incorrectos en carpetas</li>
  <li>Errores en la configuración de la base de datos</li>
</ul>

<h3>Errores más comunes</h3>

<p>
A continuación se describen los errores más frecuentes durante la instalación y configuración de Roundcube,
así como sus soluciones recomendadas.
</p>

<hr>

<h4>1. Extensiones de PHP <span style="color:red;">NOT OK</span></h4>

<p>
Si el instalador marca en rojo varias librerías, significa que faltan extensiones de PHP o no están habilitadas.
Las más críticas suelen ser <code>intl</code>, <code>mbstring</code> y el conector de base de datos.
</p>

<p><strong>Instalar extensiones faltantes:</strong></p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo apt install php-intl php-mbstring php-gd php-xml php-mysql php-zip -y</code></pre>
</div>

<p><strong>Habilitar extensiones ya instaladas:</strong></p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo phpenmod intl mbstring pdo_mysql
sudo systemctl restart apache2</code></pre>
</div>


<hr>

<h4>2. Permisos en <code>/temp</code> y <code>/logs</code> (<span style="color:red;">NOT WRITABLE</span>)</h4>

<p>
Roundcube necesita escribir archivos temporales y logs. Si aparecen como <em>Not Writable</em>,
Apache no tiene permisos suficientes.
</p>

<p><strong>Aplica los siguientes comandos dentro del directorio de Roundcube:</strong></p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>cd /var/www/html/roundcube
sudo chown -R www-data:www-data temp/ logs/
sudo chmod -R 775 temp/ logs/</code></pre>
</div>

<hr>

<h4>3. Error de base de datos (<span style="color:red;">DSN: NOT OK</span>)</h4>

<p>
Este error indica que los datos configurados en el instalador no coinciden con los existentes en MariaDB.
Roundcube no logra autenticarse contra la base de datos.
</p>

<h5>3.1 Asegurar los datos en MariaDB</h5>

<p>
Accede a MariaDB y recrea el usuario para evitar conflictos
(<strong>recuerda la contraseña que establezcas</strong>):
</p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo mysql -u root
DROP USER IF EXISTS 'roundcube'@'localhost';
CREATE USER 'roundcube'@'localhost' IDENTIFIED BY 'tu_password_aqui';
GRANT ALL PRIVILEGES ON roundcubemail.* TO 'roundcube'@'localhost';
FLUSH PRIVILEGES;
EXIT;</code></pre>
</div>

<h5>3.2 Actualizar el instalador de Roundcube</h5>

<ul>
  <li><strong>Database name:</strong> roundcubemail</li>
  <li><strong>Database user:</strong> roundcube</li>
  <li><strong>Database password:</strong> tu_password_aqui</li>
</ul>

<h5>3.3 Paso final (Importante)</h5>

<p>
Si después de pulsar <em>Update Config</em> el error continúa, edita manualmente el archivo:
</p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo nano /var/www/html/roundcube/config/config.inc.php</code></pre>
</div>

<p>
Verifica que la línea sea exactamente así:
</p>

<pre><code>$config['db_dsnw'] = 'mysql://roundcube:tu_password_aqui@localhost/roundcubemail';</code></pre>

<p>
Guarda con <strong>Ctrl+O</strong>, Enter y sal con <strong>Ctrl+X</strong>.
</p>

<hr>

<h4>4. Error de Zona Horaria (<code>date.timezone</code>)</h4>

<ol>
  <li>Edita el archivo PHP:
    <code>sudo nano /etc/php/8.x/apache2/php.ini</code>
  </li>
  <li>Busca <code>date.timezone</code> y define tu zona, por ejemplo:
    <code>date.timezone = America/Mexico_City</code>
  </li>
  <li>Reinicia Apache:
    <code>sudo systemctl restart apache2</code>
  </li>
</ol>

<hr>

<h4>5. Error al enviar correo SMTP</h4>

<p>
Este error ocurre cuando Postfix y Dovecot no están correctamente integrados.
Roundcube intenta autenticarse, pero Postfix no puede validar al usuario.
</p>

<hr>

<h4>6. Error: no se puede enviar correo SMTP (Access denied)</h4>

<h5>6.1 Configurar Postfix para usar Dovecot (SASL)</h5>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo nano /etc/postfix/main.cf

# Habilitar autenticación SASL vía Dovecot
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination</code></pre>
</div>

<h5>6.2 Configurar Dovecot</h5>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo nano /etc/dovecot/conf.d/10-master.conf

service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}</code></pre>
</div>

<h5>6.3 Permitir autenticación en texto plano (solo laboratorio)</h5>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo nano /etc/dovecot/conf.d/10-auth.conf
disable_plaintext_auth = no</code></pre>
</div>

<h5>6.4 Reiniciar servicios</h5>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo systemctl restart postfix
sudo systemctl restart dovecot</code></pre>
</div>

<h5>6.5 Verificar configuración en Roundcube</h5>

<ul>
  <li><strong>SMTP Server:</strong> localhost o 127.0.0.1</li>
  <li><strong>SMTP Port:</strong> 25 o 587</li>
  <li><strong>SMTP Auth:</strong> "Check if authentication is required"</li>
</ul>

<p>
Si el error persiste, revisa el log en tiempo real:
</p>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo tail -f /var/log/mail.log</code></pre>
</div>

<hr>

<h3>7. Acceso Web y Redirección Automática</h3>

<p>
Para que al acceder a <strong>http://localhost/</strong> se cargue directamente la interfaz de correo web (Roundcube),
realiza los siguientes pasos:
</p>

<ol>
  <li><strong>Crear el archivo de redirección:</strong></li>
</ol>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo nano /var/www/html/index.php</code></pre>
</div>

<ol start="2">
  <li><strong>Pega el siguiente código PHP:</strong></li>
</ol>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>&lt;?php
header("Location: /roundcube/");
exit;
?&gt;</code></pre>
</div>

<ol start="3">
  <li><strong>Elimina el archivo HTML original para evitar conflictos:</strong></li>
</ol>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo rm /var/www/html/index.html</code></pre>
</div>

<p>
A partir de este momento, cualquier acceso a <code>http://localhost/</code> redirigirá automáticamente
a la interfaz web de Roundcube.
</p>

<hr>

<h2>8. PRUEBAS Y TROUBLESHOOTING</h2>

<h3>Prueba SMTP con Swaks</h3>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>swaks --to user2@localhost \
--from "Soporte &lt;soporte@localhost&gt;" \
--header "Subject: Prueba" \
--body "Hola" --server IP_SERVIDOR</code></pre>
</div>

<h3>Ver logs en tiempo real</h3>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo tail -f /var/log/mail.log</code></pre>
</div>
