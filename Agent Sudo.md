# Agent Sudo

Room Description: You found a secret server located under the deep sea. Your task is to hack inside the server and reveal the truth.

We can see a few ports open

![95de5057c607dc795965ab1df86ea62f.png](images/95de5057c607dc795965ab1df86ea62f.png)

I'll browse to the http port to see what the site is about.

We get a hint here about changing the codename as user-agent. And agent R sent this.

![6c5f6637e62abed9fd9ba28f6086504e.png](images/6c5f6637e62abed9fd9ba28f6086504e.png)

I'll send this over to ZAP and see what happens when we start changing the user-agent.

First I'll try agent R as per the web-page.

![fa6d62355b47bb253250fee76caeada6.png](images/fa6d62355b47bb253250fee76caeada6.png)

Our response is different now, so we're on the right track!

![a63f56380a9cc1a528e052e15149ff3f.png](images/a63f56380a9cc1a528e052e15149ff3f.png)

Considering it is 1 letter, I can try fuzz this with the alphabet.

I've setup some payloads from A-Z in caps.

![4b1962a4e7f18ade34eb124aeedf96ce.png](images/4b1962a4e7f18ade34eb124aeedf96ce.png)

![960eee30701680fa29bf8968c2fb5e1a.png](images/960eee30701680fa29bf8968c2fb5e1a.png)

We got a different looking response for C as the agent user.

![1af9a5caa34a3191c8cd1379eccc3465.png](images/1af9a5caa34a3191c8cd1379eccc3465.png)

We have a redirect here:

![8129b75baa63ee8e698e63b7fad17095.png](images/8129b75baa63ee8e698e63b7fad17095.png)

Browsing to this gives us some more information!

![efd74dd6b5f48ef09455f06b6f7d4533.png](images/efd74dd6b5f48ef09455f06b6f7d4533.png)

Looks like chris has a weak password.

Attempting to browse to agent J as above didn't provide any further details. So I'll try an brute force now against FTP as the room suggests. I'll assume the username is chris.

Using hydra we find a password! crystal.

![88f26e899fbe36d8aaf6e1081567e710.png](images/88f26e899fbe36d8aaf6e1081567e710.png)

Now to login and have a look.

We can see some files in the ftp login

![c62b3919f8d18b616b8372526b8bcf49.png](images/c62b3919f8d18b616b8372526b8bcf49.png)

I also tried chris' credentials for ssh - but no avail.

![8985d61769c5989eeb47a75157201c58.png](images/8985d61769c5989eeb47a75157201c58.png)

I downloaded all the files with the FTP command `mget *`

Looking at the 'To_agentJ.txt' file we have some information that mentions J's password is in a picture.

![52c3e9645f04a760ac7555b0fc1b485c.png](images/52c3e9645f04a760ac7555b0fc1b485c.png)

Now lets check out the images.

So we have two cute looking aliens

![e199f2f62475e82a72d8bbd31236f3b2.png](images/e199f2f62475e82a72d8bbd31236f3b2.png)

Since we know that something is stored in one of the images we can look into that with the tool 'steghide', which I installed using ATP.

This site is useful https://0xrick.github.io/lists/stego/ 

![1c96549e1390c490c6d108939676849b.png](images/1c96549e1390c490c6d108939676849b.png)

So we certainly have something here in the first image, we just need a passphrase, I'll check the next image. The second image is a PNG which is not supported. Looking at the site above, there is a tool for PNG called foremost.

Looks like we might have something here too...

![d2e6736da8c1fc9b274e0ee1a564ece1.png](images/d2e6736da8c1fc9b274e0ee1a564ece1.png)

So we've extracted a zip file from the png!

![72104c706acecaca4365c7f4bba3eaa9.png](images/72104c706acecaca4365c7f4bba3eaa9.png)

Looks like the zip file is password protected, I can try and brute force this.

![a6876f5f9e853bbb1decdf5f8099177f.png](images/a6876f5f9e853bbb1decdf5f8099177f.png)

Using John we run a brute force with our password list.

![421aa9598dc1b00de23f73bdd325641c.png](images/421aa9598dc1b00de23f73bdd325641c.png)

Looks like we have a hit!

Extracting the zip gives us another file / clue

![ed061e8e70838c5a784ed8df86479339.png](images/ed061e8e70838c5a784ed8df86479339.png)

The string above in the message doesn't decrypt the other image, however after some playing with the Burp decode, it turns out it's Base64 encoded.

![d9b3783e1d230d64b9a731a6d1df2c95.png](images/d9b3783e1d230d64b9a731a6d1df2c95.png)

Here we have a potential password: Area51

Bingo! That decrypted the JPG steg image.

![444f4eb53b1b385429414e56c17e278a.png](images/444f4eb53b1b385429414e56c17e278a.png)

So now to extract and view the file.

![1ac729b8889978c4aec30289092e11c9.png](images/1ac729b8889978c4aec30289092e11c9.png)

Looks like we have a password for james!

I'll try this on ssh.

So we're in as james now!

![96249f8c4a97acc725f46b351e9fd07f.png](images/96249f8c4a97acc725f46b351e9fd07f.png)

We can see the user flags here:

![ce72a0f1e7b84093a161927655d9dc88.png](images/ce72a0f1e7b84093a161927655d9dc88.png)

The image in the folder is of the roswell alien autopsy ;)

Next I'll see what james can do as sudo (if anything)

![9b0557702ba0f21486686cb608588d4a.png](images/9b0557702ba0f21486686cb608588d4a.png)

Looks like we can only run /bin/bash as sudo, but not as root. 

Let's check the sudo version.

![4df4e5b6d6ff7ed71b9183215aec67d0.png](images/4df4e5b6d6ff7ed71b9183215aec67d0.png)

Looks like we have a potential hit here! 

![799d0f754736e6d6213082681d9b268e.png](images/799d0f754736e6d6213082681d9b268e.png)

Reviewing the exploit here: https://www.exploit-db.com/exploits/47502 we have some things to try.

We can enter this command `sudo -u#-1 /bin/bash`

Which exploits this vulnerability.

![3753e13f349fa9564a69d1087effc222.png](images/3753e13f349fa9564a69d1087effc222.png)

Now we are root!

And for the flag:

![d4dbecd7a7b10e2d01ed3d0a56a245e0.png](images/d4dbecd7a7b10e2d01ed3d0a56a245e0.png)

We're done!