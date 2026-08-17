---
title: "How Simple Documents Can Be Weaponized: Simulating a Host-Based Attack Using Malicious PDFs"
date: 2026-08-08
description: "A controlled exercise embedding a reverse shell payload in a PDF to test host-based defenses."
author: "seif"
draft: false
---

*Originally published by [seif](https://medium.com/@seiferboado101) on Medium*



PDFs are pretty much everywhere (in both professional and personal communication). They’re used for invoices, contracts, resumes, and countless other purposes.

However, their widespread use and inherent trust make them a prime target for exploitation.

Today, I embarked on a hands-on project to explore just how attackers weaponize PDFs to deliver malicious payloads and bypass defenses. Here’s a detailed breakdown of what I did and what I learned along the way.

## Objective: Weaponizing PDFs for Security Awareness

The goal of this project was to simulate a realistic attack scenario using a malicious PDF. This exercise was conducted in a controlled environment to better understand the techniques attackers use and the defenses modern systems employ. It also served as a valuable exercise in debugging and adapting to evolving security measures.

I used a vulnerable Windows 10 machine as the “victim” and a Kali VM for my “adversary”.

![image](/images/articles/malicious-pdf-weaponization/img0.png)

Now, moving on to actual steps in the attack:

## Step 1: Setting the Stage

### Generating the Payload

To start, I used the powerful Metasploit Framework to create a reverse shell payload embedded in a PDF file. The reverse shell would give me control of the target system if successfully executed.

Here’s the command I used:

![image](/images/articles/malicious-pdf-weaponization/img1.png)

The Metasploit Modules Used

![image](/images/articles/malicious-pdf-weaponization/img2.png)

Setting the Options

![image](/images/articles/malicious-pdf-weaponization/img3.png)

The Malicious PDF

  * **Payload** : `windows/x64/meterpreter/reverse_tcp` – Chosen for its compatibility with modern 64-bit Windows systems.
  * **LHOST and LPORT** : Set to my Kali machine’s IP and a listening port.
  * **Output Format** : PDF.

The generated file, `malicious.pdf`, was ready for testing.

### Hosting the PDF

To make the attack more realistic, I hosted the malicious PDF on a web server with a convincing interface mimicking a legitimate document portal. I created a simple HTML file with a download button

![image](/images/articles/malicious-pdf-weaponization/img4.png)

![image](/images/articles/malicious-pdf-weaponization/img5.png)

I hosted this on my Kali machine using Python’s HTTP server

Victims would visit `http://<Your_Kali_IP>`and download the file.

## Step 2: Setting Up the Listener

To catch the reverse shell connection when the PDF was opened, I set up a Metasploit listener:

  1. Start Metasploit Framework:
  2. Configure the multi-handler module:

![image](/images/articles/malicious-pdf-weaponization/img6.png)

  1. The listener waited for the target to execute the payload.

## Step 3: Testing on Windows 10

### Execution

On the victim machine (a Windows 10 VM), I downloaded and opened the PDF. However, the payload failed to execute as intended. The Meterpreter session briefly appeared in the Metasploit console before dying with the error:

![image](/images/articles/malicious-pdf-weaponization/img7.png)

The Downloaded Malicious PDF

![image](/images/articles/malicious-pdf-weaponization/img8.png)

The Malicious PDF Being Ran
    
    
    [*] Sending stage (176198 bytes) to 10.0.2.12  
    [*] Meterpreter session 1 closed.  Reason: Died

### Troubleshooting

**Payload Compatibility** :

  * I confirmed the target system was 64-bit and switched to the `windows/x64/meterpreter/reverse_tcp` payload.

When that finally worked, I finally “got in”.

![image](/images/articles/malicious-pdf-weaponization/img9.png)

Stage being sent

![image](/images/articles/malicious-pdf-weaponization/img10.png)

Now I am able to enumerate and learn more about the victim system

![image](/images/articles/malicious-pdf-weaponization/img11.png)

This next step is only a simulation, but the real damage can start once the adversary is in. There are multiple ways that they can move laterally (to other systems), maintain persistence, and even gain access to confidentials info (like in the example below where sample credentials are discovered by the “adversary”, which could be used to try against different services)

![image](/images/articles/malicious-pdf-weaponization/img12.png)

## Step 4: Lessons Learned

### Modern Defenses Work

Windows 10’s built-in protections and application sandboxing are highly effective at thwarting traditional PDF-based exploits. Attackers must continuously innovate to bypass these defenses. Given this, defenders also must adapt and not rely on built-in defense mechanisms for hosts.

### Awareness is Key

Users should always verify the source of documents before downloading or opening them. Training on recognizing phishing attempts and suspicious file behaviors is essential.

### Adapting Tactics

For redteamers, when one method fails, switching payload types, trying another route, and refining delivery techniques can allow one to understand the layers of security in play, and when vulnerabilities are found, exploit them.

## Conclusion

This project was a valuable exercise in understanding how PDFs can be weaponized and how modern defenses counter these threats. It reinforced the importance of:

  1. Keeping systems updated to leverage the latest security features.
  2. Educating users about phishing and file-based exploits.
  3. Testing defenses in a controlled environment to identify and mitigate potential weaknesses.

Have you ever thought about how everyday files like can be used maliciously (in other different ways)? Let’s discuss in the comments below!
