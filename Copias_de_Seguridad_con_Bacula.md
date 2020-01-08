# Copias de Seguridad con la herramienta Bacula.

#### Implementar un sistema de copias de seguridad para las instancias del cloud, teniendo en cuenta las siguientes características:

* Selecciona una aplicación para realizar el proceso: bacula, amanda, shell script con tar, rsync, dar, afio, etc.

* Utiliza una de las instancias como servidor de copias de seguridad, añadiéndole un volumen y almacenando localmente las copias de seguridad que consideres adecuadas en él.

* El proceso debe realizarse de forma completamente automática.

* Selecciona qué información es necesaria guardar (listado de paquetes, ficheros de configuración, documentos, datos, etc.)

* Realiza semanalmente una copia completa.
  
* Realiza diariamente una copia incremental o diferencial (decidir cual es más adecuada)
  
* Implementa una planificación del almacenamiento de copias de seguridad para una ejecución prevista de varios años, detallando qué copias completas se almacenarán de forma permanente y cuales se irán borrando.

* Añade tu sistema de copias a coconut cuando esté disponible.
  
* Selecciona un directorio de datos "críticos" que deberá almacenarse cifrado en la copia de seguridad, bien encargándote de hacer la copia manualmente o incluyendo la contraseña de cifrado en el sistema.
  
* Incluye en la copia los datos de las nuevas aplicaciones que se vayan instalando durante el resto del curso.
  
* Utiliza saturno u otra opción que se te facilite como equipo secundario para almacenar las copias de seguridad. Solicita acceso o la instalación de las aplicaciones que sean precisas.


#### La corrección consistirá tanto en la restauración puntual de un fichero en cualquier fecha como la restauración completa de una de las instancias la última semana de curso.

----------------------------------------------------
----------------------------------------------------
# Bacula

#### En esta práctica vamos a utilizar Bacula, que es un gestor de copias de seguridad fácil de manejar, gracias a que utiliza cliente-servidor.

#### Vamos a crear una nueva instancia con el nombre de *Serranito*, en el cual, vamos a utilizar como servidor de copias de seguridad dirgido por bacula. Este puede trabajar con base de datos como MySQL, PostgreSQL y SQLite.

## Instalación de Bacula

-------------------------------

* Para la gestión de Bacula, al tener tantas opciones, podemos instalar una aplicación web llamada **webmin**, para saber como se instala mira esta [página]().

* Para instalar **webmin** tenemos que realizar la instalación de un LAMP (Linux, Apache, MySQL, PHP). Si no sabes realizar la instalación y configuración de un LAMP, mira en esta [página]().

---------------------------------

Si no vamos a instalar **webmin** tenemos que instalar de todas formas el servidor de MySQL, ya que bacula necesita un gestor de base de datos para almacenar las copias de seguridad.

###### Instalamos MySQL

~~~
sudo apt install mariadb-server mariadb-client
~~~

Cuando instalemos la base de datos, el siguiente paso es instalar los paquetes necesarios para la configuración de **bacula**

###### Instalación de bacula

~~~
sudo apt install bacula bacula-client bacula-common-mysql bacula-director-mysql bacula-server
~~~

> **NOTA**: durante la instalación del paquete `bacula-director-mysql`, nos pedirá que que indiquemos si queremos configurar de forma automatica la base de datos. Le indicamos `Yes`.

###### Aceptamos la configuración autamatica de MySQL

~~~
 ┌───────────────────────────────┤ Configuring bacula-director-mysql ├────────────────────────────────┐
 │                                                                                                    │ 
 │ The bacula-director-mysql package must have a database installed and configured before it can be   │ 
 │ used. This can be optionally handled with dbconfig-common.                                         │ 
 │                                                                                                    │ 
 │ If you are an advanced database administrator and know that you want to perform this               │ 
 │ configuration manually, or if your database has already been installed and configured, you should  │ 
 │ refuse this option. Details on what needs to be done should most likely be provided in             │ 
 │ /usr/share/doc/bacula-director-mysql.                                                              │ 
 │                                                                                                    │ 
 │ Otherwise, you should probably choose this option.                                                 │ 
 │                                                                                                    │ 
 │ Configure database for bacula-director-mysql with dbconfig-common?                                 │ 
 │                                                                                                    │ 
 │                          [[<Yes>]]                                <No>                             │ 
 │                                                                                                    │ 
 └────────────────────────────────────────────────────────────────────────────────────────────────────┘ 
~~~

###### Indicamos la contraseña de usuario del administrados de MySQL

~~~
 ┌───────────────────────────────┤ Configuring bacula-director-mysql ├────────────────────────────────┐
 │ Please provide a password for bacula-director-mysql to register with the database server. If left  │ 
 │ blank, a random password will be generated.                                                        │ 
 │                                                                                                    │ 
 │ MySQL application password for bacula-director-mysql:                                              │ 
 │                                                                                                    │ 
 │ *************______________________________________________________________________________________ │ 
 │                                                                                                    │ 
 │                            <Ok>                                <Cancel>                            │ 
 │                                                                                                    │ 
 └────────────────────────────────────────────────────────────────────────────────────────────────────┘ 
~~~

###### Comprobamos que esta configurada la base de datos

~~~
debian@serranito:~$ sudo mariadb
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 61
    Server version: 10.3.18-MariaDB-0+deb10u1 Debian 10

    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | bacula             |
    | information_schema |
    | mysql              |
    | performance_schema |
    +--------------------+
    4 rows in set (0.001 sec)
~~~

## Configuración de Bacula

Ahora vamos a configurar varios ficheros de bacula para definir:

* Tipo de copia de seguridad
* Tipo de tarea a realizar
* Clientes
* Directorios que vamos a copiar
* Directorios que no vamos a copiar

El primer fichero que tenemos que modificar es `/etc/bacula/bacula-dir.conf`. Este fichero es el fichero principal, donde se configura la gran parte de las opciones de bacula.

Vamos a añadir un campo principal, indicandole cual es el `Director` y dentro de este varias opciones.

###### Añadimos apartado `Director`

~~~
Director {
  Name = serranito-dir
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "*************"
  Messages = Daemon
  DirAddress = 10.0.0.17
}
~~~

> **NOTA**: Cambiamos la `Password` y `DirAddress`. Lo demás lo dejamos por defecto.

Ahora en el mismo fichero, tenemos que modificar el apartado `JobDefs`, donde indicaremos varios campos, como si fueran variables, y las ampliaremos mas adelante.

###### Añadimos apartado `JobDefs`

~~~
JobDefs {
  Name = "Tarea-Serranito"
  Type = Backup
  Level = Incremental
  Client = serranito-fd
  FileSet = "CopiaCompleta"
  Schedule = "Programa"
  Storage = Vol-Serranito
  Messages = Standard
  Pool = Vol-Backup
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}
~~~

> **Name**: Nombre que le asignamos a la tarea `JobDefs`.
> 
> **Type**: Indicamos que el tipo de tarea son copias de seguridad, `Backup`.
>
> **Level**: Indicamos como queremos realizar las copias de seguridad, en nuestro caso `Incremental`, que es una copia de los datos creados y modificados desde la última ejecución de la copia de seguridad.
>
> **Client**: El nombre del cliente en el que se va a ejecutar esta tarea.
>
> **FileSet**: Le indicamos el nombre que vamos a utilizar en el apartado siguiente, donde expandiremos el campo de `CopiaCompleta`, indicandole los ficheros que queremos que se realicen la copia.
>
> **Schedule**: Indicamos el nombre que vamos a utilizar para extender el campo `Programa`, donde indicaremos la fecha y la regularidad de las copias de seguridad.
>
> **Storage**: Indicamos el nombre del apartado `Vol-Serranito`, el cual se extenderá mas adelante e indicaremos el tipo de almacenamiento vamos a utilizar.
>
> **Messages**: Tipo de mensaje que se dejará en `Standar` que es la opción por defecto, el cual indicará como mandará los mensajes de sucesos de bacula.
>
> **Pool**: Indicaremos el nombre del apartado `pool` que se configurará mas adelante y en él estamos indicando el volumen de almacenamiento donde se creará y almacenará las copias.
>
> **SpoolAttributes**: Esta opción permite trabajar con los atributos del spool en un fichero temporal, así que dejaremos la opcion por defecto.
>
> **Priority**: Indica el nivel de prioridad en el cual si ponemos un nùmero menos, quiere decir que tiene más prioridad e indicará cual se ejecutará antes, dejaremos el valor por defecto.
>
> **Write Bootstrap**: En este apartado indica donde esta el fichero bacula, dicha opción dejamos la que esta por defecto.

Ahora vamos a definir los trabajos de los clientes que vamos a utilizar para realizar la copia de seguridad, tenemos que indicarle `Name` (indicamos el nombre del trabajo), `JobDefs` (Definimos en cada cliente la tarea creada anteriormente, `Tarea-Serranito`) y `Client` (le vamos a indicar los nombre de los equipos)

###### Añadimos los apartado `Job` para realizar copias de seguridad

~~~
Job {
 Name = "Backup-Serranito"
 JobDefs = "Tarea-Serranito"
 Client = "serranito-fd"
}

Job {
 Name = "Backup-Croqueta"
 JobDefs = "Tarea-Serranito"
 Client = "croqueta-fd"
}

Job {
 Name = "Backup-Tortilla"
 JobDefs = "Tarea-Serranito"
 Client = "tortilla-fd"
}

Job {
 Name = "Backup-Salmorejo"
 JobDefs = "Tarea-Serranito"
 Client = "salmorejo-fd"
}
~~~

Si realizamos copias de seguridad es para que, en caso de fallo o descuido, podemos reataurarlas. Para esto vamos a definir de nuevos trabajos de los clientes pero en este caso indicamos que tipo es `Restore`. Además tenemos que indicar el `Name`, `Client`, `FileSet`, `Storage`, `Poll`, `Messages`.

###### Añadimos los apartado `Job` para restaurar las copias de seguridad

~~~
Job {
 Name = "Restore-Serranito"
 Type = Restore
 Client=serranito-fd
 FileSet="CopiaCompleta"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Restore-Croqueta"
 Type = Restore
 Client=croqueta-fd
 FileSet="CopiaCompleta"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Restore-Tortilla"
 Type = Restore
 Client=totilla-fd
 FileSet="CopiaCompleta"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Restore-Salmorejo"
 Type = Restore
 Client=salmorejo-fd
 FileSet="CopiaCompleta"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}
~~~

El siguiente apartado es `FileSet`, donde indicaremos los ficheros y directorios que queremos realizar la copia de seguridad y los que queremos excluir.

###### Añadimos el apartado `FileSet`

~~~
FileSet {
 Name = "CopiaCompleta"
 Include {
    Options {
        signature = MD5
        compression = GZIP
    }
    File = /home
    File = /etc
    File = /var
 }
 Exclude {
    File = /var/lib/bacula
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /var/cache
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }
}
~~~

> **Include**: Apartado en el que se indicaran los ficheros y las opciones que se incluirán al realizar las copias de seguridad.
>
> **Exclude**: Apartado donde se indicaran los ficheros que queremos que no se añadan en las copias de seguridad.
>
> **Options**: Apartado donde indicaremos `signature` (el tipo de encriptación) y `compression` (tipo de almacenamiento de forma comprimida).
>
> **File**: Indicaremos la ruta completa del los ficheros.

Ahora vamos a definir la programación para la realización de las copias de seguridad. Vamos a indicarle `Name`, y los diferentes `Run`, para programar el nivel si es completo o incremental y el día/hora que queremos que se realicen dichas copias.

###### Añadimos el apartado `Schedule`

~~~
Schedule {
 Name = "Programa"
 Run = Full 1st sat at 23:59
 Run = Level=Incremental sat at 23:59
}
~~~

> En el primer `Run` le indicamos que realice una copia completa, `Full`, los primeros Sábados de cada mes a las 23:59.
>
> El segundo `Run` le indicamos que realice una copia incremental, todos los Sábados a las 23:59.

-----------------------------------

> Aquí os dejo como especificar un programa de todas las formas posibles.

~~~
<void-keyword>    = on
<at-keyword>      = at
<week-keyword>    = 1st | 2nd | 3rd | 4th | 5th | first |
                    second | third | fourth | fifth
<wday-keyword>    = sun | mon | tue | wed | thu | fri | sat |
                    sunday | monday | tuesday | wednesday |
                    thursday | friday | saturday
<week-of-year-keyword> = w00 | w01 | ... w52 | w53
<month-keyword>   = jan | feb | mar | apr | may | jun | jul |
                    aug | sep | oct | nov | dec | january |
                    february | ... | december
<daily-keyword>   = daily
<weekly-keyword>  = weekly
<monthly-keyword> = monthly
<hourly-keyword>  = hourly
<digit>           = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0
<number>          = <digit> | <digit><number>
<12hour>          = 0 | 1 | 2 | ... 12
<hour>            = 0 | 1 | 2 | ... 23
<minute>          = 0 | 1 | 2 | ... 59
<day>             = 1 | 2 | ... 31
<time>            = <hour>:<minute> |
                    <12hour>:<minute>am |
                    <12hour>:<minute>pm
<time-spec>       = <at-keyword> <time> |
                    <hourly-keyword>
<date-keyword>    = <void-keyword>  <weekly-keyword>
<day-range>       = <day>-<day>
<month-range>     = <month-keyword>-<month-keyword>
<wday-range>      = <wday-keyword>-<wday-keyword>
<range>           = <day-range> | <month-range> |
                          <wday-range>
<date>            = <date-keyword> | <day> | <range>
<date-spec>       = <date> | <date-spec>
<day-spec>        = <day> | <wday-keyword> |
                    <day> | <wday-range> |
                    <week-keyword> <wday-keyword> |
                    <week-keyword> <wday-range> |
                    <daily-keyword>
<month-spec>      = <month-keyword> | <month-range> |
                    <monthly-keyword>
<date-time-spec>  = <month-spec> <day-spec> <time-spec>
~~~

-----------------------------------

Ahora tenemos que definir los clientes, tenemos que tener tantos como indicamos en el apartado de `Job`.

###### Añadimos los apartados `Client`

~~~
Client {
 Name = serranito-fd
 Address = 10.0.0.17
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "*************"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = croqueta-fd
 Address = 10.0.0.6
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "*************"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = tortilla-fd
 Address = 10.0.0.9
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "*************"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = salmorejo-fd
 Address = 10.0.0.14
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "*************"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}
~~~
> **Catalog**: Esto especifica el nombre del recurso de catálogo que se utilizará para este cliente.
>
> **File Retention**: Si `AutoPrune` esta definido como `Yes`, bacula mantendrá los registros de los ficheros, el tiempo que se le indica en el catalogo.
>
> **Job Retention**: Si `AutoPrune` esta definido como `Yes`, bacula mantendrá los registros de los trabajos, el tiempo que se le indica en el catalogo.
>
> **AutoPrune**: Si se establece en `Yes`, Bacula aplicará automáticamente `File Retention` y `Job Retention` para el cliente al final del trabajo. Si se establece en `No`, no se realizará la eleminación y su Catálogo aumentará de tamaño cada vez que ejecute un Trabajo. (Esto no tiene nada que ver con los volumenes).

Ahora toca definir el apartado de `Storage`, que es donde indicaremos el tipo de almacenamiento donde queremos guardar las copias de seguridad.

###### Añadimos el apartado `Storage`

~~~
Storage {
 Name = Vol-Serranito
 Address = 10.0.0.17
 SDPort = 9103
 Password = "*************"
 Device = FileAutochanger1
 Media Type = File
 Maximum Concurrent Jobs = 10
}
~~~

> **Device**: Es donde se define el nombre del `AutoChanger` y no el nombre del dispositivo físico.
>
> **Media Type**: Esta directiva especifica el tipo de medio que se utilizará para almacenar los datos.

> **Maximun Concurrent Jobs**: Se indica el número máximo de trabajos, con el recurso de almacenamiento actual, que puede ejecutarse simultáneamente.

Definimos el apartado de  `Catalog`, que hace referencia a la base de datos que estamos utilizando, indicandole los parametros necesarios para apuntar a ella.

###### Añadimos el apartado `Catalog`

~~~
Catalog {
 Name = mysql-bacula
 dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "bacula"
}
~~~

Por último, tenemos que definir el apartado `Pool` con el nombre `Vol-Backup` que indicamos anteriormente para indicarle algunas opciones del volumen para las copias de seguridad.

###### Añadimos el apartado `Pool`

~~~
Pool {
 Name = Vol-Backup
 Pool Type = Backup
 Recycle = yes 
 AutoPrune = yes
 Volume Retention = 365 days 
 Maximum Volume Bytes = 50G
 Maximum Volumes = 100
 Label Format = "Remoto"
}
~~~

El resto de apartados `Messages`, `Pool` y `Console` se quedarían por defecto, como muestro a continuación.

###### Añadimos los apartados `Messages`, `Pool` y `Console`

~~~
# Reasonable message delivery -- send most everything to email address
#  and to the console
Messages {
  Name = Standard
#
# NOTE! If you send to two email or more email addresses, you will need
#  to replace the %r in the from field (-f part) with a single valid
#  email address in both the mailcommand and the operatorcommand.
#  What this does is, it sets the email address that emails would display
#  in the FROM field, which is by default the same email as they're being
#  sent to.  However, if you send email to more than one address, then
#  you'll have to set the FROM address manually, to a single address.
#  for example, a 'no-reply@mydomain.com', is better since that tends to
#  tell (most) people that its coming from an automated source.

#
  mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: %t %e of %c %l\" %r"
  operatorcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: Intervention needed for %j\" %r"
  mail = root = all, !skipped
  operator = root = mount
  console = all, !skipped, !saved
#
# WARNING! the following will create a file that you must cycle from
#          time to time as it will grow indefinitely. However, it will
#          also keep all your messages if they scroll off the console.
#
  append = "/var/log/bacula/bacula.log" = all, !skipped
  catalog = all
}


#
# Message delivery for daemon messages (no job).
Messages {
  Name = Daemon
  mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula daemon message\" %r"
  mail = root = all, !skipped
  console = all, !skipped, !saved
  append = "/var/log/bacula/bacula.log" = all, !skipped
}

# Default pool definition
Pool {
  Name = Default
  Pool Type = Backup
  Recycle = yes                       # Bacula can automatically recycle Volumes
  AutoPrune = yes                     # Prune expired volumes
  Volume Retention = 365 days         # one year
  Maximum Volume Bytes = 50G          # Limit Volume size to something reasonable
  Maximum Volumes = 100               # Limit number of Volumes in Pool
}

# Scratch pool definition
Pool {
  Name = Scratch
  Pool Type = Backup
}

#
# Restricted console used by tray-monitor to get the status of the director
#
Console {
  Name = serranito-mon
  Password = "rPNmB3G7UEOX7ztuXzmvjr08SRH5HJCMT"
  CommandACL = status, .status
}
~~~

#### El fichero final quedaría de la siguiente manera. [AQUÍ](https://github.com/MoralG/Copias_de_Seguridad_con_Bacula/blob/master/bacula-dir.conf.md)

Una vez que hayamos terminado de configurar el fichero, guardamos y ejecutamos el siguiente comando para realizar una comprobación, si no devuelve nada, es que todo esta bien.

###### Comprobación del fichero `bacula-dir.conf`

~~~
sudo bacula-dir -tc /etc/bacula/bacula-dir.conf
~~~

## Configuración del volumen externo

Vamos a configurar ahora el volumen donde vamos a guardar todas las copias de seguridad.
Tendremos que crearle una partición, con un sistema de ficheros y configurar el fichero `/etc/fstab` para que monte el volumen al iniciar el sistema.

En primer lugar vamos a ver como es llamo el volumen en el sistema con el comando `lsblk -f`.

~~~
lsblk -f
  NAME   FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
  vda                                                                     
  └─vda1 ext4         6197e068-a892-45cb-9672-a05813e800ee      8G    14% /
  vdb                   
~~~

Como podemos ver, el unico que esta libre es el `vdb`. Sabiendo esto, vamos a utilizar `fdisk` para crear una partición en el volumen completo.

###### Creamos la partición `vdb1`

~~~
sudo fdisk /dev/vdb 

  Welcome to fdisk (util-linux 2.33.1).
  Changes will remain in memory only, until you decide to write them.
  Be careful before using the write command.

  Device does not contain a recognized partition table.
  Created a new DOS disklabel with disk identifier 0xdfddf621.

  Command (m for help): n
  Partition type
     p   primary (0 primary, 0 extended, 4 free)
     e   extended (container for logical partitions)
  Select (default p): p
  Partition number (1-4, default 1): 1
  First sector (2048-20971519, default 2048): 
  Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971519, default 20971519): 

  Created a new partition 1 of type 'Linux' and of size 10 GiB.

  Command (m for help): p
  Disk /dev/vdb: 10 GiB, 10737418240 bytes, 20971520 sectors
  Units: sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disklabel type: dos
  Disk identifier: 0xdfddf621

  Device     Boot Start      End  Sectors Size Id Type
  /dev/vdb1        2048 20971519 20969472  10G 83 Linux

  Command (m for help): w
  The partition table has been altered.
  Calling ioctl() to re-read partition table.
  Syncing disks.
~~~

Creada la partición, tenemos que formatearla con un sistema de ficheros `ext4`.

###### Formateamos la partición `vdb1`

~~~
sudo mkfs.ext4 /dev/vdb1 
  mke2fs 1.44.5 (15-Dec-2018)
  Creating filesystem with 2621184 4k blocks and 655360 inodes
  Filesystem UUID: faed35cd-a6d0-4ff3-842d-fc347ccbd75b
  Superblock backups stored on blocks: 
  	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

  Allocating group tables: done                            
  Writing inode tables: done                            
  Creating journal (16384 blocks): done
  Writing superblocks and filesystem accounting information: done 
~~~

Ahora tenemos el volumen listo para montarlo. Creamos el directorio, le cambiamos los permisos y el propietario,para que sean del bacula, y configuramos el fichero `/etc/fstab`.

###### Creamos el directorio `/bacula/Copias_de_Seguridad`

~~~
sudo mkdir -p /bacula/Copias_de_Seguridad
~~~

###### Cambiamos los permisos a `755` y el propietario a `bacula`

~~~
sudo chown bacula:bacula /bacula -R
sudo chmod 755 /bacula -R
~~~

###### Copiamos el `UUID` para utilizarlo en el fichero `fstab`

~~~
lsblk -f | egrep "vdb1 *" | cut -d" " -f11
  faed35cd-a6d0-4ff3-842d-fc347ccbd75b
~~~

###### Añadimos al fichero `/etc/fstab` la siguiente linea

~~~
UUID=faed35cd-a6d0-4ff3-842d-fc347ccbd75b     /bacula/Copias_de_Seguridad     ext4     defaults     0     0
~~~

###### Con la opción `-a` del comando `mount`, hacemos que el sistema vuelva a leer el fichero 'fstab' sin tener que reiniciar

~~~
sudo mount -a
~~~

###### Comprobamos que se ha montado correctamente

~~~
lsblk -l | egrep "^vdb1 *"
  vdb1 254:17   0  10G  0 part /bacula/Copias_de_Seguridad
~~~