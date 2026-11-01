---
layout: post
title: "Setting up a Linux lab on Windows: WSL, virtualization, and what broke"
---

## The idea in one paragraph

Most security tools are built for Linux, but my laptop runs Windows. Windows Subsystem for Linux (WSL) lets you run a real Linux system inside Windows without dual booting or a separate virtual machine app. The newer version, WSL2, runs a full Linux kernel inside a small, lightweight virtual machine. That detail matters, because it means WSL2 depends on your computer's hardware virtualization being switched on. Mine wasn't, and finding that out taught me more than a smooth install would have.

## How it works

WSL2 relies on two layers, and both have to be working:

1. **Hardware virtualization in the CPU.** Intel calls it VT-x, AMD calls it AMD-V (often labelled "SVM Mode"). It lets the processor run a virtual machine efficiently. It is controlled in the BIOS/UEFI firmware, before Windows even starts, and many laptops ship with it turned off.
2. **The "Virtual Machine Platform" Windows component.** This is the Windows feature that uses the CPU's virtualization to host the WSL2 virtual machine.

If either layer is missing, the Linux distribution installs fine but crashes the moment you open it.

My Ubuntu window kept closing instantly, with no visible error. The fix started with getting WSL to tell me what was wrong:

```powershell
wsl --status
```

The output was clear once I actually looked: WSL2 could not start because virtualization was not enabled, and it asked me to enable the Virtual Machine Platform component and turn virtualization on in the firmware settings.

What fixed it:

1. Enabled the Windows component (PowerShell as administrator), then restarted:
   ```powershell
   wsl.exe --install --no-distribution
   wsl --update
   ```
2. Checked **Task Manager → Performance → CPU**, where "Virtualization" showed whether the BIOS setting was on, and turned it on in the BIOS.
3. Removed the broken install and installed a fresh one:
   ```powershell
   wsl --unregister Ubuntu-24.04
   wsl --install -d Ubuntu-24.04
   ```

After that, Ubuntu opened, asked me to create a Linux user, and dropped me at a working prompt.

## Try it yourself

- Check your virtualization status first: Task Manager → Performance → CPU → "Virtualization".
- Install WSL with `wsl --install` in an administrator PowerShell, then restart.
- If your distro crashes, run `wsl --status` and `wsl -l -v` before trying anything else.
- To catch an error that flashes and disappears, launch Linux by typing `wsl` in PowerShell instead of clicking the Start menu icon. The error stays on screen.

**Warning:** `wsl --unregister` permanently deletes everything inside that Linux install. Back up anything you care about first.

## Mistakes I made

My first instinct when Ubuntu kept crashing was to uninstall and reinstall it. That would not have helped, because the problem was never Ubuntu itself. It was a Windows setting and a firmware setting. Running `wsl --status` first would have saved me a lot of guessing.

Even after Task Manager showed virtualization as enabled, Ubuntu still crashed. Fixing one layer was not enough; the Virtual Machine Platform component also had to be on, and it needed a restart.

When I did reinstall, I ran `wsl --unregister` and then couldn't find Ubuntu in the Start menu at all. I thought something else had broken, but unregistering had simply removed it, and I hadn't run the install command yet.

Finally, my first working prompt opened in `/mnt/c/WINDOWS/system32`, because I had launched it from an administrator PowerShell. I didn't recognise that path at first. It turned out to be my Windows system folder seen from inside Linux, which is not somewhere to work. `cd ~` took me to my Linux home directory instead.

The lesson I'm keeping: read the error before trying fixes.

## Why it matters in security

Virtualization is a security tool in its own right, not just a setup hurdle. Analysts open malware samples inside virtual machines so that anything harmful stays contained, and Windows itself uses the same hardware features for protections like Virtualization-Based Security. Understanding what isolates what is a core security skill.

WSL also taught me an isolation lesson in reverse. When my first prompt opened at `/mnt/c/WINDOWS/system32`, I realised that WSL mounts the Windows drive inside Linux. So WSL is **not** a sealed sandbox: a tool running in WSL can read and change my Windows files. That's convenient for everyday work, but it means untrusted tools or malware samples belong in a properly isolated virtual machine, not in WSL.
