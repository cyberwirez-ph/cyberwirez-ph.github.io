---
title: "Arcana: Curated List of Useful Offensive Tradecraft Resources"
date: 2026-07-11
description: "A curated collection of offensive tradecraft, reverse engineering, and vulnerability research resources."
author: "grit"
draft: false
---

*Originally published on [grit8086.github.io](https://grit8086.github.io/posts/arcana/)*

### Low-Level Languages

  * [C](https://www.learn-c.org/)



### Low-Level Concepts

Build core knowledge around system internals, cryptography, and debugging.

  * Obfuscation, encryption, decryption, and common algorithms
  * Learn to use debuggers (e.g., x64dbg)



#### Windows Internals

  * [Introduction to Malware Development](https://anteiku.fun/papers/an-introduction-to-modern-malware-development-for-red-teams/02_portable_executables/)
  * [cr0w – Processes, Threads & Handles](https://www.youtube.com/watch?v=aNEqC-U5tHM&t=527s)
  * Download [**Windows Internals, Part 1**](https://empyreal96.github.io/nt-info-depot/Windows-Internals-PDFs/Windows%20System%20Internals%207e%20Part%201.pdf) and use it as a reference whenever you want to dive deeper into a topic you’re currently learning.



#### Project Ideas

Try to replicate a malware behavior and ask:

  * What is being performed?
  * How is it being performed?
  * Are there alternative techniques?
  * Can we replicate the API usage?



#### Content Creators

  * [Lseqt](https://www.youtube.com/@Lsecqt)
  * [cr0w](https://www.youtube.com/@crr0ww)
  * [ActiveXSploit](https://www.youtube.com/@ActiveXSploit)



#### Blogs

  * [0xPat](https://0xpat.github.io/)
  * [mr.d0x](https://mrd0x.com/)
  * [pre.empt](https://pre.empt.dev/)
  * [0xRick](https://0xrick.github.io/misc/c2/)
  * [Capt.Meelo](https://captmeelo.com/)



#### GitHub Repositories

  * [Outpacket](https://github.com/n00py/Outpacket)



#### Books & Papers

  * [Ultimate Anti-Reversing Reference](https://anti-reversing.com/Downloads/Anti-Reversing/The_Ultimate_Anti-Reversing_Reference.pdf)
  * [VXUG Papers](https://vx-underground.org/Papers)
  * [VXLabInfo Library](https://github.com/vxlabinfo/lib/tree/master)
  * [Evading EDR – Matt Hand](https://www.amazon.com/Evading-EDR-Definitive-Defeating-Detection/dp/1718503342)



#### References

  * [VX-API](https://github.com/vxunderground/VX-API)
  * [VX-API GitBook](https://vx-api.gitbook.io/vx-api/)
  * [Unprotect Project](https://unprotect.it/)
  * [FileSec](https://filesec.io/)
  * [MalAPI](https://malapi.io/)



#### Operational Security

  * [Red Team Operational Security Explained](https://www.youtube.com/watch?v=S7-P4rmWPAc)
  * [Black Hills – Operational Security Fundamentals](https://www.youtube.com/watch?v=AHwfV3NFlno)
  * [When Cybercriminals with Good OPSEC Attack](https://www.youtube.com/watch?v=zXmZnU2GdVk)



#### Detection Awareness

  * [The DFIR Report](https://thedfirreport.com/) – Learn from adversaries’ OPSEC failures.
  * [EDR Telemetry](https://www.edr-telemetry.com/)
  * [Sigma Rules](https://socprime.com/blog/sigma-rules-the-beginners-guide/) – Learn how blue teams build detection logic.
  * [Sysmon Configuration](https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml) – Familiarize yourself with common logging configurations. If a TTP is covered by Atomic Red Team, consider it a strong hint that defenders may have detection coverage for it.
  * [Sigma Rules – Windows](https://github.com/SigmaHQ/sigma/tree/master/rules/windows)
  * [Cobalt Strike OPSEC Considerations](https://www.cobaltstrike.com/blog/opsec-considerations-for-beacon-commands)



#### Useful Pages

  * [LOTS Project](https://lots-project.com/)
  * [LOLBAS](https://lolbas-project.github.io/)
  * [HijackLibs](https://hijacklibs.net/)
  * [ired.team](https://www.ired.team/)
  * [TLDRBins](https://tldrbins.github.io/)
  * [The Hacker Recipes](https://www.thehacker.recipes/)



#### Training

  * [SpecterOps – Red Team Operations](https://specterops.io/training/red-team-operations/)
  * [RogueLabs – ROPTS RT1](https://www.roguelabs.io/rops-rt1)
  * [Binary Offensive](https://binary-offensive.com/guild)



#### Books

  * [Red Team Development and Operations – Joe Vest](https://redteam.guide/)



#### Practical EDR Evasion

Deploy the client’s EDR in a lab and test your payload against it. When something gets detected, identify the behavior or artifact that triggered the alert, research the detection mechanism, then modify your tradecraft to address it.

The workflow is essentially:

> **Deploy EDR → Run payload → Observe detection → Identify the cause → Research → Address the detection surface → Retest**

Don’t blindly apply evasion techniques. Understand what is being detected and why, then solve that specific detection problem. Keep useful telemetry such as Sysmon, ETW, Windows Event Logs, memory and call-stack visibility, and network telemetry available so you can correlate EDR alerts with what actually happened.

For example, if a loader is detected, determine whether the signal came from the executable itself, process behavior, memory characteristics, call stacks, or network activity. Address the relevant detection surface rather than changing unrelated parts of the payload.

The same principle applies to techniques such as Kerberoasting: understand the underlying telemetry and expected baseline first, then determine what behavior distinguishes your activity from normal operations.

#### Resources from Colleagues

  * [Cerbersec Notes](https://github.com/Cerbersec/notes)
  * [Maldev Links by CodeXTF2](https://github.com/CodeXTF2/maldev-links)
  * [Lavender](https://github.com/stars/Lavender-exe/lists/malware-development)



#### Archives

  * [elhacker](https://elhacker.info/)



* * *

### Reverse Engineering & Pwn Resources

#### Foundational Learning

  * [Dayzerosec – Getting Started](https://dayzerosec.com/blog/2024/07/11/getting-started-2024.html)
  * [Secnate – Exploit Development](https://secnate.github.io/resources/exploit-development/)
  * [OpenSecurityTraining2 – x86-64 Assembly](https://p.ost2.fyi/courses/course-v1:OpenSecurityTraining2+Arch1001_x86-64_Asm+2021_v1/about)
  * [Z0F Reverse Engineering Course](https://www.debugxp.com/posts/RECourse/)
  * [Nightmare](https://guyinatuxedo.github.io/00-intro/index.html)
  * [pwn.college](https://pwn.college/)



#### Applied Reverse Engineering

  * [SpecterOps – Vulnerability Research for Operators](https://specterops.io/training/vulnerability-research-for-operators/)
  * [wetw0rk](https://wetw0rk.github.io/)
  * [Wargames by RET2](https://wargames.ret2.systems/)



#### Advanced & Kernel Exploitation

  * [HEVD](https://mdanilor.github.io/posts/hevd-0/)
  * [HackSys Extreme Vulnerable Driver](https://github.com/hacksysteam/HackSysExtremeVulnerableDriver)
  * [OSED Resource Collection](https://github.com/nop-tech/OSED/tree/main)



#### Hands-On Practice

Challenge-based platforms and exercises to reinforce your skills.

  * [crackmes.one](https://crackmes.one/)
  * [pwnable.tw](https://pwnable.tw/challenge/)
  * [PicoCTF](https://picoctf.org/)
  * [Exploit Education](https://exploit.education/)



#### Blogs

  * [exploits.club](https://blog.exploits.club/)



### Contributors

Thanks to the following people:

  * [@Lattice23](https://github.com/Lattice23) – Helped with resources and structure.


