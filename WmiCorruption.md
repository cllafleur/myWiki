# WMI Corruption

See also `WMIDiag` tool.

1. Disable and stop the WMI service.
```shell     
sc config winmgmt start= disabled
net stop winmgmt
```

2. Run the following commands.
```shell
Winmgmt /salvagerepository %windir%\System32\wbem
```
(I noticed that you have run this command, but I would suggest that you try it again)
```shell
Winmgmt /resetrepository %windir%\System32\wbem
```

3. Re-enable the WMI service and then reboot the server to see how it goes.
     sc config winmgmt start= auto

### References
From [Microsoft](https://social.technet.microsoft.com/Forums/windows/en-US/8ed26d46-9994-4052-a307-5b071805aea8/wmi-corrupt-how-to-reinstallrepair)

