---
title: "Philanthropy"
date: "2023-09-18"
draft: false
---
Type: #WEB #blackbox 
Difficulty: #medium 
SOLVED by: #writeup 
TOOL USED: #burp #devtools

---

Writeup Date:2023-09-18
URL = http://web.csaw.io:14180/web/home

---------
i couldn't solve this challenge myself so here is my attempt tries solving it
and the solution.

and i will reference the writeup at the end of this

the challenge description 
![[Pasted_image_20230918222655.png]](/screenshots/Pasted_image_20230918222655.png)

upon visiting the challenge url
![[Pasted_image_20230918222955.png]](/screenshots/Pasted_image_20230918222955.png)
WE SEE login and register functionality. hit ctrl+u
![[Pasted_image_20230918223314.png]](/screenshots/Pasted_image_20230918223314.png)
we view source page we see js file that react.js .. that loads every page from the dom ig upon requesting it so i don't see anything useful in source page for any of page in this site.

![[Pasted_image_20230918223407.png]](/screenshots/Pasted_image_20230918223407.png)
so basically js file is useless for us. (for the most part)

i visit the about page
![[Pasted_image_20230918224415.png]](/screenshots/Pasted_image_20230918224415.png)
i read through it and found 2 interesting piece of information.

1.) Vigilance is key; at How to Identify a Metal Gear?
2.)otacon@protonmail.com at how can i help

so maybe we can bruteforce octan user knowing his email?

i go and register a user with burp on
![[Pasted_image_20230918225010.png]](/screenshots/Pasted_image_20230918225010.png)
so we see that we are **POST** to an api endpoint */identify/register*

then we login to */identify/login* endpoint
with email and password 

after login in i see upgrade page
so i visit it 
![[Pasted_image_20230919001316.png]](/screenshots/Pasted_image_20230919001316.png)
 so i try random number aaand
 ![[Pasted_image_20230919001632.png]](/screenshots/Pasted_image_20230919001632.png)
we are sending a json in the body and POST method to */identify/upgrade*

after i sent the request i don't see burp **catching the response**  but I see "incorrect code!" in the page now
 
so maybe  code checking mechanism part of it on the client side ?

and i see JWT in the request i made 
![[Pasted_image_20230919002218.png]](/screenshots/Pasted_image_20230919002218.png)
decoding it i see that **member is set to false**
so i thought it maybe a **weak secret** attack on the JWT

## THE JWT RABBIT HOLE
i tried cewl the about page but it didn't work because
cewl read the source code and does not find any text except Philanthropy.
i added some word manually to a wordlist
	Philanthropy
	Vigilance
	etc..
i used 
```bash
hashcat -a 0 -m 16500 <JWT> wordlist.txt
```
and failed to crack it. 
## reflection
i should have stopped here and moved on another thing other than JWT attack especially falling the weak secret attack.

## THE JAVASCRIPT DEBUGGING RABBIT HOLE
i for some reason i thought that i maybe caan look at the js file and  look how it process the member code submission
and setting different break point using devtool 
it was nightmare and i wasted so much time.

the only thing i found useful that i have found all the api endpoints without using any tool.

and here is stopped trying.

## solution 
at profile tab u can edit ur first/last name
and will post the new information to 
/identify/update at this point u can edit the request and add 
,"member":true
to the json and forward the request

now u are a member
![[Pasted_image_20230919035038.png]](/screenshots/Pasted_image_20230919035038.png)
![[Pasted_image_20230919035102.png]](/screenshots/Pasted_image_20230919035102.png)
navigating to identify and seeing the network activity from devtool (always use it if burb and can't catch some response ) we see different emails 
but snake was mentioned in the challenge description 
![[Pasted_image_20230919035355.png]](/screenshots/Pasted_image_20230919035355.png)
we see filename:blaalala.png
if we copied that and went to /images/blaalala.png
![[Pasted_image_20230919035458.png]](/screenshots/Pasted_image_20230919035458.png)
And logging in, we can access the flag!
![[Pasted_image_20230919035528.png]](/screenshots/Pasted_image_20230919035528.png)
And our flag is `csawctf{K3pt_y0u_Wa1t1ng_HUh}`!

> Technology is anything that wasn't around when you were born.
> — <cite>Man Ray</cite>