---
title: "Detection Rules That Matter: Writing Bad, Good, and Great Sigma/YARA Rules"
date: 2026-07-20
description: "What separates poor detection rules from great ones — precise logic, context, and continuous upkeep."
author: "seif"
draft: false
---

*Originally published by [seif](https://medium.com/@seiferboado101) on Medium*


![image](/images/articles/detection-rules-sigma-yara/img0.png)


**Core Thesis:**

A detection rule is not great because it runs. It is great because the SOC can trust it: the logic is precise, the context is clear, the false positives are understood, and the rule is maintained as the environment changes.

### Detection Rules That Matter: How to Write Bad, Good, and Great Sigma/YARA Rules

Detection rules are easy to write. Good detection rules are harder. Great detection rules are what actually matter.

A bad rule creates noise. A good rule detects suspicious activity. A great rule gives the security team a high-confidence signal, enough context to investigate quickly, and a structure that can be maintained as the environment changes.

**_That difference is the heart of detection engineering._**

In this blog, I’ll do my best to break down what makes a detection rule bad, good, and great, with examples using Sigma and YARA. We will also cover why organizational context matters, why rules need continuous maintenance, and how defenders can move from simple alerting to reliable detection coverage.

### What Detection Engineering Really Means

Detection engineering is the process of designing, building, testing, deploying, tuning, and maintaining detections that identify suspicious or malicious activity. It is not just writing a SIEM query. It is not just copying logic from a threat report. It is not just matching an indicator of compromise.

A detection engineer needs to understand the attacker behavior, the telemetry that proves the behavior happened, whether the activity is actually suspicious in the organization, what legitimate activity may look similar, what context an analyst needs during investigation, and how the rule will be maintained over time.

The final detection rule is only the output. The real work is the thinking behind it.

**The operating question:**

A detection rule should answer one key question: why should the SOC care about this activity? If the rule cannot answer that, it probably is not ready.

### Sigma and YARA: Same Mission, Different Targets


![image](/images/articles/detection-rules-sigma-yara/img1.png)


### Sigma: Detection Logic for Logs

[Sigma ](https://github.com/SigmaHQ/sigma)is commonly used to describe suspicious behavior in logs. It is written in YAML and designed to be SIEM-agnostic, which means the same detection idea can be translated into different SIEM query languages depending on the organization’s tooling.

Sigma is useful when the question is: what happened in the environment? It is commonly used for Windows Event Logs, Sysmon, PowerShell logs, EDR telemetry, authentication logs, cloud logs, process creation events, and network connection events.

A Sigma rule is usually focused on behavior: suspicious PowerShell usage, abnormal parent-child process relationships, credential dumping attempts, suspicious scheduled task creation, remote service creation, or risky authentication patterns.

### YARA: Detection Logic for Files and Memory

[YARA ](https://docs.virustotal.com/docs/what-is-yara)is commonly used to identify and classify files, malware samples, and memory artifacts based on strings, byte patterns, metadata, and logical conditions. It is useful when the question is: what does this artifact contain?

YARA is often used for malware family detection, suspicious binary triage, webshell detection, memory scanning, file triage, malware research, threat hunting, and incident response validation.

A YARA rule is usually focused on file or memory characteristics: unique strings, suspicious PE structure, hardcoded command-and-control patterns, debug paths, embedded configuration markers, shellcode patterns, or reused attacker toolmarks.

### The Core Framework: Bad, Good, and Great


![image](/images/articles/detection-rules-sigma-yara/img2.png)


Not all detection rules are equal. A rule can be syntactically valid and still be operationally useless. A rule can technically detect something and still create more work than value. A rule can match malicious behavior in a lab but fail in a real enterprise.

This is why we need to think beyond “does the rule work?” and ask a better question: **_does the rule work well enough for the environment it will live in?_**

### What Makes a Bad Detection Rule?

A bad detection rule is usually **too broad, too shallow, too noisy, or disconnected from the real environment**. It might detect something, but it does not give defenders enough confidence or context to act.

### A Bad Rule Detects a Tool Instead of a Threat

Here is a common example: detecting PowerShell execution by itself.
    
    
    title: PowerShell Execution  
    logsource:  
    product: windows  
    category: process_creation  
    detection:  
      selection:  
      Image|endswith: '\powershell.exe'  
    condition: selection  
    level: low

At first glance, this looks like a detection. But what is it really detecting? PowerShell. That is LITERALLY it.

PowerShell is used by administrators, deployment tools, security tools, scripts, maintenance tasks, and attackers. Detecting PowerShell by itself is not enough.

This rule does not tell us whether the activity is malicious, suspicious, unusual, or expected. It detects a tool, not a threat.

### A Bad Rule Uses Generic Strings

The same issue appears in YARA when a rule relies on weak, generic strings.
    
    
    rule Bad_Generic_Detection  
    {  
      strings:  
        $a = "http"  
        $b = "cmd.exe"  
      condition:  
        any of them  
    }

This rule will match many legitimate files. The strings are too generic. “http” and “cmd.exe” can appear in countless benign programs, scripts, installers, and system tools. This rule does not identify a malware family, unique behavior, or meaningful pattern. It is not a detection. It is a noise generator.

### A Bad Rule Has No Context

A bad rule might alert with a vague title like “Suspicious Activity Detected.” That title is not useful. Suspicious how? Suspicious where? Suspicious because of what?

An analyst should not have to reverse-engineer the rule just to understand why it fired. **A detection rule should clearly explain what behavior was detected, why the behavior matters, what data source produced the alert, what the analyst should investigate, and what false positives are expected.**

### A Bad Rule Ignores False Positives

A bad rule does not consider legitimate activity. For example, a rule detecting encoded PowerShell may be useful, but in some environments endpoint management tools or automation frameworks may also use encoded PowerShell.

If the rule does not account for that, the SOC may be flooded with alerts from IT administration scripts, software deployment tools, patch management systems, security scanners, backup tools, remote monitoring platforms, developer activity, service accounts, and scheduled tasks.

**_A rule that ignores false positives will not survive production._**

### A Bad Rule Has No Maintenance Plan

This is one of the biggest problems. A rule may be useful when first deployed, but environments change. New tools are added. Logging pipelines change. Field mappings change. Business processes change. Attackers adapt. SOC priorities shift.

**_A detection rule that is not reviewed, tuned, and maintained will eventually become stale._** Bad detection engineering treats rules as one-time work. Great detection engineering treats rules as living security content.

### What Makes a Good Detection Rule?

A good detection rule moves beyond broad matching and begins detecting meaningful suspicious behavior. It has clearer logic, better metadata, and more useful context.

### Good Sigma Example: Encoded PowerShell

title: PowerShell Encoded Command Execution
    
    
    id: 2e5f4a32–4c43–4a72-ae3d-encoded-example  
    status: experimental  
    description: Detects PowerShell execution using encoded command arguments, which may be used to hide command content during execution.  
    author: Detection Engineering Team  
    date: 2026/05/19  
    logsource:  
    product: windows  
    category: process_creation  
    detection:  
      selection_image:  
        Image|endswith:  
          - '\powershell.exe'  
          - '\pwsh.exe'  
       selection_encoded:  
        CommandLine|contains:  
          - '-enc'  
          - '-encodedcommand'  
          - '/enc'  
    condition: selection_image and selection_encoded  
    falsepositives:  
    - Administrative scripts  
    - Endpoint management tooling  
    - Software deployment automation  
    level: medium  
    tags:  
    - attack.execution  
    - attack.t1059.001

This is better because it does not just detect PowerShell. It detects PowerShell being used in a suspicious way. Encoded commands are commonly used to hide command content. That does not automatically mean malicious activity, but it is more meaningful than simply detecting powershell.exe.

The rule also includes a specific title, description, log source, detection logic, false positives, severity, and ATT&CK mapping. That makes it operationally stronger. But it can still be improved.

### Good YARA Example: Suspicious Downloader Traits
    
    
    rule Suspicious_Downloader_Generic  
    {  
      meta:  
        description = "Detects files with multiple suspicious downloader-like strings"  
        author = "Detection Engineering Team"  
        date = "2026–05–19"  
        confidence = "medium"  
      strings:  
        $s1 = "DownloadFile" ascii  
        $s2 = "User-Agent" ascii nocase  
        $s3 = "cmd.exe /c" ascii  
        $s4 = "powershell" ascii nocase  
      condition:  
        2 of ($s*)  
    }

This is better than matching one generic string because it requires multiple traits to be present. That reduces false positives. But it is still generic. It may be useful for hunting or triage, but it may not be specific enough for high-confidence malware detection.

### What Makes a GREAT Detection Rule?

A great detection rule is not just accurate. It is operationally mature. It is written with the organization in mind. It is tested, tuned, documented, owned, and maintained.

**Great rule standard**

A great rule does not only ask, “Can we detect this?” It asks, “Can we detect this reliably in our environment, with enough context for the SOC to act?”

Great rules have behavioral precision, telemetry alignment, organizational context, a false positive strategy, analyst-ready guidance, proper severity, accurate ATT&CK mapping, test evidence, known limitations, maintenance ownership, review cadence, and version control.

### Anatomy of a Great Sigma Rule

Let us improve the encoded PowerShell rule. Instead of only detecting encoded PowerShell, we can make the rule more context-aware. PowerShell launched by Office applications with encoded or download behavior is much more suspicious than PowerShell launched by an approved IT automation platform.

### Great Sigma Example: Context-Aware Suspicious PowerShell

title: Suspicious PowerShell Execution from Office Application with Encoded or Download Behavior
    
    
    id: 8b67f243–75d2–4c1e-9a28-great-sigma-example  
    status: test  
    description: >  
    Detects PowerShell or PowerShell Core execution from Microsoft Office parent processes  
    with encoded command or download-related arguments. This behavior may indicate macro-based  
    execution, payload staging, or malicious script execution. The rule prioritizes higher-risk  
    PowerShell activity by combining process lineage, command-line indicators, and organizational  
    false positive context.  
    author: Detection Engineering Team  
    date: 2026/05/19  
    modified: 2026/05/19  
    logsource:  
    product: windows  
    category: process_creation  
    detection:  
      selection_image:  
        Image|endswith:  
          - '\powershell.exe'  
          - '\pwsh.exe'  
      selection_parent_office:  
        ParentImage|endswith:  
          - '\winword.exe'  
          - '\excel.exe'  
          - '\powerpnt.exe'  
          - '\outlook.exe'  
          - '\msaccess.exe'  
          - '\onenote.exe'  
      selection_suspicious_args:  
        CommandLine|contains:  
          - '-enc'  
          - '-encodedcommand'  
          - '/enc'  
          - 'FromBase64String'  
          - 'DownloadString'  
          - 'DownloadFile'  
          - 'Invoke-WebRequest'  
          - 'iwr '  
          - 'curl '  
          - 'wget '  
          - 'IEX'  
          - 'Invoke-Expression'  
      filter_known_admin_tools:  
        ParentCommandLine|contains:  
          - 'Approved_Office_Addin_Name'  
          - 'Known_Internal_Automation'  
        filter_known_service_accounts:  
        User|contains:  
          - 'svc_softwaredeploy'  
          - 'svc_endpointmgmt'  
    condition: selection_image and selection_parent_office and selection_suspicious_args and not 1 of filter_known_*  
    fields:  
    - UtcTime  
    - Computer  
    - User  
    - Image  
    - CommandLine  
    - ParentImage  
    - ParentCommandLine  
    - ProcessId  
    - ParentProcessId  
    - CurrentDirectory  
    - Hashes  
    falsepositives:  
    - Approved Office add-ins that launch PowerShell for business workflows  
    - Internal automation using Office-generated scripts  
    - Security testing or phishing simulation activity  
    - Endpoint management activity using known service accounts  
    level: high  
    tags:  
    - attack.execution  
    - attack.t1059.001  
    - attack.t1204  
    - attack.t1566.001

### Why This Sigma Rule Is Stronger

This rule does not alert on PowerShell alone. It requires PowerShell or PowerShell Core execution, an Office parent process, suspicious command-line behavior, and exclusions for known approved tools or service accounts.

That creates a much stronger signal. PowerShell is not the detection. Office-spawned PowerShell with encoded or download behavior is the detection. The rule is now closer to an adversary behavior than a generic tool match.

It also gives analysts useful fields such as command line, parent process, user, host, current directory, process IDs, and hashes. That matters because the alert should help the analyst move faster, not force them to start from zero.

### Organizational Context: The Missing Piece

Many detection rules fail because they are technically correct but organizationally blind. A rule from the internet may be valid YAML, but that does not mean it is right for your environment.

In one company, encoded PowerShell may be rare and highly suspicious. In another company, encoded PowerShell may be used every day by IT automation, software deployment, endpoint management, or internal scripts. The same rule can be high fidelity in one environment and noisy in another.

A great detection rule should understand which teams use the behavior legitimately, which service accounts commonly trigger similar activity, which hosts are expected to run the command, whether the behavior is normal on servers but suspicious on workstations, and whether known business applications generate similar logs.

It should also consider crown jewel assets, maintenance windows, approved tools, and the quality of telemetry available. Without that context, a rule may be technically correct but operationally weak.

### Great Rules Need Continuous Maintenance


![image](/images/articles/detection-rules-sigma-yara/img3.png)


A great detection rule is not finished when it is deployed. Deployment is only the beginning. Detection rules need continuous maintenance because the environment changes.

Every rule should have an owner or team responsible for it. If a rule breaks, someone needs to fix it. If false positives increase, someone needs to tune it. If the log source changes, someone needs to validate it. No owner means no accountability.

Rules should also have a review cadence. High-priority or high-noise rules may need monthly review. Stable production rules may need quarterly review. Any major logging, EDR, SIEM, infrastructure, incident response, red team, or purple team change should trigger rule review.

Alert volume should be monitored after deployment. A team should know how often the rule fires, how many alerts are true positives, how many are false positives, which users and hosts trigger it, how much time analysts spend investigating it, and whether the rule leads to meaningful escalation.

False positive tuning should be evidence-based. Broad exclusions can hide risk. Narrow exclusions based on known tools, known paths, known service accounts, or verified business workflows preserve detection value while reducing noise.

Rules should also be version controlled. This allows the team to track who changed the rule, what changed, why it changed, and whether the change improved or weakened the detection. Detection rules should be treated like security code.

### Great Sigma Rules Should Include Analyst Guidance

A great rule should help the SOC investigate. The alert should not just say “Suspicious PowerShell detected.” It should help answer who executed it, which host executed it, what parent process launched it, what command line was used, whether the command was encoded, whether a file was downloaded, whether a network connection followed, and whether the same host triggered other detections.

The faster the analyst understands the alert, the faster they can make a decision. That is why rule metadata, fields, descriptions, and investigation notes matter. A technically clever rule that produces confusing alerts is not a great rule.

### Anatomy of a Great YARA Rule

The same maturity model applies to YARA. A bad YARA rule matches generic strings. A good YARA rule uses multiple suspicious strings. A great YARA rule combines unique indicators, structural traits, metadata, and confidence.

### Great YARA Example
    
    
    import "pe"  
    rule GREAT_Suspicious_Loader_Toolmark_Detection  
    {  
    meta:  
      description = "Detects a suspicious loader based on unique configuration marker, mutex pattern, debug path, and PE structure"  
      author = "Detection Engineering Team"  
      date = "2026–05–19"  
      modified = "2026–05–19"  
      confidence = "high"  
      severity = "high"  
      rule_type = "malware_triage"  
      reference = "Internal malware analysis case"  
      notes = "Use for file triage, malware hunting, and incident response validation"  
    strings:  
      $config_marker = "CONFIG_START::mutex_id::" ascii  
      $debug_path = "C:\Users\dev\source\repos\loader\" ascii  
      $mutex = "Global\{A17F9D2C-Loader-Lock}" ascii  
      $user_agent = "Mozilla/5.0 LoaderClient/3.1" ascii  
      $cmd_pattern = "cmd.exe /c timeout /t" ascii  
    condition:  
      uint16(0) == 0x5A4D and  
      pe.number_of_sections >= 4 and  
      pe.imports("kernel32.dll", "VirtualAlloc") and  
      pe.imports("wininet.dll", "InternetOpenA") and  
      2 of ($config_marker, $debug_path, $mutex, $user_agent, $cmd_pattern)  
    }

This rule is stronger because it does not rely on one weak string. It combines file type validation using the MZ header, PE structure checks, suspicious imports, unique strings, multiple required conditions, metadata, confidence, and intended use.

It is also clear about purpose. This is not just a generic suspicious file rule. It is designed for malware triage, hunting, and incident response validation.

### YARA Rules Also Need Operational Context

A YARA rule used for malware research may not be appropriate for production blocking. A rule used for hunting may be intentionally broader. A rule used for automated quarantine should be much more precise.

Before deploying a YARA rule, the team should understand whether it is for hunting, triage, blocking, or classification. They should know what happens when it matches, whether the confidence is high enough for automated action, whether it has been tested against benign files, whether it matches one sample or a broader family, and whether the strings are unique enough.

YARA rules also age. Malware families change. Packers change. Strings disappear. Code gets refactored. Threat actors reuse tools. Benign software may introduce similar strings. A YARA rule that is never reviewed can become stale or noisy.

### Bad vs Good vs Great in Plain Terms

**Bad rule:**

A bad detection rule detects a tool, generic string, single IOC, common command, or weak behavior without context. The result is noise, and the SOC learns to ignore it.

**Good rule:**

A good detection rule detects suspicious behavior or a known technique with useful metadata, false positives, severity, and basic testing. The result is something the SOC can investigate.

**Great rule:**

A great detection rule detects meaningful adversary behavior in the right organizational context, using reliable telemetry, with analyst guidance, tested logic, ownership, and continuous maintenance. The result is trust.

### A Practical Framework for Writing Great Rules

Start with behavior, not tools. Instead of asking how to detect PowerShell, ask how to detect suspicious PowerShell execution that may indicate payload staging or script-based execution. Instead of asking how to detect a malware hash, ask what unique file traits or behaviors identify that malware family even if the hash changes.

Then identify the right telemetry. For Sigma, that might mean process creation logs, PowerShell Script Block logs, Sysmon events, EDR telemetry, authentication logs, or cloud audit logs. For YARA, that might mean file samples, memory dumps, malware repositories, sandbox output, EDR file scanning, or incident response artifacts.

Once telemetry is confirmed, write specific logic. Avoid single-condition rules when possible. Combine process lineage, command-line indicators, file paths, user context, network activity, file writes, or artifact traits when those combinations better represent the malicious behavior.

After that, add organizational context. This is what separates generic rules from enterprise-ready rules. The rule should know what is normal in the environment, which systems and accounts are expected to perform similar behavior, which business tools may trigger the logic, and which assets deserve higher priority.

False positives should be defined early, not after the SOC complains. Tuning should be narrow and evidence-based. The goal is not to suppress everything noisy. The goal is to preserve detection value while removing expected behavior.

Finally, test, deploy carefully, and maintain the rule. Use known malicious samples or simulations, real benign data, historical logs, analyst feedback, and alert volume metrics. Deploy first in test or monitor mode when possible, then promote the rule when the signal is understood.

### Common Mistakes Detection Engineers Should Avoid

**The first mistake is writing rules from headlines**. A threat report says attackers used PowerShell, so someone writes a rule to detect PowerShell. That is not enough. The better question is how the attackers used PowerShell and what telemetry proves that behavior happened.

**The second mistake is treating indicators of compromise as long-term detections**. Hashes, domains, IP addresses, and filenames are useful for blocking, enrichment, triage, scoping, and retrospective searches. But durable detections should focus more on behaviors, techniques, and toolmarks.

**The third mistake is over-mapping to MITRE ATT &CK. ATT&CK mapping is useful only when accurate**. If a rule detects PowerShell execution, mapping it to command and scripting interpreter activity may make sense. But that does not mean the same rule detects persistence, privilege escalation, credential access, and exfiltration.

**The fourth mistake is making everything high severity**. If every alert is high, nothing is high. Severity should be based on confidence, behavior risk, asset criticality, user context, attack stage, correlation with other events, and potential impact.

**The fifth mistake is ignoring the analyst experience**.

_If the SOC cannot understand the alert quickly, the rule is incomplete_.

Analysts need context, useful fields, investigation steps, and a clear explanation of why the alert matters.

**The final mistake is never retiring rules.** Not every rule should live forever. Rules should be retired when the log source no longer exists, the threat is no longer relevant, the logic is replaced by better coverage, the rule produces no value, or the detection has moved to another control.

### Final Thoughts

Writing detection rules is not the hard part. Writing detection rules that matter is the hard part.

A bad rule creates noise. A good rule detects suspicious behavior. A great rule gives the security team confidence, context, and clarity.

Sigma and YARA are powerful tools, but the language is not what makes a rule great. The thinking does. Great detection rules are built from adversary behavior, supported by reliable telemetry, tuned to the organization, tested against real data, and maintained over time.

That is the standard detection engineering should aim for.

**_The goal is not to alert on everything. The goal is to alert on the right things, with the right context, at the right time._**
