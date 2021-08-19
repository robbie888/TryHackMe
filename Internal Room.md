# Internal Room

## Overview

This room is a pen test simulation. Here is the brief:

You have been assigned to a client that wants a penetration test conducted on an environment due to be released to production in seven days.

**Scope of Work**

The client requests that an engineer conducts an assessment of the provided virtual environment. The client has asked that minimal information be provided about the assessment, wanting the engagement conducted from the eyes of a malicious actor (black box penetration test). The client has asked that you secure two flags (no location provided) as proof of exploitation:

- User.txt
- Root.txt

Additionally, the client has provided the following scope allowances:

- Any tools or techniques are permitted in this engagement, however we ask that you attempt manual exploitation first
- Locate and note all vulnerabilities found
- Submit the flags discovered to the dashboard
- Only the IP address assigned to your machine is in scope
- Find and report ALL vulnerabilities

## Let's begin with some recon

NMap scan with -sV -sC

![6e6ac6edb1b34cbba8036052f530d1f9](https://user-images.githubusercontent.com/60744763/130014492-e3ab2c3c-31fb-48be-8fda-ec308a8faa7c.png)

the Nmap vuln script shows us a blog, wordpress and login pages

![ae88779b3f6e44d788b8ee90c6c4f1e8](https://user-images.githubusercontent.com/60744763/130014512-582eaf51-1a11-4259-be65-2108c3dd896e.png)

The blog brings us to a simple webpage, I'll start looking around it and perhaps start with a web site vulnerability scan with nikto.

![37376585e7fa4bbaa7ca6685a7c46080](https://user-images.githubusercontent.com/60744763/130014531-506cde03-81f4-428a-857f-7df2c236341b.png)

I had to set a host dns record to internal.thm for the page to load correctly.

![765c7ffd907c4e9184461d7677e9394a](https://user-images.githubusercontent.com/60744763/130014636-c2013af5-2a59-483d-a241-6ee64bc8af80.png)

The wordpress recover password link advises me if a user name exists. So I could try to enumerate users this way.

![6701422e124d4eb59a828b3187893824](https://user-images.githubusercontent.com/60744763/130014677-19e6560c-847e-4842-907b-36bca5885af8.png)

But the user 'admin' seems to exist.

![303a00f3e5384da68a2a3997d4893942](https://user-images.githubusercontent.com/60744763/130014706-69b5ceb6-c165-4876-85ed-fd4bf47d7377.png)

A WPSCAN shows XML RPC enabled, this might lead to something.

![38b849aab18248ffbbf8282959588049](https://user-images.githubusercontent.com/60744763/130014729-ed2f5ad0-5901-4de5-b22b-75723a1a6b3f.png)

wp_cron is also enabled, not sure if this is helpful yet.

![e3db0ae76e59407f9584e39fb8cc3e0b](https://user-images.githubusercontent.com/60744763/130014754-cdd59f78-194d-433a-87c2-2fc9d264c564.png)

So I did some research on the XML RPC and found that it can be an easy way to brute force for passwords.

The following link helped me with the XML code to use.

https://the-bilal-rizwan.medium.com/wordpress-xmlrpc-php-common-vulnerabilites-how-to-exploit-them-d8d3c8600b32

Using the following post I was able to see what methods are available

![b1d8db6968714b28ab741b640f01ae10](https://user-images.githubusercontent.com/60744763/130014772-0c40d91c-61e6-4d93-98bd-3d6e9560cb28.png)

Then with the following POST I used the OWASP ZAP Fuzzer feature to brute force the password with the rockyou password list.

![579b3ed868144868a340a55c130121d4](https://user-images.githubusercontent.com/60744763/130014801-f4a3a992-6a41-4d8b-a794-93bfcb514f74.png)


After a while it identified the password!

Here is the password: my2boys

![1d380d9860f14a8bacdccc699b79e441](https://user-images.githubusercontent.com/60744763/130014836-e9771e61-6101-4699-9d81-10377e0db8c8.png)

Next I'll try and upload a reverse shell plugin to get shell access to the server.

Our NMAP scan told us the server is linux, so I'll use this plugin I found online.![23f825bc6ed941ae9905773fc410234d](https://user-images.githubusercontent.com/60744763/130014878-4696ff35-aadb-40de-894e-77a83e919534.png)

![92a5be3c300240549b6bcca4355196e3](https://user-images.githubusercontent.com/60744763/130014887-764aa5fe-75ff-4e0a-bec9-f655c2c3afea.png)

I receive an error when uploading, perhaps the folder permissions are not correct for uploading plugins.

So I'll need to look for another method to inject code.

![dccd9a4781da4937a5081f4dc737b348](https://user-images.githubusercontent.com/60744763/130014912-faa2e444-709f-436e-985b-7ee4abc9e832.png)

This time I tried to use the plug-in edit and edited an existing plugin, instead of installing a new plugin.

![8e9ddd59a674456f8155091da9c936d3](https://user-images.githubusercontent.com/60744763/130014933-276edefa-8d85-4f70-b7b7-da9c76f5dcc4.png)

That also didn't work as the file is not writeable. I'll try a few more files before moving on.

![f920a912ef9a4bed8efc8c1e8fb7ee2d](https://user-images.githubusercontent.com/60744763/130014965-36848d2b-11e7-4ac7-aaea-d3368b1ed869.png)

clicking around on the site I found the following private post on the admin's portal.

![d4426a195f934428a23b7d2f9c401701](https://user-images.githubusercontent.com/60744763/130014992-fceb1681-dc42-43f2-bb7d-9e71729f5f61.png)

I'll try this user/pass combo with ssh.

![fabea57f5cf64b84ba567077887b7e66](https://user-images.githubusercontent.com/60744763/130015020-6cb54aef-6f83-4619-84f8-186b8be68c91.png)

They didn't work, but I might have a username now. So will try to crack it over ssh with the rockyou password list again.

I left that run for a while, It didn't seem to produce any results (it's still running!)

Clicking around the wordpress site I found the theme editor section and will try to inject some code here (the php reverse shell I tried earlier in the plugin section)

![7fcc2dbfc2df48778b25055947b45a6a](https://user-images.githubusercontent.com/60744763/130015054-65b85521-7f3a-4973-907f-9e8d11ca1f7b.png)

Some googling about this suggested attacking the 404.php file. So i'll add the code there.

Here I've added the code and will save the file.

![f79e023ace19453b953db127bcef9688](https://user-images.githubusercontent.com/60744763/130015078-c56729cc-afa5-40bf-a1b9-4b3ce432a3a5.png)

That saved successfully

![71b27b65d02843b592b6589fa63d9efc](https://user-images.githubusercontent.com/60744763/130015099-a8e9664d-f794-4d15-b5a1-9ac621b0d6f7.png)

After trying to load a page the didn't exist, I got a shell from my netcat listener!

![9330b2cc1d254df2a0a5e6496870aa89](https://user-images.githubusercontent.com/60744763/130015119-df4bbbca-24af-4bef-9321-c50fce04c186.png)

![9dd84d4daf074160880d2b1559acb76c](https://user-images.githubusercontent.com/60744763/130015135-1b7546e6-cd57-47c5-a7b4-51eb9a33a0ca.png)

First I tried to look around the home folders, but couldn't access much at this stage.

![4c6618fba1c94f3080421e9b7e978f89](https://user-images.githubusercontent.com/60744763/130015156-5f89244d-879f-4cca-8525-1ae8f0625972.png)

I'll upgrade from the simple shell and go from there.

![6138e28410034d158013463179bfa45d](https://user-images.githubusercontent.com/60744763/130015179-8803ebf8-3c93-45a1-b16c-4ce6ef289cbd.png)

sudo is locked down as expected.

![7ad40a684d36440b853e141b54b91aa1](https://user-images.githubusercontent.com/60744763/130015202-0e892b4a-e534-4f1f-969d-969d0605efef.png)

I started looking around the server and found something interesting in the /opt folder.

![f16d49b42c7743c899127287752d5017](https://user-images.githubusercontent.com/60744763/130015231-12b878b7-2988-4473-be31-4808e33c6e10.png)


I'll try these creds over ssh now. aubreabba / bubb13guM!@#123

That was a good find, I am in as a user with ssh now!

![41f5284313dc464aa08f78c1e285323a](https://user-images.githubusercontent.com/60744763/130015252-36cfae5b-09d0-4e9c-8b33-330d3694350e.png)

Here is the first flag!![f7ada3baad564a3284c659744ba8e433](https://user-images.githubusercontent.com/60744763/130015270-2cac24c7-1138-474d-ae8f-014984c44b6f.png)

That jenkins file also looks interesting

![29ce224e5c034488bad633b3d0ef0564](https://user-images.githubusercontent.com/60744763/130015285-c700f872-02a9-431e-b0ef-ea781a0d262c.png)

Let's check if this user is a sudo user

No luck

![5f1fc37946f443a8ab90070d4e58e041](https://user-images.githubusercontent.com/60744763/130015308-2c499aab-d0d5-4920-9c29-fab23e9774b9.png)

I'll see what that Jenkins service is, first I'll nc to it.

![f4c57696272747b6a95dd04169d2c4df](https://user-images.githubusercontent.com/60744763/130015330-f4e7f640-5787-44a6-9595-9f4feed22b3b.png)

Looks like a http service, I'll ssh and tunnel that 8080 port and see if i can browse it from my attack box.

![5bb0e72cae4f44279986ebced1f74438](https://user-images.githubusercontent.com/60744763/130015348-92a25848-b9ea-433a-94a5-62b024a94a47.png)

![7cc582a183f64d70a6b5756bdb0ddd44](https://user-images.githubusercontent.com/60744763/130015364-2d4e62f4-45d0-49ef-89f1-d22f28c31644.png)

We have another web portal. I'll try the creds we used for ssh.

They didn't work, how about the ones we found in the admin's wordpress note...

they didn't work either...

![c3da6c8e5637476d85b22029eb05e0af](https://user-images.githubusercontent.com/60744763/130015394-74cac154-b34f-41e7-8ae7-a4090dc397c5.png)

Searching the server for some information on jenkins with the ps aux command.

![7aedab1b2e714c76a7f0db11103ba345](https://user-images.githubusercontent.com/60744763/130015432-6100c825-cfff-48bc-b237-727e9e85886d.png)

Looks like it is a docker container.

![79f916c56e7646baa5d7e92e0c6f4943](https://user-images.githubusercontent.com/60744763/130015464-9375958c-2e3b-41c8-9b2d-fdb64b246a50.png)

I'll try and attack the web portal with OWASP ZAP as I did earlier with the wordpress site. I'll assume the username is admin for my first attempt.

![67f499bcff834a888fe91f1e9f0bae84](https://user-images.githubusercontent.com/60744763/130015503-7ba9e1cd-ae49-4fdd-8b4a-8e751db1a3e8.png)

After using the Fuzz tool with the rockyou password list, I saw this response from the server is a different size than the other password failure responses, so lets try that password

![6fec0289b30e4d3da5690434b6f04dbd](https://user-images.githubusercontent.com/60744763/130015544-52d69280-ad14-4f2f-8f6f-930bbb5c15b1.png)

Looks like I'm in with the username admin and password spongebob.

![79b19f11dbfc49079e84f6eae3460fcf](https://user-images.githubusercontent.com/60744763/130015570-2449bb36-0c7c-4bd9-bfa1-0fcc3dd97040.png)

I'll disable OWASP now and look around the site to see what looks interesting.

In the management section there is a script and CLI console, perhaps I can upload a reverse shell then get a shell into the docker container...

![3f2776da01c149888bbd4d0ddc8a6a69](https://user-images.githubusercontent.com/60744763/130015605-57f94e7a-73bf-450a-b949-241f342467fd.png)

The script console uses 'groovy-script' I'll google search if there are any reverse shell scripts online or how to inject OS commands.

![02717ef60e974702890f17233331270c](https://user-images.githubusercontent.com/60744763/130015630-b0206c32-befe-4e35-adcb-e27cb5a67648.png)

This link provides some options, but it seems to be windows based, I'll try adjust it for linux.

https://coldfusionx.github.io/posts/Groovy_RCE/

It seems to work running an ls command.

![c3d4873688ad45a999e28bd402093672](https://user-images.githubusercontent.com/60744763/130015657-e20a4df2-ed4d-4bc9-9f2a-0a58c059b195.png)

Now for a reverse bash.

![ace8fb321b53429d8030f08155cbf10d](https://user-images.githubusercontent.com/60744763/130015684-8cc776bd-b9bf-4cf0-a1ad-1efe6fafb160.png)

It didn't work to my attack box (makes sense...), but might work to my local ssh session on the host box.

I'll try both these IPs

![2f014f239b284c3a845b6cb1a82540ff](https://user-images.githubusercontent.com/60744763/130015718-d3b86635-a61c-4906-b058-e35ec9420668.png)

I couldn't run the shell with the above command, I'll try another injection option from the Groovy_RCE link above

![8b1aa705d68c459fa44a1170b7c10569](https://user-images.githubusercontent.com/60744763/130015738-e00500c2-d624-4b59-b5d0-edb2c9c26fe3.png)

Bingo!

![5ab1d90bec444df295f5e97030c18dab](https://user-images.githubusercontent.com/60744763/130015764-ebb45a10-5dcd-441a-81e9-97f796da9876.png)

Looking around the server, there is another note in the OPT folder including some credentials.
In hindsight, I could have just navigated through the server using the Script Console on the web portal...

![674a4aff821f4bef98a72d3c84d3f2bf](https://user-images.githubusercontent.com/60744763/130015787-885f8b03-f864-4687-b566-bef837f87760.png)

First I'll try them on the host server with ssh.

And I'm in as root!

![3d4073ac1e0e4116b48ee85d8588b85e](https://user-images.githubusercontent.com/60744763/130015807-897d7d7a-4436-4245-9279-096dc13b8c7d.png)

Now for the flag

![5465dd3c839f44c88340ea05d6c41cb0](https://user-images.githubusercontent.com/60744763/130015831-726bf02a-5428-41f4-a6a3-30e123d6c3fb.png)

