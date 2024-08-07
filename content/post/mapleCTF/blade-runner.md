---
title: "blade-runner"
date: "2023-10-01"
draft: false
---
Type: #WEB #whitebox 
Difficulty: #easy
SOLVED by: #myself 
TOOL USED: docker redis burpsuite 
TOPIC: prototype pollution 

---

Writeup Date:2023-10-01
URL = https://ctf.maplebacon.org/instances

---------

the challenge description 
![[Pasted_image_20231001161241.png]](/screenshots/Pasted_image_20231001161241.png)
we have source code so we can see what happening in the backend + we can run our docker container instead of trying to solve with 10 min time window before the instance shutdown

`uznip blade-runner.zip` to extract the src
### index.js
![[Pasted_image_20231001161829.png]](/screenshots/Pasted_image_20231001161829.png)
import some js stuff and import ./util from local folder
so this is custom code and maybe there is vulnerability some where caused by human 

in index.js we see that the flag is stored in the environment so if we can have RCE / LFI we can read /proc/self/environ and get the flag --1
i also see the flag present in **/joi endpoint.**

so how can we access the joi endpoint ? 
![[Pasted_image_20231001162434.png]](/screenshots/Pasted_image_20231001162434.png)
we will have to see what is util.auth doing. 
first
we can see docker files but there is something interesting in those ![[Pasted_image_20231001162956.png]](/screenshots/Pasted_image_20231001162956.png)
let's see our challenge live and up
`sudo docker-compose up` make sure u are in the same path as docker-compose.yml file.
and will let it build the challenge 

when i saw redis i used searchspolit and found ![[Pasted_image_20231001163755.png]](/screenshots/Pasted_image_20231001163755.png)
but while building the image i saw that he uses the latest build which doesn't have any know CVE (YET) 
and we can confirm that with ![[Pasted_image_20231001164053.png]](/screenshots/Pasted_image_20231001164053.png)
**notes** redis 4.6.10 is not reflection that the app use redis version 4.x and prone to RCE but to what the app pull from npm  ![[Pasted_image_20231001164441.png]](/screenshots/Pasted_image_20231001164441.png)
no rce for us :'(

our challenge is up let's give it a visit
u should be able to visit it at http://localhost:6969

![[Pasted_image_20231001173818.png]](/screenshots/Pasted_image_20231001173818.png)
if we try access joi endpoint `login required ` message is shown 

let's register account then![[Pasted_image_20231001174252.png]](/screenshots/Pasted_image_20231001174252.png)
once we hit submit we are redirect to http://localhost:6969/user/login?username=a&password=a and we get noting 
so inspect the request and i see that they are using get method so this ring alarm to me  

i did some changes ![[Pasted_image_20231001174804.png]](/screenshots/Pasted_image_20231001174804.png)
so i dig in the source code for "invalid body" error.
![[Pasted_image_20231001175143.png]](/screenshots/Pasted_image_20231001175143.png)
if the username and password are not objects will return invalid body
so let's change or request to json format i used content-type-converter extension
 from burpsuite u could do it manually too i just love not to rebuild any wheels (thank you foss community )
![[Pasted_image_20231001190041.png]](/screenshots/Pasted_image_20231001190041.png)
and it redirect me to the login endpoint 
how to know we registered account successfully ?
if this is blackbox challenge we can try and login (we will sent the data in json as well) and see what is the response 
OR 
we can monitor redis (redies is datebase in abstract form)

first we have to know our redis ip to connect from our machine 
`sudo docker ps`
![[Pasted_image_20231001190535.png]](/screenshots/Pasted_image_20231001190535.png)
`sudo docker exec <redis container id> ip a s`
![[Pasted_image_20231001190719.png]](/screenshots/Pasted_image_20231001190719.png)
on our machine `redis-cli -h 172.20.0.2`default port at 6379 if not in that case u can use -p \<non-stander-port>   
![[Pasted_image_20231001191015.png]](/screenshots/Pasted_image_20231001191015.png)
use command `Monitor` in redis to monitor everything happening in the database
![[Pasted_image_20231001191707.png]](/screenshots/Pasted_image_20231001191707.png)
set foo a = registered successfully 
now what ?
![[Pasted_image_20231001175143.png|600]](/screenshots/Pasted_image_20231001175143.png|600)
2.) we are blocked from registering as username "admin" 
as hackers do they don't go by the rules

### util
**user_route.js** 
![[Pasted_image_20231007132808.png]](/screenshots/Pasted_image_20231007132808.png)
if we logged with vaild username and password
our **req.session.user is set to our username**  
and redirect to /joi where is the flag is in the responses
**util.auth.js**
![[Pasted_image_20231007133147.png]](/screenshots/Pasted_image_20231007133147.png)
we see that if our req.session.user is not "admin"
we will get ADMIN REQUIRED message when  accessing /joi
so we have register username as "admin"

so how we would bypass
```js 
if (k.toLowerCase() == "username" && req.body[k].toLowerCase() == "admin") {
            return res.status(400).send("You can't use that username."); 
```
.toLowerCase() so our input is case-insensitive 
i tried to use unicode in the json so admin be something like
'\\u0061\\u0064\\u006D\\u0069\\u006E' the filter caught me still.

why? becasue json will force utf-8 in the content-type header and if u changed to unicode will cause an error.

if u just passed it in "username":"\\u0061\\u0064\\u006D\\u0069\\u006E" 
json will return it to utf-8 and will be blocked by the filter

we are left to [pototype pollution](https://www.youtube.com/shorts/HzedAeTppHI)
![[Pasted_image_20231007143711.png]](/screenshots/Pasted_image_20231007143711.png)
we registered username "admin" successfully 
and we will try to login
![[Pasted_image_20231007143854.png]](/screenshots/Pasted_image_20231007143854.png)
follow redirecting 
![[Pasted_image_20231007144046.png]](/screenshots/Pasted_image_20231007144046.png)
we get the flag locally if it doesn't show flag first time just keep sending the request in the repeater  or keep hitting F5 in the broswer(use the connect.sid cookie) due to 
![[Pasted_image_20231007144335.png]](/screenshots/Pasted_image_20231007144335.png)
which will not render the full content length.
![[Pasted_image_20231007155513.png]](/screenshots/Pasted_image_20231007155513.png)
