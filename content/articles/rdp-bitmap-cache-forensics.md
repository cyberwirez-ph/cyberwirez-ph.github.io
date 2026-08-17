---
title: "Reconstructing Adversary Activity from Visual Artifacts: RDP Bitmap Cache Forensics"
date: 2026-06-29
description: "How RDP bitmap cache artifacts give investigators visual context on attacker activity when logs are incomplete."
author: "seif"
draft: false
---

*Originally published by [seif](https://medium.com/@seiferboado101) on Medium*


![image](/images/articles/rdp-bitmap-cache-forensics/img0.png)


This month I wanted to write about something that doesn’t get talked about much, but has recently caught my eye during my studies on lateral movement: **RDP bitmap cache artifacts**.

RDP shows up everywhere in the kill-chain: initial access, lateral movement, post-exploitation. We usually talk about logs, registry keys, or authentication events. But what often gets overlooked is that RDP clients don’t just log activity. They **cache parts of what was rendered on screen**.

And sometimes, those cached fragments are the closest thing you’ll get to seeing what an attacker actually did.

This very short and sweet post is about what RDP bitmap cache artifacts are, where they live, and how they can add context during DFIR investigations, especially when other telemetry is missing or incomplete.

### What Is the RDP Bitmap Cache?

RDP bitmap caching exists for performance reasons.

Instead of constantly retransmitting static visual elements (window borders, icons, backgrounds), the RDP client stores bitmap tiles locally and reuses them when needed.

From a DFIR perspective, this means fragments of the remote screen may persist locally, including:

  * Desktop elements
  * Application windows
  * File Explorer views
  * Command prompts
  * Tool interfaces



They are **not screenshots** , and they’re not complete images, **BUT** they are real visual artifacts created during interactive sessions.

### Where These Artifacts Live (Windows)

On Windows systems, RDP bitmap cache files are typically stored under the user profile that initiated the RDP session:
    
    
    %LOCALAPPDATA%\Microsoft\Terminal Server Client\Cache\

You’ll usually see files with extensions like:

  * .bmc
  * .bin
  * .dat



Each file contains compressed bitmap fragments generated as the RDP session renders content.

### Why Bitmap Cache Artifacts Matter in DFIR

Logs can tell you **that** RDP occurred.

Bitmap cache artifacts can help you understand **what happened during the session**.

In practice, they can:

  * Confirm interactive attacker presence
  * Provide visual context for tools executed
  * Support findings when logs are cleared or incomplete
  * Help explain timeline gaps



They’re especially useful when attackers rely heavily on living-off-the-land tools or deliberately minimize telemetry.

### Limitations & Challenges

Bitmap cache forensics isn’t easy, and it’s not always rewarding.

Some realities to keep in mind:

  * Cache files are fragmented and unordered
  * Tiles represent partial screen elements
  * No native Windows viewer exists
  * Reconstruction can be time-consuming
  * Results vary heavily by activity



Because of this, bitmap cache analysis works best as **supporting evidence** , not a primary source of truth.

### High-Level Reconstruction Workflow


![image](/images/articles/rdp-bitmap-cache-forensics/img1.jpg)

![image](/images/articles/rdp-bitmap-cache-forensics/img2.png)

![image](/images/articles/rdp-bitmap-cache-forensics/img3.jpg)


At a high level, a practical workflow looks like this:

  1. **Collection**


  * Acquire cache files carefully (with a tool like [BMC-Tools)](https://github.com/ANSSI-FR/bmc-tools)
  * Preserve timestamps and directory structure



**2\. Identification**

  * Locate bitmap headers and compressed segments
  * Separate individual fragments



**3\. Extraction**

  * Carve bitmap tiles
  * Normalize size and color depth where possible



**4\. Reconstruction**

  * Manually or programmatically piece tiles together
  * Look for recognizable UI elements



**5\. Correlation**

  * Align findings with timelines
  * Correlate with logs, Prefetch, Amcache, and network data



This step is where most of the value comes from — correlation matters more than perfect reconstruction.

### What You Can (and Can’t) Expect to Recover

**You might recover:**

  * Portions of command-line output
  * File browsing activity
  * Application interfaces
  * Desktop elements



**You should not expect:**

  * Full session replays
  * Continuous activity visibility
  * Perfect clarity



Bitmap cache artifacts**are only useful when paired with other data.**

### Why Should I Still Check Bitmap Cache Artifacts Then?

In multiple investigations, bitmap cache artifacts have helped:

  * Confirm hands-on keyboard activity
  * Identify tooling not visible elsewhere
  * Strengthen incident reports with visual artifacts
  * Support scope expansion decisions



They’ve been particularly helpful in slower intrusions, where attackers are cautious, and RDP usage is limited.

Not every case benefits from this analysis, but when it does, it’s worth the effort.

### When Bitmap Cache Analysis Is Worth Your Time

I would prioritize bitmap cache analysis when:

  * RDP usage is suspected but poorly logged
  * Credential abuse indicators exist
  * Multiple remote access tools are involved
  * There are unexplained gaps in the timeline



If someone interacted with the system through RDP, there’s a chance something visual was cached.

### Final Thoughts

RDP bitmap cache forensics isn’t flashy.

It’s fragmented, imperfect, and easy to overlook, but it can add something many investigations lack: **visual context**.

In DFIR, context changes conclusions.

And sometimes the most useful artifacts are the ones no one remembers to check.
