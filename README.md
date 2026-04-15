# Cosas-de-teleco
Para crear una instancia.

Agregar nombre: 
Seleccionar el sistema operativo: 
Crear nuevo par de claves, no hay que perder la clave y que se guarde el archivo descargado ya que sera utilizado.
Tipo de clave rsa, seleccionar.pem  (Bebeate21#) <- contraseña mia
En config de red, hay que habilitar asignar automaticamente la ip publica.
Seleccionar un grupo de segurudad existente y usar el default.

Ver panel, instancia y que este inicializando. 
Ver que nuestro grupo de seguridad tenga las reglas correspondeintes o crear una, voy a panel, grupos de seguridad
nombre de grupo, tienen regla de entrada y salida 
agregar regla en tcp perzoanlizado, seleccionar todo el treafico
el origen, anywahe edicion 4 (para ponder tener concetividad con cualquier trefico)
guardar regla

para conectar 
se seleciona el id, y al cosntado dice conectar, nos conecta

mos por ssh 
el archivo de seguridad que se descarga, vamos a la rita y escribimos powershwll para abrir 

la opcion -i, es para ver que claves vamos a usar 
en el ssh nos vamos donde dice ejemplo y al abrir el power shel coppiamos eso y nos abre el linux.


estas web tienen una ip puclica

nic.cl, daniel.cl y ahi apuntamos a la ip publica 
piwershell
pwd
ls (para ennumerar los directorios y archibvos dentro de la carpeta)
cd.. (para volver hacia atras)
cd (para avanzar hacia dentro de las carpetas)
pwd (para saber donde esstoy)
cd.. vuelvo atras
ls 8entro al esqueleto de linux y me muetra todas la optiones que hay 
cat (se usa para leer archivos), cat espacio porque cat es la herramneita ->/etc/passwd (para ver donde se encuentra la raiz de los archivos. 

con sudo su queda como admin privilegiado 
markdown 





para poder crear una web con la instancia debemos de intalar mysql sistema de gestión de bases de datos relacionales
tambien debemos de instalar "php" en el panel de aws el php proporciona funciones para leer, escribir y procesar archivos en el servidor, lo cual es útil para administrar el contenido del sitio web.

Comandos para instalar en panel linux..................................

pwd: Print Working Directory Muestra la ruta de la carpeta donde estás parado.
ls: List: Enumera los archivos y directorios en la ubicación actual.
cd ..: Retroceder un nivel en el árbol de directorios.
cd [nombre]: Avanzar o entrar en una carpeta específica.
cat [archivo]: Visualizar el contenido de un archivo de texto en la terminal.
sudo su: Cambiar al usuario root (superusuario) para obtener privilegios totales.
cat /etc/passwd: Para ver la base de datos de usuarios creada por el sistema.
Crear Grupos: sudo groupadd tecnicos
Crear Usuarios: sudo useradd -m -s /bin/bash admin_red
Contraseña: sudo passwd admin_red
🏗️ 1. Instalación de MySQL (Base de Datos)
-Primero, instalaremos el motor donde se guardará la información.
sudo apt-get update
sudo apt-get install mysql-server
Membresía: sudo usermod -aG tecnicos admin_red
Carpeta: sudo mkdir /home/datos_servidor
Dueño y Grupo: sudo chown admin_red:tecnicos /home/datos_servidor
Archivo: sudo touch /home/datos_servidor/configuracion.conf
ls -ld /home/datos_servidor: Para ver si el dueño es admin_red y el grupo es tecnicos.
cat /etc/passwd: Para confirmar que los usuarios fueron creados correctamente.


🌐 2. Instalación de PHPMyAdmin (Panel Web)
-Esto sirve para ver tu base de datos desde un navegador (Chrome o Edge) en lugar de usar comandos.
Instalar paquetes: sudo apt install phpmyadmin.
Selecciona apache2 (usa la tecla Espacio para marcar el asterisco y luego Enter).
Selecciona NO cuando pregunte por dbconfig-common.
Luegoooooooo.
sudo apt install php libapache2-mod-php
sudo systemctl restart apache2: Para poder reiniciar y refrescar.
http://TU_IP_PUBLICA/phpmyadmin

Para entrar al mysql: sudo mysql -u root -p
Codigo ingresaso en mysql para la conf de la pagina web de central. 

"CREATE DATABASE central_alameda;
CREATE USER 'admin_central'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Atecentral206';
GRANT ALL PRIVILEGES ON central_alameda.* TO 'admin_central'@'localhost';
FLUSH PRIVILEGES;

USE central_alameda;
CREATE TABLE servicios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    patente VARCHAR(15),
    nombre_cliente VARCHAR(100),
    telefono_cliente VARCHAR(20),
    modelo_auto VARCHAR(50),
    producto_instalar VARCHAR(100),
    observaciones TEXT,
    observacion_trabajador TEXT,
    foto_entrega VARCHAR(255),
    llego_al_taller TINYINT(1) DEFAULT 0,
    estado_pago TINYINT(1) DEFAULT 0,
    fecha_ingreso DATETIME DEFAULT CURRENT_TIMESTAMP
);
EXIT;"

🛠️Configuración de Acceso Remoto:
cd /etc/mysql/mysql.conf.d: Ir a la carpeta de configuración.
sudo nano mysqld.cnf: Editar el archivo.


htop: Para ver el rendimiento de lapgaina a traves de linux.

sudo systemctl restart apache2: para refrescar la web 
cat /var/www/html/index.php: para ver el codigo y copiarlo
sudo nano /var/www/html/index.php: para editar el codigo
sudo rm /var/www/html/index.php: para borrar cod
