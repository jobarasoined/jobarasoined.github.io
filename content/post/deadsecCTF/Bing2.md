---
title: "Bing2"
date: "2024-07-27"
draft: false
---
Type: #WEB #whitebox 
Difficulty: #easy 
SOLVED by: #myself 
TOOL USED: #burp 
TOPIC: #command_injection

---

Writeup Date:2024-07-27
URL = https://00b46582d765094ac90bb4db.deadsec.quest/bing.php

---------

the challenge description 
source code was provide through [link](https://drive.proton.me/urls/JFNPCV77V4#dLqn62g51E4N)on discord
first let's see the website 
![[Pasted_image_20240727040922.png]](/screenshots/Pasted_image_20240727040922.png)
i clicked on CTRL-U to see sourcecode of the page
![[Pasted_image_20240727041135.png]](/screenshots/Pasted_image_20240727041135.png)
no functionality was found so i looked at the sourcecode provided  
![[Pasted_image_20240727041628.png]](/screenshots/Pasted_image_20240727041628.png)
found dockers file and fake lag to run the challenge locally
---
but the interesting thing we found bing.php file so we don't have to brute force files/directory on the instance 


## bing.php
---
upon inspecting bing.php
we found if statement that require parameter "Submit" to be set and if so it checks another parameter (ip) and remove any white space in it
and the substitutions array
![[Pasted_image_20240727043626.png]](/screenshots/Pasted_image_20240727043626.png)
![[Pasted_image_20240727043854.png]](/screenshots/Pasted_image_20240727043854.png)
so it replace arraykeys with arrayvalue in the target we set (ip paramater) and then pass it to shell_exec to ping it
if we can add to the ping command another command we will get RCE and can read the flag

### TO DO LIST
- first we need to create POST request to bing.php and pass 2 parameters Submit (first letter must be as challenge stated in this case capital ) and ip (website to ping it and BTW the macine has no connection to the internet so you will have to ping localhost)
- we will need to bypass the filter somehow to inject command that's allow us to read the flag
---


using burbsuite we will intercept request to /bing.php
so our initial request look like this 
![[Pasted_image_20240727045006.png]](/screenshots/Pasted_image_20240727045006.png) 
we will make a lot of changes and tries/errors
so send the request to the Reapter CTRL-R
and change method to POST Instate OF GET
and add our Submit and ip paramters
![[Pasted_image_20240727045941.png]](/screenshots/Pasted_image_20240727045941.png)
### bypass the filter

as we can see a lot of command and command operators eg 
`| & ; || && () % ~ <> / \ - 
are gonna be replaced with nothing
and trim in php will remove any white space
but there is mistake in the implementations 
![[Pasted_image_20240727051035.png]](/screenshots/Pasted_image_20240727051035.png)
spotted ? 
if not yet the code will look `;+space` not just `;`
so if we append the command without anyspace in between it should work fine but a lot of commands are banned 
but there is mistake in the implementations too 
`;ls` ==> wont work because "ls" is banned 
`;l''s` ==> will work because `''` in  bash gonna eval to empty string and ls gonna be concatenated you can also do it with `""`
![[Pasted_image_20240727052524.png]](/screenshots/Pasted_image_20240727052524.png)
great now we can use `ca''t /flag.txt` to get flag right ?
WRONG 
for two reasons
1. you put space between cat and /flag it will get removed and error 
2. you typed "flag" which also banned in the array
we will resolve the first issue by replacing every space we wanna type with special variable in bash called IFS 
anytime you want SPACE type ${IFS}
second flag is banned but we can type fl??.txt or \*.txt

Bash shell supports three wildcards, one of which is the question mark (?). You use wildcards to replace characters in filename templates. A filename that contains a wildcard forms a template that matches a range of filenames, rather than just one.
for REFERENCE check this [article](https://www.howtogeek.com/439199/15-special-characters-you-need-to-know-for-bash) 

so final request gonna look like this 
```bash
POST /bing.php HTTP/2
Host: 00b46582d765094ac90bb4db.deadsec.quest
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/114.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Dnt: 1
Sec-Gpc: 1
Sec-Ch-Ua-Platform: "Windows"
Sec-Ch-Ua: "Google Chrome";v="112", "Chromium";v="112", "Not=A?Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Te: trailers
Content-Type: application/x-www-form-urlencoded
Content-Length: 63

Submit=anyValueWillEvaluteToTrue&ip=127.0.0.1;ca''t${IFS}/fl?g.txt


```

![[Pasted_image_20240727060621.png]](/screenshots/Pasted_image_20240727060621.png)
> Science investigates; religion interprets. Science gives man knowledge which is power; religion gives man wisdom which is control.
> — <cite>Martin Luther King Jr.</cite>