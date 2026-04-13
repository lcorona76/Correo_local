MANUAL DE INSTALACIÓN Y CONFIGURACIÓN: LABORATORIO DE CORREO LOCAL
Versión: 1.0
Uso: Laboratorio de Ciberseguridad, Phishing y Pruebas de Red.
________________________________________

1. INTRODUCCIÓN
El presente manual describe el procedimiento técnico para el despliegue de un sistema de mensajería unificado en entorno local, diseñado específicamente para laboratorios de ciberseguridad, pruebas de penetración y simulaciones de ingeniería social.
A diferencia de un servidor de producción orientado a Internet, este entorno se centra en la interoperabilidad entre herramientas de auditoría (como Gophish o Swaks) y un servidor de correo interno robusto basado en el estándar MTA/MDA (Message Transfer Agent / Message Delivery Agent). La arquitectura utiliza Postfix para la transferencia de mensajes y Dovecot para la gestión de buzones, integrando una interfaz web mediante Roundcube para facilitar la interacción del usuario final.
Objetivos del Laboratorio:
•	Simulación de Vectores de Ataque: Permitir el envío de campañas de phishing controladas sin riesgo de filtración a redes externas.
•	Validación de Protocolos: Analizar el comportamiento de las cabeceras SMTP y los mecanismos de autenticación SASL en un entorno aislado.
•	Gestión de Usuarios: Administrar múltiples cuentas de correo locales para la validación de entrega y respuesta.
________________________________________
2. ARQUITECTURA Y FLUJO DE DATOS
El correo sigue el siguiente trayecto técnico:
1.	Emisión: Gophish/Swaks conecta vía SMTP (Puerto 25) con Postfix.
2.	Recepción: Postfix valida la IP de origen y entrega el mensaje a Dovecot.
3.	Almacenamiento: El mensaje se guarda en formato Maildir en la carpeta del usuario.
4.	Visualización: El usuario accede vía HTTP (Apache) a Roundcube, que lee el correo mediante IMAP (Puerto 143).
________________________________________
3. ARQUITECTURA Y FLUJO DE DATOS

El siguiente esquema describe el trayecto de un correo electrónico desde su generación en la herramienta de auditoría hasta su visualización por el usuario final.
Esquema de Red (Flujo SMTP/IMAP)

 
Descripción del Proceso:
1.	Originación (SMTP): El emisor (Gophish o Swaks) se conecta al servidor Postfix a través del puerto 25. Al estar en la misma red y autorizada la IP, el servidor acepta el mensaje sin requerir autenticación externa.
2.	Procesamiento y Filtro: Postfix analiza el destinatario. Al detectar que el dominio es local (ej. localhost o servidor.local), lo transfiere al agente de entrega.
3.	Almacenamiento (Maildir): El correo se guarda físicamente en el directorio del usuario dentro del sistema Linux bajo la estructura de carpetas Maildir.
4.	Acceso Final (IMAP/HTTP): El usuario accede a la URL de Roundcube. PHP se comunica con Dovecot vía IMAP (puerto 143) para "leer" los archivos en el servidor y mostrarlos gráficamente en el navegador.
________________________________________

1. Preparación del Sistema
Instalar los paquetes base para el MTA (Postfix), el MDA (Dovecot) y herramientas de gestión.

bash
sudo apt update
sudo apt install postfix dovecot-imapd mailutils php-intl php-mbstring php-gd php-xml php-mysql php-zip apache2 mariadb-server -y
Usa el código con precaución.

Nota: Durante la instalación de Postfix, selecciona "Internet Site" y define tu dominio (ej. servidor.local).

2. Configuración de Usuarios

Crea los usuarios de Linux que funcionarán como cuentas de correo:

bash
sudo adduser user1
sudo adduser user2

Usa el código con precaución.

3. Configuración de Postfix (Envío)

Edita /etc/postfix/main.cf para permitir conexiones de red y autenticación:
Escucha: inet_interfaces = all
Confianza: mynetworks = 127.0.0.0/8, [::1]/128, IP_MAQUINA_GOPHISH
Formato de buzón: home_mailbox = Maildir/
Autenticación SASL:
conf
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination

Usa el código con precaución.

Reiniciar: sudo systemctl restart postfix

4. Configuración de Dovecot (Recepción)

Configura Dovecot para que Postfix pueda validar usuarios y habilitar el acceso local.
En /etc/dovecot/conf.d/10-mail.conf: mail_location = maildir:~/Maildir
En /etc/dovecot/conf.d/10-auth.conf: disable_plaintext_auth = no
En /etc/dovecot/conf.d/10-master.conf:
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

5. Instalación de Webmail (Roundcube)

Base de Datos:

sql
CREATE DATABASE roundcubemail;
GRANT ALL PRIVILEGES ON roundcubemail.* TO 'roundcube'@'localhost' IDENTIFIED BY 'tu_password';
FLUSH PRIVILEGES;

Usa el código con precaución.

Archivos: Descarga Roundcube en /var/www/html/roundcube y asigna permisos:

bash
sudo chown -R www-data:www-data /var/www/html/roundcube/
sudo chmod -R 775 /var/www/html/roundcube/temp/ /var/www/html/roundcube/logs/

Usa el código con precaución.

Inicialización: Accede a http://localhost/roundcube/installer, configura el DSN de la base de datos y ejecuta el botón "Initialize Database".

6. Pruebas de Conectividad

Desde la red con Swaks (Simulación simple):
bash
swaks --to user2@localhost --from user1@localhost --server IP_SERVIDOR_POSTFIX
Usa el código con precaución.

Desde Gophish (Phishing simulado):

Host: IP_SERVIDOR_POSTFIX:25
From: Cualquier Nombre <user1@localhost>
Auth: Desactivada (ya que la IP de Gophish está en mynetworks).
