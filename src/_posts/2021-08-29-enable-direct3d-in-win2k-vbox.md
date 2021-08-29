---
layout:	post
title:	"Enable Direct3D acceleration in Windows 2000 in VirtualBox (Updated)"
date:	2021-08-29 13:42:00 +0800
categories: windows undocd
permalink:  enable-direct3d-in-win2k-vbox
---

Yes, you read it right. This is done by backporting files for WinXP back to Win2000.

I have tested this on VirtualBox versions below:
- v5.0.10.r104061 (... in 2017, [see archived post here](https://raymai97.github.io/myblog_archived/enable-direct3d-in-win2k-vbox))
- v5.2.44.r139111 (Thanks to Nicolas for prompting me to update this)

## Part 0: A few notes

- Win2000 refers to "Windows 2000 Professional with SP4". I didn't test on earlier version of Win2000.

- WinXP refers to "32-bit Windows XP Professional with SP3". I didn't test on earlier version of WinXP.

- The WinXP installation must be **a guest OS in VirtualBox with working Direct3D acceleration**. It would not work if your VirtualBox version does not support Direct3D acceleration for WinXP guest OS.

## Part 1: Copy related DLL files

First, you need to copy related DLL files from WinXP guest OS.

For v5.0.10.r104061, they are
```
- d3d8.dll
- d3d9.dll
- VBoxD3D8.dll
- VBoxD3D9.dll
- VBoxOGL.dll
- VBoxOGLarrayspu.dll
- VBoxOGLcrutil.dll
- VBoxOGLerrorspu.dll
- VBoxOGLfeedbackspu.dll
- VBoxOGLpackspu.dll
- VBoxOGLpassthroughspu.dll
- wined3d.dll
```

For v5.2.44.r139111, they are
```
- d3d8.dll
- d3d9.dll
- VBoxD3D8.dll
- VBoxD3D9.dll
- VBoxOGL.dll
- wined3d.dll
```

## Part 2: Mod related DLL files to load kernel31.dll instead

- Some `kernel32` APIs required by these DLL files are only present on WinXP.

- To make them work on Win2000, we need to mod them to load alternative DLL instead, which I named as `kernel31.dll`. It forwards most API calls to the real `kernel32.dll`, and provides stub for APIs that are not supported by Win2000.

- If you are interested in creating such DLL by yourself, you may refer Part X. Otherwise, please refer text below.

### Download `kernel31.dll` to Win2000 guest `C:\WINNT\System32`

For v5.0.10.r104061, please [use this]({% asset_path Kernel31.7z %}).

For v5.2.44.r139111, please [use this](https://github.com/Raymai97/vbox-d3d-on-w2k/releases/download/20210825.1/kernel31.zip).

### Mod DLL file

To mod DLL file, you need a hex editor. I recommend [HxD](https://mh-nexus.de/en/hxd/) as it is free, small and serves the purpose well.

1. Open the file with hex editor.
2. Search (Ctrl+F) for text-string `kernel32.dll`.  
Be careful. Sometimes there are more than one occurence in a file.
  - This is the `kernel32.dll` we are looking for:
  ![img]({% asset_path true-kernel32-string.png %})  
  - This is **NOT** the one we are looking for:  
  ![img]({% asset_path false-kernel32-string.png %})

3. Overwrite so it become `kernel31.dll` and save the file.  
![img]({% asset_path overwrite-like-this.png %})

## Part 3: Configure your Win2000 guest OS

Checklist:
- Install VirtualBox Guest Addition as usual.  
No need to tick "Direct3D Support".
- Install DirectX 9.0 redist.  
If you hit issue, try older version such as "DirectX 9.0c (Dec 2006)".
- Copy the related DLL files to `C:\WINNT\System32`.  
You may want to backup the original files to somewhere before overwriting them.
- Open your guest OS settings, go to Display and
  - Tick "Enable 3D Acceleration"
  - Set video memory to 128MB
![img]({% asset_path vm-cfg-enable-d3d.png %})

Finally, run `dxdiag` to see if the Direct3D acceleration is working.  
It is expected to fail in Direct3D 7, but succeed in Direct3D 8 and 9 test.

Here is the result:
![img]({% asset_path result.png %})

## Part X: Creating the Kernel31.dll

If you read this, I assume you already know how to create a DLL and export C functions. Here I'm just going to list out the important points. For complete source code, please refer to my [repo](https://github.com/Raymai97/vbox-d3d-on-w2k).

- First, find out what `kernel32` APIs are needed by the related DLLs.

- Then, find out what `kernel32` APIs are already available in Win2000.

- For APIs that are already available in Win2000, you just need to forward API calls.  
To forward API calls, use pragma to instruct the linker to do so.  
For example, if the API name is `AddVectoredExceptionHandler`, you write the pragma like this:  
```c
#pragma comment(linker, "/export:AddVectoredExceptionHandler=kernel32.AddVectoredExceptionHandler")
```
  to forward the API call to the real `kernel32.dll`.

- For APIs that are required but not available in Win2000, you need to write the function yourself.  
Then, forward the API call to your own exported function, with pragma like this:
```c
#pragma comment(linker, "/export:AddVectoredExceptionHandler=kernel31.My_AddVectoredExceptionHandler")
```
  And remember to export the function you specified. In this example, it is `My_AddVectoredExceptionHandler`.

- Make sure you use .def file, or the exported symbol will be decorated and your DLL wouldn't work.

![img]({% asset_path bad-example.png %})
![img]({% asset_path good-example.png %})

Pro Tips:
- To build a lean DLL for Win2000, try older MSVC such as "Visual C++ 5.0".
- Use Dumpbin to get the EXPORTS and IMPORTS of DLL.  
  IMPORTS let you know what APIs are needed by a DLL.  
  EXPORTS let you know what APIs are provided by a DLL.
