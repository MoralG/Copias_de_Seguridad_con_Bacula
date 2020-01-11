``` shell
Director {
  Name = serranito-dir
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/run/bacula"
  Maximum Concurrent Jobs = 20
  Password = "**********"
  Messages = Daemon
  DirAddress = 10.0.0.17
}


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


Schedule {
 Name = "Programa"
 Run = Full 1st sat at 23:59
 Run = Level=Incremental sat at 23:59
}


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


Storage {
 Name = Vol-Serranito
 Address = 10.0.0.17
 SDPort = 9103
 Password = "**********"
 Device = FileAutochanger1
 Media Type = File
 Maximum Concurrent Jobs = 10
}


Catalog {
 Name = mysql-bacula
 dbname = "bacula"; DB Address = "localhost"; dbuser = "bacula"; dbpassword = "bacula"
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

#### Por defecto

Messages {
  Name = Standard
  mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: %t %e of %c %l\" %r"
  operatorcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: Intervention needed for %j\" %r"
  mail = root = all, !skipped
  operator = root = mount
  console = all, !skipped, !saved
  append = "/var/log/bacula/bacula.log" = all, !skipped
  catalog = all
}


Messages {
  Name = Daemon
  mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula daemon message\" %r"
  mail = root = all, !skipped
  console = all, !skipped, !saved
  append = "/var/log/bacula/bacula.log" = all, !skipped
}


Pool {
  Name = Default
  Pool Type = Backup
  Recycle = yes                       # Bacula can automatically recycle Volumes
  AutoPrune = yes                     # Prune expired volumes
  Volume Retention = 365 days         # one year
  Maximum Volume Bytes = 50G          # Limit Volume size to something reasonable
  Maximum Volumes = 100               # Limit number of Volumes in Pool
}


Pool {
  Name = Scratch
  Pool Type = Backup
}


Console {
  Name = serranito-mon
  Password = "rPNmB3G7UEOX7ztuXzmvjr08SRH5HJCMT"
  CommandACL = status, .status
}
```