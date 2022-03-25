# Log for Jodie (Log4Shell CVE)

Hi there!
Here is how I've solved the "Log for Jodie" challenge in LNC CTF 2022.

# WARNING
This writeup is for EDUCATIONAL PURPOSES only. I do not have any responsibility if you face any legal consequences arising from 

## Challenge description

I found a chatroom for coffee lovers. Right now it's going strong but it would be better with more people! So tell your friends about it now! Oh, btw did you hear? Minecraft got hacked.
`nc c2.lagncrash.com 8003`

## Solution & step-by-step walkthrough
This challenge seems to be using Log4Shell, an exploit for Apache Log4J that was released in late 2021.  
https://en.wikipedia.org/wiki/Log4Shell  
It allows one to RCE into other servers just by using JNDI paired with LDAP. 

Knowledge of Java (and Python) is useful, although not strictly required.

Firstly, let us netcat into the server provided.
![Screenshot from 2022-03-25 15-00-11](https://user-images.githubusercontent.com/58442255/160070697-9cb70585-3270-4e1b-bbae-2490d89bfc9d.png)
The server has spawned a link for us to connect. Let us connect to that as well.
![Screenshot from 2022-03-25 15-02-05](https://user-images.githubusercontent.com/58442255/160070820-4a6a2ca1-1983-46b3-bd62-fff3746e865c.png)
Create a nickname and start messaging.

Let's test if the application is vulnerable. You can use any Log4J testing tool on the Internet to test.  
I found this one: https://log4shell.tools/  
Here's a screenshot:
![Screenshot from 2022-03-25 14-59-12](https://user-images.githubusercontent.com/58442255/160070457-918b9ed3-be83-4572-8e96-2cdcccc5892a.png)
Let's enter that into the chat app:
![Screenshot from 2022-03-25 15-07-12](https://user-images.githubusercontent.com/58442255/160071615-8d93a4bd-7d0e-465e-805f-f34beb1c3be6.png)
The server is <b>vulnerable.</b>

Now that we know that the server is vulnerable, how do we RCE into it and get the flag?   
A quick search returns several Log4Shell PoCs, which I used this one: https://github.com/kozmer/log4j-shell-poc
