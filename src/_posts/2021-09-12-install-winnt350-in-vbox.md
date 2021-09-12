---
layout:	post
title:	"Install Windows NT 3.50 in VirtualBox"
date:	2021-09-12 19:00:00 +0800
categories: windows winnt3x vm vbox
permalink:  install-winnt350-in-vbox
---

![img]((% winnt350_in_vbox.png %))

Environment:
```
ISO name: en_winnt_3.5_wks.iso
ISO size: 258,607,104 bytes
ISO MD5: b7205f611100bd19643329fd0ce2e6c3
ISO SHA1: ae0e3c862d8d6662e882fc4ba7963eb6cf893d91
VirtualBox: version 5.2.44 r139111
```

1) Boot into MS-DOS 6.2 with CD-ROM support.

2) Make sure C drive is formatted as FAT16.

3) Assume CD-ROM drive letter is D, run:
```
D:\i386\winnt.exe /B
```

4) Go through the text-mode setup as per usual.  
![img]((% textmode_ing.png %))

5) When the text-mode setup is done, it will ask to press any key to reboot.
Whatever you do, DO NOT boot into WinNT yet, shut it down first.  
![img]((% textmode_done.png %))

6) WinNT 3.50 is very old. Its CPU detection will not work with modern CPU.  
If we boot into WinNT without any tweak, it will say something like
> Setup cannot install on the current processor.

To overcome this issue, we need to manually specify the CPU ID.  
This can be done by modifying "initial.inf" and "setup.inf".

![img]((% mod1.png %))

![img]((% mod2.png %))  

> While we can boot into MS-DOS, use EDIT.COM to modify the files, I prefer mounting VHD, modify and dismount VHD.

After these files are modified, boot the VM.

7) If your see GUI mode setup as below, congratulations!
![img]((% congraz.png %))

From this point, there's nothing particular to take note.  
Just complete installation as usual and enjoy your WinNT 3.50 in VirtualBox. :)
