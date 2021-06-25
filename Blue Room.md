
# Blue Room #

This room is about hacking a Windows machine, the room is guided quite well.
It turned out I only needed to use two tools to complete this CTF, then I was able to get admin access and search the machine for the flags.

#### Tools Used ####
- NMap
- Metasploit Framework Console


## Recon ##

First lets start by scanning the machine with Nmap.
I'll use the -sV option right away to see if we can get the details servies running.

![image](https://user-images.githubusercontent.com/60744763/123351971-adf68f80-d5a1-11eb-9fe0-b748301caab5.png)

Here we can see we have a Windows machine, running quite a few services.

Now lets scan for known vulnterabilities with the nmap 'vuln' script.

![image](https://user-images.githubusercontent.com/60744763/123352013-c8306d80-d5a1-11eb-88f3-7e9ee9edc6ec.png)

Here we can see that it has identified a remote code execution vulnerability MS17-010.

Lets see if we can exploit that...

## Compromise the machine ##

We'll load up our Metasploit Framework Console in Kali.
Then search for the vulnerability we identified.

![image](https://user-images.githubusercontent.com/60744763/123352146-0af24580-d5a2-11eb-9cbf-c686a34f9382.png)

Let's try the first exploit in the list.

![image](https://user-images.githubusercontent.com/60744763/123352197-278e7d80-d5a2-11eb-90b4-d2073af7cabd.png)

I've loaded the exploit and configured the target's IP address in metasploit.

### Payload delivery ###

Now we run the attack...

![image](https://user-images.githubusercontent.com/60744763/123352250-3ecd6b00-d5a2-11eb-9b2b-64c3b13c1268.png)


Shortly after we get a shell

![image](https://user-images.githubusercontent.com/60744763/123352270-4e4cb400-d5a2-11eb-9e44-60ea50c4e33b.png)

Let's see who we are running the shell as

![image](https://user-images.githubusercontent.com/60744763/123352292-57d61c00-d5a2-11eb-9e1e-5b741d7aa91a.png)

We're the system account! So we should have full access to the machine now.

## Flag Hunting ##

Now I'll search the machine for the flags.
One was easily found in the root C:\ directory:

![image](https://user-images.githubusercontent.com/60744763/123352369-7b00cb80-d5a2-11eb-9b35-438a208b0eb9.png)

There are more hidden around the system.
The fastest way to find them is with a quick search from the shell.
I'll use `dir flag*.txt /s` from the root of C: to see what we get.

![image](https://user-images.githubusercontent.com/60744763/123352429-979d0380-d5a2-11eb-8aef-253ead294f4f.png)

And presto we have found all the flags.



