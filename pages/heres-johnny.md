## Challenge

The clue: "Okay, so we found some important looking files on a linux computer. Maybe they can be used to get a password to 
the process."

Along with this, I got two files, "shadow" and "passwd"

## Discovery

First I checked the contents of the two files.

```
cat passwd
>>root:x:0:0:root:/root:/bin/bash
```

```
cat shadow
>>root:$6$LcvKHioa$67O1HA8Ti.KHeNbD4rE79ZMl1RbiCw4V7eM.r6AURp2wGnapUpXC.VdVB4WGoS2J5eVKP/1MFeMmXIdveJeOS0:17695:0:99999:7:::
```

I also read a little bit to understand more about shadow and passwd files.
- [Understanding etcpasswd file format](https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format/)
- [Linux shadow file explained](https://web.archive.org/web/20131225075704/http://www.thegeekscope.com/linux-shadow-file-explained/)

## Solution

I started by "unshadowing" the shadow file, storing the result in a new file I called `mypasswd`

```
unshadow passwd shadow > mypasswd
```

Next, I opened Kali Linux and tried to brute force the solution with John the Ripper. This tries "single crack" mode first,
then uses a wordlist with rules, then goes to "incremental" mode

```
john mypasswd
```

After about 3 hours, well into "incremental" mode, I gave up. After some searching online, I found a wordlist called "rockyou.txt"
(one of the clues for the challenge was "rock you"). I downloaded the file (~130MB) and ran John the Ripper with this list.

```
john mypasswd —wordlist=rockyou.txt
```

In less than a minute, john the ripper had found the password

```
john --show mypasswd
>>root:thematrix:0:0:root:/root:/bin/bash
```

___

Pico CTF 2018 - October 2018
