# Blaster Room

Date attempted 5th July 2021

This CTF is marked as an easy room. It doesn't tell use what target we're attempting to compromise.

My approach will be to use NMAP to discover open ports, looking for an attack vector and move forward from there.

### Tools

For this CTF I used the following tools:

Kali Linux
Nmap
Gobuster
Remmina (for RDP access)

## Recon

First I will start with NMAP, with basic scanning the target IP and looking for open ports.

![image](https://user-images.githubusercontent.com/60744763/124467430-31eb1b80-dddb-11eb-95b0-c82bce7492de.png)

Nothing returned, maybe ICMP is blocked. Let's use the -Pn flag, I'll add the -sV flag too to see if we can get some service information right away.

That's better, we can see 2 ports open with some information about the services running.

![image](https://user-images.githubusercontent.com/60744763/124467542-53e49e00-dddb-11eb-97fe-81dcb80ef323.png)

It appears that we have a windows machine, with a web server and RDP access enabled.

Lets browse to that and see what we see...

We have a basic IIS welcome page.

![image](https://user-images.githubusercontent.com/60744763/124467579-5e069c80-dddb-11eb-8f0d-1a37fe3cbff9.png)

I'll use gobuster and see if we can find any directories of interest hosted on the web server.

I'll use one of the wordlists available in Kali with gobuster.

After trying a few word lists, we can see a directory that looks interesting: "retro"

![image](https://user-images.githubusercontent.com/60744763/124467602-68c13180-dddb-11eb-895b-202ab287360f.png)

Let's browse to it and see what we can see!

![image](https://user-images.githubusercontent.com/60744763/124467629-737bc680-dddb-11eb-8f3d-0115e139316f.png)

Found a nice looking website called 'Retro Fantatics', by 'Wade'. Wade could be a username?!

Looking at the bottom of the page, there is a login option... It goes to a wordpress site.

![image](https://user-images.githubusercontent.com/60744763/124467652-7d052e80-dddb-11eb-82b4-994c3a55cd2c.png)

Reading through the posts, Wade potentially gives away his password, stating it's his favourite  character of a movie coming out soon...

![image](https://user-images.githubusercontent.com/60744763/124467680-85f60000-dddb-11eb-9f6d-8292271d8771.png)

Now off to google to look up the movie 'ready player one' and see what Wade's avatar is called.

Google is helpful!

![image](https://user-images.githubusercontent.com/60744763/124467744-9ad29380-dddb-11eb-80a2-70de350a64df.png)

## Compromise the target

Let's try our potential password on the wordpress login page.

Success! The password is all lower case, just the name.

Now we have the username / password combo of wade / parzival

And we're into wordpress.

![image](https://user-images.githubusercontent.com/60744763/124467773-a32ace80-dddb-11eb-8d32-8621e3a2959f.png)

I might as well try this username and password combo on the RDP connection too...

It logs us in!

![image](https://user-images.githubusercontent.com/60744763/124467792-ad4ccd00-dddb-11eb-8bad-cf7c40218013.png)

Our recon was quite successful, we're into the machine.

## Escalation

Looking at the user account wade, it appears he is not a local admin.

![image](https://user-images.githubusercontent.com/60744763/124467824-b63d9e80-dddb-11eb-9fca-5537df68b905.png)

We'll need to find a vulnerability if we want to escalate our privileges.

There is an interesting looking EXE file on the desktop, hhupd.exe, I'll google this to see what it's about.

![image](https://user-images.githubusercontent.com/60744763/124467849-bfc70680-dddb-11eb-9206-abe5d3af8505.png)

It appears to be related to a know vulnerability CVE-2019-1388.

I'll research this issue and see how it is exploited.

This youtube video explains the process well.

https://www.youtube.com/watch?v=3BQKpPNlTSo

Essentially we run the exe file hhupd, which initiates a UAC screen, from here we can view the information about the publisher's certificate.

![image](https://user-images.githubusercontent.com/60744763/124467875-c9e90500-dddb-11eb-9cc1-ace446a5df0f.png)

Now we click the link to the VeriSign issuer, which launches an Internet Explorer window under the System account.

From here we use the File->Open option in Internet Explorer to open the file browser, then execute cmd.exe through the file browser

![image](https://user-images.githubusercontent.com/60744763/124467896-d2414000-dddb-11eb-8026-d106f7abb46b.png)

Now we have a command prompt running as System!

![image](https://user-images.githubusercontent.com/60744763/124467943-dcfbd500-dddb-11eb-9666-a8bba00fbf88.png)

## The flag

We can see the flag in the Administrator's user profile on the desktop!

![56ccd6cf88274b258717cb85acf563cd](https://user-images.githubusercontent.com/60744763/124468022-f1d86880-dddb-11eb-8878-766829bdee20.png)

I'll view its contents be running `type root.txt` from the command prompt. Notepad would have been a fine option too.

