---
title: "Memory Forensics Workshop"
date: 2026-10-01
draft: false
tags: ["presentation", "forensics", "blue-team"]
summary: "Designed and ran an intro-to-memory-analysis presentation and hands-on Volatility challenge for the WPI Cybersecurity Club."
---

## Overview
I designed and ran a presentation on Memory Analysis with Volatility for the Cyber Security Club at Worcester Polytechnic Institute. This presentation was designed to give an introduction to memory analysis along with a brief overview of prerequisite concepts such as memory for the benefit of those new to cybersecurity. The presentation I gave is provided below.


## Slides

{{< slides src="https://docs.google.com/presentation/d/e/2PACX-1vQQSNLY0zGgkaZ1ovI9XftPWTEhaPaFeiR_vKWnVZCQ3GzRYPIMSS-0XcCmxPpMwXzT6EPlh9v7xOLI/pubembed?start=false&loop=false&delayms=3000" >}}

## The Challenge
For the challenge portion of the workshop, done after the presentation, I provided members of the meeting with an `.elf` memory dump from a VirtualBox environment along with a series of questions to answer.
- What Operating System is the Memory Dump from?
- When was this dump taken?
- What is the name and PID of the suspicious process running on this system?
- What process spawned it, and what is that process’s PID?
- What is the full command line used to launch the suspicious process?
- What remote IP address and port did this process connect to?
- What file did the process have open?
- What does the file contain?

This memory dump was created by myself to replicate a user having ran a suspicious executable masquerading as a legitimate Windows process. Said process, connected to a C2 server and held open a sensitive password locked file, whose password was saved as an obfuscated environment variable. A full writeup can be found linked at the bottom of this page.



## Takeaways
This presentation and the creation of the memory dump was both a valuable experience in refreshing myself in the area of memory analysis and a good experience in presenting to a group and acting as a guide for those looking for help on the challenge. 


## Links

- [Writeup: walking through the challenge]({{< relref "writeups/memory-analysis-walkthrough" >}})

