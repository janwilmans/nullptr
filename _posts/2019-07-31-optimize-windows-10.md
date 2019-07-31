---
ID: 1854
post_title: >
  Optimize Windows 10 for Software
  Development
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: >
  http://nullptr.nl/2019/07/optimize-windows-10/
published: true
post_date: 2019-07-31 20:01:46
---
This blog is about how I use windows as a professional software engineer. What windows settings I customize and what tools I use to make my daily work as pleasing and productive as possible.

## Performance and usability enhancements

Some of these settings are particularly well suited for virtual machines, use common sense to judge whether is it safe (enough) to turn off these features on real machines. In embedded development and hardware integration work that I do, I often find myself in situations where the network is already restricted enough by firewalls and anti-virus measures that having these options on only gets in the way.

Developer mode - Handle With Care:

*   turn off UAC (see reg file below)
*   turn off Windows Defender (see reg file below)
*   turn off Firewall: Win-S, Firewall, Turn Windows defined firewall on or off, private -> off, public -> off

After the firewall is turned off, you will get regular [reminders][1] about it, I have not found out how to turn these off yet.

## Paranoid software engineers:

As a software engineer, my job is to be paranoid, if something changed on my system, I have to assume I have to re-test ALL my assumptions. This means that if an update is installed during my debug session I can basically start over. This is not acceptable for me, so to prevent this I turn off updates during debug sessions. [Winupdate-stop][2] is a handy tool to turn update off and on again.

## Windows 10 annoyances

The following settings are my personal defaults:

> Responsiveness improvements / low latency HMI:

*   turn off UAC
*   turn off Windows Defender 
*   lower cl.exe priority to keep machine responsive
*   allow all powershell scripts
*   turn of sliding, animation and transparency
*   show all tray icons

> Pleasing to work with:

*   keep shadows under windows
*   keep font optimizations

> Productivity:

*   show all file extensions
*   write minidumps to c:\crashdumps when any program crashes
*   disable phone home to microsoft when any program crashes

> And here is all of it packed into one registry patch:

[<img src="http://nullptr.nl/wp-content/uploads/2019/07/registry_developer_settings.png" alt="" width="651" height="271" class="alignnone size-full wp-image-1903" />][3]

# Aside from windows settings there are tools

Aside from windows settings there are tools that can make you developer life easier. Lets start with installation of tools. I regularly reinstall my whole system, every few months or so. Installing updating all tools to the latest version and manually installing them can be a hassle. Fortunately, [chocolatey.org][4] can install lots of things completely unattended.

*   first run this script as admin to install choco itself [1_install_chocolatey_as_admin.bat][5]
*   now open a **new** administrator command prompt and run [update_packages.bat][6]

The content of packages.bat is just an example, customize it to your wishes and save it for next time.

https://www.howtogeek.com/356642/how-to-fix-all-windows-10%E2%80%99s-annoyances/

# Tooling, roughly in order of awesome

*   [FileLocator Lite][7]
*   [GNU for Windows][8]
*   [Putty][9]
*   [chocolatey][4] 

# More tooling, that are bread and butter

*   [ccleaner][10] is really useful to remove lots of unused stuff, clear caches, remove broken registrations etc.
*   [defrag(gler)][11] harddisks (yea its still a thing, but only on large non-ssd harddisks with lots of small-ish ~1-2 MB files.)
*   [Notepad++][12] (Settings -> Preferences -> Backup -> Remember current session -> disable)
*   [Free Hex Editor Neo][13] 
*   [Ultraedit][14] (paid, I've been a user for 20 years, just the best, but admittedly, Notepad++ and FHE Neo cover most use cases)
*   [git][15] need I say more ?
*   [TortoiseGit][16] (Settings -> Icon Overlays -> Status cache -> None)

 [1]: http://nullptr.nl/wp-content/uploads/2019/07/firewall_off_reminders.png
 [2]: https://www.novirusthanks.org/products/win-update-stop/
 [3]: https://github.com/janwilmans/windows-docker/blob/master/autoinstall_vsbuildtools/developer_settings.reg
 [4]: https://chocolatey.org/
 [5]: https://github.com/janwilmans/windows-docker/blob/master/autoinstall_vsbuildtools/1_install_chocolatey_as_admin.bat
 [6]: https://github.com/janwilmans/windows-docker/blob/master/autoinstall_vsbuildtools/update_packages.bat
 [7]: https://www.mythicsoft.com/filelocatorlite/
 [8]: http://%28https://github.com/bmatzelle/gow
 [9]: https://www.putty.org/
 [10]: https://www.ccleaner.com/ccleaner
 [11]: https://www.ccleaner.com/defraggler
 [12]: https://notepad-plus-plus.org/
 [13]: https://www.hhdsoftware.com/free-hex-editor
 [14]: https://www.ultraedit.com/
 [15]: https://git-scm.com/download/win
 [16]: https://tortoisegit.org/