# Instalar OTRS en Debian
## Instalar Apache y PostgreSQL

Conectar al servidor mediante ssh:

        $ ssh root@192.168.43.75

Actualizar los repositorios:

        $ sudo apt-get update

Instalar Apache2 y PostgreSQL:

        $ sudo apt-get install -y apache2 libapache2-mod-perl2 postgresql
        
Verificar si Apache y Postgres estan corriendo en los puertos :80 y :5432

        # netstat -plntu
        
## Instalar Modulos de Perl

OTRS esta basado en Perl, por lo que se necesita instalar modulos Perl para cubrir los requerimientos necesarios.

        $ sudo apt-get install -y libapache2-mod-perl2 libdbd-pg-perl libnet-dns-perl libnet-ldap-perl libio-socket-ssl-perl libpdf-api2-perl libsoap-lite-perl libgd-text-perl libgd-graph-perl libapache-dbi-perl libarchive-zip-perl libcrypt-eksblowfish-perl libcrypt-ssleay-perl libencode-hanextra-perl libjson-xs-perl libmail-imapclient-perl libtemplate-perl libtemplate-perl libtext-csv-xs-perl libxml-libxml-perl libxml-libxslt-perl libpdf-api2-simple-perl libyaml-libyaml-perl

Activar los modulos Perl para apache, luego reiniciar servicios de apache:

        # a2enmod perl
        # systemctl restart apache2
        
Verificar que los modulos esten activos:

        apachectl -M | sort
        
En la sección de `Loaded Modules` debe existir `perl_module (shared)`

## Crear nuevo usuario para OTRS

OTRS es una aplicacion web que corre bajo apache. para mayor seguridad se necesita un usuario normal, no un usuario root.

Crear un usuario llamado 'otrs'

        # useradd -r -d /opt/otrs -c 'OTRS User' otrs

   `-r` : crear un usuario como una cuenta de sistema
   `-d /opt/otrs` : define el directorio home para el nuevo usuario
   `-c` :  comentario

## Crear y configurar la Base de Datos


## Descargar y configurar OTRS

module
## Importar Base de Datos de ejemplo


## Iniciar OTRS


## Configurar OTRS Cronjob


## Probando OTRS


## Iniciar Daemon

