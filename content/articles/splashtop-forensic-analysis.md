---
title: "Deep Dive Forensic Analysis: Splashtop in Incident Response"
date: 2026-07-06
description: "Forensic artifacts left behind by Splashtop remote access software — install, usage, and file-transfer traces."
author: "seif"
draft: false
---

*Originally published by [seif](https://medium.com/@seiferboado101) on Medium*


![image](/images/articles/splashtop-forensic-analysis/img0.jpg)


### Introduction

New month, new blog as usual.

Remote administration tools show up in IR investigations more often than people expect. They’re widely used by IT teams for legitimate purposes, but they’re also frequently abused by attackers because they blend into normal administrative activity, especially over the recent years.


![image](/images/articles/splashtop-forensic-analysis/img1.png)
Any.Run Infographic on the Top RMM Tools Exploited by Threat Actors

This time, I wanted to cover an RMM tool I came across with the labs I work with, and have had some trouble with the first time, which is **Splashtop**


![image](/images/articles/splashtop-forensic-analysis/img2.png)


We usually see the usual threat actor favorites like AnyDesk and TeamViewer, but one RMM tool that I’ve seen that’s been a bit more trending is **Splashtop.**

Regarding RMMs, from an incident response perspective, this creates an interesting challenge. When you find a remote access tool on a system, the question is rarely just _“is this malicious?”_ The real question becomes _“_** _how was this tool used?_**_”_

**Understanding the forensic artifacts** left behind by these tools makes a big difference when reconstructing attacker activity. Installation traces, service creation, log files, network artifacts, and registry entries can all help determine whether a tool was installed legitimately or used as part of an intrusion.

In this article, I’m going to take a deeper look at **Splashtop** from a forensic perspective. Using a small lab environment, I walked through the installation and usage of Splashtop and documented the artifacts it leaves behind across the system.

The goal here is to highlight the **artifacts that are actually useful during an investigation.**

I’m talking about the ones that can help answer questions like:

  * When was Splashtop installed?
  * Was a remote session initiated? And when?
  * Was file transfer used?
  * What remote system is connected to the host? (IP Address, User Name, Workstation Name, etc.)



### Reference Research

This research was heavily inspired by the excellent work from **Synacktiv (author: Théo Letailleur)** , who conducted a comprehensive forensic analysis of legitimate remote administration tools.


![image](/images/articles/splashtop-forensic-analysis/img3.png)


Their research paper:

**Legitimate RATs: A Comprehensive Forensic Analysis of the Usual Suspects**

provides an extensive technical breakdown of artifacts left by various legitimate remote access tools.

Reference:  
<https://www.synacktiv.com/publications/legitimate-rats-a-comprehensive-forensic-analysis-of-the-usual-suspects>

Their work serves as an excellent foundation for defenders investigating environments where legitimate administrative tools may have been abused by threat actors.

This article builds upon their findings and demonstrates how these artifacts appear in practice through a controlled forensic lab.

### Installation

During installation, Splashtop deploys several components responsible for remote connectivity, system communication, and automatic updates.

The main executables installed include:

Splashtop Remote ServiceC:\Program Files (x86)\Splashtop\Splashtop Remote\Server\SRService.exe

Splashtop Remote AgentC:\Program Files (x86)\Splashtop\Splashtop Remote\Server\SRAgent.exe

Splashtop UpdaterC:\Program Files (x86)\Splashtop\Splashtop Software Updater\SSUAgent.exe

Utility binariesSRUtility.exe, SRFeature.exe


![image](/images/articles/splashtop-forensic-analysis/img4.png)


Once installed, Splashtop establishes persistence primarily through **Windows services** and system-level configuration changes.

These services ensure the remote agent launches automatically on system startup and remains available for remote connections.

### Logs Generated

One of the most useful sources of forensic evidence when investigating Splashtop is the set of **logs generated during remote sessions**.

When Splashtop installs its service (SplashRemoteService), it creates two **Windows Event Log channels** :

### Splashtop Remote Session Log
    
    
    Splashtop-Splashtop Streamer-Remote Session/Operational

This log records events related to remote activity such as:

  * Remote session creation
  * File transfers
  * Client hostname
  * Session identifiers



Example event:


![image](/images/articles/splashtop-forensic-analysis/img5.png)
Remote Session Connection from Splashtop’s Web Client
![image](/images/articles/splashtop-forensic-analysis/img6.png)
Remote Session Connection from Splashtop Business Revealing the Connecting Device Name
![image](/images/articles/splashtop-forensic-analysis/img7.png)
Remote Session End Event

This information is extremely valuable during incident response because it allows investigators to determine:

  * When remote sessions occurred
  * Which files were transferred
  * The remote system initiating the connection



### Splashtop Status Log
    
    
    Splashtop-Splashtop Streamer-Status/Operational

This event log tracks the operational state of the Splashtop service.

Example entry:


![image](/images/articles/splashtop-forensic-analysis/img8.png)
Splashtop Event of Addition to a Team

These entries help determine:

  * When the service became active
  * Which relay servers were used
  * When the system connected to Splashtop infrastructure



### Log Files

In addition to Windows event logs, Splashtop creates several log files on disk.

### File Transfer Logs


![image](/images/articles/splashtop-forensic-analysis/img9.png)
Sample File Transfer Using Splashtop Business

Location:
    
    
    %PROGRAMDATA%\Splashtop\Temp\log\FTCLog.txt


![image](/images/articles/splashtop-forensic-analysis/img10.png)
Sample Log of a File Transfer

This log tracks file transfers performed during remote sessions.

From a forensic standpoint, this log reveals:

  * The file transferred
  * The remote user account
  * The remote IP address
  * Timestamp of the transfer



This makes it extremely useful for identifying potential **data exfiltration or tool deployment**.

### Agent Logs

Location:
    
    
    C:\Program Files (x86)\Splashtop\Splashtop Remote\Server\log\agent_log.txt

This log functions primarily as a **debug log** for the Splashtop agent.

Although less useful for direct investigation, it can help confirm agent activity and troubleshooting events.

### General Agent Logs

Location:
    
    
    C:\Program Files (x86)\Splashtop\Splashtop Remote\Server\log\SPLog.txt

This log contains a wide range of operational information, including:

  * Client hostname
  * User display name
  * Client IP address
  * Relay servers used
  * File transfer operations
  * Chat functionality



Example log entries:


![image](/images/articles/splashtop-forensic-analysis/img11.png)
SPLog User Name Connection
![image](/images/articles/splashtop-forensic-analysis/img12.png)
An Attempted File Transfer Event on SPLog

These logs provide investigators with strong indicators of **who connected, from where, and what actions were performed**.

### Chat Logs

If the chat functionality is used during a remote session, Splashtop allows users to save chat transcripts locally.

Default format:
    
    
    Splashtop_Chat_YYYYMMDD_HHMM.txt

Example:


![image](/images/articles/splashtop-forensic-analysis/img13.png)


These logs may provide **direct conversational evidence** between an attacker and a victim system.

### Artifacts Created and Use Cases

### Network Artifacts

During operation, Splashtop communicates with relay servers hosted by Splashtop infrastructure.

Example server:
    
    
    *.splashtop.com (api.splashtop.com, relay.splashtop.com)

These connections are typically encrypted and may occur over common ports such as HTTPS, allowing them to blend into normal network traffic.

However, investigators can still identify:

  * Relay server domains
  * Client public IP addresses
  * Remote session timing



These indicators may be extracted from both **network captures and application logs**.

### Executables

Several Splashtop binaries are deployed on installation:


![image](/images/articles/splashtop-forensic-analysis/img14.png)

    
    
    SRService.exe  
    SRAgent.exe  
    SRUtility.exe  
    SRFeature.exe  
    SSUAgent.exe

Each of these executables plays a role in maintaining remote connectivity and updating the agent.

From a forensic perspective, execution of these binaries may appear in several forensic artifacts such as:

  * **BAM (Background Activity Monitor)**
  * **UserAssist**
  * **ShimCache**
  * **AmCache**
  * **JumpLists**
  * **Prefetch**



These artifacts allow investigators to determine whether the binaries were executed and potentially reconstruct timelines of activity.

### Registry Artifacts

Splashtop installation creates several registry keys that can confirm its presence on a system.

Key locations include:
    
    
    HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WINEVT\Channels\Splashtop-Splashtop Streamer-Remote Session/Operational  
    HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WINEVT\Channels\Splashtop-Splashtop Streamer-Status/Operational  
    HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\Splashtop Software Updater  
    HKLM\SOFTWARE\WOW6432Node\Splashtop Inc.  
    HKLM\SYSTEM\ControlSet001\Services\SplashtopRemoteService  
    HKLM\SYSTEM\ControlSet001\Control\SafeBoot\Network\SplashtopRemoteService  
    HKU\.DEFAULT\Software\Splashtop Inc.  
    HKU\SID\Software\Splashtop Inc.

These registry entries are strong indicators that Splashtop was installed and configured on the host.

### Service Creation

Splashtop installs multiple services to maintain persistence.

These can be observed through **Windows Event ID 7045**.

### Splashtop Software Updater
    
    
    ServiceName: Splashtop Software Updater Service  
    ImagePath: C:\Program Files (x86)\Splashtop\Splashtop Software Updater\SSUService.exe  
    StartType: Automatic

### Splashtop Remote Service
    
    
    ServiceName: Splashtop Remote Service  
    ImagePath: C:\Program Files (x86)\Splashtop\Splashtop Remote\Server\SRService.exe  
    StartType: Automatic


![image](/images/articles/splashtop-forensic-analysis/img15.png)


These services ensure the remote agent remains active and reconnects automatically after system reboot.

### Databases

Splashtop also maintains **encrypted SQLite databases** that store agent data.

Locations include:
    
    
    C:\Program Files (x86)\Splashtop\Splashtop Remote\Server\db\SRAgent.sqlite3  
    %PROGRAMDATA%\Splashtop\Splashtop Remote Server\Credential\<random>.sqlite3

Additionally, several encryption keys are stored in the credential directory:
    
    
    SDCredPubKey  
    SDCredPriKey  
    SDValidationKey

According to [Synacktiv’s report](https://www.synacktiv.com/publications/legitimate-rats-a-comprehensive-forensic-analysis-of-the-usual-suspects), these databases appear to contain mostly **host information such as installed Windows updates and system metadata**.

### Conclusion

Splashtop is a legitimate remote administration tool widely used across enterprise environments. However, its capabilities also make it attractive for attackers seeking persistent remote access to compromised systems.

From a forensic standpoint, Splashtop leaves behind a significant number of artifacts that can assist investigators in reconstructing attacker activity. These artifacts include:

  * Dedicated Windows Event Log channels
  * Detailed log files containing session information
  * File transfer records
  * Service creation events
  * Registry keys indicating installation
  * Execution traces in common forensic artifacts
  * Encrypted SQLite databases storing system metadata



From a forensic perspective, Splashtop leaves behind a decent amount of artifacts that can help during an investigation. Between the event logs, local log files, service creation events, registry keys, and execution traces across common forensic artifacts,**there are multiple ways to confirm** that the tool was installed and understand how it was used on a system.

In environments where legitimate remote administration tools are common, the challenge isn’t just identifying the tool.

It’s figuring out **how it was used and whether the activity makes sense for the environment**.

This was a small lab exercise to validate some of the artifacts documented in previous research and see how they actually appear on a system. There’s a good chance I missed something or that there are additional artifacts worth looking at.

**If you’ve come across other Splashtop artifacts during investigations or have additional findings, feel free to share or reach out**. The more we document these tools from a forensic perspective, the easier it becomes for the community to investigate when they’re abused.
