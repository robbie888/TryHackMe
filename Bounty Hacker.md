# Bounty Hacker

Description: You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!

Let's begin with a standard nmap scan.

![9e90c8eaadc5e5eec9c0c61fa2300806.png](images/9e90c8eaadc5e5eec9c0c61fa2300806.png)

Getting a bit more detail:

![ecf5f8086baf867995442d8aae83de56.png](images/ecf5f8086baf867995442d8aae83de56.png)

Looks like we have a few ports we can look into. I'll see about the web-site and check if FTP allows for anonymous login.

FTP first:

![e72cc6b88dc090becd9e1d0a6c20626a.png](images/e72cc6b88dc090becd9e1d0a6c20626a.png)

We do have anonymous access, and a few files to check. I'll download them and see what we see.

![d48ffa2766914f6d01fb4cee3f075b06.png](images/d48ffa2766914f6d01fb4cee3f075b06.png)

This information looks very interesting, I'm going to assume this could be a password list. And in the tasks file we have a name 'lin', could this be a username too?

Let's try hydra on ftp to start with.

![341440b922f8f6952d57e8297f2dd705.png](images/341440b922f8f6952d57e8297f2dd705.png)

That didn't yield any results, how about ssh?

![737bff9da6fecf4b1d10af845e76dade.png](images/737bff9da6fecf4b1d10af845e76dade.png)

That worked out well

We have lin / RedDr4gonSynd1cat3

![7060457fef98da24cc3c34aef50a1676.png](images/7060457fef98da24cc3c34aef50a1676.png)

We're in!

Our first flag

![3c9436bd522ee70d142d4391028401c9.png](images/3c9436bd522ee70d142d4391028401c9.png)

Now for the root flag, I'll check out sudo first.

![6b60571ab5887296315c6c28d1bf81fa.png](images/6b60571ab5887296315c6c28d1bf81fa.png)

Looks like we can use tar as sudo, let's check https://gtfobins.github.io/

This looks good from gtfobins

![73bdd8300db704992bd47b41d1cb89b1.png](images/73bdd8300db704992bd47b41d1cb89b1.png)

Success!

![350a16c84655ade92e9f19be33576345.png](images/350a16c84655ade92e9f19be33576345.png)

And now for the flag

![9218f4ad399836df757b2297e6ba621a.png](images/9218f4ad399836df757b2297e6ba621a.png)

That's it for this room, turns out I didn't need to check the web site, but I did anyway and it;'s a Cowboy Beepop scene :)