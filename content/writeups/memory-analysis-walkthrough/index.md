---
title: "Memory Analysis Walkthrough"
date: 2026-10-03
draft: false
tags: ["forensics", "volatility"]
summary: "My walkthrough of the memory forensics challenge I built for the Cybersecurity Club workshop."
---

This is my walkthrough for the challenge I put together for the
[memory forensics workshop]({{< relref "projects/memory-forensics-workshop" >}})
I ran for the Cybersecurity Club. Everyone was given an `.elf` memory dump
(`CaseFile.elf`) taken from a VirtualBox VM along with eight questions to answer,
and the whole point was to work through them using Volatility 3. Below I go
through how each one gets answered.

I made this dump to replicate a user having run a suspicious executable that was
masquerading as a legitimate Windows process, which in turn connected out to a C2
server and was holding open a sensitive password-locked file, whose password was
saved as an obfuscated environment variable. Every command below is run as
`vol -f CaseFile.elf <plugin>`.

## What OS is the dump from?

The first thing I did was figure out what I was even looking at, which tells
Volatility which set of plugins to use:

```bash
vol -f CaseFile.elf windows.info
```

This comes back as Windows, 64-bit, and the build fields line up with Windows 10.
If you just want a quick guess at the OS, `banners.Banners` works too.

## When was the dump taken?

That same `windows.info` output has a `SystemTime` field, which is the system
clock at the moment the dump was captured, so there was no need to run anything
else for this one.

## What is the suspicious process and its PID?

To see everything that was running I pulled up the process tree:

```bash
vol -f CaseFile.elf windows.pstree
```

The one that stands out is `svccccchost.exe`, running as PID 4700. It stood out to
me for two reasons: the name is a typo-squat of the real `svchost.exe`, and it's
running out of a user's Desktop instead of `C:\Windows\System32\` where the real
one lives.

## What spawned it, and what is that process's PID?

The process tree answers this too. Directly above `svccccchost.exe` is
`explorer.exe`, which is the Windows shell, so this is what you'd expect to see if
a user double-clicked something to launch it rather than running it from a
terminal.

## What is the full command line?

To see exactly how it was launched I checked the command line for that PID:

```bash
vol -f CaseFile.elf windows.cmdline --pid 4700
```

```
"C:\Users\vboxuser\Desktop\svccccchost.exe" -ExecutionPolicy Bypass -File "C:\Users\vboxuser\Desktop\svcccchost.ps1"
```

The `-ExecutionPolicy Bypass` here is a good thing to point at, since it's telling
PowerShell to ignore its script restrictions, which is not something you'd
normally see on a legitimate process.

## What remote IP and port did it connect to?

Next I looked at the network connections to see where it was reaching out to:

```bash
vol -f CaseFile.elf windows.netscan
```

Filtering down to PID 4700 shows an established connection out to
`192.168.56.1:4443`, which is the C2 server I set up.

## What file did it have open?

Rather than guessing at filenames, I listed the handles that the suspicious
process itself was holding, which shows everything it had open at the time of the
dump:

```bash
vol -f CaseFile.elf windows.handles --pid 4700
```

Looking through the file handles in that list, the one that matters is
`Passwords_Backup.zip`. This is where the filename actually comes from, since at
this point I didn't know it ahead of time.

## What does the file contain?

This last one took two steps. Now that I knew the filename, I scanned for its file
object to get the physical offset I needed to carve it out of memory:

```bash
vol -f CaseFile.elf windows.filescan | grep -i Passwords_Backup
vol -f CaseFile.elf windows.dumpfiles --physaddr <offset>
```

The archive is password-protected, but the password is sitting right there in the
process's environment variables, just base64-encoded:

```bash
vol -f CaseFile.elf windows.envars --pid 4700
```

Among the normal Windows variables there's a `SYNC_TOKEN` that doesn't belong.
Decoding it gives the password:

```bash
echo '<base64 string>' | base64 -d
```

Using that password to open the archive gives up the `flag.txt` inside, which
wraps up the challenge.
