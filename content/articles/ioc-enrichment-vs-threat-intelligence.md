---
title: 'Stop Calling IOC Enrichment "Threat Intelligence": How to Properly Use CTI for Alert Triage'
date: 2026-08-03
description: "Why effective alert triage needs behavioral context and campaign awareness, not just reputation-score lookups."
author: "seif"
draft: false
---

*Originally published by [seif](https://medium.com/@seiferboado101) on Medium*


![image](/images/articles/ioc-enrichment-vs-threat-intelligence/img0.jpg)


Most SOC teams say they use Cyber Threat Intelligence during alert triage.

What they usually mean is this: an alert fires, an analyst copies an IP, hash, URL, or domain into VirusTotal, AbuseIPDB, URLScan, Shodan, or a threat intel platform, sees a reputation score, adds “known malicious” or “no hits,” and moves on.

**That is not threat intelligence.**

**That is enrichment.**

Enrichment answers: “ _What do other sources say about this indicator?_ ”

Threat intelligence should answer: “ _What does this activity mean in our environment, how confident are we, what else should we look for, what decision does this support, and what should happen next?_ ”

That distinction matters because alert triage is not just about proving whether something is bad. It is about deciding what kind of incident you may be dealing with, how far it may have gone, what evidence is missing, what containment actions are justified, and what future detections or hunts should be created from the case.

The reading that triggered this blog framed it well: reputation lookups provide a verdict, while CTI adds context such as related infrastructure, campaigns, actor behavior, techniques, and whether the organization has seen similar activity before.

It also made the L1/L2 distinction clear: L1 may stop at “is this bad?”, but L2 has to contextualize the activity and turn it into a better investigation path.

### Enrichment Is Useful. It Is Just Not Enough.


![image](/images/articles/ioc-enrichment-vs-threat-intelligence/img1.png)


Let’s be fair. Enrichment has value.

Looking up a hash, IP, URL, ASN, certificate, passive DNS record, WHOIS record, malware family name, or sandbox result can quickly support triage. It can help an analyst decide whether an alert deserves escalation. It can reduce time spent on obvious false positives. It can reveal related artifacts.

**_But enrichment is still raw material._**

A VirusTotal hit does not automatically mean your host is compromised.

A Cloudflare IP with malicious detections does not mean every endpoint that connected to it is infected.

A legitimate service such as GitHub, Discord, Telegram, Dropbox, Pastebin, api.github.com, or api.myip.com can appear in both benign and malicious workflows.

The presence of a shared infrastructure indicator is not proof of compromise by itself.

**This is where analysts get burned.**

They treat external reputation as **truth** instead of treating it as a **lead**.

They block shared infrastructure because a vendor page looks red.

They close alerts because the hash has zero detections.

They copy IOCs from a report without asking whether the report describes the same behavior, same attack stage, same target type, same timeline, or same environment.

That is not intelligence-led triage. **_That is outsourced judgment._**

[FIRST’s CTI SIG](https://www.first.org/global/sigs/cti/curriculum/introduction) describes CTI as something meant to support effective DFIR and incident response operations, including tactical decision-making during cyber incidents. It also explicitly separates the idea of a “data feed” from intelligence and emphasizes the relationship between intelligence, forensics, evidence, indicators, and response.

That is the standard: intelligence should improve the decision, not decorate the ticket.

### The Difference Between Lookup, Enrichment, and Contextualization

A clean way to separate the three:

**Lookup** asks: “Has anyone seen this indicator before?”

Example:  
evil-domain[.]com has 18 vendor detections.

**Enrichment** asks: “What metadata can we add?”

Example:  
The domain resolves to this IP, uses this registrar, was first seen yesterday, shares a certificate with these domains, and appears in this malware sandbox report.

**Contextualization** asks: “How does this change our investigation?”

Example:  
The domain appears in a recent npm supply chain campaign targeting developer environments.


![image](/images/articles/ioc-enrichment-vs-threat-intelligence/img2.png)


Our alert fired from a build runner after npm install. The process accessed .npmrc, .aws/credentials, .env, and GitHub tokens. This is consistent with credential theft from a compromised dependency. We should isolate the runner, rotate exposed secrets, review package lockfiles, hunt for affected packages across CI/CD logs, and check for unauthorized GitHub repositories or workflow changes.

**_That last paragraph is CTI doing work._**

It connects the indicator to behavior, behavior to campaign, campaign to affected assets, and affected assets to response actions.

### Why CTI Belongs in Alert Triage

Alert triage is usually treated as a queue-management problem: validate, classify, escalate, close.

But stronger analysts use triage as the first intelligence checkpoint.

A single alert can tell you what happened. CTI helps you ask what it may represent.


![image](/images/articles/ioc-enrichment-vs-threat-intelligence/img3.jpg)


A suspicious node.exe accessing .env, .npmrc, SSH keys, and cloud credentials is not just “credential access by untrusted process.” In a developer workstation or CI/CD runner, that pattern may indicate software supply chain compromise, malicious dependency execution, or credential theft from a build environment.

A suspicious PowerShell command is not just “encoded PowerShell.” It might match an intrusion set’s initial access playbook, a commodity loader, a phishing payload, or an internal admin script gone wrong.

A domain hit is not just a domain hit. It may be part of a campaign, a sinkhole, a CDN, a parking page, a scanner, a redirect chain, or old infrastructure that has changed hands.

The point is to answer: “What is the most likely threat scenario, and what evidence would prove or disprove it?”

[MITRE ATT&CK](https://attack.mitre.org/) helps here because it gives defenders a common language for adversary tactics and techniques based on real-world observations. It is useful not because mapping something to ATT&CK magically makes the alert correct, but because it forces the analyst to describe behavior instead of stopping at atomic indicators.


![image](/images/articles/ioc-enrichment-vs-threat-intelligence/img4.jpg)


David Bianco’s Pyramid of Pain makes the same point from another angle: hashes, IPs, and domains are easier for adversaries to change, while tools and TTPs are more disruptive to detect and deny. The higher you move, the more useful your CTI becomes for detection and response.

### A Practical CTI Workflow for Alert Triage

Here is the workflow I actually practically use in our SOC.

### 1\. Start with internal evidence first

Before touching external CTI, understand what your own telemetry says.

Ask:

  * What triggered the alert?
  * What user, host, process, parent process, command line, file, registry key, network connection, cloud action, or identity event is involved?
  * Is this expected for that user, host, role, application, or time window?
  * What happened before and after the alert?
  * What asset type is involved: workstation, server, domain controller, CI/CD runner, cloud workload, developer machine, finance laptop?



This matters because internal evidence outweighs external intelligence. If your SIEM shows malicious behavior, CTI should supplement and scope it, not override it. If CTI says an IP is malicious but your logs show normal CDN traffic to a legitimate SaaS app, the context wins.

### 2\. Convert the alert into behaviors

Do not jump straight from alert to IOC.

Translate the event into behaviors.

Example:

Alert: node.exe accessed credential files

Behavioral interpretation:

  * A Node.js process executed after package installation.
  * The process accessed developer secrets.
  * It queried cloud or GitHub-related resources.
  * It may represent supply chain execution through package lifecycle scripts or build tooling.
  * Potential ATT&CK areas include supply chain compromise, credential access, discovery, exfiltration, and persistence depending on evidence.



This is where the analyst starts moving from atomic indicators to composite indicators. FIRST’s CTI curriculum describes this exact progression: indicators often need further analysis and contextualization, and composite information is richer and more useful for security operations than isolated atomic data.

### 3\. Enrich the indicators, but label what they are

Now use enrichment.

Look up:

  * Hashes
  * Domains
  * URLs
  * IP addresses
  * Certificates
  * JA3/JA4 or TLS fingerprints
  * User agents
  * File paths
  * Mutexes
  * Registry keys
  * Package names
  * GitHub repositories
  * Cloud account IDs
  * Email senders
  * Command-line fragments



But classify each result:

  * Confirmed malicious indicator
  * Suspicious but weak indicator
  * Legitimate service abused by attacker
  * Shared infrastructure
  * Historical indicator
  * Expired or low-confidence indicator
  * Internal-only indicator
  * Unknown



This prevents the common mistake of treating all IOCs as equal.


![image](/images/articles/ioc-enrichment-vs-threat-intelligence/img5.png)


Platforms like MISP support this operationally through warning lists, decay models, sightings, tags, and filtering so analysts can reduce false positives and evaluate the freshness and quality of indicators.

MISP warning lists are specifically designed to flag well-known indicators that may represent false positives, errors, or mistakes, which is exactly what SOC teams need when they are dealing with shared infrastructure or noisy public feeds.

### 4\. Search for campaign-level context

After enrichment, ask the better question:

“Does this behavior match a known campaign, technique, malware family, intrusion set, or public report?”

Search for unique behavior, not just indicators.

Weak search:  
1.2.3.4 malicious

Better search:  
"node.exe" ".npmrc" ".aws/credentials" "npm install"

Better still:  
"binding.gyp" "credential" "GitHub" "npm install"

This is where CTI starts paying off.


![image](/images/articles/ioc-enrichment-vs-threat-intelligence/img6.png)


**The 2026 Miasma npm supply chain activity is a strong example**. Microsoft reported that the campaign affected 32 maliciously modified packages across more than 90 versions under the @redhat-cloud-services npm scope, with trojanized packages published through a legitimate GitHub Actions OIDC workflow. The payload harvested credentials from GitHub, npm, AWS, Azure, GCP, HashiCorp Vault, Kubernetes, and developer systems.

Snyk later described a related Node-gyp supply chain compromise where malicious packages abused binding.gyp and node-gyp to execute attacker-controlled code during npm install, harvested developer and CI/CD credentials, exfiltrated through GitHub repositories, injected GitHub Actions workflows, and propagated across ecosystems. StepSecurity tracked the wider Miasma wave as a self-spreading worm using “Phantom Gyp,” noting that the technique bypassed tools that only checked package.json lifecycle scripts.

That kind of reporting does more than give an IOC list. It tells an analyst where to look next.

Instead of only asking, “Was this hash seen before?”, the analyst can ask:

  * Did the host run npm install shortly before the alert?
  * Was binding.gyp, preinstall, postinstall, or node-gyp involved?
  * Were package lockfiles updated?
  * Were GitHub tokens, npm tokens, cloud keys, or Kubernetes configs accessed?
  * Were new repositories created under the user or organization?
  * Were GitHub Actions workflows modified?
  * Were CI/CD runners affected?
  * Do we need credential rotation, not just endpoint cleanup?



That is CTI shaping triage.

### 5\. Validate the report against your logs

This is the step weak analysts skip.

A public report is not your incident.

A vendor writeup may be accurate and still incomplete for your case. It may describe one wave of activity while your environment saw another. It may include indicators that are already dead. It may mention legitimate services that are only suspicious when combined with other behavior. It may over-focus on the malware and under-focus on the intrusion path.

So validate claims one by one.

If the report says the malware used GitHub for exfiltration, check your proxy, DNS, EDR, GitHub audit logs, and cloud logs for matching activity.

If the report says the malware touched .npmrc, .env, .aws/credentials, and SSH keys, check file access telemetry.

If the report says it used a package lifecycle script, check process lineage, package manager logs, and build output.

If the report lists affected package versions, check lockfiles, SBOMs, package cache, artifact builds, CI logs, and developer workstations.

CTI does not replace evidence. CTI tells you what evidence to go get.

### 6\. Weigh source reliability and confidence


![image](/images/articles/ioc-enrichment-vs-threat-intelligence/img7.png)


Not all CTI sources deserve equal weight.

A confirmed internal incident report from your own DFIR team is stronger than a random public pulse.

A vendor report with technical depth, timestamps, affected versions, behavioral indicators, and remediation guidance is stronger than a reposted IOC list.

A behavior observed in your logs is stronger than a reputation score.

A linked indicator that belongs to a campaign chain is stronger than a lone IP address with no context.

FIRST recommends communicating uncertainty in CTI with clear confidence language, including levels such as unlikely, likely, and highly likely. The point is simple: CTI consumers should know whether a claim is proven, assessed, inferred, or speculative.

In triage notes, that means writing like this:

High confidence: node.exe accessed .npmrc, .aws/credentials, and .env on WKSTN-047 after npm install.

Medium confidence: Activity is consistent with npm supply chain credential theft based on process lineage and accessed files.

Low confidence: Campaign overlap with Miasma is possible based on behavior, but no matching package/version has been confirmed yet.

That is better than: “Looks like Miasma. Escalating.”

### 7\. Turn CTI into action

** _If the CTI does not change the action, it is trivia._**

Good CTI should lead to at least one of these:

  * A sharper severity decision
  * A clearer incident classification
  * A hunt query
  * A containment recommendation
  * A credential rotation scope
  * A detection engineering task
  * A blocklist or allowlist decision
  * A case note for future analysts
  * A report to leadership
  * A new internal intelligence entry
  * A MISP/OpenCTI update
  * A Sigma/YARA/SIEM rule improvement
  * A change to the playbook


![image](/images/articles/ioc-enrichment-vs-threat-intelligence/img8.jpg)


OpenCTI is built around this kind of relationship-based intelligence work: structuring and visualizing technical and non-technical threat information, managing observables, and helping teams connect actors, malware, TTPs, indicators, and reports. MISP, meanwhile, is designed for collecting, storing, distributing, and sharing security indicators and threat data in a structured way for incident analysts, security professionals, and malware reversers.

Used properly, these platforms should not be expensive IOC buckets. They should become the connective tissue between SOC triage, threat hunting, detection engineering, vulnerability management, and incident response.

### Example: Bad vs Good CTI Triage Note

### Weak note

>  _Hash checked in VirusTotal. 12 vendors flagged malicious. Domain also malicious. Escalating as true positive._

This note gives a verdict but no intelligence. It does not explain what happened, what it means, what evidence supports it, what is missing, or what should happen next.

### Better note

>  _Alert fired for_ _node.exe accessing developer credential files on_ _WKSTN-047 under user_ _CORP\j.okafor. Process lineage shows_ _code.exe →_ _npm install →_ _node index.js. Accessed files include_ _.npmrc,__.env,__.aws/credentials, and SSH private key material. This behavior is consistent with npm supply chain credential theft patterns reported in recent Miasma/Shai-Hulud-style campaigns, where malicious packages executed during install and harvested GitHub, npm, cloud, and CI/CD secrets. Current confidence is medium because behavioral overlap is strong, but the affected package/version has not yet been confirmed. Recommended next steps: collect package lockfile and npm cache, identify package installed at execution time, review GitHub and npm audit logs, rotate exposed user/cloud/dev tokens, hunt for the same package and process chain across developer endpoints and CI/CD runners, and create a detection for package install spawning credential file access._

That is triage with CTI.

It does not just say “bad.”  
It says what is known, why it matters, how confident we are, and what to do next.

### Common CTI Mistakes in Alert Triage

### Mistake 1: Treating reputation as evidence

Reputation is a lead. Evidence comes from your telemetry.

A domain can be malicious in one context and benign in another. An IP can host thousands of unrelated services. A public report can mention a legitimate API because malware abused it, not because the API itself is malicious.

### Mistake 2: Copy-pasting IOCs without understanding behavior

IOC lists are often incomplete, stale, or overbroad.

Use them, but do not worship them.

Ask what behavior generated the indicator. Ask where in the attack chain it belongs. Ask whether it is unique, shared, expired, or easily changed.

### Mistake 3: Over-attribution

Attribution is useful when it changes the response.

But during triage, you usually do not need to prove an APT name to act. You need to know the likely intrusion path, exposed assets, affected accounts, required containment, and next hunts.

Filigran’s 2026 CTI fallacies article makes the point bluntly: CTI is not a feed; it is decision support grounded in context and requirements. It also warns that attribution should be a tool, not the goal, because the value is in actions such as detection, hardening, and prioritization.

### Mistake 4: Ignoring indicator lifetime

Some indicators age badly.

Hashes can remain useful for known malware samples. Domains may remain useful for detection or retrospective hunting. Cloud IPs, CDN IPs, VPS IPs, and shared hosting infrastructure may become stale fast.

MISP playbooks account for this through decay handling: indicators can receive decay scores, be tagged, or have the to_ids flag disabled when they are considered decayed.

The lesson: do not treat all indicators as permanently blockable.

### Mistake 5: Not creating internal intelligence

Every real case should make the next case easier.

If your team confirms a malicious package, command line, parent-child process chain, suspicious user agent, registry path, cloud API call, phishing sender pattern, or lateral movement technique, capture it.

Internal CTI should include:

  * What was observed
  * Where it was observed
  * What it means
  * Confidence level
  * Source
  * First seen / last seen
  * Affected assets
  * Related alerts
  * Related MITRE techniques
  * Detection logic
  * Recommended response
  * Expiration or review date



This is how a SOC matures. You stop solving the same problem from zero every time.

### A Simple CTI-Driven Triage Checklist

When an alert fires, ask:

  * What is the alert actually detecting?
  * What behavior does this represent?
  * What evidence do we have internally?
  * What indicators can be enriched?
  * Which indicators are strong, weak, shared, stale, or legitimate-but-abused?
  * Does the behavior match a known report, campaign, malware family, or technique?
  * What parts of the report match our logs?
  * What parts do not match?
  * What additional evidence would increase or decrease confidence?
  * What assets, accounts, secrets, or systems could be affected?
  * What hunts should we run?
  * What containment is justified now?
  * What should be documented for future cases?



That is the muscle.

### Final Takeaway

CTI is not the act of checking whether an IOC is red or green.

**CTI is the discipline of turning scattered evidence into a decision.**

A lookup tells you: “ _This hash is bad_.”

Enrichment tells you: “ _This hash is connected to these domains, IPs, reports, and detections._ ”

Contextualized CTI tells you: “This alert is likely part of this intrusion pattern, affecting this type of asset, requiring these validation steps, these hunts, and these response actions, with this confidence level.”

That is the difference between an analyst who closes tickets and an analyst who improves the security program.

**Modern SOCs do not need more screenshots of VirusTotal.**

They need analysts who can **take an alert, weigh internal evidence against external intelligence, understand attacker behavior, communicate uncertainty, and leave behind better detections** than they found.

That is how CTI should be used in alert triage.

### References

[FIRST CTI SIG materials on CTI’s role in supporting DFIR, intelligence tradecraft, indicators, data feeds versus intelligence, and tactical decision-making during incidents.](https://www.first.org/global/sigs/cti/curriculum/introduction)

[MITRE ATT&CK documentation describing ATT&CK as a knowledge base of adversary tactics and techniques based on real-world observations.](https://attack.mitre.org/)

[David Bianco’s Pyramid of Pain, which explains why different indicator types have different defensive value and why TTP-level detection is more disruptive to adversaries.](https://www.attackiq.com/glossary/pyramid-of-pain-2/)

[Microsoft Threat Intelligence’s June 2026 writeup on the Red Hat npm Miasma credential-stealing campaign.](https://www.microsoft.com/en-us/security/blog/2026/06/02/preinstall-persistence-inside-red-hat-npm-miasma-credential-stealing-campaign/)

[Snyk’s June 2026 writeup on the Node-gyp supply chain compromise and self-propagating npm worm behavior.](https://snyk.io/blog/node-gyp-supply-chain-compromise-self-propagating-npm-worm-binding-gyp/)

[Filigran’s 2026 article on CTI fallacies, especially the point that CTI is decision support, not merely a feed.](https://filigran.io/blog/the-10-fallacies-of-cti/)
