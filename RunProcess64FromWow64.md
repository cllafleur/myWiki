# Run 64bit process from wow64

Although 64-bit clients can be managed, LANDesk client processes are 32-bit processes, so they are compatible with legacy 32-bit systems. On 64-bit systems, processes running in 32-bit address space are run within a sandbox (WOW64) for backwards compatibility. As a result, VB scripts, Powershell scripts and Batch files that are run as distribution packages or custom scripts, would run as 32-bit processes within the WOW64 sandbox. The method below describes how to break out of the sandbox and run scripts and programs as native 64-bit processes on 64-bit systems.

A sample Batch file and VB wrapper-script is also provided that checks the CPU architecture and runs a command-line as a native 32-bit or 64-bit program (with WOW64-redirection disabled for 64-bit).

#### Support for 64-bit distribution packages (LDMS 9.0 SP2 and later)
In LD 9.0 SP2, a new option was introduced for Batch file, Windows Script Host and Powershell distribution packages, that allows the user to specify that they want the package to run as a 64-bit application. This option is ignored on 32-bit systems. The method used to run scripts as 64-bit applications is described below.


This method is not available for custom scripts. It’s not available for distribution packages before LDMS 9.0 SP2. In these cases, the method described below can be used to run Batch files, VB-scripts and Powershell scripts within 64-bit address space.

#### The WOW64 sandbox
On 64-bit OS’es, 32-bit processes are executed within a sandbox (WOW64) for backwards compatibility. The features implemented by WOW64 are filesystem and registry redirection for certain directories.
The reasoning behind this is that many programs have the `%SystemRoot%\system32` directory hardcoded, and more importantly, unlike the transition from 16-bit to 32-bit Windows, virtually all of the DLLs implementing the Windows API retained their same name (and path) across the 32-bit to 64-bit transition.

Clearly this creates a conflict, as there can’t be both 32-bit and 64-bit DLLs with the same name in the same path. The solution that Microsoft devised, filesystem redirection, is a layer that intercepts access to the system32 directory made by 32-bit applications, and redirects I/O to the `%SystemRoot%\SysWOW64` directory, which holds the 32-bit variant of Windows DLL’s and utilities. Similarly, the `HKLM\SOFTWARE` registry key is redirected to `HKLM\SOFTWARE\Wow6432Node` for 32-bit processes running on 64-bit systems.

Normally, this works fairly well, but there are times when a program needs to access the real `%SystemRoot%\System32` directory from a script. When LANDesk executes a 64-bit binary, it already runs this as a 64-bit application (with WOW64 redirection turned off), but Batch files or VB/Powershell scripts run as a 32-bit applications by default, because the 32-bit versions of `cmd.exe`, `cscript.exe` and `powershell.exe` are used by default.

#### Backdoor
For Windows Vista/7 and later, Microsoft has provided a “backdoor” to WOW64 filesystem/registry redirection, in the form of a pseudo-directory `%SystemRoot%\Sysnative` (eg. `C:\Windows\Sysnative`). This pseudo-directory is not visible in directory listings and does not exist for native 64-bit processes. It exists for WOW64 processes only and can be used to access the native 64-bit system32 directory.

By using `%SystemRoot%\Sysnative\cscript.exe` and `%SystemRoot%\Sysnative\cmd.exe`, one can run scripts and batch files (and other Windows utilities, eg. `msiexec.exe`) in native 64-bit mode.

#### Detection matrix – how to detect process bitness
Processes run in one of 3 CPU architecture modes:

* Native 32-bit processes on a 32-bit OS
* Native 64-bit processes on a 64-bit OS
* Legacy 32-bit processes in the WOW64 sandbox on a 64-bit OS

The following environment variables help to identify which mode a process is running in:

+ PROCESSOR_ARCHITECTURE:     reports the native processor architecture, EXCEPT for WOW64 where it reports x86.
+ PROCESSOR_ARCHITEW6432:     only used for WOW64 (32-bit processes on 64-bit OS), where it reports the original native processor architecture.

| 	|32-bit native|64-bit native|WOW64|
|--|--|--|---|
PROCESSOR_ARCHITECTURE|X86|AMD64(*)|X86
PROCESSOR_ARCHITEW6432|	Not defined|Not defined|AMD64 (*)|

(*)Replace AMD64 with IA64 for Itanium processors

#### Wrapper scripts to run Batch files and VB scripts in native 64-bit mode
If you want to run Batch files and scripts in native 64-bit mode, the following wrapper scripts could be used. If the `%PROCESSOR_ARCHITEW6432%` environment variable is defined, the script calls itself recursively, but using the native 64-bit version of the interpreter (cmd/cscript.exe).

__Sample Batch file:__
```shell
IF "%PROCESSOR_ARCHITEW6432%"=="" GOTO native
%SystemRoot%\Sysnative\cmd.exe /c %0 %*
exit
:native

... the rest of the script ...
```
 
__Sample VB Script:__
```vb
Set WshShell = CreateObject("WScript.Shell")
 
If (Len(WshShell.Environment("PROCESS")("PROCESSOR_ARCHITEW6432")) > 0) Then
        exe = "%SystemRoot%\Sysnative\cmd.exe /c " & WScript.ScriptFullName
                Set objArgs = WScript.Arguments
                For Each strArg in objArgs
                    exe = exe & " " & strArg
                Next
                retcode = WshShell.Run(exe, 0, True)
                WScript.Quit (retcode)
 
End If
 
... the rest of the script ...
```

From <https://community.landesk.com/support/docs/DOC-26359> 


