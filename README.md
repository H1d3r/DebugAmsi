# DebugAmsi
## TL;DR

Just run the file from Release and get access to powershell without AMSI!


https://github.com/MzHmO/DebugAmsi/assets/92790655/e56f48e7-074c-47f8-87b1-8aa3cf4b59bb


## How It Works
One day I've discovered an interesting function [DebugActiveProcess](https://learn.microsoft.com/en-us/windows/win32/api/debugapi/nf-debugapi-debugactiveprocess) which allows us to become a debugger for a process. Full-fledged debugging will be available if our process has the SeDebug privilege or the ability to call OpenProcess() with the PROCESS_ALL_ACCESS mask. 

Once our process becomes a debugger, it can handle the LOAD_DLL_DEBUG_EVENT event, which is generated by the Windows system when any DLL is loaded into the process address space.

Thus, we can start powershell.exe, then become a debugger for it and intercept an attempt to load amsi.dll . And then patch it at the moment of loading.

The problem is that we may miss the point of loading amsi.dll into the powershell.exe process, so in my code, the process starts in a suspended state (at that point, it will only have ntdll.dll in its address space). Then installs the debugger and resumes the main thread of the process, which causes powershell.exe to resume and load the necessary libraries.
![изображение](https://github.com/MzHmO/DebugAmsi/assets/92790655/1807cea2-e118-45e0-a749-f0cb7c934a09)

![изображение](https://github.com/MzHmO/DebugAmsi/assets/92790655/5a3ccb31-7c42-4c6b-a254-894559d98421)

After generating the LOAD_DLL_DEBUG_EVENT event, I find amsi.dll and parsing its EAT to find the AmsiOpenSession and AmsiScanBuffer functions.
![изображение](https://github.com/MzHmO/DebugAmsi/assets/92790655/edcaa824-820a-4ad1-9b96-f6410af765fe)

EAT parsing is done by reading the memory of the amsi.dll library loaded in powershell.exe . This is done so as not to load amsi.dll into our own process.
![изображение](https://github.com/MzHmO/DebugAmsi/assets/92790655/82f6e142-41db-423d-b4d9-6ea1083e6278)

After that, a simple patch is applied that renders AMSI useless. This results in a running powershell.exe process with amsi disabled
