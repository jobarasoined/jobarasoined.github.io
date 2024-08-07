---
title: "one-for-all"
date: "2023-09-10"
draft: false
---
type: #WEB #blackbox
difficulty: #easy
SOLVED by: #myself and biogenisis

writeup on how we-- aced first blood on one-for-all challenge
patriotCTF 2023

was rated easy in first but later PatriotCTF Rated it hard as u can see in the screenshot

the challenge
![[Pasted_image_20230910014131.png]](/screenshots/Pasted_image_20230910014131.png)

the first thing we see is a field require from us a username
![[Pasted_image_20230910015024.png]](/screenshots/Pasted_image_20230910015024.png)

as any fellow hacker i typed the normal thing and hit the big button
![[Pasted_image_20230910015148.png]](/screenshots/Pasted_image_20230910015148.png)

![[Pasted_image_20230910015232.png]](/screenshots/Pasted_image_20230910015232.png) 

No such user exists (keep that in mine)


i noticed something on the main page in top corner and and it bothred me so i went back and clicked on it

![[Pasted_image_20230910015707.png]](/screenshots/Pasted_image_20230910015707.png)

it sent me to http://chal.pctf.competitivecyber.club:9090/user?id=1
![[Pasted_image_20230910015823.png]](/screenshots/Pasted_image_20230910015823.png)
id = 1 is wilson so maybe different id number can lead to different name 
and i can collect them to try on the main page huh i'm so smart

tried id=2 aaaaand 
![[Pasted_image_20230910020015.png]](/screenshots/Pasted_image_20230910020015.png)

nothing changed ..
so maybe it's three then (said with hope)
nope not three ethier 

so i sent the request to the intruder in burpsuite and set the payload from 0 to 100 and let it run

while i was waiting burp to finish i went back on the main page and tried wilson

and got the No user exist error.

so i thought maybe  it's fetching from a database so it's maybe vulnerable
to sqli 

i tried `wilson" or 1=1 -- -`
and 
![[Pasted_image_20230910020741.png]](/screenshots/Pasted_image_20230910020741.png)

YES it vulnerable to SQLi and maybe we can dump the flag 

before we do so let's take look on burp
at the same moment i hear discord notification so i change tabs and see BioGensis

tellin me he grabbed <\div description in the source code
(hit ctrl + u to see source code)


he got the part of the flag in user?id=0 :)

and burp confirm that because the only content length different happen 
at user?id=0
the part he got was `ev3rYtH1nG}`

so maybe my SQLi we will be the rest and we solve the challenge to move on with our teammate on  **Pick Your Starter** challenge 

hint NO we won't
not even close 💀

so i tried to go to hacktricks  sqli page 

we aleardy no it's sqllite database because the commect -- - worked 
yet u can confirm with last_insert_rowid()=last_insert_rowid() which will equal
to ture in a SQLlite database

anyway
`a" union select 1,1,1,1 -- -`
and keep increasing "1," if u don't get true response 
that is the count of columns (4)
inject `(select sql from sqlite_schema)`
into one of the columns
so its like that 
`a" union select 1,1,(select sql from sqlite_schema),1 -- -`

all that i didn't know and biogenisis was the one who solved that part 
i googled everything later tho
so i don't be script kidyy 
-but we all are right 💀-
![[Pasted_image_20230910050059.png]](/screenshots/Pasted_image_20230910050059.png)
so we know that the name of the table name is accounts contains id,email,username,password 

`a" union select id,email,username,password from accounts limit 1 offset 2 -- - `

why offset 2 and limit 1 ??
first if u don't know limit and offset see [this](https://www.sqltutorial.org/sql-limit/) 

offset 2 will skip  kairn
-we already know from a" or 1=1 -- trial-

and limit 1 because the page can't dumb all the table in one time
so we go only by 1
and increment offset till we find what we want

![[Pasted_image_20230910051133.png]](/screenshots/Pasted_image_20230910051133.png)
offset 2 get us that
![[Pasted_image_20230910051810.png]](/screenshots/Pasted_image_20230910051810.png)
part of the flag
so we now have `and_Adm1t_ev3rYtH1nG}`

offset 3 get us something interesting
![[Pasted_image_20230910052042.png]](/screenshots/Pasted_image_20230910052042.png)

a path ..  secret one 
![[Pasted_image_20230910052236.png]](/screenshots/Pasted_image_20230910052236.png)
we get you don't have permission to acccess /secretsforyou/
just like a 403 error 
but it's just html page not real 403

i tried maybe if i added x-forward-from: 127.0.0.1
maybe it will work. it didn't, i tried different local ips 10.12.x.x 192.168.x.x
nothing worked. 

so i ran [dontgo403](https://github.com/devploit/dontgo403) great tool to bypass 403 written in golang so it's super fast
it can try different headers too like  x-forward-from and more

![[403.png]](/screenshots/403.png)
so 178 bytes stood out to my eyes

visiting the page we get the third part of the flag
![[Pasted_image_20230910053320.png]](/screenshots/Pasted_image_20230910053320.png)

flag so far `l00s3_and_Adm1t_ev3rYtH1nG}`

so we missing part
hinting the name one-FOR{UR}-all

i just sent a get request in burp 
previously name=karin i changed to name=admin earlier 
and it was the solution 
```
 GET / HTTP/1.1
Host: chal.pctf.competitivecyber.club:9090
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: Flag 5/5=e4a541}; name=admin
Upgrade-Insecure-Requests: 1
DNT: 1
Sec-GPC: 1
Content-Length: 0
```
and
![[burp.png]](/screenshots/burp.png)
boom
`PCTF{Hang_l00s3_and_Adm1t_ev3rYtH1nG}`
![[firstblood.png]](/screenshots/firstblood.png)

was nice challenge combined bunch of vulnerability
IDOR, SQLi, 403 bypass !! the latter was just amazing idea tbh

checkmate challenge was a lot funnier but that for another writeup
see you guys later.