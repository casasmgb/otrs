# Instalar OTRS en Debian
## Instalar Apache y PostgreSQL

Conectar al servidor mediante ssh
        $ ssh root@192.168.43.75
Actualizar los repositorios
        $ sudo apt-get update
Instalar Apache2 y PostgreSQL
        $ sudo apt-get install -y apache2 libapache2-mod-perl2 postgresql
Verificar si Apache y Postgres estan corriendo en los puertos :80 y :5432
        # netstat -plntu
        
## Instalar Modulos de Perl
    
## Crear nuevo usuario para OTRS

## Crear y configurar la Base de Datos

## Descargar y configurar OTRS

## Importar Base de Datos de ejemplo

## Iniciar OTRS

## Configurar OTRS Cronjob

## Probando OTRS

## Iniciar Daemon

