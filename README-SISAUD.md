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
    
    $sudo apt update

Instalar Java8:

    $sudo apt install oracle-java8-installer

  Aceptar la instalacion y los terminos de Java.
  
Para ver la Version instalada:

    S java -version
    
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
        <role rolename="role1"/>                                                  
        <role rolename="admin-gui"/>                                              
        <user username="admin" password="password" roles="manager-gui,admin-gui"/>
    </tomcat-users>

Para Administrar app:
  
    $ sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml

Para administrar Host:

    sudo nano /opt/tomcat/webapps/host-manager/META-INF/context.xml

Reiniciar El servicio:

    $ sudo systemctl restart tomcat

 o
 
    $ /opt/tomcat/bin/shutdown.sh
    $ /opt/tomcat/bin/startup.sh
    
Ya esta configurado para uar el servidor y manager en:
  
    http://server_domain_or_IP:8080
![tomcat manager](https://assets.digitalocean.com/articles/tomcat8_1604/manager.png)



## Instalar Apache
## Instalar Php











## Instalar Apache y PostgreSQL

Conectar al servidor mediante ssh:

        $ ssh root@192.168.33.14

Actualizar los repositorios:

        $ sudo apt-get update

Instalar Apache2 y PostgreSQL:

        $ sudo apt-get install -y apache2 libapache2-mod-perl2 postgresql
        
Verificar si Apache y Postgres están corriendo en los puertos :80 y :5432

        # netstat -plntu
        
## Instalar módulos de Perl

OTRS esta basado en Perl, por lo que se necesita instalar módulos Perl para cubrir los requerimientos necesarios.

        $ sudo apt-get install -y libapache2-mod-perl2 libdbd-pg-perl libnet-dns-perl libnet-ldap-perl libio-socket-ssl-perl libpdf-api2-perl libsoap-lite-perl libgd-text-perl libgd-graph-perl libapache-dbi-perl libarchive-zip-perl libcrypt-eksblowfish-perl libcrypt-ssleay-perl libencode-hanextra-perl libjson-xs-perl libmail-imapclient-perl libtemplate-perl libtemplate-perl libtext-csv-xs-perl libxml-libxml-perl libxml-libxslt-perl libpdf-api2-simple-perl libyaml-libyaml-perl

Activar los módulos Perl para apache, luego reiniciar servicios de apache:

        # a2enmod perl
        # systemctl restart apache2
        
Verificar que los módulos estén activos:

        apachectl -M | sort
        
En la sección de `Loaded Modules` debe existir `perl_module (shared)`

## Crear nuevo usuario para OTRS

OTRS es una aplicación web que corre bajo apache. para mayor seguridad se necesita un usuario normal, no un usuario root.

Crear un usuario llamado 'otrs'

        # useradd -r -d /opt/otrs -c 'OTRS User' otrs

   `-r` : crear un usuario como una cuenta de sistema
   `-d /opt/otrs` : define el directorio home para el nuevo usuario
   `-c` :  comentario

Agregar el usuario 'otrs' al grupo 'www-data' porque  apache esta corriendo bajo el grupo 'www-data':

        # usermod -a -G www-data otrs
        
Verificar que el usuario 'otrs' este habilitado en el archivo `/etc/passwd`

        greep -rin otrs /etc/passwd
        
La respuesta debe ser similar a: `35:otrs:x:999:999:OTRS User:/opt/otrs:`

El nuevo usuario esta creado.


## Crear y configurar la Base de Datos

Es necesario crear una nueva base de datos PostgreSQL para el sistema OTRS, y configurarla de modo que este lista.

Login con el usuario `postgres` y acceder al shell de PostgreSQL:

        # su - postgres
        $ psql
        
Crear un nuevo rol llamado 'otrs' con el password 'otrsclave' y la opción 'nosuperuser':

        =# create user otrs password 'otrsclave' nosuperuser;
        
Crear una base de datos llamada 'otrs' con privilegios del usuario 'otrs':

        =# create database otrs owner otrs;
        =# \q
        
Editar el archivo de configuración de PostgreSQL para la autenticación del rol otrs:

        nano /etc/postgresql/9.6/main/pg_hba.conf
        
Pegar la siguiente configuración en el apartado `# Database administrative login by Unix domain socket`

        local   otrs            otrs                                    password
        host    otrs            otrs            127.0.0.1/32            password
        
guardar el archivo y salir de nano.

Volver con privilegios de root y reiniciar PostgreSQL:

        $ exit
        # systemctl restart postgresql

PostgreSQL esta listo para la instalación de OTRS.

## Descargar y configurar OTRS

La versión de OTRS que se esta instalando es la 5.0.16.

        # cd /opt/
        # wget http://ftp.otrs.org/pub/otrs/otrs-5.0.16.tar.gz
        
        
Extraer los archivos de OTRS, renombrar el directorio y cambiar el dueño de todos los archivos del directorio al usuario 'otrs':

        # tar -xzvf otrs-5.0.16.tar.gz
        # mv otrs-5.0.16 otrs
        # chown -R otrs:otrs otrs
        
Verificar que los módulos para OTRS estén instalados:

        # /opt/otrs/bin/otrs.CheckModules.pl
        
Es probable que en el listado nos muestren módulos con 'not installed' nos queda instalarlos con el comando `apt-get install -y modulo-que-falta`

OTRS esta descargado y el servidor esta listo para la instalación.

Modificar el archivo de configuración de OTRS:

        # cd /opt/otrs/
        # cp Kernel/Config.pm.dist Kernel/Config.pm
        
Editamos con nano: 

        # nano Kernel/Config.pm

Cambiar el password de la base de datos:

        $Self->{DatabasePw} = 'otrsclave';

Comentar el soporte para base de datos MySQL

        # $Self->{DatabaseDSN} = "DBI:mysql:database=$Self->{Database};host=$Self->{DatabaseHost};";
        
Quitar el comentario del soporte para bases de datos PostgreSQL

        $Self->{DatabaseDSN} = "DBI:Pg:dbname=$Self->{Database};";
        
Guardar el archivo y salir de nano.

Modificar el archivo apache startup para habilitar soporte a PostgreSQL:

        nano scripts/apache2-perl-startup.pl
        
Quitar el comentario de las lineas después de `# enable this if you use postgresql` quedando de la siguiente manera:

        # enable this if you use postgresql
        use DBD::Pg ();
        use Kernel::System::DB::postgresql;

Guardar el archivo y salir de nano.

Finalmente verificar que no exista error en las dependencias:

        perl -cw /opt/otrs/bin/cgi-bin/index.pl
        perl -cw /opt/otrs/bin/cgi-bin/customer.pl
        perl -cw /opt/otrs/bin/otrs.Console.pl

Las respuestas a cada script deben devolver `OK`.

## Importar Base de Datos de ejemplo

Login con el usuario 'postgres' y dirigirse al directorio otrs:

        # su - postgres
        $ cd /opt/otrs/
        
Insertar bases de datos y esquemas  de tablas con el comando psql como usuario otrs :

        $ psql -U otrs -W -f scripts/database/otrs-schema.postgresql.sql otrs
        $ psql -U otrs -W -f scripts/database/otrs-initial_insert.postgresql.sql otrs
        $ psql -U otrs -W -f scripts/database/otrs-schema-post.postgresql.sql otrs

Escribir el password `otrsclave` para cada comando.

## Iniciar OTRS

La base de datos y OTRS están configurados, queda iniciar OTRS.

Dar permisos para los archivos y directorios de otrs para el usuario y grupo `www-data`:

        # /opt/otrs/bin/otrs.SetPermissions.pl --otrs-user=www-data --web-group=www-data

Habilitar la configuración otrs apache, creando un enlace simbólico del archivo al directorio virtual host de apache:

        # ln -s /opt/otrs/scripts/apache2-httpd.include.conf /etc/apache2/sites-available/otrs.conf
        

Habilitar otrs virtual host y reiniciar apache:

        # a2ensite otrs
        # systemctl restart apache2

## Configurar OTRS Cronjob

Login con el usuario 'otrs', luego posicionarse en el directorio 'var/cron':

        # su - otrs
        $ cd var/cron/
        $ pwd
          /opt/otrs/var/cron
          
Copiar todos los scripts cronjob.dist:

        $ for foo in *.dist; do cp $foo `basename $foo .dist`; done
        
Volver a los privilegios de root, e iniciar los cron scripts:

        $ exit
        # /opt/otrs/bin/Cron.sh start otrs
        
debe devolver `(using /opt/otrs) done`.

Crear manualmente un cronjob para PostMaster que busque los correos electrónicos cada 2 minutos:

        # su - otrs
        $ crontab -e

Copiar y pegar o siguiente, al final del archivo:

        */2 * * * *    $HOME/bin/otrs.PostMasterMailbox.pl >> /dev/null
        
guardar y salir.

Ahora detenemos el Daemon de otrs y lo volvemos a iniciar:

        $ bin/otrs.Daemon.pl stop
        $ bin/otrs.Daemon.pl start
        
Deve devolver `Daemon stopped` y también `Daemon started`

## Probando OTRS

Para probar el sistema OTRS abra el navegador e ingrese a la dirección:

http://localhost/otrs/    ó    http://192.168.33.14/otrs/

Con el usuario: `root@localhost` y contraseña: `root`

![otrs 5s](https://www.howtoforge.com/images/how-to-install-otrs-opensource-trouble-ticket-system-on-ubuntu-16-04/big/12.png)

## Debug OTRS

Si tiene un error como 'OTRS Daemon no se esta ejecutando' puede habilitar la depuracion de Daemon:

        # su - otrs
        $ cd /opt/otrs/
        $ bin/otrs.Daemon.pl stop
        $ bin/otrs.Daemon.pl start --debug
        
