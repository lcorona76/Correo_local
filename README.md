MANUAL DE INSTALACIÓN Y CONFIGURACIÓN: LABORATORIO DE CORREO LOCAL
Versión: 1.0
Uso: Laboratorio de Ciberseguridad, Phishing y Pruebas de Red.
________________________________________
1. INTRODUCCIÓN
El presente manual describe el despliegue de un sistema de mensajería unificado en entorno local, diseñado para laboratorios de auditoría. La arquitectura utiliza Postfix (MTA) para la transferencia de mensajes, Dovecot (MDA) para la gestión de buzones e IMAP, y Roundcube como interfaz web de usuario. El objetivo es permitir simulaciones controladas con herramientas como Gophish o Swaks sin salida a Internet.
________________________________________
2. ARQUITECTURA Y FLUJO DE DATOS
El correo sigue el siguiente trayecto técnico:
1.	Emisión: Gophish/Swaks conecta vía SMTP (Puerto 25) con Postfix.
2.	Recepción: Postfix valida la IP de origen y entrega el mensaje a Dovecot.
3.	Almacenamiento: El mensaje se guarda en formato Maildir en la carpeta del usuario.
4.	Visualización: El usuario accede vía HTTP (Apache) a Roundcube, que lee el correo mediante IMAP (Puerto 143).
________________________________________
3. INSTALACIÓN DEL SERVIDOR (LAMP STACK)
Primero, instalamos las dependencias de servidor web, base de datos y PHP:
bash
sudo apt update
sudo apt install apache2 mariadb-server php php-mysql libapache2-mod-php php-xml php-mbstring php-intl php-zip php-gd -y
Usa el código con precaución.
________________________________________
4. CONFIGURACIÓN DE USUARIOS Y CORREO
4.1 Creación de cuentas locales
bash
sudo adduser user1
sudo adduser user2
Usa el código con precaución.
4.2 Configuración de Postfix
Edita /etc/postfix/main.cf:
•	inet_interfaces = all
•	mynetworks = 127.0.0.0/8, [::1]/128, IP_DE_GOPHISH
•	home_mailbox = Maildir/
Añade al final para habilitar la comunicación con Dovecot:
conf
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination
Usa el código con precaución.
Reiniciar: sudo systemctl restart postfix
4.3 Configuración de Dovecot
1.	Archivo /etc/dovecot/conf.d/10-auth.conf: disable_plaintext_auth = no
2.	Archivo /etc/dovecot/conf.d/10-mail.conf: mail_location = maildir:~/Maildir
3.	Archivo /etc/dovecot/conf.d/10-master.conf:
conf
service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}
Usa el código con precaución.
Reiniciar: sudo systemctl restart dovecot
________________________________________
5. INSTALACIÓN DE ROUNDCUBE (WEBMAIL)
1.	Base de Datos:
sql
CREATE DATABASE roundcubemail;
CREATE USER 'roundcube'@'localhost' IDENTIFIED BY 'tu_password';
GRANT ALL PRIVILEGES ON roundcubemail.* TO 'roundcube'@'localhost';
FLUSH PRIVILEGES;
Usa el código con precaución.
2.	Despliegue de Archivos:
Descarga Roundcube en /var/www/html/roundcube y asigna permisos:
bash
sudo chown -R www-data:www-data /var/www/html/roundcube/
sudo chmod -R 775 /var/www/html/roundcube/temp/ /var/www/html/roundcube/logs/
Usa el código con precaución.
3.	Configuración Web: Accede a http://localhost/roundcube/installer. Usa los datos de la DB creados arriba. Al finalizar, borra la carpeta installer.
________________________________________
6. ACCESO WEB Y REDIRECCIÓN AUTOMÁTICA
Para que al entrar a http://localhost/ cargue directamente el correo:
1.	Crea el archivo de redirección:
bash
sudo nano /var/www/html/index.php
Usa el código con precaución.
2.	Pega este código:
php
<?php header("Location: /roundcube/"); exit; ?>
Usa el código con precaución.
3.	Elimina el archivo original: sudo rm /var/www/html/index.html
________________________________________
7. PRUEBAS Y TROUBLESHOOTING
Prueba con Swaks (Simulación de Phishing):
bash
swaks --to user2@localhost --from "Soporte <soporte@localhost>" --header "Subject: Prueba" --body "Hola" --server IP_SERVIDOR
Usa el código con precaución.
Solución de Errores Comunes:
•	Error SMTP Auth: Verifica que la IP de la máquina que envía esté en mynetworks (Postfix).
•	Error Permisos Roundcube: Asegúrate de que temp/ y logs/ sean propiedad de www-data.
•	Log de errores en tiempo real: sudo tail -f /var/log/mail.log


DIAGRAMA DE FLUJO Y ARQUITECTURA
El siguiente esquema describe el trayecto de un correo electrónico desde su generación en la herramienta de auditoría hasta su visualización por el usuario final.
Esquema de Red (Flujo SMTP/IMAP)

 
Descripción del Proceso:
1.	Originación (SMTP): El emisor (Gophish o Swaks) se conecta al servidor Postfix a través del puerto 25. Al estar en la misma red y autorizada la IP, el servidor acepta el mensaje sin requerir autenticación externa.
2.	Procesamiento y Filtro: Postfix analiza el destinatario. Al detectar que el dominio es local (ej. localhost o servidor.local), lo transfiere al agente de entrega.
3.	Almacenamiento (Maildir): El correo se guarda físicamente en el directorio del usuario dentro del sistema Linux bajo la estructura de carpetas Maildir.
4.	Acceso Final (IMAP/HTTP): El usuario accede a la URL de Roundcube. PHP se comunica con Dovecot vía IMAP (puerto 143) para "leer" los archivos en el servidor y mostrarlos gráficamente en el navegador.
________________________________________

