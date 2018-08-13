## Challenge

In this challenge, we were given an IP Address and cryptic clue: "Welcome to Hacker2, where uptime is a
main priority".

## Discovery

When we tried to hit the address in our browser, we received a `403 Forbidden`. We poked around in the network calls
and hit it with curl. Then turned to good ol' Google. This article was extremely enlightening:

[Enumerating IPs in X-Forwarded Headers to Bypass 403 Restrictions](https://shubs.io/enumerating-ips-in-x-forwarded-headers-to-bypass-403-restrictions/)

## Solution

As described above, the `X-Forwarded-For` header contains an ordered list of IP addresses. These can be unlimited in number, and are separated by a comma. The furthest left IP address represents the client's address, and all others represent intermediate proxies. When a server receives this list of IPs, it traverses the list in reverse in order to determine the trustworthiness of the client's address.

We set the X-Forwarded-For header in our request to include a list of four IPs that all matched the original local IP of the website.

```
X-Forwarded-For: <Local IP>, <Local IP>, <Local IP>, <Local IP>
```

This request allowed us to bypass the 403 and get the flag.
