<section>
    <h1>MANUAL DE INSTALACIÓN Y CONFIGURACIÓN: LABORATORIO DE CORREO LOCAL</h1>
    <p><strong>Versión:</strong> 1.0<br>
    <strong>Uso:</strong> Laboratorio de Ciberseguridad, Phishing y Pruebas de Red.</p>
    <hr>

    <h2>1. INTRODUCCIÓN</h2>
    <p>El presente manual describe el despliegue de un sistema de mensajería unificado en entorno local, diseñado para laboratorios de auditoría. La arquitectura utiliza <strong>Postfix (MTA)</strong> para la transferencia de mensajes, <strong>Dovecot (MDA)</strong> para la gestión de buzones e IMAP, y <strong>Roundcube</strong> como interfaz web de usuario. El objetivo es permitir simulaciones controladas con herramientas como Gophish o Swaks sin salida a Internet.</p>

    <hr>

    <h2>2. ARQUITECTURA Y FLUJO DE DATOS</h2>
    <p>El correo sigue el siguiente trayecto técnico:</p>
    <ol>
        <li><strong>Emisión:</strong> Gophish/Swaks conecta vía SMTP (Puerto 25) con Postfix.</li>
        <li><strong>Recepción:</strong> Postfix valida la IP de origen y entrega el mensaje a Dovecot.</li>
        <li><strong>Almacenamiento:</strong> El mensaje se guarda en formato Maildir en la carpeta del usuario.</li>
        <li><strong>Visualización:</strong> El usuario accede vía HTTP (Apache) a Roundcube, que lee el correo mediante IMAP (Puerto 143).</li>
    </ol>

    <hr>

    <h2>3. INSTALACIÓN DEL SERVIDOR (LAMP STACK)</h2>
    <p>Primero, instalamos las dependencias de servidor web, base de datos y PHP:</p>
    <pre><code>sudo apt update
sudo apt install apache2 mariadb-server php php-mysql libapache2-mod-php php-xml php-mbstring php-intl php-zip php-gd -y</code></pre>

    <hr>

    <h2>4. CONFIGURACIÓN DE USUARIOS Y CORREO</h2>
    <h3>4.1 Creación de cuentas locales</h3>
    <pre><code>sudo adduser user1
sudo adduser user2</code></pre>

    <h3>4.2 Configuración de Postfix</h3>
    <p>Edita <code>/etc/postfix/main.cf</code>:</p>
    <ul>
        <li><code>inet_interfaces = all</code></li>
        <li><code>mynetworks = 127.0.0.0/8, [::1]/128, IP_DE_GOPHISH</code></li>
        <li><code>home_mailbox = Maildir/</code></li>
    </ul>
    <p>Añade al final para habilitar la comunicación con Dovecot:</p>
    <pre><code>smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination</code></pre>
    <p>Reiniciar: <code>sudo systemctl restart postfix</code></p>

    <h3>4.3 Configuración de Dovecot</h3>
    <ol>
        <li>Archivo <code>/etc/dovecot/conf.d/10-auth.conf</code>: <code>disable_plaintext_auth = no</code></li>
        <li>Archivo <code>/etc/dovecot/conf.d/10-mail.conf</code>: <code>mail_location = maildir:~/Maildir</code></li>
        <li>Archivo <code>/etc/dovecot/conf.d/10-master.conf</code>:</li>
    </ol>
    <pre><code>service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}</code></pre>
    <p>Reiniciar: <code>sudo systemctl restart dovecot</code></p>

    <hr>

    <h2>5. INSTALACIÓN DE ROUNDCUBE (WEBMAIL)</h2>
    <p><strong>1. Base de Datos:</strong></p>
    <pre><code>CREATE DATABASE roundcubemail;
CREATE USER 'roundcube'@'localhost' IDENTIFIED BY 'tu_password';
GRANT ALL PRIVILEGES ON roundcubemail.* TO 'roundcube'@'localhost';
FLUSH PRIVILEGES;</code></pre>

    <p><strong>2. Despliegue de Archivos:</strong></p>
    <p>Descarga Roundcube en <code>/var/www/html/roundcube</code> y asigna permisos:</p>
    <pre><code>sudo chown -R www-data:www-data /var/www/html/roundcube/
sudo chmod -R 775 /var/www/html/roundcube/temp/ /var/www/html/roundcube/logs/</code></pre>

    <p><strong>3. Configuración Web:</strong> Accede a <code>http://localhost/roundcube/installer</code>. Usa los datos de la DB creados arriba. Al finalizar, borra la carpeta installer.</p>

    <hr>

    <h2>6. ACCESO WEB Y REDIRECCIÓN AUTOMÁTICA</h2>
    <p>Para que al entrar a <code>http://localhost/</code> cargue directamente el correo:</p>
    <ol>
        <li>Crea el archivo de redirección: <code>sudo nano /var/www/html/index.php</code></li>
        <li>Pega este código:
            <pre><code>&lt;?php header("Location: /roundcube/"); exit; ?&gt;</code></pre>
        </li>
        <li>Elimina el archivo original: <code>sudo rm /var/www/html/index.html</code></li>
    </ol>

    <hr>

    <h2>7. PRUEBAS Y TROUBLESHOOTING</h2>
    <p>Prueba con Swaks (Simulación de Phishing):</p>
    <pre><code>swaks --to user2@localhost --from "Soporte &lt;soporte@localhost&gt;" --header "Subject: Prueba" --body "Hola" --server IP_SERVIDOR</code></pre>
    
    <p><strong>Solución de Errores Comunes:</strong></p>
    <ul>
        <li><strong>Error SMTP Auth:</strong> Verifica que la IP de la máquina que envía esté en <code>mynetworks</code> (Postfix).</li>
        <li><strong>Error Permisos Roundcube:</strong> Asegúrate de que <code>temp/</code> y <code>logs/</code> sean propiedad de <code>www-data</code>.</li>
        <li><strong>Log de errores en tiempo real:</strong> <code>sudo tail -f /var/log/mail.log</code></li>
    </ul>

    <hr>

    <h2>DIAGRAMA DE FLUJO Y ARQUITECTURA</h2>
    <p>El siguiente esquema describe el trayecto de un correo electrónico desde su generación en la herramienta de auditoría hasta su visualización por el usuario final.</p>
    
    <div style="background: #f4f4f4; padding: 15px; border-left: 5px solid #ccc;">
        <p><strong>Descripción del Proceso:</strong></p>
        <ol>
            <li><strong>Originación (SMTP):</strong> El emisor (Gophish o Swaks) se conecta al servidor Postfix a través del puerto 25. Al estar en la misma red y autorizada la IP, el servidor acepta el mensaje sin requerir autenticación externa.</li>
            <li><strong>Procesamiento y Filtro:</strong> Postfix analiza el destinatario. Al detectar que el dominio es local (ej. localhost o servidor.local), lo transfiere al agente de entrega.</li>
            <li><strong>Almacenamiento (Maildir):</strong> El correo se guarda físicamente en el directorio del usuario dentro del sistema Linux bajo la estructura de carpetas Maildir.</li>
            <li><strong>Acceso Final (IMAP/HTTP):</strong> El usuario accede a la URL de Roundcube. PHP se comunica con Dovecot vía IMAP (puerto 143) para "leer" los archivos en el servidor y mostrarlos gráficamente en el navegador.</li>
        </ol>
    </div>
</section>
