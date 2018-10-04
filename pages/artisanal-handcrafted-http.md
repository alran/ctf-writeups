## Challenge

The clue: We found a hidden flag server hiding behind a proxy, but the proxy has some... _interesting_ ideas of what qualifies 
someone to make HTTP requests. Looks like you'll have to do this one by hand. Use the proxy to send HTTP requests to 
"flag.local" 

I was also given a username and password to login.

## Discovery

I hit the proxy endpoint using netcat `nc <address> <port>` and was immediately prompted to "Commence HTTP". I tried
something basic first:

```http
GET / HTTP/1.1
Host: flag.local
```

This returned HTML with a link `<a href="/login">Login</a>` to a login page.

```http
GET /login HTTP/1.1
Host: flag.local
```

The login page returned an HTML form with named inputs for "user" and "pass"

```html
<form method="POST" action="login">
   <input type="text" name="user" placeholder="Username" />
   <input type="password" name="pass" placeholder="Password" />
   <input type="submit" />
</form>
```

I spent some time reading about how to create a post request with form data in HTTP.
- [Mozilla: Sending and Receiving Form Data](https://developer.mozilla.org/en-US/docs/Learn/HTML/Forms/Sending_and_retrieving_form_data)
- [Stack Overflow: How Are Parameters Sent in an HTTP Post Request](https://stackoverflow.com/questions/14551194/how-are-parameters-sent-in-an-http-post-request)


## Solution

I sent a post request to `/login` with the credentials given in the clue. I spent some time making sure I got the
correct syntax for this. Without the `Content-Length` header, the HTTP request would send prematurely, before I got a
chance to add the username and password. If the Content-Length was too short, the request would hang until there
were enough characters or until I force quit. 

```http
POST /login HTTP/1.1
Host: flag.local
Content-Type: application/x-www-form-urlencoded
Content-Length: 38

user=realbusinessuser&pass=potoooooooo
```

This returned a cookie via the `set-cookie` header.

```http
HTTP/1.1 302 Found
x-powered-by: Express
set-cookie: real_business_token=PHNjcmlwdD5hbGVydCgid2F0Iik8L3NjcmlwdD4%3D; Path=/
location: /
vary: Accept
content-type: text/plain; charset=utf-8
content-length: 23
date: Thu, 04 Oct 2018 01:48:32 GMT
connection: close

Found. Redirecting to /
```

I added this cookie to the original get request and was served HTML containing the flag!

```http
GET / HTTP/1.1
Host: flag.local
Cookie: real_business_token=PHNjcmlwdD5hbGVydCgid2F0Iik8L3NjcmlwdD4%3D; Path=/
```


___

Pico CTF 2018 - October 2018
