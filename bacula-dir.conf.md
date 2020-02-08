
```shell
# Indicar quien es el director

Director {
  Name = serranito-dir
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "**************"
  Messages = Daemon
  DirAddress = 10.0.0.17
}

# Indicar la tarea y sus opciones

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


# Definir los trabajos de los clientes que vamos a utilizar para realizar la copia de seguridad

# Diariamente

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

# Semanalmente

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

# Mensualmente

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

# Definir los trabajos de los clientes que vamos a utilizar para restaurar la copia de seguridad

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
 Client=tortilla-fd
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

# Indicar la ruta y configuración de los ficheros de la copia de seguridad

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
    File = /bacula
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

# Indicar la programación

Schedule {
 Name = "Programa-Diario"
 Run = Level=Incremental Pool=Daily daily at 20:59
}

Schedule {
 Name = "Programa-Semanal"
 Run = Level=Full Pool=Weekly sat at 23:50
}

Schedule {
 Name = "Programa-Mensual"
 Run = Level=Full Pool=Monthly 1st sun at 23:50 
}

# Indicar los cliente

Client {
 Name = serranito-fd
 Address = 10.0.0.17
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "**************"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = croqueta-fd
 Address = 10.0.0.6
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "**************"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = tortilla-fd
 Address = 10.0.0.9
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "**************"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = salmorejo-fd
 Address = 10.0.0.14
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "**************"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}



Storage {
 Name = Vol-Serranito
 Address = 10.0.0.17
 SDPort = 9103
 Password = "**************"
 Device = FileAutochanger1
 Media Type = File
 Maximum Concurrent Jobs = 10
}

# Indicar la base de datos

Catalog {
 Name = mysql-bacula
 dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "**************"
}


Pool {
 Name = Daily
 Use Volume Once = yes
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 10d
 Recycle = yes
}


Pool {
 Name = Weekly
 Use Volume Once = yes
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 30d
 Recycle = yes
}


Pool {
 Name = Monthly
 Use Volume Once = yes
 Pool Type = Backup
 AutoPrune = yes
 VolumeRetention = 365d
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


# Por defecto
# -------------------------------------------------------

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

```