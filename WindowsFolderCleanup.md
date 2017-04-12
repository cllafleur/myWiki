# Windows folder cleanup 

Permet de nettoyer le répertoire WinSxS

Analyze et rapport
```shell
Dism.exe  /online /cleanup-image /analyzecomponentstore
```

Lance la procédure de cleanup
```shell
Dism.exe /online /Cleanup-Image /StartComponentCleanup
```

En cas de corruption
```shell
Dism /Online /Cleanup-Image /RestoreHealth
Dism /Online /Cleanup-Image /scanHealth
```


Le contenu du répertoire `C:\Windows\SoftwareDistribution\Download` est supprimable
En désactivant au préalable le service Windows update
```shell
net stop wuauserv
```

Compresser le répertoire `C:\Windows\Installer`



Windows Vista or Windows Server 2008 Service Pack 1:  `VSP1CLN.EXE`
•Windows Vista or Windows Server 2008 Service Pack 2:  `Compcln.exe` 
•Windows 7 or Windows Server 2008 R2 Service Pack 1:  `DISM /online /Cleanup-Image /SpSuperseded` or Disk Cleanup Wizard (cleanmgr.exe)

### References
From [Microsoft](https://social.technet.microsoft.com/forums/windowsserver/en-us/75b7edb0-4bb2-4337-af23-85e3ff179d92/dism-command-in-windows-2008-sp2)

