## Challenge

In this challenge, we were given a url to a website

## Discovery

When we tried to hit the website in our browser, we received a `403 Forbidden`. We poked around in the network calls
and hit it with curl.

Then turned to good ol' Google. This article was extremely enlightening:

https://shubs.io/enumerating-ips-in-x-forwarded-headers-to-bypass-403-restrictions/

## Solution

We implemented the solution described in the url above.
