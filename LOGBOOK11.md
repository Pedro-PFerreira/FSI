# Work on week #11

## Task 1

* Before start this first task we need to configure the file my_CA_opensl.cnf.

* Our goal on this task is create a Certificate Authority.CA is a trusted authority which creates and sigb digital certificates.

* To do this task we need to use one command demonstrated in next picture with command: openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -keyout ca.key -out ca.crt -subj "/CN=www.modelCA.com/O=Model CA LTD./C=US" -passout pass:dees



![](https://i.imgur.com/ZYpkoNP.png)
![](https://i.imgur.com/CJoU4zd.png)


* What part of the certificate indicates this is a CA’s certificate?

In the file "ca.crt" have a section "basic constraints" there is an attribute "certificate aithority" is set to "Yes". This shows that it is the certificate for the CA.


* What part of the certificate indicates this is a self-signed certificate? 

 A self-signed certificate is on the file "ca.crt" we can see that the subject and the issuer are the same. On top of that in the command we ran was include the flag "-x509" (used to create self-signed certificates).

## Task 2

* On this task,the goal is to create a "Certificate Signing Request",CSR.This will be generate by the CA created in task1

* For acomplish this task we need to use one command demonstrated in next two pictures:
 
* ![](https://i.imgur.com/lBIneqr.png)

* The next step is using another command to see the information of CSR demonstrated in next picture:
![](https://i.imgur.com/FLFX2qn.png)



![](https://i.imgur.com/wsSU8HB.png)

* The next commands will generate a pair of public/private key, and then create a certificate signing request from the public key. 
  
  openssl req -in server.csr -text -noout
  openssl rsa -in server.key -text -noout
  
![](https://i.imgur.com/WbHmLMl.png)


* We finished our task with success.


## Task 3

* In this task we needed to generate a certificate. So, we used the following command to do that:

* This command allow us to convert the "server.csr" to a "server.crt".

openssl ca -config myCA_openssl.cnf -policy policy_anything -md sha256 -days 3650 -in server.csr -out server.crt -batch -cert ca.crt -keyfile ca.key


## Task 4

* For this task 4, we need to deploy a webiste with a certificate.

* First step to accomplish this task is modified ssl.conf to the configuration of the picture: 

![](https://i.imgur.com/bLEahGW.png)


* In order to make the website work, we need to enable Apache’s ssl module and then enable this site. They can be done using the following commands, which are already executed when the container is built.

![](https://i.imgur.com/FUQF3HK.png)


* When we acess the site www.bank32.com we have a warning this happens because while our certificate is being recognized it is from an unknown CA.

 * To solve this warning we need to add the file "ca.crt" to preferences -> Pravate & Security -> View Certificate.

![](https://i.imgur.com/7q09M56.png)



![](https://i.imgur.com/LoCjl0c.png)



## Task 5

* The main goal of this task is to make a "Man-in-the-Middle" Attack. This type of the attack consists in impersonate a website and convince the victim that is using an authentical one. For this we will be trying to try and create a fake "www.example.com".

* First we need to do a setup like we do in www.bank32.com but with one change in ssl.conf.

![](https://i.imgur.com/OcNwwJW.png)


* We need to go file /etc/hosts and change it:
![](https://i.imgur.com/MQtyOVA.png)


## Task 6




# CTF's

## CTF 1 

* In this challenge, we need to attack a service using the RSA function. We know a couple of information in advance:

1) e = 0x10001 = 17;

2) p is the next prime of 2^512 and q is the next prime of 2^513;

3) n = p * q;

* The first step is to find the p and q values. So we simply searched them on the Internet and we obtained the information needed: 

> Value of p:

![](https://i.imgur.com/hK3B3ja.png)

> Value of q:

![](https://i.imgur.com/R1m8D4o.png)

* The next step is to find d value. This number is the one that the equation```d * e % ((p-1)*(q-1)) = 1``` is true.

* This is equivalent to:

```d * e = 1 mod((p-1)* (q-1)) <=> d = (1 mod((p-1)* (q-1))) / e```

* This means that d is inverse modular multiplicative inverse funtion of thos expression.

* We found that we can using the ```pow()```function to obtain this value:

```d = pow(e, -1, (p-1) * (q-1))```


![](https://i.imgur.com/BWIJaTl.png)


* Finally we just need to decrypt the message of the service using but first, we need to obtain it. In order to do that we just used the command ```netcat ctf-fsi.fe.up.pt 6000```, and we obtained it:

![](https://i.imgur.com/KvcJ6q5.png)

* Then, we used the template given to make the attack and put the values in the respective variables and we ran the code.

> Script of the attack:

```python

from binascii import hexlify, unhexlify
p = 13407807929942597099574024998205846127479365820592393377723561443721764030073546976801874298166903427690031858186486050853753882811946569946433649006084171# next prime 2**512
q = 26815615859885194199148049996411692254958731641184786755447122887443528060147093953603748596333806855380063716372972101707507765623893139892867298012168351# next prime 2**513
n = p*q
e = 0x10001 # a constant
d = pow(e, -1, (p-1) * (q-1))

# Print the result
print("d = " + str(d))# a number such that d*e % ((p-1)*(q-1)) = 1


enc_flag = "0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001a63beee9e42f852d291797d2ef52258fc22d1d6e4a1db600adf37ca87a1afa51fc4f13e98e72d5e25f4867dabd168637712de6ca070a2aa5a47a9b024f232e7f65294740c68ee38c691fab136010dde2ec7d06a92c771d9f8c8096d6aa8f4eb513673628eec4acc57263e3a9084c44cf132f28cda6bc18ea75c79a904e859dda"

def enc(x):
	int_x = int.from_bytes(x, "big")
	y = pow(int_x,e,n)
	return hexlify(y.to_bytes(256, 'big'))

def dec(y):
	int_y = int.from_bytes(unhexlify(y), "big")
	x = pow(int_y,d,n)
	return x.to_bytes(256, 'big')

y = dec(enc_flag)
print(y.decode())

```

* Fortunately, we obtained the flag:

![](https://i.imgur.com/zgB2UKb.png)


## CTF 2

* This challenge is quite similar to the last one; however there is a small difference: this time, instead of having one 'e', we have 2 values for 'e', since we are intercepting a communication now. So, we need to decrypt 2 messages to obtain the flag.

* Firstly, we analysed the information we have:

1) 2 e's, instead of 1, which e1 = 0x10001 = 17 and e2 = 0x10003 = 19;

2) n is the same for both messages, which is: 29802384007335836114060790946940172263849688074203847205679161119246740969024691447256543750864846273960708438254311566283952628484424015493681621963467820718118574987867439608763479491101507201581057223558989005313208698460317488564288291082719699829753178633499407801126495589784600255076069467634364857018709745459288982060955372620312140134052685549203294828798700414465539743217609556452039466944664983781342587551185679334642222972770876019643835909446132146455764152958176465019970075319062952372134839912555603753959250424342115581031979523075376134933803222122260987279941806379954414425556495125737356327411;

* After that, we searched for types of RSA attacks that could be successful with the kind of the information we have, and we found one: The "Common Modulus" RSA attacks.

* The main concept "Common Modulus" attacks is decrypting 2 different ciphertextexts of the same message using different exponents, that is, different e's, but the same commun modulus, which corresponds to n. In order to do this attack is using the euclidean algorithm in other to obtain the x and y that ```x*e1 + y*e2 = 1```. This would help us to show the plaintext of the encrypted messages.

* This is the code we used to obtain the plaintext

> Script used to make the attack

```python
import gmpy
   
class RSAModuli:
    def __init__(self):
        
       self.a = 0
       self.b = 0
       self.m = 0
       self.i = 0
    def gcd(self, num1, num2):
       if num1 < num2:
           num1, num2 = num2, num1
       while num2 != 0:
           num1, num2 = num2, num1 % num2
       return num1
    def extended_euclidean(self, e1, e2):
       self.a = gmpy.invert(e1, e2)
       self.b = (float(self.gcd(e1, e2)-(self.a*e1)))/float(e2)
    def modular_inverse(self, c1, c2, N):
       i = gmpy.invert(c2, N)
       mx = pow(c1, self.a, N)
       my = pow(i, int(-self.b), N)
       self.m= mx * my % N
    def print_value(self):
       print("Plain Text: ", self.m)
   
   
def main():
   c = RSAModuli()
   N = 29802384007335836114060790946940172263849688074203847205679161119246740969024691447256543750864846273960708438254311566283952628484424015493681621963467820718118574987867439608763479491101507201581057223558989005313208698460317488564288291082719699829753178633499407801126495589784600255076069467634364857018709745459288982060955372620312140134052685549203294828798700414465539743217609556452039466944664983781342587551185679334642222972770876019643835909446132146455764152958176465019970075319062952372134839912555603753959250424342115581031979523075376134933803222122260987279941806379954414425556495125737356327411
   c1 = 0x41ee28ab466c4fbf96ccb9d809cc41c915be9534fe8df58a535ee6161dc8b90ac5615df89d6eda71d47022e9811d5af45ee5e01036b1938d0ef9bd08644bc376836f6bc7a7998c59706f884ad02083c60545eb615447ca9cb3c5fae21726a697342ccf99d2ee3fc59972810b60b1fee751351213ac5fd57b90ee05229f87af243db55b4e59bd552bf02823782b7aaba94918a819dd29ef002e2640f2e503e8f1c79508598b9bab21eb99ac07522b46aa6a0f41b07ca1b39e6cd60e7f6086258ad17f8e4d4951c3825e4052d478632395ec2103bcca7c26998cff1a591dc159f5d7da8c912d98c8f5f0cb1b5ea4df3e5e7dcea223d965013982874b2f90c1b47f
   c2 = 0x2112ac814e3885a50de43fa0a750d2e98839e175af17c4d8c2295963d5d923e297c3bd331ebe6d6dc3f2441f46968eba2199f7c6a4609eea5134feea2fb968dadc40e0cbd52ae497e15bc36ee88d2c76e9c6c334c66ce95ff816d6ac2cd7f3dac4c49aced0385e6a95833b4ab8d24cdd8f3ec281009542756ab09f0d0a59c698348bc3588357b49884906f19093a98943010143e27b30b77e44d291cc34824a90c903cbfdab97e6a39c1d848fb7cb170d453a4d5c64a637167e7ceb942281352de18c1a8f5d5d8941052deb4f338698e67d71d8db82e57d6ccc4cd4539a67786c13ed939c1a4bbf1d2cfbef21eca736d78961a2984e35c301e4aec03bf79f615
   e1 = 65537
   e2 = 65539
   c.extended_euclidean(e1, e2)
   c.modular_inverse(c1, c2, N)
   c.print_value()

if __name__ == '__main__':
   main()
```

* It should be noted that the value of n corresponds to the value of N obtained with the command ```netcat ctf-fsi.fe.up.pt 6001```

![](https://i.imgur.com/uxX3VCN.png)

* We ran the code and we obtained the plained the encrypted as output:

![](https://i.imgur.com/BV7VcNa.png)

* However, this is not the real content of the message, it is encrypted in decimal. So, we used a tool in web to convert it to plain text. First, we need to convert it to hexadecimal and then to plaintext:

![](https://i.imgur.com/OD4LvEI.png)


* And we obtained the flag:

![](https://i.imgur.com/cFEA4It.png)

