# Instalar Kamailio con sqlite3

## Actualizar sistema

    sudo apt update && sudo apt upgrade -y

## Instalación

Instalar Kamailio junto con el módulo SQLite y sqlite3:

    sudo apt-get install kamailio kamailio-sqlite-modules sqlite3

## Configurar kamctlrc

Editar el archivo de configuración de kamctl:

    sudo vim /etc/kamailio/kamctlrc

Buscar y modificar las siguientes variables:

    DBENGINE=SQLITE

    DB_PATH="/tmp/kamailio.db"

## Crear base de datos

Ejecutar el script de creación de base de datos:

    sudo kamdbctl create

Esto creará el archivo `/tmp/kamailio.db` con todas las tablas necesarias.

Asignar permisos correctos para que Kamailio pueda acceder a la base de datos:

    sudo chown kamailio:kamailio /tmp/kamailio.db

## Configurar kamailio.cfg

Editar la configuración principal de Kamailio:

    sudo vim /etc/kamailio/kamailio.cfg

Asegurarse de que las siguientes directivas están presentes (o agregarlas si no existen):

Añadir al inicio para cargar el módulo db_sqlite:

    #!define WITH_DB
    loadmodule "db_sqlite.so"

Configurar el alias de la base de datos SQLite (después de cargar los módulos):

    modparam("usrloc", "db_url", "sqlite:///tmp/kamailio.db")
    modparam("subscriber", "db_url", "sqlite:///tmp/kamailio.db")

## Iniciar servicio

Habilitar e iniciar Kamailio:

    sudo systemctl enable kamailio
    sudo systemctl start kamailio

Verificar que el servicio está corriendo:

    sudo systemctl status kamailio

## Verificar instalación

Comprobar que kamailio está escuchando:

    sudo ss -tlnp | grep kamailio

Listar las tablas creadas en la base de datos:

    sqlite3 /tmp/kamailio.db .tables

## Agregar usuario de prueba

    kamctl add test 1234

## Firewall

Si hay un firewall activo, permitir el tráfico SIP (puerto 5060 UDP):

    sudo ufw allow 5060/udp
