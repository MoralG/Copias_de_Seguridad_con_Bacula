``` shell
# Indicar quien es el director

Director {
  Name = serranito-dir
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "MoralG630789"
  Messages = Daemon
  DirAddress = 10.0.0.17
}

# Indicar la tarea y sus opciones

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

# Definir los trabajos de los clientes que vamos a utilizar para realizar la copia de seguridad

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

# Definir los trabajos de los clientes que vamos a utilizar para restaurar la copia de seguridad

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
 Client=tortilla-fd
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

# Indicar la ruta y configuración de los ficheros de la copia de seguridad

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

# Indicar la programación

Schedule {
 Name = "Programa"
 Run = Full 1st sat at 23:59
 Run = Level=Incremental sat at 23:59
}

# Indicar los clientes

Client {
 Name = serranito-fd
 Address = 10.0.0.17
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "MoralG630789"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = croqueta-fd
 Address = 10.0.0.6
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "MoralG630789"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = tortilla-fd
 Address = 10.0.0.9
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "MoralG630789"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Client {
 Name = salmorejo-fd
 Address = 10.0.0.14
 FDPort = 9102
 Catalog = mysql-bacula
 Password = "MoralG630789"
 File Retention = 90 days
 Job Retention = 6 months
 AutoPrune = yes
}

Storage {
 Name = Vol-Serranito
 Address = 10.0.0.17
 SDPort = 9103
 Password = "MoralG630789"
 Device = FileAutochanger1
 Media Type = File
 Maximum Concurrent Jobs = 10
}

# Indicar la base de datos

Catalog {
 Name = mysql-bacula
 dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "MoralG630789"
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