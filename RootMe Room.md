#RootMe Room

Another TryHackMe box, we need to try and get root on this device. It appears to be a Linux machine based on the description.

## Tools

- Kali Linux
- Nmap
- GoBuster
- OWASP ZAP
- PHP Reverse Shell
- Python

## Recon

I'll start with an NMAP scan using the default script and -sV to get any version information on the services running.

![fc87141f83ef498c4d454b93c5400707.png](images/485cd95ce77f45c3943ec593cf141769.png)

We have two ports open, ssh and http.

There is a website setup:

![1ff27dcbe18916986e2c16144f7bc95b.png](images/5ac59d6bedf54be48b1ee42daa34712b.png)

The source of this page doesn't provide much.

I'll try gobuster to see if there are any interesting folders we can find.

![6692040f1648c3de0fc4f380a6b7a6c6.png](images/7e3be08a693c4056899c297ff9f4f726.png)

It found some interesting folders. Uploads and panel.

## Exploitation

Panel brings me to an upload portal, which is a good place to try my reverse PHP shell.

![07077af542dfc55abdd317c1a1036690.png](images/08ca06f9260947d28ad551f6bfbeea5c.png)

PHP uploads are blocked, so I will try to determine what files are allowed.

![165a9fa60931522493c09f3516431e15.png](images/a2d1966572814f2da3288a6018792d1a.png)

![7e4e78ffacea7f18c4b5d9a62d725d4f.png](images/9a3790c36e3e410283962fd332c278af.png)

Using OWASP ZAP Fuzzer tool I'll try several alternative PHP related extensions.

![12dc402e6727bbbbedd09f6532b878ee.png](images/4c2acbd16cc543d487ac421b69a7befc.png)

![c38d3377e12b5b56555a2443b425a81c.png](images/5ff6280ee13449b9afef9d0abd1c88ba.png)

That seemed quite successful,quite a few extensions were allowed except php.

![4865aff3ddb105ab7299b2708d0460ad.png](images/e89ce97f0ed5496eaec621e95a9d294f.png)

Here is the response of the phtml, I'm guessing successo means success! ;)

![eb50c70a5155b02c71ab9ab72782b2b0.png](images/6ee7397dc195457180b086fbe79a16bb.png)

I can see it has been uploaded to the /uploads folder.

Now Ill try and run the script while having an NC listener active.

![05bbc471cb0bad2dc94510e4c9e17571.png](images/98f3593a3beb47fd8814c0843e15f910.png)

Successo! We have a reverse shell:

![10ded4375a8ebdd3a8e5f809f0ae0cbf.png](images/82d0db223aa54d7983a68a82e5255d97.png)

We can see my OWASP ZAP attack uploaded several reverse shells here:

![f58a4901b62251a4f235f08a9fec4b13.png](images/2eed9bf532b34800a38578bbf207c562.png)

First I'll upgrade from the simple shell with `python -c 'import pty; pty.spawn("/bin/bash")'`

![6ccf453fcb35b89af751f99febdaccaa.png](images/490f4ab289164a2c833a77ce45ed664f.png)

Now let's look for a user flag.

Here it is:

![e0f58ffc02ec92b793a89d7b97fe7b7f.png](images/fc831971099541fc9c23b7533722fa03.png)

## Privilege Escalation

Now for privilege escalation, I'll start by searching for SUID files.

Python looks interesting

![aad747c0b6d1058f909a5591d05875b8.png](images/1807b1deea1540fc9bea01a5f6de6079.png)

After some research the site below advises a way to priv escalate with python

https://gtfobins.github.io/gtfobins/python/

This should do it `python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`

And now we have it! My effective ID is root!

![29fb93a88e7d203b9e6b0f21eb249edf.png](images/10e3091487bc4c94a7cfff68114f544a.png)

Now for the flag

![48d6776f11bc67d29e3602f5b0f5b8a1.png](images/97ad8bac87dd445294eaf4d48205defa.png)