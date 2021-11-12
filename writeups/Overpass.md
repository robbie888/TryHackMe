# Overpass

Description: What happens when some broke CompSci students make a password manager?

A quick scan shows us two open ports. 22 and 80.

![43b2df24a5bb33eca7bf0b4ee640d0b7.png](images/43b2df24a5bb33eca7bf0b4ee640d0b7.png)

There is a website accessible on the machine.

![a841d5d6d6a5a8777e9a74a8f70c960c.png](images/a841d5d6d6a5a8777e9a74a8f70c960c.png)

While I click around the site, I'll kick of a gobuster scan for hidden folders.

We're getting some hits right away, downloads, admin etc.

![f2f4aa7fa8ccf221475b8ce54ce220c6.png](images/f2f4aa7fa8ccf221475b8ce54ce220c6.png)

There is an admin page here with a login. It also looks like the username might be 'administrator'.

![d772c37965d9ce5b701b3573a91a7780.png](images/d772c37965d9ce5b701b3573a91a7780.png)

I'll fire up burp and see what the login request looks like.

![ba15a5225ba4fad68e53ae9cf207bc3c.png](images/ba15a5225ba4fad68e53ae9cf207bc3c.png)

It looks like there is an API folder as well. I'll try some SQL injections to see what we get. The normal payloads of `'--` and `'-- or 1=1` didn't yield any results or errors. I could try fuzzing some SQL injection payloads, but will look around the site a bit more first.

Looking in the source code of the login page we see some js files that look interesting.

![7470d8fec48e3b807326039a54aa3a40.png](images/7470d8fec48e3b807326039a54aa3a40.png)

Looking at the js file it looks like it is looking for a specific string response from the server to deny a login.

![db7537a739a8ec10341609e8c3eb4f86.png](images/db7537a739a8ec10341609e8c3eb4f86.png)

So if we can change this response from the server to the browser, we might be able to login.

![7a7be5afca8de05a4f81a24b911927ea.png](images/7a7be5afca8de05a4f81a24b911927ea.png)

I changed the text to 'ok', and forward it to the client and we got further access!

![4422728e0c2d988909eb8abf06ca2ae7.png](images/4422728e0c2d988909eb8abf06ca2ae7.png)

Looks like this might be James's ssh login key, but it is password protected. I'll try to crack it with John.

First I've downloaded the key and tested ssh with it:

![a0c336ffd09e2fb3ed7c0873cbf3cd83.png](images/a0c336ffd09e2fb3ed7c0873cbf3cd83.png)

We can see it's asking for a password. Now over to John.

First I'll covert it to a suitable format for john.

![3d905377101988d94aaf1bad3efbb97f.png](images/3d905377101988d94aaf1bad3efbb97f.png)

Now run the crack with rockyou.txt

![2af5037d8f89321b167d3c8a7a0517e2.png](images/2af5037d8f89321b167d3c8a7a0517e2.png)

Looks like we have a hit 'james13'.

Now we're in ssh as james!

![b346fcb6d36255bc4091ebd9ecbb57fb.png](images/b346fcb6d36255bc4091ebd9ecbb57fb.png)

So we have out first flag, and a possible hint?

![8e70190eaca996ce128a49a257d25b82.png](images/8e70190eaca996ce128a49a257d25b82.png)

Looks like they're using their application for the password manager, I can see an overpass file in the user's files.

![c4c40f3ddd0840d196cd6a04773df59b.png](images/c4c40f3ddd0840d196cd6a04773df59b.png)

I'll have a look at the source code for their app, which is available on their website!

![618b7d790493c1537690b9afaf63ad60.png](images/618b7d790493c1537690b9afaf63ad60.png)

Looking at this function, it may be encrypting with a rot47

In the user's overpass file we can see the encrypted code.

![2dd1cd8835d31de5374ecdafc213ef3e.png](images/2dd1cd8835d31de5374ecdafc213ef3e.png)

Now to try and decrypt it, this site is helpful https://rot47.net/

![a754bb858e3349e8937c8647da5d22df.png](images/a754bb858e3349e8937c8647da5d22df.png)

Here we have some new things to try!

The password above worked for james's account! So I tried it with sudo, but James doesn't have access to sudo.

![5d3c4034ec3a5f2c7e57eeffcadc9020.png](images/5d3c4034ec3a5f2c7e57eeffcadc9020.png)

Next I looked for SUID bit files, but didn't see anything useful.

The todo hint talks about some automated script building something, so maybe I need to look into that. A common method for automated scripts is cron.

![b6915b9f53409abcf43b37dbf74e1b5f.png](images/b6915b9f53409abcf43b37dbf74e1b5f.png)

Looking at the crontab file I can see that a script is updated regularly. I tried to find the www folder in the server but it must be in a protected folder.

But the cron job grabs the file from overpass.thm. Perhaps I can exploit this, if I can change the dns resolver to my own webserver and pass it a reverse shell code - I should be able to get in as a root shell.

This is interesting, the hosts file has write access!

![d55053889753e59fad30aefbb9413541.png](images/d55053889753e59fad30aefbb9413541.png)

Now I'll try this out. 

I'll make a reverse shell payload with netcat

![37ebc749cbc7090253fd88b96c0aa9ef.png](images/37ebc749cbc7090253fd88b96c0aa9ef.png)

Put it in a folder nested under downloads/src/ and share that root folder on an python webserver. Using the same naming convention as the cron job.

![284868a184fdc4ce1db269a55a46fa55.png](images/284868a184fdc4ce1db269a55a46fa55.png)

Now to load the server.

![e2980458c63b5a56e38c94a5662fff65.png](images/e2980458c63b5a56e38c94a5662fff65.png)

Finally we change the host file, pointing the overpass.thm to my IP.

![311489bd6129acefd56b1cace5cce851.png](images/311489bd6129acefd56b1cace5cce851.png)

Now we wait...

This is working, I got a hit on my server.

![53f0183dff60c49729beda7282ce5bca.png](images/53f0183dff60c49729beda7282ce5bca.png)

But the reverse shell didn't load on my nc listener. Maybe the version of nc on the server doesn't support the -e flag.

![9c828382d4f24f16166b3c6abcdffb73.png](images/9c828382d4f24f16166b3c6abcdffb73.png)

I'll try a different payload / reverse shell in my http server script.

![fcf92ba10504ac10b672994a3d5b74c4.png](images/fcf92ba10504ac10b672994a3d5b74c4.png)

That one worked! :D

![97caa2e451df9bd608b0b5b1c2138f50.png](images/97caa2e451df9bd608b0b5b1c2138f50.png)

I am (g)root!

![bdd05bdc426ec753eb53ba1ad887cae1.png](images/bdd05bdc426ec753eb53ba1ad887cae1.png)

Now for the flag.

![8c535a74d58cfa674380bf288f8cb555.png](images/8c535a74d58cfa674380bf288f8cb555.png)