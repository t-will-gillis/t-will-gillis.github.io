---
layout: post
title: 'Cyber Skyline: Forensics'
date: 2025-05-05 19:16 -0700
description: CySky Notes Forensics
image:
  path: ../assets/img/site_images/cyber_skyline/cs_0r2.png
  alt: Cyber Skyline Forensics
category: [Cyber Skyline]
tags: [notes, cyber skyline, cybersecurity, forensics]
---


# **Forensics**

### Challenges
- Version Control (easy) n/i
- [**File Carving (medium)**](#file-carving)
- PDF (easy) n/i
- [**Magic Bytes (medium)**](#magic-bytes)
- Docter (medium) n/i
- [**The Book (hard)**](#the-book)


### Main Tools  
The main tools used in these challenges:
- **binwalk** (File Carving)
- **www.Metadata2go.com** (PDF)
- **hexed.it** (Magic Bytes)
- **volatility3** (The Book)
- **crackstation.net** (The Book)


### File Carving
From Cyber Skyline: *The security team has found a rather strange file exiting the network. We're not sure if it's containing any sensitive information. Help us identify what's in it.*

#### Tools used
- **binwalk**

#### Process
You are given a file to download called `green_file`. Your OS might open this and show a small field of green pixels. In Linux, type `file green_file`. `file` identifies this file as a PNG image.

Run **binwalk** on `green_file` to search through binary file for magic bytes.

<figure>
  <img src="../assets/img/site_images/cyber_skyline/file_carving_5.png" alt="" title="Use binwalk to analyze `green_file` and extract hidden files" style="max-width:67%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Use binwalk to analyze `green_file` and extract hidden</figcaption>
</figure> 

Five PNG files, one CAB (gzip) file, and a `/flags` directory can be extracted from `green_file`. The `/flags` directory contains the `flags.txt`, printed below.

<figure>
  <img src="../assets/img/site_images/cyber_skyline/file_carving_8.png" alt="" title="Contents of `_green_file.png.extracted`" style="max-width:67%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Contents of `_green_file.png.extracted`</figcaption>
</figure>


> What is the format of the file?
>
> **`PNG`**
{: .prompt-tip }


> How many files can be extracted from the binary?
>
> **`Six`**
{: .prompt-tip }

> What is the hidden flag in the file?
>
> **`SKY-RWCI-4291`**
{: .prompt-tip }

<sub>[Back to top](#challenges)</sub>

### Magic Bytes
From Cyber Skyline: *This file appears to be changed in some way. Can you recover the original?*

#### Tools used
- **HexEdIt** (https://heded.it)


#### Process
We are provided an image named `flag.jpeg`. Of course, this file appears to be a JPG file. However, when you attempt to open this, an message appears that there is an error interpreting the JPEG image because of an "Unsupported marker type 0xf6". 

With some light Googling, it appears this specific error "...is usually related to a wrong file format and extension" per Stellar Data Recovery (www[.]stellarinfo[.]com).

Using [Gary Kessler's reference](www.garykessler.net/library/file_sigs_GCK_latest.html), a JFIF/JPE/JPEG/JPG file should have the following bytes  
&emsp;`FF D8 FF E0 xx xx 4A 46 49 46 00`.  
Using [Wikipedia's List of File Signatures](en.wikipedia.org/wiki/List_of_file_signatures), we see a similar list:  
&emsp;`FF D8 FF E0 00 10 4A 46 49 46 00 01`  
Using **HexEdIt** (see below), we can see that our actual header file signature is:  
&emsp;`FF D8 FF EO 00 10 4A 46 49 46 00 0D`

<figure>
  <img src="../assets/img/site_images/cyber_skyline/magic_bytes_22.png" alt="" title="Use HexEdIt on file" style="max-width:50%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Use HexEdIt on file: top</figcaption>
</figure> 
<figure>
  <img src="../assets/img/site_images/cyber_skyline/magic_bytes_25.png" alt="" title="Use HexEdIt on file" style="max-width:50%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Use HexEdIt on file: bottom</figcaption>
</figure> 

Therefore, this might be a matter of changing the last byte from `0D` to `01`, but notice also that the trailing bytes `60 82` do not match the trailer for a JPG, `FF D9`. According to Kessler, the header and trailing bytes on a PNG should be:  
&emsp;`89 50 4E 47 0D 0A 1A 0A` header  
&emsp;`49 45 4E 44 AE 42 60 82 (IEND®B`‚...)` trailer  

The existing trailer is:  
&emsp;`49 45 4E 44 AE 42 60 82`

Thus it appears we have either a JPG with a malformed header and trailer, or a PNG with a malformed header. 

- For JPG: Changing the bytes in the header and the trailer to match a JPG still results in the same error. 
- For PNG: Changing the header to match a PNG does not immediately fix the problem: "Invalid IHDR length". If we open a known good PNG file, the header through the previous JPG bytes is:  
&emsp;&emsp;`89 50 4E 47 0D 0A 1A 0A 00 00 00 0D`

Replacing the remaining bytes, the PNG opens and shows the flag. 

<figure>
  <img src="../assets/img/site_images/cyber_skyline/magic_bytes_35.png" alt="" title="Editing the magic bytes for a PNG signature" style="max-width:67%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Editing the magic bytes for a PNG signature</figcaption>
</figure> 

> What is the original file type?
>
> **`PNG`**
{: .prompt-tip }

> What is the flag?
>
> **`SKY-DSFG-5792`**
{: .prompt-tip }

### The Book
From Cyber Skyline: *We have obtained a live system memory dump from a hacker's computer before it fried itself. The hacker was looking at a suspicious document. Can you retrieve the lost information?*

#### Tools used
- **volatility3**
- **crackstation.net**

#### Process
The memory dump file is named `memdump.mem.xz`. You will want to unzip the `.xz` file using the command `xz -d memdump.mem.xz`, which will unpack `memdump.mem`. 

Run **volatility3** to analyze `memdump.mem`.

> What OS was this dump taken from?
>
> From `file memdump.mem`:
> **`Windows`**
{: .prompt-tip }

Since we now know this memory dump comes from a Windows environment, we can use the command `./vol.py -f ./memdump.mem windows.envars.Envars`. 

<figure>
  <img src="../assets/img/site_images/cyber_skyline/the_book_10.png" alt="" title="Use volatility3 to analyze `memdump.mem`" style="max-width:67%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Use volatility3 to analyze `memdump.mem`</figcaption>
</figure> 

<figure>
  <img src="../assets/img/site_images/cyber_skyline/the_book_12.png" alt="" title="Use volatility3 to analyze `memdump.mem` cont." style="max-width:67%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Use volatility3 to analyze `memdump.mem` cont.</figcaption>
</figure> 

<figure>
  <img src="../assets/img/site_images/cyber_skyline/the_book_14.png" alt="" title="Use volatility3 to analyze `memdump.mem` final" style="max-width:67%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Use volatility3 to analyze `memdump.mem` final</figcaption>
</figure>

> What is the name of the computer?
>
> **`DESKTOP-OT97GG3`**
{: .prompt-tip }

> What is the name of the user that was logged in?
>
> **`liber8hacker`**
{: .prompt-tip }

Next, run a scan for files `./vol.py -f ./memdump.mem windows.filescan.FileScan` to print out all file objects present. These can be reviewed, and if any files are of interest they can be extracted by using this command and the virtual address:
`./vol.py -f ./memdump.mem -o ./dump windows.dumpfiles.DumpFiles --virtaddr <file address>`

We are given a hint that there is a SQLite file of interest. SQLite file extensions can be `.sqlite`,`.db`, and `.db3`. There is one potential database file of interest:

<figure>
  <img src="../assets/img/site_images/cyber_skyline/the_book_20.png" alt="" title="File of interest in FileDump` final" style="max-width:67%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">File of interest in FileDump</figcaption>
</figure>

> What is the full filepath and file of the file of interest?
>
> **`\Users\liber8hacker\Desktop\black_book.db`**
{: .prompt-tip }

The recovered files can be opened in SQLite and reveal info from the `Black Book`:

> What is the real name of "cloud"?
>
> "Cloud" is alias #8, which corresponds to:
> **`gloria hampton`**
{: .prompt-tip }

We can use **volatility3** one last time to recover hashes:
`./vol.py -f ./memdump.mem windows.hashdump.HashDump`  

In my case, this results in an `argument PLUGIN: invalid choice` error. To understand the error, we need to add `-vv` (verbose) to the previous command:
`./vol.py -f ./memdump.mem windows.hashdump.HashDump -vv`

<figure>
  <img src="../assets/img/site_images/cyber_skyline/the_book_40.png" alt="" title="Use volatility3 to analyze `memdump.mem`" style="max-width:67%; margin:auto" >
  </figure>
<figure>
  <img src="../assets/img/site_images/cyber_skyline/the_book_41.png" alt="" title="Missing module in volatility3's `HashDump`" style="max-width:67%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Missing module in volatility3's `HashDump`</figcaption>
</figure> 

This problem took some Googling. The module that `HashDump` needs is named `pycryptodome`. After some false-starts, the solution is to open a virtual environment, then to re-install `volatility3` and `pycryptodome` in that environment. 
  ```
  python3 -m venv venv
  source venv/bin/activate
  pip install volatility3
  pip install pycryptodome
  ```  
We can rerun the previous command (without the `-vv`) and **volatility3** successfully returns the hashes, and we can use `https://crackstation.net` to crack the hash:

<figure>
  <img src="../assets/img/site_images/cyber_skyline/the_book_50.png" alt="" title="After creating venv and installing pycryptodome" style="max-width:67%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">After creating venv and installing pycryptodome</figcaption>
</figure> 

<figure>
  <img src="../assets/img/site_images/cyber_skyline/the_book_55.png" alt="" title="Use crackstation.net to crack the hash" style="max-width:67%; margin:auto" >
  <figcaption style="font: italic small sans-serif; text-align:center">Use crackstation.net to crack the hash</figcaption>
</figure> 

> What is password of the currently logged in user?
>
> 
> **`avatar2`**
{: .prompt-tip }


<sub>[Back to top](#challenges)</sub>

