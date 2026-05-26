---
title: "Redirection of Non-Existent .ph and .com.ph Domains"
date: 2025-03-04
author: "drew-byte"
tags: ["incident-response", "threat-intelligence", "infostealer", "malware", "phishing"]
draft: false
---

*Originally published by [drew-byte](https://drew-byte.github.io)*

An incident response report covering the FakeCaptcha/ClickFix campaign — a social engineering attack that redirects users visiting non-existent Philippine domains to a fake CAPTCHA page, ultimately delivering Lumma Stealer.

Key findings:
- A suspicious XOR-encrypted PowerShell script was triggered on a user's machine
- The infection chain traced back to a deceptive CAPTCHA page impersonating a legitimate site
- Non-existent `.ph` and `.com.ph` domains are being weaponized via a `bouncy.php` redirection mechanism
- The payload (`Erratic1Crank1Banshee1Drainpipe.enx`) was analyzed and linked to Lumma Stealer
- IOCs including hashes and defanged URLs are documented

[Read the full report →](https://drew-byte.github.io/posts/irreport/)
