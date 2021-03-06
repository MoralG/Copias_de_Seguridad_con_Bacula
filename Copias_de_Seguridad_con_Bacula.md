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

## Esquema del escenario

![Tarea1.1](image/Tarea1.1_Bacula.png)

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

## Configuración del fichero `bacula-dir.conf`

Ahora vamos a configurar varios ficheros de bacula para definir:

* Tipo de copia de seguridad
* Tipo de tarea a realizar
* Clientes
* Directorios que vamos a copiar
* Directorios que no vamos a copiar

Otros datos a tener en cuenta son los puertos:

* bacula-dir: puerto 9101
* bacula-fd: puerto 9102
* bacula-sd: puerto 9103
* Base de datos MySQL: puerto 3306

El fichero que tenemos que modificar es `/etc/bacula/bacula-dir.conf`. Este fichero es el fichero principal, donde se configura la gran parte de las opciones de bacula.

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
  Password = "************"
  Messages = Daemon
  DirAddress = 10.0.0.17
}
~~~

> **NOTA**: Cambiamos la `Password` y `DirAddress`. Lo demás lo dejamos por defecto.

Ahora en el mismo fichero, tenemos que añadir los apartado s`JobDefs`, donde indicaremos varios campos, como si fueran variables, y las ampliaremos mas adelante. El apartado `JobDefs` agrupa un grupo de `Jobs`, donde los parametros que se apliquen a este, se aplicaran a todos los `Jobs` que esten en el.

Creamos tres `JobDefs`, uno para cada tipo de tarea.
* Copia de seguridad diariamente
* Copia de seguridad semanalmente
* Copia de seguridad mensualmente

###### Añadimos apartado `JobDefs`

~~~
JobDefs {
  Name = "Tarea-Diaria"
  Type = Backup
  Level = Incremental
  Client = serranito-fd
  Schedule = "Programa-Diario"
  Pool = Daily
  Storage = Vol-Serranito
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}


JobDefs {
  Name = "Tarea-Semanal"
  Type = Backup
  Client = serranito-fd
  Schedule = "Programa-Semanal"
  Pool = Weekly
  Storage = Vol-Serranito
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}


JobDefs {
  Name = "Tarea-Mensual"
  Type = Backup
  Client = serranito-fd
  Schedule = "Programa-Mensual"
  Pool = Monthly
  Storage = Vol-Serranito
  Messages = Standard
  SpoolAttributes = yes
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}
~~~

> **Name**: Nombre que le asignamos a la tarea `JobDefs`.
> 
> **Type**: Indicamos que el tipo de tarea son copias de seguridad, `Backup`.
>
> **Level**: Indicamos como queremos realizar las copias de seguridad, en nuestro caso `Incremental`, ya que es una copia de los datos creados y modificados desde la última ejecución de la copia de seguridad.
>
> **Client**: El nombre del cliente en el que se va a ejecutar esta tarea.
>
> **Schedule**: Indicamos el nombre que vamos a utilizar para extender cada campo, donde indicaremos la fecha y la fecha de realización de las copias de seguridad.
>
> **Storage**: Indicamos el nombre del apartado `Vol-Serranito`, el cual se extenderá mas adelante e indicaremos el tipo de almacenamiento vamos a utilizar.
>
> **Messages**: Tipo de mensaje que se dejará en `Standar` que es la opción por defecto, el cual indicará como mandará los mensajes de sucesos de bacula.
>
> **Pool**: Indicaremos el nombre del apartado `pool` que se configurará mas adelante y en él indicaremos el volumen de almacenamiento donde se creará y almacenará las copias, además de las retenciones de estas.
>
> **SpoolAttributes**: Esta opción permite trabajar con los atributos del spool en un fichero temporal, así que dejaremos la opcion por defecto.
>
> **Priority**: Indica el nivel de prioridad en el cual si ponemos un nùmero menos, quiere decir que tiene más prioridad e indicará cual se ejecutará antes, dejaremos el valor por defecto.
>
> **Write Bootstrap**: En este apartado indica donde esta el fichero bacula, dicha opción dejamos la que esta por defecto.

Ahora vamos a definir los trabajos de los clientes que vamos a utilizar para realizar la copia de seguridad, para esto utilizamos los apartados `Job`. 

Tenemos que indicarle `Name` (indicamos el nombre del trabajo), `JobDefs` (Definimos en cada cliente la tarea creada anteriormente para cada tipo de `Job` y `Client` (le vamos a indicar los nombre de los equipos)

También le indicamos el `FileSet`, el cual, expandiremos el apartado mas adelante y donde indicaremos los ficheros que queremos que se realicen la copia.

Vamos a crear un `Job` para cada cliente y para cada fecha de la realización de la copia.

###### Añadimos los apartado `Job` para realizar copias de seguridad

##### Diariamente
~~~
Job {
 Name = "Daily-Backup-Serranito"
 JobDefs = "Tarea-Diaria"
 Client = "serranito-fd"
 FileSet= "Copia-Serranito"
}

Job {
 Name = "Daily-Backup-Croqueta"
 JobDefs = "Tarea-Diaria"
 Client = "croqueta-fd"
 FileSet= "Copia-Croqueta"
}

Job {
 Name = "Daily-Backup-Tortilla"
 JobDefs = "Tarea-Diaria"
 Client = "tortilla-fd"
 FileSet= "Copia-Tortilla"
}

Job {
 Name = "Daily-Backup-Salmorejo"
 JobDefs = "Tarea-Diaria"
 Client = "salmorejo-fd"
 FileSet= "Copia-Salmorejo"
}
~~~

##### Semanalmente
~~~
Job {
 Name = "Weekly-Backup-Serranito"
 JobDefs = "Tarea-Semanal"
 Client = "serranito-fd"
 FileSet= "Copia-Serranito"
}

Job {
 Name = "Weekly-Backup-Croqueta"
 JobDefs = "Tarea-Semanal"
 Client = "croqueta-fd"
 FileSet= "Copia-Croqueta"
}

Job {
 Name = "Weekly-Backup-Tortilla"
 JobDefs = "Tarea-Semanal"
 Client = "tortilla-fd"
 FileSet= "Copia-Tortilla"
}

Job {
 Name = "Weekly-Backup-Salmorejo"
 JobDefs = "Tarea-Semanal"
 Client = "salmorejo-fd"
 FileSet= "Copia-Salmorejo"
}
~~~

##### Mensualmente
~~~
Job {
 Name = "Monthly-Backup-Serranito"
 JobDefs = "Tarea-Mensual"
 Client = "serranito-fd"
 FileSet= "Copia-Serranito"
}

Job {
 Name = "Monthly-Backup-Croqueta"
 JobDefs = "Tarea-Mensual"
 Client = "croqueta-fd"
 FileSet= "Copia-Croqueta"
}

Job {
 Name = "Monthly-Backup-Tortilla"
 JobDefs = "Tarea-Mensual"
 Client = "tortilla-fd"
 FileSet= "Copia-Tortilla"
}

Job {
 Name = "Monthly-Backup-Salmorejo"
 JobDefs = "Tarea-Mensual"
 Client = "salmorejo-fd"
 FileSet= "Copia-Salmorejo"
}
~~~

Si realizamos copias de seguridad es para que, en caso de fallo o descuido, podemos reataurarlas. Para esto vamos a definir, de nuevo, `Job` para clientes pero en este caso indicamos que el tipo es `Restore`. Además tenemos que indicar el `Name`, `Client`, `FileSet`, `Storage`, `Poll`, `Messages`.

###### Añadimos los apartado `Job` para restaurar las copias de seguridad

~~~
Job {
 Name = "Restore-Serranito"
 Type = Restore
 Client=serranito-fd
 FileSet= "Copia-Serranito"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Restore-Croqueta"
 Type = Restore
 Client=croqueta-fd
 FileSet= "Copia-Croqueta"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Restore-Tortilla"
 Type = Restore
 Client=totilla-fd
 FileSet= "Copia-Tortilla"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}

Job {
 Name = "Restore-Salmorejo"
 Type = Restore
 Client=salmorejo-fd
 FileSet= "Copia-Salmorejo"
 Storage = Vol-Serranito
 Pool = Vol-Backup
 Messages = Standard
}
~~~

Los siguientes apartados son los `FileSet`, donde indicaremos los ficheros y directorios que queremos realizar la copia de seguridad y los que queremos excluir.

###### Añadimos el apartado `FileSet`

~~~
FileSet {
 Name = "Copia-Serranito"
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
    File = /nonexistant/path/to/file/archive/dir
    File = /proc
    File = /var/cache
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }

FileSet {
 Name = "Copia-Croqueta"
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
    File = /var/tmp
    File = /tmp
    File = /sys
    File = /.journal
    File = /.fsck
 }
}

FileSet {
 Name = "Copia-Tortilla"
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

FileSet {
 Name = "Copia-Salmorejo"
 Include {
    Options {
        signature = MD5
        compression = GZIP
    }
    File = /home
    File = /etc
    File = /var
    File = /usr/share/nginx
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
 Name = "Programa-Diario"
 Run = Level=Incremental Pool=Daily daily at 23:05
}

Schedule {
 Name = "Programa-Semanal"
 Run = Level=Full Pool=Weekly sat at 23:50
}

Schedule {
 Name = "Programa-Mensual"
 Run = Level=Full Pool=Monthly 1st sun at 23:50 
}
~~~

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
 Password = "**********"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = croqueta-fd
 Address = 10.0.0.6
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "**********"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = tortilla-fd
 Address = 10.0.0.9
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "**********"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = salmorejo-fd
 Address = 10.0.0.14
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "**********"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}
~~~
> **Catalog**: Esto especifica el nombre del recurso de catálogo que se utilizará para este cliente.
>
> **File Retention**: Si `AutoPrune` esta definido como `Yes`, bacula mantendrá los registros de los ficheros en el catalogo.
>
> **Job Retention**: Si `AutoPrune` esta definido como `Yes`, bacula mantendrá los registros de los trabajo en el catalogo.
>
> **AutoPrune**: Si se establece en `Yes`, Bacula aplicará automáticamente `File Retention` y `Job Retention` para el cliente al final del trabajo. Si se establece en `No`, no se realizará la eleminación y su Catálogo aumentará de tamaño cada vez que ejecute un Trabajo. (Esto no tiene nada que ver con los volumenes).

Ahora toca definir el apartado de `Storage`, que es donde indicaremos el tipo de almacenamiento donde queremos guardar las copias de seguridad.

###### Añadimos el apartado `Storage`

~~~
Storage {
 Name = Vol-Serranito
 Address = 10.0.0.17
 SDPort = 9103
 Password = "**********"
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

Por último, tenemos que definir los apartados de `Pool` que indicamos anteriormente para indicarle algunas opciones del volumen para las copias de seguridad e indicarle las retenciones para cada uno de los tipos de programación indicados anteriormente.

###### Añadimos los apartados `Pool`

~~~
Pool {
 Name = Daily
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 10d
 Volume Use Duration = 5d
 Recycle = yes
}


Pool {
 Name = Weekly
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 30d
 Volume Use Duration = 5d
 Recycle = yes
}


Pool {
 Name = Monthly
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 365d
 Volume Use Duration = 5d
 Recycle = yes
}


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

> **Use Volume Once**: Tenemos que indicar `yes` para indicar bacula que el volumen esta completo, esto hará que empiece a contar la retención en el momento que se haga el priemr `job`. Porque por defecto bacula no activa la retención de ficheros hasta que el volumen este completo.
>
> **VolumeRetention**: Indicamos el tiempo que va a guardarse las copias de seguridad en el volumen.
>
> **Recycle**: Este campo es para que realice un purgue después de que termine el tiempo de retención.

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

Como podemos ver, el unico que esta libre es el `vdb`.

###### Formateamos la partición `vdb`

~~~
sudo mkfs.ext4 /dev/vdb
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
lsblk -f | egrep "vdb *" | cut -d" " -f11
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
lsblk -l
  NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
  vda  254:0    0  10G  0 disk 
  vda1 254:1    0  10G  0 part /
  vdb  254:16   0  10G  0 disk /bacula/Copias_de_Seguridad
~~~

## Configuración del fichero `bacula-sd.conf`

Con este fichero, que se encuentra en `/etc/bacula/bacula-sd.conf`, vamos a modificar la configuración del `Director`, el cual, lo indicamos en el fichero `bacula-dir.conf`. Este se encargará de realizar las copias en el volumen configurado en el anterior punto.

Indicamos el primer apartado, que es `Storage`, y sus campos serán parecidos a los campos del `Director` pero con el nombre diferente, como vemos a continuación:

###### Configuramos el apartado `Storage`

~~~
Storage { 
 Name = serranito-sd
 SDPort = 9103 
 WorkingDirectory = "/var/lib/bacula"
 Pid Directory = "/run/bacula"
 Maximum Concurrent Jobs = 20
 SDAddress = 10.0.0.17
}
~~~

> **NAME**: En este caso, le indicamos `-sd`, por defecto bacula con esa coletilla hace referencia al fichero donde se encentra.

Ahora vamos a indicar a que director hace referencia este fichero, para esto, tenemos que indicar el mismo `Name` y `Password` que en el fichero `bacula-dir.conf`.

###### Configuramos el apartado `Director` de referencia

~~~
Director {
 Name = serranito-dir
 Password = "**********"
}
~~~

Ahora indicamos de nuevo el `Director` pero esta vez es para otorgar privilegios sobre el estado del proceso de almacenamiento de las copias de seguridad. Para realizar esto, tenemos que indicarle la coletilla `-mon`.

###### Configuramos el apartado `Director` para poder ver el estado del proceso

~~~
Director {
 Name = serranito-mon
 Password = "bacula"
 Monitor = yes
}
~~~

Vamos a configurar el cargador automático virtual del volumen montado en el punto anterior, para realizar esto, tenemos que indicar el `Name` del campo `Device` del apartado `Storage` del fichero `bacula-dir.conf`.

###### Configuramos el apartado `Autochanger` para configurar un nuevo `Device`

~~~
Autochanger {
 Name = FileAutochanger1
 Device = DispositivoCopia
 Changer Command = ""
 Changer Device = /dev/null
}
~~~

Y por último configuramos un `Device` con el mismo nombre que en apartado anterior con la confuración necesaría para el volumen de copias de seguridad.

###### Configuramos el apartado `Device`

~~~
Device {
 Name = DispositivoCopia
 Media Type = File
 Archive Device = /bacula/Copias_de_Seguridad
 LabelMedia = yes;
 Random Access = Yes;
 AutomaticMount = yes;
 RemovableMedia = no;
 AlwaysOpen = no;
 Maximum Concurrent Jobs = 5
}
~~~

Dejamos el apartado `Messages` por defecto.

~~~
Messages {
  Name = Standard
  director = serranito-dir = all
}
~~~

#### El fichero final quedaría de la siguiente manera. [AQUÍ](https://github.com/MoralG/Copias_de_Seguridad_con_Bacula/blob/master/bacula-sd.conf.md)

Una vez que hayamos terminado de configurar el fichero, guardamos y ejecutamos el siguiente comando para realizar una comprobación, si no devuelve nada, es que todo esta bien.

###### Comprobación del fichero `bacula-sd.conf`

~~~
sudo bacula-sd -tc /etc/bacula/bacula-sd.conf
~~~

También tenemos que reiniciar los servicios de los dos ficheros modificados.

###### Reiniciamos los servicios

~~~
sudo systemctl restart bacula-sd.service
sudo systemctl restart bacula-director.service
~~~

###### Comprobamos que se ha iniciado correctamente
~~~
debian@serranito:~$ sudo systemctl status bacula-sd.service
  ● bacula-sd.service - Bacula Storage Daemon service
     Loaded: loaded (/lib/systemd/system/bacula-sd.service; enabled; vendor preset:   enabled)
     Active: active (running) since Thu 2020-01-09 10:55:25 UTC; 10s ago
       Docs: man:bacula-sd(8)
    Process: 12143 ExecStartPre=/usr/sbin/bacula-sd -t -c $CONFIG (code=exited,   status=0/SUCCESS)
   Main PID: 12148 (bacula-sd)
      Tasks: 2 (limit: 1168)
     Memory: 1.0M
     CGroup: /system.slice/bacula-sd.service
             └─12148 /usr/sbin/bacula-sd -fP -c /etc/bacula/bacula-sd.conf

  Jan 09 10:55:25 serranito systemd[1]: Starting Bacula Storage Daemon service...
  Jan 09 10:55:25 serranito systemd[1]: Started Bacula Storage Daemon service.

debian@serranito:~$ sudo systemctl status bacula-director.service
  ● bacula-director.service - Bacula Director Daemon service
     Loaded: loaded (/lib/systemd/system/bacula-director.service; enabled; vendor   preset: enabled)
     Active: active (running) since Thu 2020-01-09 10:55:28 UTC; 19s ago
       Docs: man:bacula-dir(8)
    Process: 12155 ExecStartPre=/usr/sbin/bacula-dir -t -c $CONFIG (code=exited,  status=0/SUCCESS)
   Main PID: 12157 (bacula-dir)
      Tasks: 3 (limit: 1168)
     Memory: 1.5M
     CGroup: /system.slice/bacula-director.service
             └─12157 /usr/sbin/bacula-dir -fP -c /etc/bacula/bacula-dir.conf

  Jan 09 10:55:28 serranito systemd[1]: Starting Bacula Director Daemon service...
  Jan 09 10:55:28 serranito systemd[1]: Started Bacula Director Daemon service.
~~~

## Configuración del fichero `bconsole.conf`

Tenemos que configurar el fichero `/etc/bacula/bconsole.conf` para poder configurar y gestionar bacula desde la consola de comandos. Para activar esta opción tenemos que añadir un apartado `Director`.

###### Añadimos el apartado `Director`

~~~
Director {
 Name = serranito-dir
 DIRport = 9101
 address = 10.0.0.17
 Password = "**********"
}
~~~

## Creamos un programa usando crontab para guardar el listado de paquetes

Vamos a crear un script, el cual queremos ejecutarlo todos los días a las 22:55 para que se ejecute antes de la copia diaria y que meta la lista de paquetes instalados en cada máquina.

> **NOTA**: El script en Tortilla, Croqueta y Serranito, va a ser diferente respecto a Salmorejo, ya que en centos se lista de diferente manera los paquetes.

El script lo vamos a guardar en `/var/script` y el fichero con el listado en `/var/local`.

* En Serranito, Croqueta, Tortilla, Salmorejo

~~~
sudo touch /var/local/paquetes.txt
sudo mkdir /var/script
sudo nano /var/script/ListaDePaquetes.sh
~~~

* En Serranito, Croqueta, Tortilla

Añadimos lo siguiente al fichero `ListaDePaquetes.sh`.
~~~
#!/bin/bash

sudo dpkg --get-selections > /var/local/paquetes.txt
~~~

* En Salmorejo

Añadimos lo siguiente al fichero `ListaDePaquetes.sh`.
~~~
#!/bin/bash

sudo dnf repoquery --qf '%{name}' --userinstalled  > /var/local/paquetes.txt
~~~

Le cambiamos los permisos al script para que se pueda ejecuar.
~~~
sudo chown root:root /var/script/ListaDePaquetes.sh 
sudo chmod 744 /var/script/ListaDePaquetes.sh 
~~~

Ahora vamos a crear el cron. Las tareas cron siguen una determinada sintaxis. Tienen 5 asteriscos seguidos del comando a ejecutar.

**Ejemplo:**

~~~
* * * * * <script>
~~~

Los 5 asteriscos representan, de izquierda a derecha:

* Minutos: de 0 a 59.
* Horas: de 0 a 23.
* Día del mes: de 1 a 31.
* Mes: de 1 a 12.
* Día de la semana: de 0 a 6, siendo 0 el domingo.

Si se deja un asterisco, quiere decir "cada" minuto, hora, día de mes, mes o día de la semana

Sabíendo esto, vamos a crear un cron con el comando `crontab -e`, seleccionaremos el editor que más nos guste y escribiremos el registro `55 22 * * * /var/script/ListaDePaquetes.sh`.
~~~
sudo crontab -e
no crontab for root - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /usr/bin/joe
  2. /usr/bin/jstar
  3. /usr/bin/jpico
  4. /usr/bin/jmacs
  5. /bin/nano        <---- easiest
  6. /usr/bin/vim.basic
  7. /usr/bin/rjoe
  8. /usr/bin/vim.tiny

Choose 1-8 [5]: 5
crontab: installing new crontab
~~~

Podemos comprobar que esta disponible con:
~~~
sudo crontab -l
  55 22 * * * /var/script/ListaDePaquetes.sh
~~~


Solo quedaría reiniciar el servicio de cron.

* En Serranito, Croqueta, Tortilla
~~~
sudo systemctl restart cron.service
~~~

* En Salmorejo
~~~
sudo systemctl restart crond
~~~

Si queremos restaurar una máquina en un futuro por un fallo, tendremos que restaurar la copia, en la cual se encontrará el fichero `paquetes.txt`.

* En Serranito, Croqueta y Tortilla 
~~~
sudo apt install dselect
sudo dpkg --set-selections < paquetes.txt
sudo dselect
~~~

* En Sallmorejo

~~~
< paquetes.txt xargs dnf -y install
~~~

Y se nos instalarían todos los paquetes que teniamos antes del fallo.

## Creamos un programa para guardar la base de datos de bacula.

Tenemos que añadir la siguiente linea, ya que si se produce un error en la máquina Serranito y la base de datos fallara, tendríamos las copias de seguridad pero no podriamos restaurarlas porque no tenemos los datos de la base de datos para realizar esa restauración.

Añadimos al script anterior la sigueinte linea para que realice la copia de la base de datos.

~~~
sudo mysqldump --user=bacula --password=******** bacula > /bacula/Copias_de_Seguridad/copia_seguridad.sql
~~~

## Configuración de los clientes Serranito, Croqueta, Tortilla y Salmorejo en sus respectivos ficheros `bacula-fd.conf`

La configuración de los clientes se realizan de la misma forma en los 4 clientes, con leves cambios en cada uno de ellos.

Mostraré como es la instalación del paquete necesario en cada uno de los clientes y el fichero que hay que modificar. Iré mostrando los cambios diferenciados en cada uno de los clientes añadiendo al principio de cada tarea el cliente al que se corresponde la configuración.

--------------

Empezaremos instalando el cliente de bacula en los diferente clientes. Al distros distintas de Linux, los paquetes y los paquete de instalación son diferentes.

###### Instalación del cliente de Bacula en:

* ##### Serranito (Debian) - Croqueta (Debian) - Tortilla (Ubuntu)

~~~
sudo apt install bacula-client
~~~

* ##### Salmorejo (Centos)

~~~
sudo yum install bacula-client
~~~

Ahora modificaremos el fichero `bacula-fd.conf` en la ruta `/etc/bacula/bacula-fd.conf` que es igual en todos los cliente.

En este fichero añadiremos la configuración de cada cliente, indicandole el `Director` creado en el servidor Serranito y el del acceso a la configuración por comandos. También tenemos que añadir el apartado `FileDaemon` para indicar los datos necesarios del demonio del cliente y por último el `Messages`.

###### Modificamos los ficheros `/etc/bacula/bacula-fd.conf` en:

* ##### Serranito (Debian) - Croqueta (Debian) - Tortilla (Ubuntu)


> **NOTA**: Las diferencias entre los clientes son las que van entre `< >`.

~~~
Director {
 Name = serranito-dir
 Password = "**********"
}

Director {
 Name = serranito-mon
 Password = "**********"
 Monitor = yes
}

FileDaemon {
 Name = <Nombre_Cliente>-fd
 FDport = 9102
 WorkingDirectory = /var/lib/bacula
 Pid Directory = /run/bacula
 Maximum Concurrent Jobs = 20
 Plugin Directory = /usr/lib/bacula
 FDAddress = <Dirección_IP_Cliente>
}

Messages {
 Name = Standard
 director = serranito-dir = all, !skipped, !restored
}
~~~

* ##### Salmorejo (Centos)

~~~
Director {
 Name = serranito-dir
 Password = "**********"
}

Director {
 Name = serranito-mon
 Password = "**********"
 Monitor = yes
}

FileDaemon {
  Name = salmorejo-fd
  FDport = 9102
  WorkingDirectory = /var/spool/bacula
  Pid Directory = /var/run
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib64/bacula
}

Messages {
 Name = Standard
 director = serranito-dir = all, !skipped, !restored
}
~~~

Guardamos los ficheros y reiniciamos el servicio del fichero `bacula-fd.conf` en todos los clientes.

###### Reiniciamos el servicio en:

* ##### Serranito (Debian) - Croqueta (Debian) - Tortilla (Ubuntu) - Salmorejo (Centos)

~~~
sudo systemctl restart bacula-fd.service
~~~

Ahora tenemos que reiniciar los servicios en el Servidor, del fichero `bacula-sd.conf` y del fichero `bacula-dir.conf` para que reconozca los clientes que hemos configurado.

###### Reiniciamos los servicios en:

* ##### Serranito (Debian)

~~~
sudo systemctl restart bacula-sd.service
sudo systemctl restart bacula-director.service
~~~

Tenemos que abrir los puertos, en mi caso al tener las máquinas en Openstack tengo que crear una regla de seguridad y abrir el puerto 9102/TCP, y en el cliente Salmorejo (Centos) tenemos que abrir el puerto 9102/TCP también con el `firewall-cmd`.

###### Abrir puerto en los clientes Croqueta, Tortilla y Salmorejo

![Tarea1.2](image/Tarea1.2_Bacula.png)

###### Abrir el puerto en el cliente Salmorejo

~~~
firewall-cmd --zone=public --permanent --add-port 9102/tcp
firewall-cmd --reload
~~~

Cuando tengamos reinicado todos los servicios en los clientes y en el servidor y los puertos abiertos, podemos comprobar que todo ha salido bien, listando los clientes, desde la consola de comando de bacula y viendo el estado.

##### Comprobamos la conexión

###### Iniciamos la consola de bacula
~~~
sudo bconsole
  Connecting to Director 10.0.0.17:9101
  1000 OK: 103 serranito-dir Version: 9.4.2 (04 February 2019)
  Enter a period to cancel a command.
  *
~~~

Ahora metemos los comando para listar los clientes y ver sus estados, cuando nos salga el `*`

###### Cliente Serranito

~~~
*status client
  The defined Client resources are:
       1: serranito-fd
       2: croqueta-fd
       3: tortilla-fd
       4: salmorejo-fd
  Select Client (File daemon) resource (1-4): 1
  Connecting to Client serranito-fd at 10.0.0.17:9102

  serranito-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu debian buster/sid
  Daemon started 09-Jan-20 18:32. Jobs: run=0 running=0.
   Heap: heap=114,688 smbytes=22,012 max_bytes=22,029 bufs=68 max_bufs=68
   Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
   Plugin: bpipe-fd.so 

  Running Jobs:
  Director connected at: 09-Jan-20 19:15
  No Jobs running.
  ====
~~~

###### Cliente Croqueta

~~~
*status client
  The defined Client resources are:
       1: serranito-fd
       2: croqueta-fd
       3: tortilla-fd
       4: salmorejo-fd
  Select Client (File daemon) resource (1-4): 2
  Connecting to Client croqueta-fd at 10.0.0.6:9102

  croqueta-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu debian buster/sid
  Daemon started 09-Jan-20 18:33. Jobs: run=0 running=0.
   Heap: heap=114,688 smbytes=22,010 max_bytes=22,027 bufs=68 max_bufs=68
   Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
   Plugin: bpipe-fd.so 

  Running Jobs:
  Director connected at: 09-Jan-20 19:16
  No Jobs running.
  ====
~~~

###### Cliente Tortilla

~~~
*status client
  The defined Client resources are:
       1: serranito-fd
       2: croqueta-fd
       3: tortilla-fd
       4: salmorejo-fd
  Select Client (File daemon) resource (1-4): 3
  Connecting to Client tortilla-fd at 10.0.0.9:9102

  tortilla-fd Version: 9.0.6 (20 November 2017) x86_64-pc-linux-gnu ubuntu 18.04
  Daemon started 09-Jan-20 18:34. Jobs: run=0 running=0.
   Heap: heap=110,592 smbytes=21,981 max_bytes=21,998 bufs=68 max_bufs=68
   Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
   Plugin: bpipe-fd.so 

  Running Jobs:
  Director connected at: 09-Jan-20 19:16
  No Jobs running.
  ====

  Terminated Jobs:
  ====
~~~

###### Cliente Salmorejo

~~~
*status client
  The defined Client resources are:
       1: serranito-fd
       2: croqueta-fd
       3: tortilla-fd
       4: salmorejo-fd
  Select Client (File daemon) resource (1-4): 4
  Connecting to Client salmorejo-fd at 10.0.0.14:9102

  salmorejo-fd Version: 9.0.6 (20 November 2017) x86_64-redhat-linux-gnu redhat (Core)
  Daemon started 09-Jan-20 18:34. Jobs: run=0 running=0.
   Heap: heap=102,400 smbytes=21,984 max_bytes=22,001 bufs=68 max_bufs=68
   Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
   Plugin: bpipe-fd.so 

  Running Jobs:
  Director connected at: 09-Jan-20 19:16
  No Jobs running.
  ====
~~~

## Gestión de Bacula desde la consola

Estamos llegando al final, como último punto antes de realizar las copias de seguridad y de restaurarlas, vamos a ponerle un nombre a nuestro volumen e indicarle el `Pool` que queremos que se le adjunte.

Para realizar esto vamos a iniciar la consola de Bacula y ejecutar el comando `label`, esto hará que solo reconozca nuestro `Catalog` y nuestro `Storage`, luego nombraremos a nuestro volumen y por último indicaremos el `Pool` que queremos utilizar. Esto lo hacemos por cada `Pool`, es decir, para los pool `Daily`, `Monthly` y `Weekly`.

###### Iniciamos la consola de Bacula e introducimos el comando `label`

~~~
*label
  Automatically selected Catalog: mysql-bacula
  Using Catalog "mysql-bacula"
  Automatically selected Storage: Vol-Serranito
  
  Enter new Volume name: Backup-Daily
  
  Defined Pools:
       1: Daily
       2: Default
       3: File
       4: Monthly
       5: Scratch
       6: Vol-Backup
       7: Weekly
  Select the Pool (1-7): 1
  
    Connecting to Storage daemon Vol-Serranito at 10.0.0.17:9103 ...
    Sending label command for Volume "backups" Slot 0 ...
    3000 OK label. VolBytes=226 VolABytes=0 VolType=1 Volume="backups"    Device="DispositivoCopia" (/bacula/Copias_de_Seguridad)
    Catalog record for Volume "backups", Slot 0  successfully created.
    Requesting to mount FileAutochanger1 ...
    3906 File device ""DispositivoCopia" (/bacula/Copias_de_Seguridad)" is always     mounted.
~~~

Este proceso lo repitimos para `Weekly` y `Monthly`. Podemos comprobar que se han creado correctamente ejecutando `list volume`.

~~~
*list volume
  Pool: Daily
  +---------+-------------+-----------+---------+---------------+----------+--------------+---------  +------+-----------+-----------+---------+----------+---------------------+-----------+
  | MediaId | VolumeName  | VolStatus | Enabled | VolBytes      | VolFiles | VolRetention | Recycle |   Slot | InChanger | MediaType | VolType | VolParts | LastWritten         | ExpiresIn |
  +---------+-------------+-----------+---------+---------------+----------+--------------+---------  +------+-----------+-----------+---------+----------+---------------------+-----------+
  |      25 | Backups-Daily | Append    |       1 | 2,690,083,786 |        0 |      864,000 |       1   |    0 |         0 | File      |       1 |        0 | 2020-02-11 23:05:25 |   824,480 |
  +---------+-------------+-----------+---------+---------------+----------+--------------+---------  +------+-----------+-----------+---------+----------+---------------------+-----------+
  
  Pool: Default
  No results to list.
  
  Pool: File
  No results to list.
  
  Pool: Monthly
  +---------+----------------+-----------+---------+-------------+----------+--------------+--------- +------+-----------+-----------+---------+----------+---------------------+------------+
  | MediaId | VolumeName     | VolStatus | Enabled | VolBytes    | VolFiles | VolRetention | Recycle |  Slot | InChanger | MediaType | VolType | VolParts | LastWritten         | ExpiresIn  |
  +---------+----------------+-----------+---------+-------------+----------+--------------+--------- +------+-----------+-----------+---------+----------+---------------------+------------+
  |      26 | Backup-Monthly | Append    |       1 | 569,965,264 |        0 |   31,536,000 |       1 |      0 |         0 | File      |       1 |        0 | 2020-02-02 23:52:46 | 30,721,721 |
  +---------+----------------+-----------+---------+-------------+----------+--------------+--------- +------+-----------+-----------+---------+----------+---------------------+------------+
  
  Pool: Scratch
  No results to list.
  
  Pool: Vol-Backup
  No results to list.
  
  Pool: Weekly
  +---------+---------------+-----------+---------+---------------+----------+--------------+---------  +------+-----------+-----------+---------+----------+---------------------+-----------+
  | MediaId | VolumeName    | VolStatus | Enabled | VolBytes      | VolFiles | VolRetention | Recycle |   Slot | InChanger | MediaType | VolType | VolParts | LastWritten         | ExpiresIn |
  +---------+---------------+-----------+---------+---------------+----------+--------------+---------  +------+-----------+-----------+---------+----------+---------------------+-----------+
  |      27 | Backup-Weekly | Append    |       1 | 2,350,966,937 |        0 |    2,592,000 |       1   |    0 |         0 | File      |       1 |        0 | 2020-02-08 23:52:49 | 2,296,124 |
  +---------+---------------+-----------+---------+---------------+----------+--------------+---------  +------+-----------+-----------+---------+----------+---------------------+-----------+
~~~


Ya tenemos todo configurado para que se realicen la copias de seguridad programadas, pero para no tener que esperar para probar si funciona o no, vamos a hacer que las haga de forma manual.

Iniciamos la consola de bacula y con el comando `run` nos muestra los `Jobs` que hay para activar.

~~~
*run
  A job name must be specified.
  The defined Job resources are:
       1: Backup-Serranito
       2: Backup-Croqueta
       3: Backup-Tortilla
       4: Backup-Salmorejo
       5: Restore-Serranito
       6: Restore-Croqueta
       7: Restore-Tortilla
       8: Restore-Salmorejo

  Select Job resource (1-8): 2
    Run Backup job
    JobName:  Backup-Croqueta
    Level:    Incremental
    Client:   croqueta-fd
    FileSet:  CopiaCompleta
    Pool:     Vol-Backup (From Job resource)
    Storage:  Vol-Serranito (From Job resource)
    When:     2020-01-11 17:05:53
    Priority: 10

    OK to run? (yes/mod/no): yes
    Job queued. JobId=6
~~~

Nos dice que ha iniciado el `job` llamado `Backup-Croqueta`, con esto, podemos ejecutar el comando `status` e indicarle que nos muestre el estado de los clientes y luego indicarle el de Croqueta.

~~~
*status
  Status available for:
       1: Director
       2: Storage
       3: Client
       4: Scheduled
       5: Network
       6: All

  Select daemon type for status (1-6): 3
    The defined Client resources are:
         1: serranito-fd
         2: croqueta-fd
         3: tortilla-fd
         4: salmorejo-fd

  Select Client (File daemon) resource (1-4): 2
    Connecting to Client croqueta-fd at 10.0.0.6:9102

    croqueta-fd Version: 9.4.2 (04 February 2019)  x86_64-pc-linux-gnu debian buster/ sid
    Daemon started 09-Jan-20 18:33. Jobs: run=2 running=0.
     Heap: heap=126,976 smbytes=255,814 max_bytes=468,947 bufs=97 max_bufs=155
     Sizes: boffset_t=8 size_t=8 debug=0 trace=0 mode=0,0 bwlimit=0kB/s
     Plugin: bpipe-fd.so 

    Running Jobs:
    Director connected at: 11-Jan-20 17:18
    No Jobs running.
    ====

    Terminated Jobs:
     JobId  Level    Files      Bytes   Status   Finished        Name 
    ===================================================================
         6  Full      4,556    37.09 M  OK       11-Jan-20 17:06 Backup-Croqueta
    ====
~~~

Como podemos ver, la copia se ha realizado correctamente. A partir de aquí, podemos ver los estados de los demas cliente, realizar restauraciones, etc.

> **NOTA**: Para saber que significan todos los status, puedes echar un vistazo [AQUÍ](https://www.baculasystems.com/dl/trial_experience/manual//html/main/Job_status.html)