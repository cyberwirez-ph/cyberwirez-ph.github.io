---
title: "Hunting for Criminal Infrastructures"
date: 2026-08-01
description: "Practical techniques for uncovering unreported malicious infrastructure using specific fingerprints and search queries on Censys and VirusTotal."
author: "innauwu"
draft: false
---

*Originally published on [innauwu.gitbook.io](https://innauwu.gitbook.io/blogs/randos/hunting-for-criminal-infrastructures)*

![Infrastructure hunting screenshot](/images/articles/hunting-criminal-infrastructures/img1.png)

\\\\\\\\\\\ Still under construction

Every so often I'll see someone on LinkedIn or X mention they've been tracking some nation-state cluster for the better part of a year, and I always wonder what that actually looks like day to day. Turns out for a lot of these people, that IS the day job.

I've got zero formal CTI training, just been teaching myself, I'm eyeing IntelOps' Adversary Infrastructure Hunter course soon. There's bills and rent to pay.

As of writing this blog, I’ve already found several previously unreported pieces of infrastructure and shared them with threat intel platforms. You'd be surprised how garbage their OPSEC actually is.

![Infrastructure hunting screenshot](/images/articles/hunting-criminal-infrastructures/img2.png)

## No fluff How to do it?

Basic syntax: think of something malicious, say "PureRat" or refer to this [post](https://www.linkedin.com/posts/anna-p-868921105_threatintelligence-malwareanalysis-purerat-activity-7487351279401246721-FB7f?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD58KB4Bb1raesIIsbEVbsVZsBZM3ZoQqjI) for a quickwin. Search that term on Censys, see what pops up.

<br>

Check the resulting IPs on VirusTotal. If you're already getting a bunch of hits/detections, that infra's already known, someone beat you to it, not new.

## Finding Unreported Infrastructure

So narrow it down. Instead of just the malware name, start layering in specifics: a JA3/JARM hash, a cert subject/issuer string, a specific port + banner combo, a known C2 config artifact, a very specific file name. ***The more specific the fingerprint, the fewer results, but the higher the odds you're looking at something unreported.***

Censys gives you 5 searches a day without an account. Sounds stingy until you realize how far one good query, so you better make the most of it. I start with keywords that tend to show up on threat actor infrastructure without drowning you in noise. Something like below:

* **`"telegram"`** — threat actors love using Telegram for C2, communications, exfil, or just dropping contact info.
* **`".dll"`** — it’s literally a Windows file type. Why the hell is someone randomly hosting a `.dll` online?  It doesn’t automatically mean malicious, but it’s definitely something I’d investigate.
* **`"Rubeus.exe"`** — an Active Directory/Kerberos assessment tool.

Then you try to combine these findings or "Combo".

If you're not sure about the query syntax, **have Claude draft one**, just know it'll probably hand you legacy syntax, then run it through Censys' own AI query fixer, which is weirdly good at untangling it into whatever the current syntax actually is.

Welp, If you made found this blog, you’re probably more than skilled enough to take the idea and run with it. Have fun!

## FAQ

### **How do i know if its a red team infrastructure for an engagement or a legitimate threat actor activity?**

It depends, there are a few things I’d consider when trying to determine whether an infrastructure is part of an authorized engagement or belongs to an actual threat actor. 

**a.** In a legitimate red team engagement, there is generally little reason to leave infrastructure unnecessarily exposed to the public internet when access can be restricted to the company’s known  controlled sources.

**b.** A good red teamer will typically have controls around their infrastructure and avoid leaving it running in the wild without a reason. Their infrastructure should generally be tied to a defined engagement, scope, and timeframe.

**c.** That said, this is not a definitive indicator. Red teams may intentionally deploy internet-facing infrastructure to simulate real-world threat actors and test an organization’s external detection and response capabilities.

But honestly, I rarely run into this kind of ambiguity. More often than not, when you’re actively hunting, the infrastructure you come across already has a pretty high confidence of being malicious lol. Especially when you find a [**language associated with a sanctioned country**](https://www.cyber.nj.gov/threat-landscape/2026-cyber-threat-assessment) in the Readme.txt of a c2 infrastructure, that’s already high-confidence malicious.

At that point, it’s pretty much a wrap.

PS: i've read an experience of some people encountering red team infra being open in the wild. One investigator i came across said, that in their own words "fucking python http'd their whole fucking engagement folders" absolutely bonkers.
