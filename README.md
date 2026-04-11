<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Manual Servidor de Correo Local</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; line-height: 1.6; color: #333; max-width: 900px; margin: 0 auto; padding: 20px; background-color: #f4f7f6; }
        h1 { color: #2c3e50; border-bottom: 2px solid #3498db; padding-bottom: 10px; }
        h2 { color: #2980b9; margin-top: 30px; border-left: 5px solid #3498db; padding-left: 10px; }
        h3 { color: #16a085; }
        code { background-color: #eee; padding: 2px 5px; border-radius: 4px; font-family: 'Consolas', monospace; color: #c7254e; }
        pre { background-color: #2d2d2d; color: #f8f8f2; padding: 15px; border-radius: 5px; overflow-x: auto; font-family: 'Consolas', monospace; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .note { background-color: #e7f3fe; border-left: 6px solid #2196F3; padding: 10px; margin: 20px 0; }
        .warning { background-color: #fff3cd; border-left: 6px solid #ffc107; padding: 10px; margin: 20px 0; }
        table { width: 100%; border-collapse: collapse; margin: 20px 0; }
        table, th, td { border: 1px solid #ddd; padding: 12px; text-align: left; }
        th { background-color: #3498db; color: white; }
    </style>
</head>
<body>

    <h1>Manual de Instalación: Servidor de Correo Local</h1>
    <p><strong>Versión:</strong> 1.0 | <strong>Entorno:</strong> Laboratorio de Pruebas (Postfix + Dovecot + Roundcube)</p>

    <div class="note">
        <strong>Introducción:</strong> Este entorno está diseñado para auditorías de ciberseguridad, permitiendo integrar herramientas como Gophish y Swaks en un flujo de correo controlado y local.
    </div>

    <h2>1. Requisitos del Sistema (LAMP)</h2>
    <p>Instalación de Apache, MariaDB y módulos de PHP necesarios:</p>
    <pre>sudo apt update
sudo apt install apache2 mariadb-server php php-mysql libapache2-mod-php php-xml php-mbstring php-intl php-zip php-gd -y</pre>

    <h2>2. Configuración de Postfix (MTA)</h2>
    <p>Editar el archivo <code>/etc/postfix/main.cf</code> para permitir acceso desde la red local:</p>
    <pre>inet_interfaces = all
mynetworks = 127.0.0.0/8, [::1]/128, [IP_GOPHISH]
home_mailbox = Maildir/</pre>
    <p>Añadir al final la integración con Dovecot:</p>
    <pre>smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination</pre>

    <h2>3. Configuración de Dovecot (MDA/IMAP)</h2>
    <p>Configurar el acceso a buzones y autenticación en <code>/etc/dovecot/conf.d/</code>:</p>
    <ul>
        <li><strong>10-mail.conf:</strong> <code>mail_location = maildir:~/Maildir</code></li>
        <li><strong>10-auth.conf:</strong> <code>disable_plaintext_auth = no</code></li>
        <li><strong>10-master.conf:</strong> Habilitar el socket para Postfix.</li>
    </ul>
    <pre>service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}</pre>

    <h2>4. Webmail Roundcube</h2>
    <p>Crear la base de datos y asignar permisos de escritura a las carpetas <code>temp/</code> y <code>logs/</code>:</p>
    <pre>sudo chown -R www-data:www-data /var/www/html/roundcube/
sudo chmod -R 775 /var/www/html/roundcube/temp/ /var/www/html/roundcube/logs/</pre>

    <h2>5. Redirección Automática</h2>
    <p>Para acceder directamente vía <code>http://localhost/</code>, crear <code>/var/www/html/index.php</code>:</p>
    <pre>&lt;?php header("Location: /roundcube/"); exit; ?&gt;</pre>

    <h2>6. Troubleshooting (Solución de Problemas)</h2>
    <table>
        <tr>
            <th>Error</th>
            <th>Solución</th>
        </tr>
        <tr>
            <td>Relay Access Denied</td>
            <td>Añadir la IP de la máquina atacante a <code>mynetworks</code> en Postfix.</td>
        </tr>
        <tr>
            <td>SMTP Auth Failed</td>
            <td>Verificar que el socket de Dovecot esté corriendo y tenga permisos.</td>
        </tr>
        <tr>
            <td>Database Error 1045</td>
            <td>Resetear permisos del usuario 'roundcube' en MariaDB.</td>
        </tr>
    </table>

    <div class="warning">
        <strong>Seguridad:</strong> Al finalizar la configuración, recuerda borrar la carpeta <code>/installer</code> de Roundcube.
    </div>

</body>
</html>

