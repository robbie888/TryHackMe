# Simple CTF

This is a simple CTF room.

Let's start with an Nmap scan.

![c05730d7e09734c9b20936ef965b6ada.png](images/c05730d7e09734c9b20936ef965b6ada.png)

We can see a few ports open here and ssh running on port 2222.

I'll check for vulnerabilities with an nmap scan.

![8c739e77a0b22687c2f8f70157053906.png](images/8c739e77a0b22687c2f8f70157053906.png)

We note a robots.txt file, so we can look at that.

First I'll check out the website.

![4de2776e5b0871b064d72036cdc01fb4.png](images/4de2776e5b0871b064d72036cdc01fb4.png)

We get the default Apache2 page.

Now the robots.txt, looks like it has some interesting information. A potential user name 'mike' and a folder.

![51b5ce433ee58b710791e18ceed33e5a.png](images/51b5ce433ee58b710791e18ceed33e5a.png)

The openemr folder didn't load anything.

So I'll kick of a gobuster directory scan.

![f817c54e38057e48b14ed693f760114f.png](images/f817c54e38057e48b14ed693f760114f.png)

We've found a folder /simple

This leads to a CSM called CMS made simple.

![8aefbb4479cddf3c90ab5fcee078d844.png](images/8aefbb4479cddf3c90ab5fcee078d844.png)

The version is listed at the bottom of a page

![88a900aac07a5f787b0d431741420f03.png](images/88a900aac07a5f787b0d431741420f03.png)

A quick google takes us to the exploitdb with a SQL injection vulnerability.

https://www.exploit-db.com/exploits/46635

The vulnerability is CVE-2019-9053

I've copied the exploit onto my attack box and will try it out.

![43e1887b18040df7eb68c1cfcf5f8fef.png](images/43e1887b18040df7eb68c1cfcf5f8fef.png)

![cdb6f5af4f3a6952663cd9536bd73254.png](images/cdb6f5af4f3a6952663cd9536bd73254.png)

I'll try it out with my a short password list - fasttrack

![c8c189baec6bb00e4d9ed9f3a27c901c.png](images/c8c189baec6bb00e4d9ed9f3a27c901c.png)

We got some good results here.

Now to try this to login, I'll go for ssh first.

![2bbfe8873a36b05abdfa27e2b41b660a.png](images/2bbfe8873a36b05abdfa27e2b41b660a.png)

We're in as mitch!

And we have out user flag.

![9e2c7fd8565e403220e9f1d6a7c2f381.png](images/9e2c7fd8565e403220e9f1d6a7c2f381.png)

I'll check out sudo now.

![8ff94ee365f91b104474507fe41b02e8.png](images/8ff94ee365f91b104474507fe41b02e8.png)

Looks like I can run vim as sudo, so will try and spawn a shell from vim.

Here we are in vim as sudo, I'll run the following vim command.

![e1d2f7adc6290ffe208845e618e824ab.png](images/e1d2f7adc6290ffe208845e618e824ab.png)

And here we have it, i am root!

![fa0dc3fb611ae2e449181cbdec162073.png](images/fa0dc3fb611ae2e449181cbdec162073.png)

And now for the flag!

![7b89d7a27175b6188ac57084fc1e2e44.png](images/7b89d7a27175b6188ac57084fc1e2e44.png)

That's it for this one.