## Challenge

The clue:

> I don't know why anyone worries about broken hash algorithms when there are just
-so many- of them out there! If you just stack them all together, the flaws in
each individual algorithm cover each other. It's basic Defense-in-Depth, and yet
the entire cryptographic community never considered it! Introducing the MegaHash
cryptographically secure hash algorithm. Basically, I stack together all the
available algorithms in a vanilla Python 3.6 instance together in a sequence (a
la BlockChain technology). It’s 100% unbreakable unless every single one of them
is broken! For your first futile opportunity to get points, implement the easiest
of the cryptographic attacks, a collision attack! -- https://en.wikipedia.org/wiki/Collision_attack

Along with this, I got two files, "server.py" and "MegaHash.py" and a <host><port> for the challenge.

## Discovery

I started by hitting the `<host><port>` directly in my browser. The HTML page that
loaded had two input boxes. These corresponded with the contents of "server.py",
which set up a Flask server with a single endpoint that compared the hashed
values of two user-supplied inputs.

```python
calculatedHash1 = MegaHash.hash(input1.encode('ASCII'))
calculatedHash2 = MegaHash.hash(input2.encode('ASCII'))
if(calculatedHash1 == calculatedHash2):
    return "Correct! Your flag is: " + flag
```

The "MegaHash.py" file was more complex, defining an algorithm that appeared to
run the user input string through multiple popular hashing algorithms.

However, I noted that one part of the code had a note about further investigation.

```python
MegaHashAlgorithms = ['sha3_384', 'blake2s', 'sha3_256', 'sha224', 'sha384', 'md5',
                      'sha3_224', 'shake_256', 'sha256', 'sha512', 'sha1', 'sha3_512',
                      'blake2b']
#
# ...
#
intermediateHash = self._firstAlgorithm.digest()
for nextAlg in self.MegaHashAlgorithms[1:]:
    algorithm = hashlib.new(nextAlg)
    algorithm.update(intermediateHash)
    try:
        intermediateHash = algorithm.digest()
    except TypeError:
        #TODO: Investigate. Runtime error says:
        #   TypeError: Required argument 'length' (pos 1) not found
        intermediateHash = algorithm.digest(2)
```

Next, I did some reading on collision attacks, which are possible when an attacker
can find two inputs that produce the same hash value. From the clue, we know
that something about this hash algorithm makes it possible for a collision attack
to occur. But what? Then, in my research I read an obvious quote that solved this
entire problem:

```
If the programmer of this code were a little less careful, it might have been
possible to trick their code into hashing two identical inputs (which would, of
course, produce two identical outputs).
```

## Solution

I did a test. First, I typed two different inputs into the boxes. The response
was:

```
Incorrect hash. Calculated hash is 0590b91d577b4670c4f3ebfdb8d09df6e5584b11a58ba6d1771168e81b0cd1d0da260a1a51c6ce29640f75f7dc7773ff05d7f422bf1d474d949e9b0d95949b88
for Input 1 andffe0f4e42bdb64933bcd119f6941d41b17b51eaf3dcd3ff52784065a195c830167ef887adcbcbf41ef6363f768e4161d853accca610f8501b233e3c83b3441ab
for Input 2.
```

Then, I typed the same inputs into the boxes. This produced the same hash and I
got the flag.

___

OpenCTF - August 2019
