# Work on week #7


## Task 1

* Our goal in this task is to provide an input to the server and when the server program tries to print out the user input in the myprintf() function,it will crash.If myprintf() returns,it will print out "Returned properly" as you can see in our image below. 

* In this task will use a command **$ echo hello | nc 10.9.0.5 9090**  to send a string "Hello" to a function myprintf() on the server 10.9.0.5.

![](https://i.imgur.com/aojyVHP.png)
 <p> 
    Fig.1- Output of the function myprintf() and more ...
</p>

## Task 2

### Task 2A

* In order to complete this task, we need to understand the printf works, in order to expose sensitive data from stack. Firstly, we can put some 4 B in the code and see the next variable's adress. We can use the 4 B bytes given. However, we have to change the format string in order to print the variable successfully.

* Since we want to print 4 B and each has 8 bits, we need to print 64 (4*8) registers, the 63 before and actual register. So we alter the string ```s``` to this format.

![](https://i.imgur.com/S9KoCkA.png)

* Then, we need to compile this program, using these commands:

![](https://i.imgur.com/CapMC8z.png)

* And we have the expetec result. Since we can see our bytes selected before the variable's address: 

![](https://i.imgur.com/k21HiCO.png)


### Task 2B

* Now we know what is the next address. So we can use it to know what is the content of the heap.

* Firstly, we set the value of the variable ```number``` to the secret address, which is given by the server (0x080b4008)

* The difference between this task and the previous one is that now we want the access the content of the address, not the address itself and we know this content is in the string format, So instead of read 64 registers, we read 63 and the last on we use the '%s' format to read the content of the register. So, this is build_string.py now:

![](https://i.imgur.com/Zjx3Oc1.png)

* We execute the same commands and we obtained the secret message:

![](https://i.imgur.com/bW0vxXF.png)


## Task 3


### Task 3A


 In this task we need to change ```number``` variable.
 
 * Like we have done in task 2B, we have to access the address by making it the beginning of our string into it. The address is 0x080e5068.
 
 * We can access it by using "%x" 64 times and changing the values for 63 times plus "%n".The reason for this is the same we used to crash the server and we can write to the address the pointer is pointing to.In this case pointing to where the secret variable was located.
![](https://i.imgur.com/maqsOPh.png)



![](https://i.imgur.com/93MIqlT.png)


![](https://i.imgur.com/YgS2aLu.png)


### Task 3B

The difference between task 3B and 3A is in this one we have to change the secret variable like task 3A but also change it to a specific value: 0x5000

* We know "%n" print the number of characters printed in the string so far, and it will be able to write 0x5000. We must make it so that our string has 5000 characters before it. 

 
![](https://i.imgur.com/BbsP66U.png)

* From this task we understand how printf() is used to acess or change memory from an attacked device, how it can be exploited and what cautions we need to have when using it.

* We have also learned how an attacker can use the side effects of "%x", "%s", "%n" in a formatted string to his/her advantage performing an attack.


![](https://i.imgur.com/MCGAj9i.png)

## CTF 
 
### Challenge 1

* In this task, we has to explore a vulnerability of format strings in order to create an exploit and obtain the challenge's flag. Firstly, we have to check what protections are activated. We used the command ```checksec program``` in order to obtain relevant information about that.

![](https://i.imgur.com/fpPeFhn.png)

* Comparing to the last week's tasks, we have now canaries and the and no PIE. The fact of not having PIE activated means that there is no randomness on the adresses on the executable.

* Secondly, we had to answer this questions:

1) In which code line is the vulnerability found?

2) What can we do by taking advantage of the vulnerability?

3) What is the funtionality that allow us to obatain the flag?

* Answers:

1) The vulnerability is at line 25 of "main.c", since has a ```scanf()``` that scans the bytes of the variable ```buffer```; however, the load_flag() function loads mote bytes than ```buffer``` contains. In addition of that, the "%32s" will only scan data if it fits the buffer.

![](https://i.imgur.com/PKmCR3I.png)

2) We this vulnerability, we can smash the execution stack and access values that we are not supposed to.

3) If we use a format string and using the register where ```load_flag()``` is addressed, then, we might obtain the flag value.

* After answering the questions, we started our process to otbtain the flag value:

* First, we made some debugging to the program in order to obain the ```load_flag()``` register, thanks to the command ```gdb program``` in a terminal. Fortunately, we obtained the address of the register:

![](https://i.imgur.com/p4y3L3v.png)

* Then, we need to put the value of this register in the our "exploit_example.py". Since it is an integer, the execution stack reads it differently when compared to an array, as we concluded in the task of the previous week. So, we inverted the order of the bytes of the register 0x8049256 and inserted the register "\x60\xC0\x04\x08". Besides that, we needed to concatenate it to "-%s", so that we could we read the content of the register itself. We executed the exploit file in local mode and we could access the content of "flag.txt".

![](https://i.imgur.com/HjzA73g.png)

![](https://i.imgur.com/EPsVyCY.png)

* This means we exploited the program successfully. So, we tried in the remote mode, by asserting the ```LOCAL``` variable as False and we obtained the flag value sucessfully.

![](https://i.imgur.com/214uA47.png)

![](https://i.imgur.com/7ikThKb.png)


### Challenge 2

* This second challenge is the strongest version of our challenge 1. When we use checksec we saw the same restrictions from the first challenge.

* Before making the exploit, we need to answer some questions first:

1) In which code line is the vulnerability? What does the vulnerability allow us to do?

2) Is the flag loaded to memory? Is there any functionality we can use to access the flag?

3) To we need to do to unlock the functionality?

* These are our answers:

1) The bug is in line 12 of the "main.c". It is similar to the previous task- scanf only scans what fits in the buffer and is the same size of the key. This vulnerability can be explored to access the content of some register that contains sensitive data if the attack is made successfully.

2) Yes, the key is loaded to the memory and we can access to its value by using the command ```p &key``` 

3) In order to unlock the functionality, we need to use the debugger mode in order to do that, we use the command ```gdb program```.

* So when we saw the code provided we notice the code doesn´t have a function that already opens our code, but if the key equals to 0*BEEF, we can run BASH and we can open it.

![](https://i.imgur.com/dc5nAlU.png)

* Now we run program with gdb and we find where the key is the located:

Calculating A:

* We know how to write our string: at first, we have to calculate A. To do this we know 0*BEEF=48879. We also knew that we have written 8 chars so far for our address, that left us with

  48879-8=48871. This is the number we got to substitute 'A' with this value.
 
* When we run the exploit_example.py we got the flag in the flag.txt:

![](https://i.imgur.com/X3WYBwl.png)


