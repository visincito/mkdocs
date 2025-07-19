
# Instalar Kamailio con sqlite3

## Instalación

Start by installing Kamailio and the associated SQLite module, optionally the command line interface for SQLite which can be useful for poking around the Kamailio database:

    sudo apt-get install kamailio kamailio-sqlite-modules sqlite3

## Database Creation

Next, set up the SQLite database. Open up the kamctlrc file with a preferred editor:

    sudo vim /etc/kamailio/kamctlrc

Find the DBENGINE variable and set it to SQLITE:

    DBENGINE=SQLITE
