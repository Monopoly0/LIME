---
published: false
---
---
title: SIP Bypass POC
author: cotes
date: 2022-10-26 14:10:00 +0800
categories: [POC, Mac]
tags: [mac, priv-esc]
render_with_liquid: false
---

# THM- NAX

## Overview
<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%2012.02.35%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
This box made use of web enumeration and obscure steganagrophy. There wasn't much post exploit to be done making it a pretty start to finish box once finding a foothold. 

## Notable Scans
https://github.com/LimeIncOfficial/Blog-Repo/blob/main/Nax.thm/results/10.10.91.116/scans/tcp_80_http_index.html
https://github.com/LimeIncOfficial/Blog-Repo/blob/main/Nax.thm/results/10.10.91.116/scans/tcp_80_http_nmap.txt
https://github.com/LimeIncOfficial/Blog-Repo/blob/main/Nax.thm/results/10.10.91.116/scans/_full_tcp_nmap.txt

## Road to User
The first thing I did was to see if there was a webserver running. 
<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%2012.25.06%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
I checked if `/nagiosxi` was a subdirectory of the webserver and got a 301. I'll start a dir enum scan and keep the dir in the back of my mind for the time being. The other thing I noticed was the string in the middle of the webpage.
<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%201.00.06%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />

## First Flag 
 Writing down the numbers of the elements and inputing the string into an ascii to text convertor leads to the first flag 
<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%2012.33.28%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
 
 Once we navigate to this directory we are greeted with a very obscure picture. Once we download the image and use an exiftool to see if there is any intresting metadata we find the second flag.
<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%2012.36.52%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
 
 Checking back in with our dir enum (using dirbuster) I visited the webapp and navigated to the login page.
 
<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%201.00.06%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
 
 I tried the default username and password. I also tried combinations of previous strings and usernames found with no avail. I wanted to used bruteforcing as a last resort, so I looked elsewhere .I went back to the picture since it looked very odd. It was also my only lead at the time since there were no exploits to get past the login page. A popular list of resources for stego to refer to is https://0xrick.github.io/lists/stego/#tools . 
 
 One thing that caught my eye was the programming language named Piet. The language was named after, Piet {redacted}, so after some searching I came accross this site called: https://www.bertnase.de/npiet/npiet-execute.php . I uploaded the piet file and retrieved the credentials (the password is after the % btw). Apparently the defualt login username was the new one, I thought it was gonna be one we discover after gaining access to the admin controls XD. So there goes flags 3 & 4. 
<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%201.31.36%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
 
 The 5th flag was the CVE associated with the metasploit module I'd use to exploit the box. The path of the metasploit module is the 6th flag. `linux/http/<redacted?>`
 
 While it is always good practice to use the exploit code itself instead of using a metasploit module, we will be using the module for the since the challenege specifically states the use of the module. 
<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%202.05.47%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
 
 Here is the output of the metasploit module I ran:
 
<img src="https://github.com/LimeIncOfficial/Blog-Repo/blob/main/Nax.thm/Screen%20Shot%202022-03-17%20at%202.14.08%20AM.png?raw=true"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
 
 And it turns out we were given root upon execution :)
 <img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%202.14.53%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
 
 Now all you have to do is find the flags on the box.
 
 User:
<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%202.23.36%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />
 Root:
<img src="https://raw.githubusercontent.com/LimeIncOfficial/Blog-Repo/main/Nax.thm/Screen%20Shot%202022-03-17%20at%202.17.20%20AM.png"
     alt="Markdown Monster icon"
     style="width: auto; max-width: 100%; height: auto;" />


