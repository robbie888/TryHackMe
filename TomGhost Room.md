TomGhost Room

# TomGhost Room

Identify recent vulnerabilities to try exploit the system or read files that you should not have access to.

This room doesn't have any further information other than the description above. It has two flags user.txt and root.txt

## Tools:

I used the following tools in the CTF.

- Kali Linux
- Nmap
- AjpShooter
- John

## Write Up

Note: I'll set the IP of the server in my host file to tomghost.thm on my attack box.

### Recon:

I'll start with NMAP:

![4477fd913b43d9c0a7403fa20642f042.png](images/ed049d14631342baa5ce4251bd5a7de2.png)

We can see a few services running on the target.

Looks like we have tomcat running on port 8080

![f240a1dd295c82a8ad2d9363ced4b233.png](images/fad911d101754dde84f14e82206341b6.png)

Looking around I tried the /manager path, but it I don't have access, it seems to be blocked to the localhost only.

![b35dca4c65a6c1aa98539c16a145f555.png](images/34027a41f748418a955b9b9165378526.png)

Nmap - vulns script didn't find much interesting

![6a9b3e07ac09b7a2bcac444b40bd08b0.png](images/e4836b53339c4b12ac14880ad1c9548e.png)

Doing some research about tomcat 9.03 I came across this article advising that the port 8009 might be vulnerable with CVE-202-1938

https://blog.qualys.com/product-tech/2020/03/10/detect-apache-tomcat-ajp-file-inclusion-vulnerability-cve-2020-1938-using-qualys-was

I'll see if I can find any exploits. Further research led me to ajpShooter.py which exploits this vulnerability.

Downloading the code from here: https://raw.githubusercontent.com/00theway/Ghostcat-CNVD-2020-10487/master/ajpShooter.py

I've saved it on my attack box and will try it out.

![e5212fb7c02771cb73a7644cd3926ce4.png](images/5c67fda44c614efbacf9ba87b0849974.png)

### Exploitation:

Following the instructions on the github page I run the following:

![e150eecfd1c4bc40c65b53d673717fc2.png](images/96a67031524f4c5dac4ae009586fea88.png)

This gives me some nice results:

![0df1a22d66c66af31acb76894bd59b7d.png](images/ae3d977bb1ec4fc38a54468de6ab31e7.png)

That looks like a possible username / password combo.

skyfuck:8730281lkjlkjdqlksalks

The only place to login on this machine is SSH, as the tomcat login portal is blocked to localhost.

So I'll try that first.

![a900ace74a964b0fc2db2e091d12c6ef.png](images/f52aeaa418c74a51a8736fc313918574.png)

And we're in with a user!

Having a quick look, it seems we have an encrypted file here with pgp, and the key right next to it.

![21d4f869867c36b3ecdffaf455561939.png](images/63d8bd01fde542ba90c03bcb02ae5eab.png)

I imported the key and tried to decrypt it but the password I have doesn't work.

![3baf83ec29cbd18a2e2eda1218ea94ac.png](images/aaba3775f63f4f3587851cf9c14dbfef.png)

So I'll try and brute force it with out rockyou.txt password list.

Following the instructions on the site below, I'll use john to attempt the brute force attack on the key.

https://www.ubuntuvibes.com/2012/10/recover-your-gpg-passphrase-using-john.html

This was successful as we can see

![f4cd0c5c062b4baabf00170d0f2cb81b.png](images/18b040feb66d452d8fd5ce21a3b61eab.png)

The password is: alexandru

I'll try and decrypt the file now on the target machine.

And here we have another account:

![b97164affffa8fccbb6aec64e8a5655a.png](images/424a4bc1b2c4409ab97be37fb81c3b2a.png)

merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j

Before trying those credentials I had a quick hunt around the server and found the first flag in merlin's home folder.

![eb8a6f963c209e502d89ee13f180bbe1.png](images/d475580495a6418fa6a5fb3406b65951.png)

It also looks lie merlin might be a sudo user. Which could be interesting. I'll try login to another ssh session with those credentials to see what I can do from there.

![501710283d15f5484a1bcb7e857531b8.png](images/8a3dd5468a6a46969f6029c45919212d.png)

Now I'm in as merlin.

### Privilege Escalation

Looks like merlin can run zip as root.

![84e45e1b2e11799cdf9e1a3e77b1a3ff.png](images/d0214557e8d54bab9c746b919db7c6df.png)

Referring to this site: https://gtfobins.github.io/gtfobins/zip/

I may have found some priv escalation options.

The code snippet is:

```
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
sudo rm $TF
```

And here we have it:

![069712c1cc06c3f6b9850248dfbad946.png](images/ff174bed82ec45b9b40adfacfcfe8ab2.png)

Now for the root flag

![ab5aa9e34c7bd5abfc61c0a6090a8c14.png](images/fa0c206c0f304bcbbb20ed75323d3d81.png)