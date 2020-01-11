``` shell
Storage { 
 Name = serranito-sd
 SDPort = 9103 
 WorkingDirectory = "/var/lib/bacula"
 Pid Directory = "/run/bacula"
 Maximum Concurrent Jobs = 20
 SDAddress = 10.0.0.17
}


Director {
 Name = serranito-dir
 Password = "**********"
}


Director {
 Name = serranito-mon
 Password = "bacula"
 Monitor = yes
}


Autochanger {
 Name = FileAutochanger1
 Device = DispositivoCopia
 Changer Command = ""
 Changer Device = /dev/null
}


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


Messages {
  Name = Standard
  director = serranito-dir = all
}
```