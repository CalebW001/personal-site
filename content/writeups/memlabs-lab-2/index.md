---
title: "MemLabs Lab 2"
date: 2026-09-28
draft: false
tags: ["forensics", "volatility", "memlabs"]
summary: "My writeup for MemLabs Lab 2, a three-flag Windows memory forensics challenge."
---

For this challenge I was given a `.raw` memory dump file and this description:

> One of the clients of our company lost access to his system due to an unknown
> error. He is supposedly a very popular "environmental" activist. As a part of
> the investigation, he told us that his go-to applications are browsers, his
> password managers etc. We hope that you can dig into this memory dump and find
> his important stuff and give it back to us.
>
> **Note**: This challenge is composed of 3 flags.

## My process

To establish what OS this dump was from I first ran:

```bash
vol -f MemoryDump_Lab2.raw banners.Banners
```

This returned `ntkrnlmp.pdb`, indicating the OS is Windows for this dump.

I then ran this to get some basic info about the dump:

```bash
vol -f MemoryDump_Lab2.raw windows.info
```

After which I ran:

```bash
vol -f MemoryDump_Lab2.raw windows.pslist
```

I noted 3 things running that I was interested in:

- `KeePass.exe` — likely the password manager mentioned, so I suspected this would lead to a flag.
- `chrome.exe` — multiple instances were running, and the description had mentioned the client's go-to applications are browsers.
- `notepad.exe` — while not mentioned in the description, I felt this was worth checking out.

### KeePass

I first looked into `KeePass.exe`:

```bash
vol -f MemoryDump_Lab2.raw windows.pstree | grep 1064
```

The output contained the path `C:\Users\SmartNet\Secrets\Hidden.kdbx`. After a
quick Google I found that a `.kdbx` file is the file that would hold the passwords
for a KeePass account, and could be opened with the keepassxc tool.

I dumped the `Hidden.kdbx` file:

```bash
vol -f MemoryDump_Lab2.raw windows.dumpfiles --physaddr 0x3fb112a0
```

Then ran the tool, where I was asked for a password. Not having the password at
the moment, I went back to check out the `chrome.exe` instances.

### Chrome

```bash
vol -f MemoryDump_Lab2.raw windows.cmdline | grep -i chrome
```

After which I decided to process dump one of the chrome instances:

```bash
vol -f MemoryDump_Lab2.raw -o . windows.memmap --pid 1632 --dump
```

Looking through this with the following resulted in a lot of noise:

```bash
strings -n 8 pid.1632.dmp | grep -iE "https?://" | sort -u | less
```

So I cut down on it by filtering out a lot of the unimportant links:

```bash
strings -n 8 pid.1632.dmp | grep -iE "https?://" | sort -u | grep -viE "google|gstatic|googleapis|googleusercontent|microsoft|windows|w3\.org|schema|chromium|mozilla|apple|adobe|digicert|verisign|symantec|globalsign|amazontrust|ocsp|\.crl|\.crt|bi0s|cdn" | less
```

This allowed me to first spot a link to a blog, which just turned out to be the
cybersecurity blog, not directly related to the challenge. I then noticed a link
to the MEGA home page, which inspired me to grep for it further:

```bash
strings -n 8 pid.1632.dmp | grep -iE "mega.nz/"
```

Not finding anything further, I decided to check a different `chrome.exe`
instance:

```bash
vol -f MemoryDump_Lab2.raw -o . windows.memmap --pid 2296 --dump
strings -n 8 pid.2296.dmp | grep -iE "mega.nz"
```

Which gave me an actual MEGA link to a download for a zip file called
`MemLabs_Lab2_Stage3`, which when downloaded contained a second zip file called
`Important.zip`. That one was password locked, however there was a password hint:
"Password is SHA1(stage-3-FLAG) from Lab-1. Password is in lowercase." Running
`sha1sum` on the stage 3 flag from Lab 1 gave me the password. Opening
`Important.zip` gave me `Important.png`, which displayed the flag for stage 3 of
this challenge.

### Environment variables

At this point I decided to check the environment variables, as Lab 1 had
contained some valuable information in them:

```bash
vol -f MemoryDump_Lab2.raw windows.envars > envars.txt
grep -viE "ALLUSERSPROFILE|APPDATA|CommonProgram|COMPUTERNAME|ComSpec|FP_NO_HOST|HOMEDRIVE|HOMEPATH|LOCALAPPDATA|LOGONSERVER|NUMBER_OF|OS|Path|PATHEXT|PROCESSOR|ProgramData|ProgramFiles|ProgramW6432|PROMPT|PSModulePath|PUBLIC|SESSIONNAME|SystemDrive|SystemRoot|USERDOMAIN|USERNAME|USERPROFILE|windir|windows_tracing|PID|Process" envars.txt | sort -u
```

Scrolling through the output, I saw a section
`ZmxhZ3t3M2xjMG0zX1QwXyRUNGczXyFfT2ZfTDRCXzJ9` that looked like base64. Decoding it
gave me the flag for stage 1:

```bash
echo 'ZmxhZ3t3M2xjMG0zX1QwXyRUNGczXyFfT2ZfTDRCXzJ9' | base64 -d
```

### Finding the KeePass password

Now looking back at the process list, I noticed a `cmd.exe` and a `conhost.exe`
process, and decided to process dump both of them to see if I could find a
password input for `Hidden.kdbx`:

```bash
vol -f MemoryDump_Lab2.raw -o . windows.memmap --pid 2096 --dump
vol -f MemoryDump_Lab2.raw -o . windows.memmap --pid 2068 --dump
strings -n 4 -e l pid.2068.dmp | grep -iE "password|secret|kdbx|hidden|master"
strings -n 4 -e l pid.2096.dmp | grep -iE "password|secret|kdbx|hidden|master"
```

Which gave me `/C:/Users/Alissa%20Simpson/Pictures/Password.png`. I then grepped
for the address via filescan and dumped the file:

```bash
vol -f MemoryDump_Lab2.raw windows.filescan | grep -i "Password.png"
vol -f MemoryDump_Lab2.raw windows.dumpfiles --physaddr 0x3fce1c70
```

The png contained a password for the `Hidden.kdbx` file, which gave me access to
the flag for stage 2.
