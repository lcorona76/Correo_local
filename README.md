<div style="font-family: 'Segoe UI', Arial, sans-serif; max-width: 900px; margin: 20px auto; color: #333; line-height: 1.6; border: 1px solid #ddd; padding: 40px; border-radius: 10px; background-color: #ffffff; box-shadow: 0 4px 15px rgba(0,0,0,0.1);">

    <!-- CABECERA -->
    <h1 style="text-align: center; color: #2c3e50; border-bottom: 4px solid #3498db; padding-bottom: 15px; margin-bottom: 30px;">Manual de Instalación: Servidor de Correo Local</h1>
    
    <!-- INTRODUCCIÓN TÉCNICA -->
    <div style="background-color: #f8f9fa; border: 1px solid #e9ecef; padding: 25px; border-radius: 8px; margin-bottom: 35px;">
        <h2 style="color: #2c3e50; margin-top: 0; font-size: 1.4em;">1. Introducción Técnica</h2>
        <p>Este manual describe el despliegue de un <strong>entorno de mensajería unificado</strong> para laboratorios de ciberseguridad. El objetivo es proporcionar un servidor interno aislado para capturar y validar correos electrónicos generados por herramientas como <strong>Gophish</strong> o <strong>Swaks</strong>.</p>
        <p>La arquitectura prioriza la interoperabilidad local y la visibilidad de logs, permitiendo monitorear los protocolos SMTP e IMAP en tiempo real sin necesidad de salida a internet.</p>
    </div>

    <!-- ARQUITECTURA Y DIAGRAMA -->
    <h2 style="color: #2980b9; border-left: 5px solid #2980b9; padding-left: 10px;">2. Arquitectura y Flujo de Datos</h2>
    <p>La infraestructura se divide en tres componentes críticos:</p>
    <ul style="padding-left: 20px; margin-bottom: 25px;">
        <li><strong>Postfix (MTA):</strong> Recibe el correo vía SMTP (Puerto 25).</li>
        <li><strong>Dovecot (MDA):</strong> Organiza y sirve los correos vía IMAP (Puerto 143).</li>
        <li><strong>Roundcube:</strong> Interfaz Web para el usuario final (Puerto 80).</li>
    </ul>

    <div style="margin: 30px 0; text-align: center;">
        <table style="width: 100%; border-collapse: separate; border-spacing: 10px; font-size: 0.85em;">
            <tr>
                <td style="background: #34495e; color: white; padding: 15px; border-radius: 10px; width: 30%;">
                    <strong>ORIGEN</strong><br>Gophish / Swaks<br><small>(Puerto 25 SMTP)</small>
                </td>
                <td style="vertical-align: middle; color: #3498db; font-weight: bold; font-size: 1.5em;">&rarr;</td>
                <td style="background: #3498db; color: white; padding: 15px; border-radius: 10px; width: 30%;">
                    <strong>PROCESO</strong><br>Postfix MTA<br><small>(Relay Local)</small>
                </td>
                <td style="vertical-align: middle; color: #27ae60; font-weight: bold; font-size: 1.5em;">&rarr;</td>
                <td style="background: #27ae60; color: white; padding: 15px; border-radius: 10px; width: 30%;">
                    <strong>DESTINO</strong><br>Dovecot / Maildir<br><small>(Buzón de Usuario)</small>
                </td>
            </tr>
        </table>
        <div style="margin-top: 15px; padding: 10px; border: 2px dashed #bdc3c7; display: inline-block; border-radius: 5px; color: #7f8c8d;">
            <strong>Acceso Web:</strong> Roundcube Webmail &larr; Apache HTTP
        </div>
    </div>

    <!-- INSTALACIÓN -->
    <h2 style="color: #2980b9; border-left: 5px solid #2980b9; padding-left: 10px; background: #f9f9f9;">3. Configuración del Servidor</h2>
    <p>Instalación del stack base y creación de usuarios de prueba:</p>
    <pre style="background: #2c3e50; color: #ecf0f1; padding: 15px; border-radius: 5px; overflow-x: auto; font-family: 'Consolas', monospace;">
sudo apt update && sudo apt install apache2 mariadb-server postfix dovecot-imapd php php-mysql -y
sudo adduser user1
sudo adduser user2</pre>

    <h2 style="color: #2980b9; border-left: 5px solid #2980b9; padding-left: 10px; background: #f9f9f9;">4. Configuración de Postfix & Dovecot</h2>
    <p>En <code>/etc/postfix/main.cf</code>, habilita la red y el socket de autenticación:</p>
    <pre style="background: #f4f4f4; border: 1px solid #ccc; padding: 15px; border-radius: 5px; font-family: monospace;">
inet_interfaces = all
mynetworks = 127.0.0.0/8, [::1]/128, [IP_RED_LOCAL]
home_mailbox = Maildir/
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes</pre>

    <p>En <code>/etc/dovecot/conf.d/10-auth.conf</code> y <code>10-master.conf</code>:</p>
    <pre style="background: #f4f4f4; border: 1px solid #ccc; padding: 15px; border-radius: 5px; font-family: monospace;">
disable_plaintext_auth = no
mail_location = maildir:~/Maildir

# Socket en 10-master.conf
unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
}</pre>

    <!-- WEBMAIL -->
    <h2 style="color: #2980b9; border-left: 5px solid #2980b9; padding-left: 10px; background: #f9f9f9;">5. Roundcube e Indexación</h2>
    <p>Permisos de carpeta y redirección automática en <code>/var/www/html/index.php</code>:</p>
    <pre style="background: #f4f4f4; border: 1px solid #ccc; padding: 15px; border-radius: 5px;">
sudo chown -R www-data:www-data /var/www/html/roundcube/
# Archivo index.php para redirección:
&lt;?php header("Location: /roundcube/"); exit; ?&gt;</pre>

    <!-- VERIFICACIÓN -->
    <h2 style="color: #27ae60; border-left: 5px solid #27ae60; padding-left: 10px; background: #f4faf6;">6. Verificación Final</h2>
    <table style="width: 100%; border-collapse: collapse; margin-top: 15px;">
        <tr style="background-color: #27ae60; color: white;">
            <th style="padding: 12px; border: 1px solid #ddd;">Servicio</th>
            <th style="padding: 12px; border: 1px solid #ddd;">Puerto</th>
            <th style="padding: 12px; border: 1px solid #ddd;">Comando</th>
        </tr>
        <tr>
            <td style="padding: 10px; border: 1px solid #ddd;"><strong>SMTP</strong> (Postfix)</td>
            <td style="padding: 10px; border: 1px solid #ddd; text-align: center;">25</td>
            <td style="padding: 10px; border: 1px solid #ddd;"><code>systemctl status postfix</code></td>
        </tr>
        <tr style="background-color: #f9f9f9;">
            <td style="padding: 10px; border: 1px solid #ddd;"><strong>IMAP</strong> (Dovecot)</td>
            <td style="padding: 10px; border: 1px solid #ddd; text-align: center;">143</td>
            <td style="padding: 10px; border: 1px solid #ddd;"><code>systemctl status dovecot</code></td>
        </tr>
        <tr>
            <td style="padding: 10px; border: 1px solid #ddd;"><strong>HTTP</strong> (Apache)</td>
            <td style="padding: 10px; border: 1px solid #ddd; text-align: center;">80</td>
            <td style="padding: 10px; border: 1px solid #ddd;"><code>systemctl status apache2</code></td>
        </tr>
    </table>

    <div style="margin-top: 30px; background: #fdf2f2; padding: 15px; border-radius: 5px; border: 1px solid #f5c6cb; color: #721c24;">
        <strong>Troubleshooting:</strong> Si no hay conexión, revisa los logs en tiempo real con: <br>
        <code>sudo tail -f /var/log/mail.log</code>
    </div>

</div>
