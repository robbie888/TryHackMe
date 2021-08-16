This is the pickle rick room. A web site to exploit.

This Rick and Morty themed challenge requires you to exploit a webserver to find 3 ingredients that will help Rick make his potion to transform himself back into a human from a pickle.

Tools I used where:
Kali Linux
NMAP
The rest was done in the web browser.

## Flag 1
Rick & Morty at it again...

![e77ed589d4b94979b53ce34cf7a3e75e](https://user-images.githubusercontent.com/60744763/129639815-9ce29bcc-7185-49c6-bf9f-a18e3497a421.png)

Source code indicates a username: R1ckRul3s

![c860842fa2104ba6950e5a6fca8d9320](https://user-images.githubusercontent.com/60744763/129639835-b05ac40f-892c-4cd9-b963-a7636b00c687.png)

Source code also shows a folder /assets

![faed74b2c47f4e83bf56a388c0a615b3](https://user-images.githubusercontent.com/60744763/129639860-4038eb31-befb-48b9-be11-afdaeb5caab9.png)

This folder is listable:

![7720f7e94a054207b3ab8a4f7b765d34](https://user-images.githubusercontent.com/60744763/129639872-54c96d37-01d2-4a19-a713-8f8e6c1ae8e2.png)

NMap vulnerabilities script shows some interesting options.

Robots.txt, a login page, and possible sql injection. Plus an insecure session id.

![24bb800a48ac467890c9f98dc364528e](https://user-images.githubusercontent.com/60744763/129639892-dce81e4b-dce2-4052-8aa1-e14c965ae350.png)

The robots.txt file as an interseting string, it might be a password.

Wubbalubbadubdub

![e4450f4ec0e7456490927cc8e68c4973](https://user-images.githubusercontent.com/60744763/129639907-1b72f19c-f5a8-425c-9dca-8deb73f7ccf0.png)

I tried ssh access with the username we found above, but it is protected by a public key authentication.

![6985919e5cff48a096be42aa7b0b9a1d](https://user-images.githubusercontent.com/60744763/129639925-18aabbf5-cbcb-4963-8504-715d9b75fe9a.png)

I'll try it on the login page nmap that found too...

![163c567e4580488d8488cbbc1d42e2be](https://user-images.githubusercontent.com/60744763/129639940-a1f77583-c492-45df-96a8-f8b1253fd546.png)

Looks like we're in with that username/password combo.

![3423729ac2a843e38d43315cbaf3f01e](https://user-images.githubusercontent.com/60744763/129639961-bf3ddf05-e30e-4e55-90e3-c998e492d9a5.png)

This command panel takes OS commands, I typed `ls` and received the following.

![a9846774616c42c39f46ea245aeeee87](https://user-images.githubusercontent.com/60744763/129639977-d5c934a7-3c26-4ffe-9b12-6ad637847b93.png)

Looks like our first flag is the first file in this list. I'll read it with the `cat` command.

![96df9b51a07f47888212523f20c5e596](https://user-images.githubusercontent.com/60744763/129639987-f3b5cc2e-5788-4dd3-9c67-618d4542ad24.png)

That didn't work, the `cat` command is blocked.

I'll try some more combos, like using `ls && cat <filename&>`

That didn't work either, but browsing to that filename worked!

![c7e936101c8e48a3acc2e212a9561b70](https://user-images.githubusercontent.com/60744763/129640004-63c7cc60-45be-49fc-b3cd-7fc14979f5a9.png)

First flag found.

## Flag 2

The clue.txt file looks interesting, it's advising us to look for clues on the file system.

![0a7c324604524a698e6df7c9a38e006c](https://user-images.githubusercontent.com/60744763/129640025-6b23e326-ee53-4f77-97e1-1894792cb3ec.png)

`ls /home` yields a user name called rick. I'll check that folder out.

`ls /home/rick -la` yields the following, looks like our second flag. 
I'll look around the web site for some potential IDOR to a file system path, since `cat` is not working.

![b238d05a0dcb4069876b89892bfebd62](https://user-images.githubusercontent.com/60744763/129640067-06638562-84b5-436a-9b5c-f8ac2afaa87a.png)

The rest of the site appears to be inaccessible to my current user. However 
I'll try copy the file from the home directory to the www folder so I can read it through the browser.

![1f8dfb521fdf4d948b7cf78c58083f9d](https://user-images.githubusercontent.com/60744763/129640106-caf54cb1-cddf-41ff-9cb3-802efad583e3.png)

That didn't work either, I used the following command to read the error in stdout.

![c5fa1961ce9e4a04bfad0db2ef445281](https://user-images.githubusercontent.com/60744763/129640124-de64c628-59f3-478f-821f-e16ce34a42a2.png)

The command `grep` seems to work, so I'll try that to read the file in rick's home folder.

I used `grep a <filename>`, just guessing "a" might be in the file. I got lucky!

![d7b5e44415f344eaa4684a8778c351e1](https://user-images.githubusercontent.com/60744763/129640181-ae5220ed-a182-46c7-9a9f-e8465b126722.png)

That looks like the next flag!

## Flag 3

Now I'm guessing we need to see what is in the /root folder for the final command.

First I'll look at the source of the login.php to see if anything interesting is there...

Using grep again with the -v flag and a random string that is unlikely to be in the code, it should show me the PHP code.

It looks like the browser has rendered it, but I can view source now to see what is in there.

![24951b825755405dac1fdc4ffd53382e](https://user-images.githubusercontent.com/60744763/129640214-934c44f3-bf79-4220-9cde-e11fe9f15b31.png)

![81a0329dafdf4d9b86d6d1a2aeb1b9d5](https://user-images.githubusercontent.com/60744763/129640237-c40aa9bc-581c-494a-a629-e7bf75284e5b.png)

Nothing too interesting here...

I could try my luck with the sudo command, I'll try a `sudo ls`

That seems to have worked! This server is very mis configured!

Now lets look in the root folder.

And here we have it!

![81d82fc9127d4f7ab3f5e8fde35b2ee3](https://user-images.githubusercontent.com/60744763/129640279-9544aaa3-ecb5-41be-8e9f-a3be90f77c47.png)

I'll read the file with the same grep command I used earlier to view the source code except as sudo this time.

And here we have it

![359ec3295dff475f846b8855edf56950](https://user-images.githubusercontent.com/60744763/129640298-c4f69ee7-84aa-46e3-b62f-9098f13c5f3a.png)


## Root Shell Attempt
Next for fun, I'll try a reverse shell command in the command panel box.

![48be827592b74b47a83cb688580b491b](https://user-images.githubusercontent.com/60744763/129640321-04be0d10-ed02-4720-9bd3-302721a0f6d2.png)

Win! I have a reverse shell now...

![965cdf96b8c7468bbbe4c2c39ebbac19](https://user-images.githubusercontent.com/60744763/129640342-e8e74c4a-5070-4207-aeab-6bc0e47b495d.png)

Since I already know sudo is working for my used, I can simply run bash as a sudo user.

![ba65f92de27441cb90a148b9e5978377](https://user-images.githubusercontent.com/60744763/129640367-721989bc-f744-49e1-a95f-4010f7bb5373.png)

And I am (g)root!
