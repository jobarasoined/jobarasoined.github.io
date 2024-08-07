---
title: "64bps"
date: "2023-05-13"
draft: false
---
type: #WEB #whitebox
difficulty: #easy
SOLVED by: #writeup

![[Pasted_image_20230513002425.png]](/screenshots/Pasted_image_20230513002425.png)The challenge statement contains a Unix command, which means the following.

 Create 2gb.txt and write 2 GiB (2147483648 bytes) of random data
`dd if=/dev/random of=2gb.txt bs=1M count=2048`

Append the contents of flag.txt to the end of 2gb.txt
`cat flag.txt >> 2gb.txt`

Delete flag.txt
`rm flag.txt`

So we can see that the first 2 GiB of `2gb.txt` is random data and the flag is at the end.

Now, if you try to open `2gb.txt` from the link provided in the statement, the page should stay stuck while loading. If you download the zip file in question, you will see the following statement in `nginx.conf`, indicating that the server will only let you download at a speed of 64 bps per connection.

```
limit_rate         8; # 8 bytes/s = 64 bps
```

If you try to download a 2 GiB file from the beginning, it would take 10 years to reach the end, and you would never make it to the end of WaniCTF in time.

This is where the `Range` header of HTTP comes into play. With this header, you can download just any part of a huge response.

### [HTTP range requests - HTTP MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Range_requests)

```bash
curl --range -60 https://64bps-web.wanictf.org/2gb.txt -- method by video walkthrough
here we want the last 60 byte from the file
```


```bash
curl https://64bps-web.wanictf.org/2gb.txt -i -H "Range: bytes=0-1"

HTTP/1.1 206 Partial Content
Server: nginx
Date: Fri, 12 May 2023 22:53:52 GMT
Content-Type: text/plain
Content-Length: 2
Connection: keep-alive
Last-Modified: Mon, 01 May 2023 04:40:51 GMT
ETag: "644f42d3-80000031"
Content-Range: bytes 0-1/2147483697

### WE can see the content-range and minus the last 64 byte from it 
```

```sh
In this case, we want the `2147483648`th byte of the file and beyond, so we can get the flag by using curl as follows.

curl --verbose --header 'Range: bytes=2147483648-' https://64bps-web.wanictf.org/2gb.txt

# The same can be written as follows
curl --verbose --range '2147483648-' https://64bps-web.wanictf.org/2gb.txt
```

FLAG{m@ke_use_0f_r@n0e_reques7s_f0r_l@r9e_f1les} DONE!!