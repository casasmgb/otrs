# Instalar OTRS 5s en Debian
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

(http://localhost/otrs/)

Con el usuario: `root@localhost` y contraseña: `root`

![otrs 5s](https://www.howtoforge.com/images/how-to-install-otrs-opensource-trouble-ticket-system-on-ubuntu-16-04/big/12.png)

## Debug OTRS

Si tiene un error como 'OTRS Daemon no se esta ejecutando' puede habilitar la depuracion de Daemon:

        # su - otrs
        $ cd /opt/otrs/
        $ bin/otrs.Daemon.pl stop
        $ bin/otrs.Daemon.pl start --debug
