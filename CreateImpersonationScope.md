# Create Impersonation context


[MSDN Reference](https://msdn.microsoft.com/en-us/library/system.security.principal.windowsimpersonationcontext(v=vs.110).aspx)

```csharp
internal static class CmaImpersonator
{
    [DllImport("advapi32.dll", SetLastError = true, CharSet = CharSet.Unicode)]
    private static extern bool LogonUser(
        String lpszUsername,
        String lpszDomain,
        String lpszPassword,
        int dwLogonType,
        int dwLogonProvider,
        out SafeTokenHandle phToken);
 
    public sealed class SafeTokenHandle : SafeHandleZeroOrMinusOneIsInvalid
    {
        private SafeTokenHandle()
            : base(true) { }
 
        [DllImport("kernel32.dll")]
        [ReliabilityContract(Consistency.WillNotCorruptState, Cer.Success)]
        [SuppressUnmanagedCodeSecurity]
        [return: MarshalAs(UnmanagedType.Bool)]
        private static extern bool CloseHandle(IntPtr handle);
 
        protected override bool ReleaseHandle()
        {
            return CloseHandle(handle);
        }
    }
 
    internal static T InvokeAsOtherUser<T>(Func<T> method, string domain, string userName)
    {
        T result = default(T);
 
        //http://msdn.microsoft.com/en-us/library/system.security.principal.windowsimpersonationcontext(v=vs.110).aspx
        int LOGON32_LOGON_NEW_CREDENTIALS = 9;
 
        string password = File.ReadAllText(HostingEnvironment.ApplicationPhysicalPath + userName + ".password.txt");
 
        SafeTokenHandle safeTokenHandle;
        var logonSucceed = LogonUser(userName, domain, password, LOGON32_LOGON_NEW_CREDENTIALS, 0, out safeTokenHandle);
        if (logonSucceed)
        {
            using (safeTokenHandle)
            {
                // Use the token handle returned by LogonUser. 
                using (WindowsIdentity newId = new WindowsIdentity(safeTokenHandle.DangerousGetHandle()))
                {
                    using (WindowsImpersonationContext impersonatedUser = newId.Impersonate())
                    {
                        result = method.Invoke();
                    }
                }
                // Releasing the context object stops the impersonation 
            }
        }
        else
        {
            var errorCode = Marshal.GetLastWin32Error();
            throw new Exception(string.Format(
                "Could not impersonate. LogonUser returned error code {0}.",
                errorCode));
        }
 
        return result;
    }
 
}
```