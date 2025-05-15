---
layout: post
title: 'Cyber Skyline: Log Analysis'
date: 2025-05-08 09:54 -0700
description: CySky Notes Log Analysis
image:
  path:
  alt:
category: [Cyber Skyline]
tags: [notes, cyber skyline, cybersecurity, log analysis]

---

# **Log Analysis**

### Challenges
- SSH (easy) n/i
- Nginx (Medium)
- History (Medium)
- [**Squid (Hard)**](#squid)
- Payments (Hard)
- VSFTPD (Easy)
- Login (Easy)
- [**Custom File Format (Hard)**](#custom-file-format)

### Main Tools  
The main tools used in these challenges:
- **bash** (Squid)
- **CyberChef** (Custom File Format)


### Squid
Analysis of Squid proxy log

#### Tools used
- **bash**

#### Process
Download the file named `squid_access.log`. We can use **bash** commands to analyze the file, starting with `cat squid_access.log`.

<figure>
  <img src="../assets/img/site_images/cyber_skyline/squid_10.png" alt="" title="Initial `cat` of `squid_access.log`" style="max-width:50%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Initial `cat` of `squid_access.log`</figcaption>
</figure> 

> In what year was this log saved?
>
> From the above `cat`, the log timestamps are given in Epoch time. Using [EpochConverter](www.epochconverter.com) with an input of "1286536308", we can find the start year:
>
> **`2010`**
{: .prompt-tip }

Using various `bash` commands, we can extract data from the logs. For example:  

- Extracts the second column (time elapsed) and sorts it: 
  - `cat squid_access.log | awk '{print$2}' | sort -n`
  - `awk '{print$2}' squid_access.log | sort -n` 
- Extract the third column (URLs) and return count of uniques:
  - `cat squid_access.log | awk '{print$3}' | sort | uniq | wc -l`
  - `awk '{print$3}' squid_access.log | sort -u | wc -l`
- How many GET requests were made?
  - `awk '{print$6}' squid_access.log | grep 'GET' | wc -l`

> How many milliseconds did the fastest request take?
>
> Using this command extracts the second column and sorts it:
> `cat squid_access.log | awk '{print$2}' | sort -n | head -n 1` OR//
> `awk '{print$2}' squid_access.log | sort -n | head -n 1` 
>
> **`5`** milliseconds
{: .prompt-tip }

> How many milliseconds did the longest request take?
>
> Similar to previous, except reverse
> `cat squid_access.log | awk '{print$2}' | sort -rn | head -n 1` OR//
> `awk '{print$2}' squid_access.log | sort -rn | head -n 1`
> 
> **`41762`** milliseconds
{: .prompt-tip }

> How many different IP addresses did the proxy service use?
>
> `awk '{print$3}' squid_access.log | sort -u | wc -l`
> 
> **`4`** 
{: .prompt-tip }

> How many GET requests were made?
>
> `awk '{print$6}' squid_access.log | grep 'GET' | wc -l`
> 
> **`35`** 
{: .prompt-tip }

> How many POST requests were made?
>
> `awk '{print$6}' squid_access.log | grep 'POST' | wc -l`
> 
> **`78`** 
{: .prompt-tip }

> What company created the antivirus used on the host at 192.168.0.224?
>
> `cat squid_access.log | grep '192.168.0.224'  `
> 
> **`Norton / Symantec`** 
{: .prompt-tip }

> What URL is used to download the virus definitions?
>
> `cat squid_access.log | grep '192.168.0.224'  `
> 
> **`hxxp://liveupdate[.]symantecliveupdate[.]com/streaming/norton$202009$20streaming$20virus$20definitions_1.0_symalllanguages_livetri.zip`** 
{: .prompt-tip }

<sub>[Back to top](#challenges)</sub>

### Custom File Format
Analysis of logs from a custom file format

#### Tools used
- **CyberChef**

#### Process
We are given a custom log in the SKY log file format. Per Cyber Skyline:  

<figure>
  <img src="../assets/img/site_images/cyber_skyline/cff_10.png" alt="" title="Specification of Custom File Format" style="max-width:50%; margin:auto auto 0px auto" >
  <img src="../assets/img/site_images/cyber_skyline/cff_12.png" alt="" title="Specification of Custom File Format" style="max-width:50%; margin:0px auto auto auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Specification of Custom File Format</figcaption>
</figure> 

Download the file named `Custom File Format.sky`. 

<figure>
  <img src="../assets/img/site_images/cyber_skyline/cff_12.png" alt="" title=" " style="max-width:50%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center"> </figcaption>
</figure> 

> In what year was this log saved?
>
> 
>
> **``**
{: .prompt-tip }

<sub>[Back to top](#challenges)</sub>