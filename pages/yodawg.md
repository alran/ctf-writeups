## Challenge

For this challenge, we were given a simple text file with the following content:

```
NTc1NzM4Njc1bjQ3NDYzMzVuNzk3NzY3NTM1MzQyNnM1bjU3NDY3OTVuNDM0MjM1NjI3OTQyNzM2MTU3N
zQ2cDQ5NDc1Njc1NTkzMjM5Nm82MTU3MzU2cjQ5NDg0cjc2NDk0ODY0NnA0OTQ4NDIzMTY0NDM0MjY4Nj
I2OTQyNnA2MjZxNHI3NjVuNDc1NjZvNDk0NzMxNnA2MzMzNHI2ODVuMzI1NTY3NjE1NzM0Njc2NTU3Mzg
2NzVuNTczNTZuNjIzMjUyNnA1bjQzNDI3NDVuNTg0cjduNTk1NzY0NnA0OTQ4NHI3NjQ5NDg2cDc2NDk0
NzRyNjg2MjY5NDI2bzVuNTMzMTZwNjI2cTRyNzY1bjQ3NTU2NzY0MzI2ODcwNjI0NzU1Njc2NTU3Mzg2N
zVuNDc1NTc0NW41NzM1Nm42MjMyNTI2cDRwNjczcTNx
```

## Discovery

The first thing we did was run the text through a base64 decoder.

```
echo text | base64 -D
```

This returned a string of numbers and letters that on first glance seemed random.

```
575738675n4746335n7977675353426s5n5746795n434235627942736157746p494756755932396o6
157356r49484r764948646p49484231644342686269426p626q4r765n47566o4947316p63334r685n
32556761573467655738675n57356n6232526p5n4342745n584r7n5957646p49484r7649486p76494
74r686269426o5n53316p626q4r765n4755676432687062475567655738675n4755745n57356n6232
526p4p673q3q
```

We broke this out into two character chunks after noticing some common combinations.
For example, `5n` appears multiple times, as does `3q`.

```
57 57 38 67 5n 47 46 33 5n 79 77 67 53 53 42 6s 5n 57 46 79 5n 43 42 35 62 79
42 73 61 57 74 6p 49 47 56 75 59 32 39 6o 61 57 35 6r 49 48 4r 76 49 48 64 6p
49 48 42 31 64 43 42 68 62 69 42 6p 62 6q 4r 76 5n 47 56 6o 49 47 31 6p 63 33
4r 68 5n 32 55 67 61 57 34 67 65 57 38 67 5n 57 35 6n 62 32 52 6p 5n 43 42 74
5n 58 4r 7n 59 57 64 6p 49 48 4r 76 49 48 6p 76 49 47 4r 68 62 69 42 6o 5n 53
31 6p 62 6q 4r 76 5n 47 55 67 64 32 68 70 62 47 55 67 65 57 38 67 5n 47 55 74
5n 57 35 6n 62 32 52 6p 4p 67 3q 3q
```

In doing this, we noticed that the only letters represented are N through S. The
only numbers represented are 31 through 79. Could this be an attempt at hex? 

## Solution

In hex, there are only 6 characters, A - F. In our text, there are also only
6 characters, N - F. We shifted all letters up so that the N's were A's and the
S's were F's.

Voila! This presented up with a valid base64 encoded string.

We ran this through a base64 decoder and got the flag.
