---
title: "Extract Service 1"
date: "2023-05-07"
draft: false
---
type: #WEB #whitebox
difficulty: #easy
SOLVED by: #myself

lesson learnt > after almost 2 days from trying
and not understading shit being copypasta person
if you don't understand the code playing with it isn't enough

the challenge
![[Pasted_image_20230507234311.png]](/screenshots/Pasted_image_20230507234311.png) 
## opening the web page  i see this 

![[Screenshot_from_2023-05-05_15-32-23.png]](/screenshots/Screenshot_from_2023-05-05_15-32-23.png)

uploading a file and the site spit out its content 

looking and the source code 
![[Pasted_image_20230508005421.png]](/screenshots/Pasted_image_20230508005421.png) 
it's takes a zipped file 'docx is a zipped file btw' to unzip it and read a traget
AT LINE 38 we see that we control the key-value ExtractTarget in our POST request
rn its word/document let's do poc
![[Screenshot_from_2023-05-05_16-03-47_1.png|1080x720]](/screenshots/Screenshot_from_2023-05-05_16-03-47_1.png|1080x720)
we will replace word/document.xml with etc/passwrd and see what will happen 
![[Screenshot_from_2023-05-05_16-21-17.png|1028]](/screenshots/Screenshot_from_2023-05-05_16-21-17.png|1028)
we were right Error : open /tmp/{uuid}/etc/paswrd
NOW let's try ../../../../../../../flag
![[Screenshot_from_2023-05-05_16-22-19.png|1028]](/screenshots/Screenshot_from_2023-05-05_16-22-19.png|1028)
WE GOT OUR FLAG :)