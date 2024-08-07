---
title: "Screenshot"
date: "2023-05-13"
draft: false
---
type: #WEB #whitebox
difficulty: #hard
SOLVED by: #writeup
![[Pasted_image_20230513054450.png]](/screenshots/Pasted_image_20230513054450.png)
First, review the application overview.

-   The application returns a screenshot of any page as a PNG image.
-   The URL is specified by a GET parameter like `/api/screenshot?url=https%3A%2F%2Fwww.google.com%2F`.
-   The backend is implemented in JavaScript (Node.js) and opens the page with Headless Chromium to render the content.
-   The flag is stored in `/flag.txt`.

In applications where the server accesses the URL specified by the user, an attack technique called SSRF (Server Side Request Forgery) is known.

(Note: The following article is written in Japanese; you may find a similar information in English by Googling.)

[A Thorough Introduction to SSRF (Server Side Request Forgery) | Hiroshi Tokumaru's Diary](https://blog.tokumaru.org/2018/12/introduction-to-ssrf-server-side-request-forgery.html)

Chrome can display the contents of local files using URLs starting with `file://`, such as `file:///etc/passwd`. Let's try to use this to see if we can get it to take a screenshot of `file:///flag.txt`. If you enter `file:///flag.txt` in the form and submit it, unfortunately, a "Bad Request" error shows up and the attack will fail.

Let's read the distributed source code to see why an error is displayed when we submit `file:///flag.txt`. If you open `index.js`, you will find the following code on line 20.
```javascript
if (!req.query.url.includes("http") || req.query.url.includes("file")) {
  res.status(400).send("Bad Request");
  return; }
}
```
It seems that the system inspects the value of the `req.query.url`, or `url` parameter of the query string, and returns an error if it does not contain `http` or contains `file` as a substring.

```js
console.log(req.query.url) // => "file:///flag.txt"
console.log(req.query.url.includes("http")) // => if the string coitains "http" (false)
console.log(req.query.url.includes("file")) // => if the string coitains "file" (true)
```
Actually, there is a flaw in this check.

When a query with multiple `url` parameters is passed, such as `?url=abc&url=def`, the contents of `req.query.url` will be an array. Since `Array.prototype.includes()` is a method that returns whether a particular element is included or not, it cannot check whether a particular string is included as a substring.
```js
console.log(req.query.url) // => ["abc", "def"].
console.log(req.query.url.includes("http")) // => if the **array** contains "http" (false)
console.log(req.query.url.includes("file")) // => if the **array** contains "file" (false)
```

Thus, when a query string such as `?url=file%3A%2F%2F%2Fflag.txt&url=http` is passed, it will pass the checks. Such a bug is sometimes called **[parameter type confusion](https://codeql.github.com/codeql-query-help/javascript/js-type-confusion-through-parameter-tampering/).**
![[Pasted_image_20230513073051.png]](/screenshots/Pasted_image_20230513073051.png)
Let's look at what happens to the URLs opened in Chromium when multiple `url` parameters are passed. In line 27 of `index.js`, we see the following
```js
const params = new URLSearchParams(req.url.slice(req.url.indexOf("?")));
await page.goto(params.get("url"));
```
This code extracts the back part of the URL after the `?`, and uses `URLSearchParams` to retrieve the value of the `url` parameter. If you read the specification of `URLSearchParams.prototype.get()`, you will see that when multiple identical parameters are passed, the first one is returned. That is, `?url=file%3A%2F%2F%2Fflag.txt&url=http` will return `file:///flag.txt`.
![[Pasted_image_20230513075307.png]](/screenshots/Pasted_image_20230513075307.png) it's **array** url=\["file\:///flag.txt" "http"] **insted** of **string** so the check won't work
-   [URL Standard](https://url.spec.whatwg.org/#dom-urlsearchparams-get)
-   [URLSearchParams: get() method - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/get)

Thus, accessing `/api/screenshot?url=file%3A%2F%2F%2Fflag.txt&url=http` will allow you to pass the checks and have Chrome open `file:///flag.txt`.

Note that an unintended solution found during the event is to specify `filE:///http/../flag.txt` as the URL. You can also get the flag this way (thanks for the report).

![[Pasted_image_20230513071553.png]](/screenshots/Pasted_image_20230513071553.png)
![[Pasted_image_20230513071617.png]](/screenshots/Pasted_image_20230513071617.png)
file scheme require :/// after it if you do a path relative path it will trigger error. 
so the correct way is ///path "the absloute path"

FLAG{beawre_of_parameter_type_confusion!} DONE!!