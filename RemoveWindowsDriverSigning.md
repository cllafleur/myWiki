# Remove windows driver signing enforcement

Disabling Driver Signature Enforcement In Windows 8 Permanently

## Step 1.
Open the Windows command promt as “Run as Administrator”.

## Step 2.
Run 
```shell
 bcdedit -set loadoptions DISABLE_INTEGRITY_CHECKS
```

## Step 3.
To finalize the process run 
```shell
bcdedit -set TESTSIGNING ON
```

## Step 4.
Reboot and you’re done.
To disable it do step 1 and run these commands on 
step 2 and 3:
Step 2.
```shell
 bcdedit -set loadoptions ENABLE_INTEGRITY_CHECKS
 ```
Step 3. 
```shell
bcdedit -set TESTSIGNING OFF
```
Then do step 4 and you’re done.

### References

From [Microsoft forum](http://answers.microsoft.com/en-us/windows/forum/windows_tp-hardware/how-do-i-disable-driver-signature-enforcement-win/a53ec7ca-bdd3-4f39-a3af-3bd92336d248) 

