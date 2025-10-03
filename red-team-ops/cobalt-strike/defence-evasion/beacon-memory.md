# Beacon Memory

Since Cobalt Strike [4.11](https://www.cobaltstrike.com/blog/cobalt-strike-411-shh-beacon-is-sleeping), Beacon has had a significant uplift in the number of evasions that are enabled out of the box.  Before, we had to enable a whole raft of options in Malleable C2 that are now enabled by default.  However, there are still a few bits that we can turn on for even better evasion.

Executing a new Beacon and inspecting the memory allocation, in a tool like [System Informer](https://systeminformer.com/), will reveal a single region.

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/df72d02a44b639a70fdbb7fb110ab361.png" alt="" width="100%"><figcaption></figcaption></figure>

Beacon has a feature called a 'sleep mask' which obfuscates Beacon's memory before it goes to sleep.  The memory itself is therefore quite unreadable which allows it to hide from memory scanners.  However, the memory allocation itself has some suspicious properties that we can address.

### Module stomping <a href="#el_1742403712940_389" id="el_1742403712940_389"></a>

The default behaviour of the reflective loader is just to allocate a new region of memory using an API like VirtualAlloc.  This causes the memory to have no 'use' as shown in System Informer, unlike most of the other regions that have been loaded from a legitimate DLL on disk.  Obviously, dropping a Beacon DLL to disk defeats the point of using reflective injection, which is where a technique called module stomping (or overloading) becomes useful.

This is a process where instead of allocating memory using VirtualAlloc, a legitimate DLL is loaded from disk using LoadLibrary or perhaps LdrLoadDll.  The content of that loaded DLL in memory is then overwritten with Beacon.  The upshot being that Beacon then appears to be running in a memory region that is backed by a module on disk.

This can be enabled using `stage.module_[x64/x86]` in Malleable C2 by specifying the filename of a DLL in System32.  The module that you pick should not be in use by the host application (e.g. if you're injecting Beacon shellcode into an existing process), and it needs to be big enough to comfortably hold Beacon.

```
stage {
   set module_x64 "Hydrogen.dll";
   }
```

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/b7857baa783cac8fbcfc55a7cecf10a7.png" alt="" width="100%"><figcaption></figcaption></figure>

### RWX memory <a href="#el_1742403705346_374" id="el_1742403705346_374"></a>

By default, the reflective loader will set Beacon's memory with RWX (read, write, and execute) permissions.  PowerShell, .NET, and other applications that leverage the CLR (Common Language Runtime) will legitimately have RWX regions because of how the JIT (just-in-time) compiler works.  However, seeing RWX memory in most native applications is usually a bad sign.

Setting the `stage.userwx` option in Malleable C2 to `false` instructs the reflective loader to allocate memory using RW permissions first, copy each Beacon PE section into memory, and then assign the correct final memory permission for each section.  This is usually:

* RX for the .text section.
* RW for the .data section.
* R for the .rdata, .pdata, and .reloc sections.

```
stage {
   set userwx "false";
      set module_x64 "Hydrogen.dll";
      }
```

<figure><img src="https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/7a9eb07d64de3de067190b7a27ebe955.png" alt="" width="100%"><figcaption></figcaption></figure>

### PE Headers <a href="#el_1742481151900_343" id="el_1742481151900_343"></a>

We can also instruct the reflective loader to copy Beacon into memory without its PE headers.  These headers contain information vital for loading Beacon into memory, but they're not actually required to be copied into memory with the PE.  Removing them slightly reduces the detection surface of the Beacon.

```
stage {   
set userwx "false";
   set module_x64 "Hydrogen.dll";
      set copy_pe_header "false";
      }
```
