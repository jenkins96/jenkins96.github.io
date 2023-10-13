---
layout: post
title: System.Runtime.Serialization Corrupted And Unreadable 0x8007050
subtitle: Guide
gh-repo: jenkins96/jenkins96.github.io
gh-badge: [star, fork, follow]
tags: [procmon, IIS]
comments: true
---

## Behavior

When browsing to specific resources, such as “http(s)://<domain>/default.asp”, site comes up and is working as expected”.
  
However, when browsing to a directory, such as “http(s)://<domain>/folder/”, it gives an error.
  
True error cannot be seen until “customErrors mode=”Off” in “web.config”:
  
```

   <system.web>

       <customErrors mode="Off" />

   </system.web>

```
  
After doing this we can see the actual error:
  
```

"Could not load file or assembly 'System.Runtime.Serialization’. The file or directory is corrupted and unreadable 0x80070570"

 

Source File: C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config Line:97

```
  
![](/assets/img/articles/SystemRuntimeSerialization/serializationError.png)
  
Found in Event Viewer same thing:
  
```
Event code: 3008

Event message: A configuration error has occurred.

Process information: 

   Process ID: 97096 

   Process name: w3wp.exe 

   Account name: IIS APPPOOL\DefaultAppPool 

Exception information: 

   Exception type: ConfigurationErrorsException 

   Exception message: Could not load file or assembly 'System.Runtime.Serialization, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089' or one of its dependencies. The file or directory is corrupted and unreadable. (Exception from HRESULT: 0x80070570) (C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config line 97)

  at System.Web.Configuration.CompilationSection.LoadAssemblyHelper(String assemblyName, Boolean starDirective)

  at System.Web.Configuration.CompilationSection.LoadAssembly(AssemblyInfo ai)

  at System.Web.Compilation.BuildManager.GetReferencedAssemblies(CompilationSection compConfig)

  at System.Web.Compilation.BuildManager.GetPreStartInitMethodsFromReferencedAssemblies()

  at System.Web.Compilation.BuildManager.CallPreStartInitMethods(String preStartInitListPath, Boolean& isRefAssemblyLoaded)

  at System.Web.Compilation.BuildManager.ExecutePreAppStart()

  at System.Web.Hosting.HostingEnvironment.Initialize(ApplicationManager appManager, IApplicationHost appHost, IConfigMapPathFactory configMapPathFactory, HostingEnvironmentParameters hostingParameters, PolicyLevel policyLevel, Exception appDomainCreationException)
```
  
Tested with new IIS site and new App Pool. Created "test.txt" and added to Default Document. Same issue. Issue is System-wide
  
First thought is that there is something wrong with “System.Runtime.Serialization.dll” or something wrong with the ROOT web.config.
  
Checked the ROOT web.config located at “C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config” around line 97.
  
However, when checked with another machine, file seems to be fine and dll is there.
  
![](/assets/img/articles/SystemRuntimeSerialization/rootwebconfig.png)
  
The other thing that we tried was to copy the “System.Runtime.Serialization.dll” from a working Server to the non-working Server.
  
Copied “C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Runtime.Serialization\v4.0_4.0.0.0__b77a5c561934e089\System.Runtime.Serialization.dll” from working to non-working Server, same location.
  
Did ``` iisreset /noforce ```
  
Tested. Issue persisted.
  
At this point, we enabled Fusion Log & Repro issue.
  
* [Enable Fusion Logs](https://stackoverflow.com/questions/255669/how-to-enable-assembly-bind-failure-logging-fusion-in-net)
    
Fusion Log shows that it finds “System.Runtime.Serialization” from GAC at:“C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Runtime.Serialization\v4.0_4.0.0.0__b77a5c561934e089\System.Runtime.Serialization.dll”.
  
However, for some reason, we ended up with:"ERR: Unrecoverable error occurred during pre-download check (hr=0x80070570)".
  
![](/assets/img/articles/SystemRuntimeSerialization/fusionLog.png)
  
This supports the theory that “System.Runtime.Serialization.dll” is corrupted. However, replacing it with the dll from working Server should have fixed this.
  
Next logical step was to use "gacutil /if C:\Windows\Microsoft.NET\Framework64\v4.0.30319\System.Runtime.Serialization.dll" to install “System.Runtime.Serialization.dll”. Although I would have expected that copying and pasting from working to non-working should have had the same effect.
  
* [Gacutil.exe](https://learn.microsoft.com/en-us/dotnet/framework/tools/gacutil-exe-gac-tool)
  
For some reason, we could not do that at the time.
  
By this point we ran ``` sfc /scannow ```. But it did not find anything to fix.
  
The very next thing to try is ``` chkdsk c: /f ```. But this runs at reboot, so we could not test at the moment.
  
* [chkdsk](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/chkdsk?tabs=event-viewer)
  
![](/assets/img/articles/SystemRuntimeSerialization/chkdsk.png)
  
In the meantime, we took PROCMON trace.
* [PROCMON](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon)
  
![](/assets/img/articles/SystemRuntimeSerialization/procmon.png)
  
Trace shows the w3wp.exe finds “System.Runtime.Serialization.dll” in GAC from “C:\Windows\Microsoft.NET\Framework64\v4.0.30319\”.
  
However it also find its native image, which has priority. It finds “System.Runtime.Serialization.ni.dll” in “ "C:\Windows\assembly\NativeImages_v4.0.30319_64\System.Runteb92aa12#\1bed881d4e49318171274a4f08663898”.
  
Runtime first checks if there is native image in GAC, if there is, it takes it, if not, if will find the actual dll.
  
Native images are created by NGEN to convert the dll to machine code in advanced. So, that your app don’t need  to convert the source code of that dll to machine code every time on start.
  
The problem here is that the native image is corrupted!
  
We need to delete the "System.Runtime.Serialization.ni.dll" located at "C:\Windows\assembly\NativeImages_v4.0.30319_64\System.Runteb92aa12#\1bed881d4e49318171274a4f08663898\System.Runtime.Serialization.ni.dll".
  
By doing so, it will force application to use the actual dll.
  
We also can delete native image, the "System.Runtime.Serialization.ni.dll", and then re-generate it with NGEN.
  
* [NGEN](https://learn.microsoft.com/es-es/dotnet/framework/tools/ngen-exe-native-image-generator)
  
CMD ADMIN and change directory to " C:\Windows\Microsoft.NET\Framework64\v4.0.30319".
  
Run:
  
```
ngen install System.Runtime.Serialization
```
  
![](/assets/img/articles/SystemRuntimeSerialization/ngen.png)
  
``` iisreset /noforce ```
  
Done.

Do not underestimate PROCMON :)