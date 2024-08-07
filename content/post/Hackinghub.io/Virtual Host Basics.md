---
title: "Virtual Host Basics"
date: "2023-09-21"
draft: false
---

## [[hackinghub.io]]
### Hubs - Virtual Host Basics
Type: #WEB #blackbox
Difficulty: #easy
SOLVED by: #myself
TOOL USED: #gobuster #host #feroxbuster
TOPIC: #api #vhost

---

Writeup Date:2023-09-20
URL = \*.nzkh4v4n.ctfio.com

---------

the challenge description 

![[Pasted_image_20230921014153.png]](/screenshots/Pasted_image_20230921014153.png)

```bash
$ rustscan -a www.nzkh4v4n.ctfio.com --ulimit 5000 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Please contribute more quotes to our GitHub https://github.com/rustscan/rustscan

[~] The config file is expected to be at "/home/musashi/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 206.189.31.119:22
Open 206.189.31.119:80
Open 206.189.31.119:9999

```

### GOBUSTER VHOST VS DNS
**VHOSTING**
```bash
─$ gobuster vhost -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://nzkh4v4n.ctfio.com  --append-domain -t 100 
===============================================================
Gobuster v3.2.0-dev
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://nzkh4v4n.ctfio.com
[+] Method:          GET
[+] Threads:         100
[+] Wordlist:        /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.2.0-dev
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
2023/09/20 07:24:31 Starting gobuster in VHOST enumeration mode
===============================================================
Found: www.nzkh4v4n.ctfio.com Status: 200 [Size: 2275]
Found: app.nzkh4v4n.ctfio.com Status: 200 [Size: 625]
Found: marketing.nzkh4v4n.ctfio.com Status: 200 [Size: 1371]
Found: zeus.nzkh4v4n.ctfio.com Status: 200 [Size: 72]
Progress: 4989 / 4990 (99.98%)===============================================================
2023/09/20 07:24:35 Finished
===============================================================

```

**DNS**
```bash
$ gobuster dns -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -d nzkh4v4n.ctfio.com   -t 100
===============================================================
Gobuster v3.2.0-dev
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     nzkh4v4n.ctfio.com
[+] Threads:    100
[+] Timeout:    1s
[+] Wordlist:   /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
===============================================================
2023/09/20 07:26:13 Starting gobuster in DNS enumeration mode
===============================================================
Found: app.nzkh4v4n.ctfio.com

Progress: 4989 / 4990 (99.98%)===============================================================
2023/09/20 07:27:04 Finished
===============================================================

```

doing
```bash
host nzkh4v4n.ctfio.com                                    nzkh4v4n.ctfio.com has address 206.189.31.119
```
we have IP address how to know if it's vhosting or not 

we can use curl and supply HEADER HOST: localhost/subdomain
and see if they are different response here's example 

```zsh
curl 206.189.31.119 -H "HOST: zeus.nzkh4v4n.ctfio.com"
EvilCorp API Server
For customer data use {host}.zeus.nzkh4v4n.ctfio.com    
```
**NOTE**: if u can't access those site through the browser 
with error like this :![[Pasted_image_20230920065939.png]](/screenshots/Pasted_image_20230920065939.png)
then edit the /etc/hosts file
with ip and the domain name
for example 
206.189.31.119  zeus.nzkh4v4n.ctfio.com marketing.nzkh4v4n.ctfio.com
```html
curl 206.189.31.119 -H "HOST: "marketing.nzkh4v4n.ctfio.com"       
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>EvilCorp Marketing Intranet</title>
    <link rel="stylesheet" href="/style.css">
</head>
<body>
<header>
    <h1>Welcome to EvilCorp Marketing Intranet</h1>
    <nav>
        <ul>
            <li><a href="#home">Home</a></li>
            <li><a href="#news">News</a></li>
            <li><a href="#events">Events</a></li>
            <li><a href="#documents">Documents</a></li>
            <li><a href="#contact">Contact</a></li>
        </ul>
    </nav>
</header>
<main>
    <section id="home">
        <h2>Flags</h2>
        <p>FLAG_TWO{925494afdb0024a6add21a5b9ee9f3f0}</p>
    </section>
    <section id="news">
        <h2>News</h2>
        <!-- Your content for the news section goes here -->
    </section>
    <section id="events">
        <h2>Events</h2>
        <!-- Your content for the events section goes here -->
    </section>
    <section id="documents">
        <h2>Documents</h2>
        <!-- Your content for the documents section goes here -->
    </section>
    <section id="contact">
        <h2>Contact</h2>
        <!-- Your content for the contact section goes here -->
    </section>
</main>
<footer>
    <p>&copy; 2023 EvilCorp Marketing. All rights reserved.</p>
</footer>
</body>
</html>
                                                                                                                       

```

```bash
curl 206.189.31.119 -H "HOST: localhost"                   
{"status":"OK","flag":"FLAG_ONE{95fe9f9e8bf849aec6a8de02fee14d57}"}
```

so we will continue on zeus.nzkh4v4n.ctfio.com domain because this message 
For customer data use {host}.zeus.nzkh4v4n.ctfio.com is interesting

```bash
gobuster vhost -w ./subdomains.txt -u http://zeus.nzkh4v4n.ctfio.com  --append-domain -t 100
===============================================================
Gobuster v3.2.0-dev
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://zeus.nzkh4v4n.ctfio.com
[+] Method:          GET
[+] Threads:         100
[+] Wordlist:        ./subdomains.txt
[+] User Agent:      gobuster/3.2.0-dev
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
2023/09/20 08:06:44 Starting gobuster in VHOST enumeration mode
===============================================================
Found: a.auth-ns.zeus.nzkh4v4n.ctfio.com Status: 200 [Size: 670]
Found: b.auth-ns.zeus.nzkh4v4n.ctfio.com Status: 200 [Size: 670]
Found: c.auth-ns.zeus.nzkh4v4n.ctfio.com Status: 200 [Size: 670]
Found: ipv6.teredo.zeus.nzkh4v4n.ctfio.com Status: 200 [Size: 670]
Found: store.zeus.nzkh4v4n.ctfio.com Status: 200 [Size: 12]
===============================================================
2023/09/20 08:06:46 Finished
===============================================================

```

store.zeus.nzkh4v4n.ctfio.com is interesting one 
and i checked all the above with curl they all returned nginx expect store 
we now can continue with content discovery api endpoints
![[Pasted_image_20230920071832.png]](/screenshots/Pasted_image_20230920071832.png)
and we have /api with 200 we visit it 
![[Pasted_image_20230920071940.png]](/screenshots/Pasted_image_20230920071940.png)
and boom we have flag number 4

flag three i tried to bruteforce app.nzkh4v4n.ctfio.com
with username and password list was provided by hackinghub.io but it would took me so long 

i tried to enumerate directory/files didn't find anything except style.css 
so i watched ben's [video](https://www.youtube.com/watch?v=lUUL2dNQI5M) and i saw him using app-dev
not just app so i tested app-dev with curl
![[Pasted_image_20230920074801.png]](/screenshots/Pasted_image_20230920074801.png)
and we got the flag there.
done

lesson learned: don't rely on one subdomain list 
because even i was provided with subdomain from the challenge it didn't contain the  app-dev

so i wanted to know where is app-dev in my seclist/DNS/*
![[Pasted_image_20230921020527.png]](/screenshots/Pasted_image_20230921020527.png)
with grep -l i  found out that my usual go to lists aka 
- /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
- /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
- /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt

didn't contain app-dev. so if you stuck on any point do more enumeration 
u will get your foothold eventually 


> Character develops itself in the stream of life.
> — <cite>Johann Wolfgang von Goethe</cite>