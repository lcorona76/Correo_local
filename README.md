<h1>MANUAL DE INSTALACIÓN Y CONFIGURACIÓN: LABORATORIO DE CORREO LOCAL</h1>
<p><strong>Versión:</strong> 1.0<br>
<strong>Uso:</strong> Laboratorio de Ciberseguridad, Phishing y Pruebas de Red</p>

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

<h2>Complementos esenciales para un servidor completo. </h2>
<p>Postfix por sí solo solo envía/recibe; para leer los correos desde un cliente o webmail, necesitarás:</p> 
<ul>
  <li><b>Dovecot:</b> Funciona como el agente de entrega (MDA) que permite acceder a los buzones mediante protocolos IMAP o POP3.</li>
  <li><b>Webmail:</b> Herramientas como Roundcube permiten visualizar los correos desde un navegador.</li>
  <li><b>Mailx:</b> Una utilidad de línea de comandos muy útil para realizar pruebas rápidas de envío: echo "Contenido" | mail -s "Asunto" usuario@localhost. </li>
</ul>

<h2>Puertos comunes</h2>

<p>Si configuras clientes externos para conectar con tu Postfix local, ten en cuenta los puertos estándar:</p>
<ul>
  <li>SMTP: Puerto 25 (sin cifrado) o 587 (con STARTTLS).</li>
  <li>SMTPS: Puerto 465 (cifrado SSL/TLS).</li>
</ul>

<hr>

<h2>5. CONFIGURACIÓN DE USUARIOS</h2>

<p>Para crear un entorno de pruebas con múltiples usuarios en un servidor Postfix local, el enfoque cambia de "solo enviar" a "gestionar buzones". Aquí tienes los pasos clave para que tus usuarios puedan enviarse correos entre sí:</p>

<h2>1. Crear los usuarios en el sistema</h2>

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

<h3>Prueba de envío</h3>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>echo "Hola usuario 2" | mail -s "Prueba Local" user2@localhost</code></pre>
</div>

<hr>

<h2>6. CONFIGURACIÓN DE WEBMAIL (Roundcube)</h2>

<h3>Instalar dependencias</h3>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo apt install apache2 mariadb-server php php-mysql libapache2-mod-php \
php-xml php-mbstring php-intl php-zip php-curl php-gd php-imagick -y</code></pre>
</div>

<h3>Crear base de datos</h3>

<div class="cmd-box">
  <button onclick="copyCmd(this)">Copiar</button>
  <pre><code>sudo mysql -u root
CREATE DATABASE roundcubemail;
CREATE USER 'roundcubeuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON roundcubemail.* TO 'roundcubeuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;</code></pre>
</div>

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
