# Steel Mountain

Description: Hack into a Mr. Robot themed Windows machine. Use metasploit for initial access, utilise powershell for Windows privilege escalation enumeration and learn a new technique to get Administrator access.

## Employee of the month

The first question is to find the employee of the month. There is a photo on the main page. The source code shows the file name which gives us the employee's name ;)

![be5919c41d92ff2dbe2fbec4cd9b21fd.png](images/be5919c41d92ff2dbe2fbec4cd9b21fd.png)

![9bed8158b0b292925484393210cf8e4d.png](images/9bed8158b0b292925484393210cf8e4d.png)

## Initial Access

I'll start off with nmap.

![a71410916df44c9bf11b113de1660eee.png](images/a71410916df44c9bf11b113de1660eee.png)

Here we can see what's running on the machine, including service versions.

There is another web server on port 8080, looks like a file server. Let's check it out!

![5119c6b77788669e4f8b800b766c687d.png](images/5119c6b77788669e4f8b800b766c687d.png)

This nmap scan and link from the web server gives us the answer's to the next few questions.

Now to look for an exploit.

![747d1462f605c170c7097cf4c5d7d593.png](images/747d1462f605c170c7097cf4c5d7d593.png)

Looks like this version is vulnerable to RCE! We should be able to take advantage of this.

Note the CVE info is available here: https://www.exploit-db.com/exploits/39161

Now over to metasploit, here we can see the exploit, I'll try this one.

![9bd47efa2d01ffe1d57883e8809e1ce1.png](images/9bd47efa2d01ffe1d57883e8809e1ce1.png)

Now I've setup the exploit with the appropriate details

![6314c88fc9ef01652030763e823aa415.png](images/6314c88fc9ef01652030763e823aa415.png)

We run the exploit and...

![d7e5e9e0a913706fea24593b46b5daac.png](images/d7e5e9e0a913706fea24593b46b5daac.png)

we have a meterpreter shell!

I've now opened a os shell, hunted around the server, and found the first user flag.

![e036ea1df0c5a86947dd7bef039c1c62.png](images/e036ea1df0c5a86947dd7bef039c1c62.png)

## Privilege Escalation

Now we'll try to escalate our privileges on this machine.

First I tried my luck with the built in metasploit option 'getsystem' but this failed.

![cc8ab71aa81e602f1d0a55c0de69d9e1.png](images/cc8ab71aa81e602f1d0a55c0de69d9e1.png)

So I'll look for other options. The room advises us to try a PowerUp script, so I'll get that and check it out.

I've downloaded it, put it on my desktop and uploaded it with my meterpreter shell.

![b7a171ce779778dbc5fd9226038f9f64.png](images/b7a171ce779778dbc5fd9226038f9f64.png)

Now I'll load powershell

![9068f36bc616cd7f897f4f56fc937ee5.png](images/9068f36bc616cd7f897f4f56fc937ee5.png)

Now to run the script.

![6868fc1a7ea243b4875e363fd2975c7c.png](images/6868fc1a7ea243b4875e363fd2975c7c.png)

So we have some interesting info here about this service. This services appears vulnerable: AdvancedSystemCareService9

![665ede7c4cf3cea5cd4f32065ae590f6.png](images/665ede7c4cf3cea5cd4f32065ae590f6.png)

Since we can restart this service, and write to the folder in it's path and the path being used is non quoted, we could replace this executable with a payload.

I've now made a reverse shell payload which I'll upload using the same process as before with meterpreter.

![0cb2e7aa0874cd7b61e74066982023a6.png](images/0cb2e7aa0874cd7b61e74066982023a6.png)

Here is the file uploaded and copied to the appropriate location, now to move it to our service file, setup our listener and restart the service.

![b4555b57370a1b5ad5825df4218f89ba.png](images/b4555b57370a1b5ad5825df4218f89ba.png)

A quick note, the above image is incorrect the correct location to place the payload is one folder down in the IObit directory, this way the service picks up the payload first.

Now to stop and start the service

![bafa287f5f2711f717aeab942cf00d63.png](images/bafa287f5f2711f717aeab942cf00d63.png)

Now we get a connection on our listener!

![a76c585b00cf175138636d73fd528cf3.png](images/a76c585b00cf175138636d73fd528cf3.png)

We are system!

![3adcb183bb6ca0461a9dfac76f15d996.png](images/3adcb183bb6ca0461a9dfac76f15d996.png)

Now to hunt for the flag :)

Here we go! In the administrator's profile.

![b3640c54d003530be16f8ccb949c3a95.png](images/b3640c54d003530be16f8ccb949c3a95.png)

That's it for this CTF!