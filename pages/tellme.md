## Challenge

We received a clue and a binary file. The clue said:

> OK, do you think you know my secret? See if you can guess what I'm expecting to hear.

## Discovery

We started by opening the binary in Ghidra, which can be installed and run
with the following commands:

```bash
$ brew cask install java
$ brew cask install ghidra
$ ghidraRun
```

This opens up the ghidra GUI. We imported the binary file and had Ghidra run
analysis on it. Then, we scrolled through the functions list, trying to find
something that looked interesting. Ghidra compiles the code into C, which is extremely,
infinitely more useful than looking at machine code.

There was one function that had print statements like "Do you know" and "Tell me".

```c
undefined8 FUN_0010141a(void)

{
  int iVar1;
  size_t local_20;
  char *local_18;
  char *local_10;

  local_18 = (char *)0x0;
  local_20 = 0;
  puts("Do you know?");
  printf("Tell me: ");
  getline(&local_18,&local_20,stdin);
  strtok(local_18,"\n");
  local_10 = (char *)FUN_00101215(s_xzyu{hfkxcbhmloaoftay}_001040b0,s_SOYOULYKECATS_001040a0);
  iVar1 = strcmp(local_18,local_10);
  if (iVar1 == 0) {
    puts("You got it!");
  }
  else {
    puts("You don\'t know, huh?");
  }
  free(local_10);
  return 0;
}
```

This gnarly, but way, way easier to read than the ASM:
```asm
00101422 48 c7 45        MOV        qword ptr [RBP + local_18],0x0
         f0 00 00
         00 00
0010142a 48 c7 45        MOV        qword ptr [RBP + local_20],0x0
         e8 00 00
         00 00
00101439 e8 22 fc        CALL       puts                                             int puts(char * __s)
         ff ff
0010144a e8 41 fc        CALL       printf                                           int printf(char * __format, ...)
         ff ff
00101456 48 8d 4d e8     LEA        RCX=>local_20,[RBP + -0x18]
0010145a 48 8d 45 f0     LEA        RAX=>local_18,[RBP + -0x10]
00101464 e8 87 fc        CALL       getline                                          __ssize_t getline(char * * __lin
         ff ff
00101477 e8 64 fc        CALL       strtok                                           char * strtok(char * __s, char *
         ff ff
0010148a e8 86 fd        CALL       FUN_00101215                                     undefined FUN_00101215()
         ff ff
001014a1 e8 fa fb        CALL       strcmp                                           int strcmp(char * __s1, char * _
         ff ff
001014a6 85 c0           TEST       EAX,EAX
001014a8 75 0e           JNZ        LAB_001014b8
001014b1 e8 aa fb        CALL       puts                                             int puts(char * __s)
         ff ff
001014bf e8 9c fb        CALL       puts                                             int puts(char * __s)
         ff ff
001014cb e8 70 fb        CALL       free                                             void free(void * __ptr)
         ff ff
001014d0 b8 00 00        MOV        EAX,0x0
         00 00
001014d6 c3              RET
```

The important line here is this one:

```c
local_10 = (char *)FUN_00101215(s_xzyu{hfkxcbhmloaoftay}_001040b0,s_SOYOULYKECATS_001040a0);
```

This code calls another function with two hard coded string values and compares
the user input with their result.

The other function looks like this:
```c
void * FUN_00101215(char *SOYOULYKECATS,char *CHANGEDFLAG)

{
  char cVar1;
  int iVar2;
  size_t __size;
  void *pvVar3;
  ushort **ppuVar4;
  uint i;

  __size = strlen(SOYOULYKECATS);
  pvVar3 = malloc(__size);
  i = 0;
  while( true ) {
    __size = strlen(SOYOULYKECATS);
    if (__size <= (ulong)i) break;
    __size = strlen(CHANGEDFLAG);
    iVar2 = toupper((int)CHANGEDFLAG[(ulong)i % __size]);
    ppuVar4 = __ctype_b_loc();
    if (((*ppuVar4)[(long)SOYOULYKECATS[(ulong)i]] & 0x100) == 0) {
      ppuVar4 = __ctype_b_loc();
      if (((*ppuVar4)[(long)SOYOULYKECATS[(ulong)i]] & 0x200) == 0) {
        cVar1 = SOYOULYKECATS[(ulong)i];
      }
      else {
        iVar2 = ((int)SOYOULYKECATS[(ulong)i] + -0x47) - (iVar2 + -0x41);
        cVar1 = (char)iVar2 + (char)(iVar2 / 0x1a) * -0x1a + 'a';
      }
    }
    else {
      iVar2 = ((int)SOYOULYKECATS[(ulong)i] + -0x27) - (iVar2 + -0x41);
      cVar1 = (char)iVar2 + (char)(iVar2 / 0x1a) * -0x1a + 'A';
    }
    *(char *)((long)pvVar3 + (ulong)i) = cVar1;
    i = i + 1;
  }
  return pvVar3;
}
```

Clearly, there is some sort of looping and transformation happening in this method.


## Solution
At the end of the day, this method is hardcoded. Yes, there is some hard-to-read
transformation happening to the starting value, but the output can be reproduced.
We rewrote the function so that would compile in C locally, and got the flag.

```c

#include <stdio.h>
#include <stddef.h>
#include <stdint.h>
#include <limits.h>
#include <sys/types.h>
#include <string.h>
#include <stdlib.h>
#include <stdbool.h>
#include <ctype.h>

void * FUN_00101215(char *pcParm1,char *pcParm2)

{
  char cVar1;
  int iVar2;
  size_t __size;
  void *pvVar3;
  ushort **ppuVar4;
  uint local_1c;

  __size = strlen(pcParm1);
  pvVar3 = malloc(__size);
  local_1c = 0;
  while( true ) {
    __size = strlen(pcParm1);
    if (__size <= (ulong)local_1c) break;
    __size = strlen(pcParm2);
    iVar2 = toupper((int)pcParm2[(ulong)local_1c % __size]);
    ppuVar4 = (short unsigned int **)__ctype_b_loc();
    if (((*ppuVar4)[(long)pcParm1[(ulong)local_1c]] & 0x100) == 0) {
      ppuVar4 = (short unsigned int **)__ctype_b_loc();
      if (((*ppuVar4)[(long)pcParm1[(ulong)local_1c]] & 0x200) == 0) {
        cVar1 = pcParm1[(ulong)local_1c];
      }
      else {
        iVar2 = ((int)pcParm1[(ulong)local_1c] + -0x47) - (iVar2 + -0x41);
        cVar1 = (char)iVar2 + (char)(iVar2 / 0x1a) * -0x1a + 'a';
      }
    }
    else {
      iVar2 = ((int)pcParm1[(ulong)local_1c] + -0x27) - (iVar2 + -0x41);
      cVar1 = (char)iVar2 + (char)(iVar2 / 0x1a) * -0x1a + 'A';
    }
    *(char *)((long)pvVar3 + (ulong)local_1c) = cVar1;
    local_1c = local_1c + 1;
  }
  return pvVar3;
}

int main() {
  char *local_10;
  local_10 = (char *)FUN_00101215("xzyu{hfkxcbhmloaoftay}","SOYOULYKECATS");
  puts(local_10);
  return 0;
}
```

___

OpenCTF - August 2019
