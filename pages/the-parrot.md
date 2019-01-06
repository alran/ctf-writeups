## Challenge

I received a locked pdf and the clue "Can you find the missing parrot?"

## Discovery

I downloaded the pdf and confirmed that it was password protected. Then, I used `pdfid`
to confirm it was encrypted.

```
> pdfid -e parrot.pdf

  PDFiD 0.2.1 parrot.pdf
   PDF Header: %PDF-1.5
   ...
   /Encrypt               1
   ...
```

This command output a long list of details about the pdf, including something called `/Encrypt`,
which was set to 1.

I read a lot about pdfs after this initial discovery, and realized that in order to properly
run this pdf through a password cracker, I would need to get the password hash from the document.
First, I tried using `pdf-parser` with the `-s` or `--search` option, passing in `/Encrypt` as my
argument.

```
> pdf-parser -s /Encrypt parrot.pdf

  obj 38 0
   Type: /XRef
   Referencing: 2 0 R, 1 0 R, 37 0 R
   Contains stream

    <<
      /Type /XRef
      ...
      /Size 39
      /ID [<8d8f274ea6bdaf47b827feced9665dff><c186f14cd43e2b76c77deb8f036269c5>]
      /Encrypt 37 0 R
    >>
```

This returned a medium sized chunk of relatively useless data, except for the last line `/Encrypt 37 0 R`.
The first number following `/Encrypt` is the location of the password hash.

However, knowing the location of the hash didn't turn out to be super helpful.

## Solution

I used [pdf2john](https://github.com/magnumripper/JohnTheRipper/blob/bleeding-jumbo/run/pdf2john.pl) to grab the 
password hash. This tool comes with the jumbo version of [John The Ripper](https://github.com/magnumripper/JohnTheRipper), 
which I did not have on the machine I was using. I ended up using a python version instead (john comes with a 
.pl version).

```
> ./pdf2john.py parrot.pdf | ./csv-cut.py -s : 1 > parrot.hash
```

This output the password hash to parrot.hash.

```
> cat parrot.hash

  $pdf$1*2*40*-8*1*16*8d8f274ea6bdaf47b827feced9665dff*32*e3789e9700b5230fc63d2b54712bd9014df1a6d
  56c6f555d0c0d9ec6b962d3d6*32*8b5f4006c0d5697bca85a008a27d1415e79d78feff796d6ec6d39c0596607b15
```

Once I got the hash, I ran it through John The Ripper, with no specific options set. It found the password
immediately.

```
> john parrot.hash --show

  ?:cracker
  1 password hash cracked, 0 left
```

I unlocked the pdf using `cracker` as the password and was greeted with a basic looking pdf with a picture of a parrot
and the words "The Parrot", but nothing that looked like a flag.

I saved the image to my computer so that I could analyze it more closely, and was surprised when I opened the file
to see that it had completely changed. Instead of a green parrot, the image was of a cartoon bird with the flag.


Additional Reading: [Cracking Encrypted PDFs](https://blog.didierstevens.com/2017/12/26/cracking-encrypted-pdfs-part-1/)

___

Code Mash CTF - January 2019
