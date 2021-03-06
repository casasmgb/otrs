                                                                                                    by: gcasas
# Instalar SISAUD en Debian

Se instalara:

## Java8
## Tomcat
## Apache
## Php

## Instalar Java8
Conectar al servidor mediante ssh:

    $ ssh root@192.168.33.14

Actualizar los repositorios:

    $ sudo apt-get update
        
Instalar  software-properties-common y dirmngr para instalar java:

    $ sudo apt install software-properties-common
    $ sudo apt-get install dirmngr
    
Agregar los paquetes para la version 8:

    $ sudo add-apt-repository ppa:webupd8team/java
    
  precione enter y verifique que las llaves gpg se agregaron correctamente como se ve a continuacion:
    
    gpg: keybox '/tmp/tmpmvi_tfzn/pubring.gpg' created             
    key C2518248EEA14886:                                          
    14 signatures not checked due to missing keys                  
    gpg: /tmp/tmpmvi_tfzn/trustdb.gpg: trustdb created             
    gpg: key C2518248EEA14886: public key "Launchpad VLC" imported 
    gpg: no ultimately trusted keys found                          
    gpg: Total number processed: 1                                 
    gpg:               imported: 1                                 
    gpg: no valid OpenPGP data found.                              

Agregar la llave a los repositorios:
    
    $ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C2518248EEA14886
  debe responder similar a:
  
    Executing: /tmp/apt-key-gpghome.bb1vIJLem4/gpg.1.sh --keyserver keyserver.ubuntu.com --recv-keys C2518248EEA14886 
    key C2518248EEA14886:                                                                                             
    14 signatures not checked due to missing keys                                                                     
    gpg: key C2518248EEA14886: public key "Launchpad VLC" imported                                                    
    gpg: Total number processed: 1                                                                                    
    gpg:               imported: 1                                                                                    
    
Actualizar los repositorios:
    
    $ sudo apt update

Instalar Java8:

    $ sudo apt install oracle-java8-installer

  Aceptar la instalacion y los terminos de Java.
  
Para ver la Version instalada:

    $ java -version
    
 respuesta:
 
    java version "1.8.0_201"
    Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
    Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)

## Instalar Tomcat

Por seguridad se debe crear un usuario sin privilegios para tomcat pero algunos sistemas operativos predentan una vulnerabilidad que quita privilegios a un usuario creado por adduser, por lo que sebe ejecutar lo siguiente:

    $ sudo apt remove unscd

Primero crearemos un grupo para el usuario tomcat:

    $ sudo groupadd tomcat

Luego creamos el usario tomcat, lo haremos mienbro del grupo tomcat, y le daremos el directorio /opt/tomcat (donde estara instalado Tomcat), y agregaremos /bin/false (nadie podra iniciar session con la cuenta).

    $ sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
  listo...

Ahora a instalar Tomcat, para escoger la version podemos visitar [La pagina de descarga de Tomcat 9](https://tomcat.apache.org/download-90.cgi), debajo de la seccion Binary Distributions -> Core, copiar el link del "tar.gz" de la version que qiere instalar.
Para descargar cambiar la ruta a /tmp

    $ cd /tmp
    
Usaremos curl, si no se tiene instalado instalar con:

    $ sudo apt install curl

Descargar los binarios de la version 8.5:
    
    $ curl -O https://www-eu.apache.org/dist/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.tar.gz
    
Instalaremos Tomcat 8.5 en /opt/tomcat. crear la direccion y extraer los archivos:

    $ sudo mkdir /opt/tomcat
    $ sudo tar xzvf apache-tomcat-8.5.37.tar.gz -C /opt/tomcat --strip-components=1

Se debe cambiar los permisos al usuario tomcat:
  el usuario tomcat sera propietario d /opt/tomcat
  dar permisos de escritura al grupo para la carpeta conf
  dar permisos de ejecucion al grupo para la carpeta conf
  el usuario tomcat sera dueño de las carpetas: webapps/ work/ temp/ logs/
  
    
    $ cd /opt/tomcat
    $ sudo chgrp -R tomcat /opt/tomcat
    $ sudo chmod -R g+r conf
    $ sudo chmod g+x conf
    $ sudo chown -R tomcat webapps/ work/ temp/ logs/

Crear el archivo systemd Service.

debemos conocer cual es el directorio de nuestro JAVA_HOME, podemos usar el comando.

    $ sudo update-java-alternatives -l
  
  la respuesta sera: 
  
    java-8-oracle                  1081       /usr/lib/jvm/java-8-oracle
    
  /usr/lib/jvm/java-8-oracle es la ruta del JAVA_HOME, puede ser diferente.
  
Crearemos el archivo tomcat.service:

    $ sudo nano /etc/systemd/system/tomcat.service
  
  y pegaremos lo siguiente, reemplazar JAVA_HOME por lo obtenido en el paso anterior:
  
    [Unit]
    Description=Apache Tomcat Web Application Container
    After=network.target

    [Service]
    Type=forking

    Environment=JAVA_HOME=/usr/lib/jvm/java-8-oracle
    Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
    Environment=CATALINA_HOME=/opt/tomcat
    Environment=CATALINA_BASE=/opt/tomcat
    Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
    Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

    ExecStart=/opt/tomcat/bin/startup.sh
    ExecStop=/opt/tomcat/bin/shutdown.sh

    User=tomcat
    Group=tomcat
    UMask=0007
    RestartSec=10
    Restart=always

    [Install]
    WantedBy=multi-user.target
    
  guardar y cerar nano (se puede usar otro editor) con ctrl+o  y ctrl+x
  
Ahora recargaremos el deminio de systemd:

    $ sudo systemctl daemon-reload
    
Iniciamos el servidor Tomcat escribiendo:
  
    $ sudo systemctl start tomcat
  
  se puede usar tambien:
  
    $ sudo /opt/tomcat/bin/startup.sh
    
Para verificar el estado del servidor:

    $sudo systemctl status tomcat
    
Se debe ajustar el firewal para probar el servisor:

  usaremos ufw si no se tiene instalado instalar con:
  
    $ sudo apt install ufw
    
  y ejecutar el comnado par aconfigurrar el puerto 8080
  
    $ sudo ufw allow 8080
  
 podremos abrir la pagina principal de tomcat en el servidor en la direccion:
 
    http://server_domain_or_IP:8080
    
para detener tomcat usar:

    $ sudo systemctl enable tomcat
  
  o
  
    $ /opt/tomcat/bin/shutdown.sh
  
    
Configurar Tomcat Web Manage Interfaz.
editar el archivo tomcat-users.xml

    $ sudo nano /opt/tomcat/conf/tomcat-users.xml

  y configurar un usuario simular a:
  
    <tomcat-users . . .>
        <role rolename="manager-gui"/>
        <role rolename="admin-gui"/>
        <user username="admin" password="password" roles="manager-gui,admin-gui"/>
    </tomcat-users>

Para Administrar app:
  
    $ sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml
    
  o en  /opt/tomcat/webapps/host-manager/META-INF/context.xml depende de la version y configuracion
  
    $ sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml

y comentar las lineas de ip debe quedar de la siguiente manera:

    <Context antiResourceLocking="false" privileged="true" >
    <!--
      <Valve className="org.apache.catalina.valves.RemoteAddrValve"
             allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
      <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1$
      -->
    </Context>

Reiniciar El servicio:

    $ sudo systemctl restart tomcat

 o
 
    $ /opt/tomcat/bin/shutdown.sh
    $ /opt/tomcat/bin/startup.sh
    
Ya esta configurado para uar el servidor y manager en:
  
    http://server_domain_or_IP:8080
![tomcat manager](https://assets.digitalocean.com/articles/tomcat8_1604/manager.png)

## Instalar Apache

Con el gestor de paquetes de debian:

    $ sudo apt install apache2
    
Permitir que el firewall acepte HTTP y HTTPS, usaremos la heramienta ufW

    $ sudo ufw app list
  
  El perfil WWW es usado para administrar los puertos por el servidor:
    
    ...
      WWW
      WWW Cache
      WWW Full
      WWW Secure
    ...
    
  verificamos el trifoc por los puertos del perfil WWW Full, que son 80,443/tcp
  
    $ sudo ufw app info "WWW Full"
      
 permitir trafico para HTTP y HTTPS con este perfil
 
    $ sudo ufw allow in "WWW Full"
    
  ahora podremos ver apache instalado en el servidor.
    
    http://your_server_ip 

  ![apache](https://i.stack.imgur.com/1NOHl.jpg)

The WWW profiles are used to manage ports used by web servers:
    

Output
Available applications:
. . .
  WWW
  WWW Cache
  WWW Full
  WWW Secure
. . .
  
## Instalar Php 7.2

Para instalar PHP usaremos apt agregando un repositorio:
    
    $ wget -q https://packages.sury.org/php/apt.gpg -O- | sudo apt-key add -
    $ echo "deb https://packages.sury.org/php/ stretch main" | sudo tee /etc/apt/sources.list.d/php.list
    $ sudo apt-get install ca-certificates apt-transport-https
    $ sudo apt-get update
    
Para instalar PHP usaremos apt:
    
    $ sudo apt-get install php7.2 libapache2-mod-php7.2

Instalamos alguna extenciones de php
  
    $ sudo apt install php7.2-cli php7.2-pdo php7.2-mbstring php7.2-tokenizer php7.2-xml php7.2-ctype php7.2-json php7.2-soap libargon2-0 libsodium23 libssl1.1 php7.2-cli php7.2-opcache php7.2-readline
    
Definimos a php7.2

    $ update-alternatives --set php /usr/bin/php7.2
    
Habilitamos php7.2 para apache y habilitamos el modulo rewrite

    $ sudo a2enmod php7.2
    $ sudo a2enmod rewrite
    
Reiniciamos apache2

    $ systemctl restart apache2
    
Argegamos la extencion .php al servidor apache, moviendo index.php al pincipio
  
    $ sudo nano /etc/apache2/mods-enabled/dir.conf
  
  debe quedar de la siguiente manera:
  
    <IfModule mod_dir.c>
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
    </IfModule>
    
  guardamos con ctrl+o y ctrl+x
  
Reiniciamos el servidor:

    $ sudo systemctl restart apache2

Verificamos el estado del servidor.
    
    $ sudo systemctl status apache2

Probamos si esta instalado con php.info
  
    $ sudo nano /var/www/html/info.php
  
  y agregar al archivo
  
    <?php
      phpinfo();
    ?>

guardar y cerrar el archivo.

probar PHP en:

    http://your_server_ip/info.php
    
## Configuracion para la aplicacion SISAUD

Copiar el proyecto en la ruta /var/www/html/SISTEMA_WEB

### Crear un virtual host para la aplicacion.

Damos permiso www-data a la carpeta /varwww/html

    $ chown -R www-data: /var/www/html
    
Creamos el archivo para la configuracion del virtualhost

    $ nano /etc/apache2/sites-available/sisaud.bo.conf
    
y en el pegamos lo siguiente:
    
    <VirtualHost *:80>                                                  
                                                                    
        DocumentRoot "/var/www/html/SISTEMA_WEB/public"                    
        ServerName 172.16.70.157                                    
        <Directory "/var/www/html/SISTEMA_WEB/public">                     
           AllowOverride  all                                       
           Order allow,deny                                         
           Allow from all                                           
        </Directory>                                                
                                                                    
        ErrorLog ${APACHE_LOG_DIR}/sisaud.bo_error.log              
        CustomLog ${APACHE_LOG_DIR}/sisaud.bo_access.log combined   
                                                                    
    </VirtualHost>                                                      

Habilitamos el sitio con:

    $ a2ensite sisaud.bo.conf
    
Reiniciamos el servidor apahe:

    $ systemctl restart apache2
    
Usaremos composer si no se tiene insatalado segir los pasos acontinuacion.

    $ curl -sS https://getcomposer.org/installer | php
    $ mv composer.phar /usr/local/bin/composer
    $ sudo chmod +x /usr/local/bin/composer
    $ composer 
    
       ______
      / ____/___  ____ ___  ____  ____  ________  _____
     / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
    / /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
    \____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                      /_/
    Composer version 1.X.X 201X-mm-dd hh:mm:ss

Eliminamos la carpeta vendor y los archivos .lock de composer 

    $ sudo rm -rf composer.lock vendor
    
y reinstalamos las dependencias:

    $ composer install
    
la pagina aparecera en el navegador con la IP del equipo

    http://your_server_ip/



