Sherlock Scenario

FrostLine Analytics, an internal data reporting firm, noticed one of their containerized services making outbound connections it had no business making. Rather than kill the container and lose whatever was sitting in memory, the security team took a checkpoint of it while it was still running and pulled it offline for analysis.

You have been handed that container checkpoint. The goal is to reconstruct the compromise.

Tasks:


**1. The security team believes the attacker had hands-on access to the container at the time the incident was declared and IR was initiated. Finding the attacker's hidden network channel is important for attribution and identifying potential exfiltration. What is the IP address and port?**

After unzipping original zip file I inspected what I was provided by.
![Pasted image 20261031225048.png](images/Pasted%20image%2020261031225048.png)

![Pasted image 20261031225111.png](images/Pasted%20image%2020261031225111.png)

![Pasted image 20261031225145.png](images/Pasted%20image%2020261031225145.png)

I saw many .img files, at this point I don't really know what any of these files actuall are so I decided to inspect some of them.
![Pasted image 20261031225448.png](images/Pasted%20image%2020261031225448.png)

Here I saw many files are **CRIU** files, checkpoint/restore in userspace. Its software tool that security team probably used for "freezing" running container, saved its current state to disk as image files (that is why we can see a lot of .img files in checkpoint folder) and restored it later on same or different machine. Also, below I installed **CRIU** tool for inspecting these files.
By going through output of this command:
![Pasted image 20261031231453.png](images/Pasted%20image%2020261031231453.png)

I noticed this suspicious dst port and dst address, while src port and src address represents containers ip and port.
![Pasted image 20261031231405.png](images/Pasted%20image%2020261031231405.png)


> 🟩 **Answer:** 185.220.101.47:14444




**2. During disk inspection of the checkpoint, nothing malicious was observed. What is the full path of the file suspected to be the one establishing a covert channel to the attacker's infrastructure?**

By inspecting files.img with criu tool crit I found the answer.
![Pasted image 20261031234122.png](images/Pasted%20image%2020261031234122.png)

Also we can see that attackers ip and port are in the same file on same spot which makes this answer more likely to be correct.

> 🟩 **Answer:** /tmp/.x




**3. What is the inode of the file from previous task?**

By inspecting dump.log file I found something interesting:
- CRIU processes file with file descriptor 14 (fd 14 from dump.log), path is /tmp/.x but label "(deleted)" means that file was already deleted from filesystem
- after that removes the label "(deleted)" so the original path can be observed
- CRIU recognizes "/tmp/.x" like "ghost file" and stores it in checkpoint ("Dumping ghost file for fd 30 id 0x24")
- CRIU stores content of the deleted file ("Dumping ghost file contents (id 0x1)")
- and lastly CRIU states that for verification purposes it could only store the file size ("Only file size could be stored for validation for file")
![Pasted image 20261101000002.png](images/Pasted%20image%2020261101000002.png)

By knowing this this indicates that inode number of /tmp/.x file is actually inode number of "ghost-file-1.img". 
![Pasted image 20261101003014.png](images/Pasted%20image%2020261101003014.png)


> 🟩 **Answer:** 323271




**4. The malicious file was used to setup the container to quietly reach a remote endpoint. What is the full URL?**

I ran this command to find something useful,
![Pasted image 20261101191530.png](images/Pasted%20image%2020261101191530.png)

by examining the output of command I found the answer.
![Pasted image 20261101191408.png](images/Pasted%20image%2020261101191408.png)


> 🟩 **Answer:** xmrpool.eternal-hashes.net:55555



**5. Connecting to that remote service needs credentials to authenticate. What are the credentials the file uses to authenticate to the remote endpoint?**
I got this answer while I was doing task 4. In task 3 I was provided with base64 string (it can be seen in 2nd picture in task 3), I imported it in cyberchef and got this result. 
![Pasted image 20261101012110.png](images/Pasted%20image%2020261101012110.png)

So I ran strings command on pages-1.img file and filter out some keywords that can potentially lead me to answer.
![Pasted image 20261101012342.png](images/Pasted%20image%2020261101012342.png)


> 🟩 **Answer:** 49PnvHq8sLab2Tn1Qk7r6Wd4eZ9yJ3cV5bN8mA1sR2tF6gH2xK4wL7pD3qE9oU1iY5aS2dG8hC6vB6nM4kQ9TfXm7Zr3D:x




**6. Identifying the initial access vector is crucial for identifying IOCs and TTPs, making it easier to determine the breach radius. What is the IP address used to compromise the container as part of the initial compromise?**

I started with inspecting files.img file,

![Pasted image 20261101193530.png](images/Pasted%20image%2020261101193530.png)

by inspecting it I found suspicious inbound TCP connection.
![Pasted image 20261101193444.png](images/Pasted%20image%2020261101193444.png)

So I ran strings against "pages-1.img" to possibly find something that can confirm my suspicion. 
![Pasted image 20261101193839.png](images/Pasted%20image%2020261101193839.png)

In the middle of output I finally found it.
![Pasted image 20261101194010.png](images/Pasted%20image%2020261101194010.png)

This confirms 27.34.64.82 as the source IP involved in the initial compromise.
Also I checked it in container.log by searching for this ip keyword.
![Pasted image 20261102154833.png](images/Pasted%20image%2020261102154833.png)


> 🟩 **Answer:** 27.34.64.82




**7. The attacker abused a normal and trusted workflow that the container was used for by FrostLine Analytics to achieve code execution. What is the name of the file that was created by the trusted workflow that gave the attacker access to the container, and what was its title?**

I used timestamp that I obtained in previous question in container.log.
![Pasted image 20261102155000.png](images/Pasted%20image%2020261102155000.png)

Then I ran strings command against pages-1.img file and filter out this timestamp.
![Pasted image 20261102155125.png](images/Pasted%20image%2020261102155125.png)


> 🟩 **Answer:** RPT-1057, Untitled




**8. The DevSecOps checklist clearly dictates that service account authentication mechanisms should be considered compromised in the event of an incident. What is the expiry time of the mechanism used in the environment, which could have given the attacker cluster-wide privileged access?**

By running strings against pages-1.img file and filtering out some keywords,
![Pasted image 20261102160700.png](images/Pasted%20image%2020261102160700.png)

I found **JWT** service account token.
![Pasted image 20261102160601.png](images/Pasted%20image%2020261102160601.png)

By copying 'sa_token' field and importing it in JWT debugger, we can find expiration time in JSON format.
![Pasted image 20261102161327.png](images/Pasted%20image%2020261102161327.png)

Expiration time is 1814239749 which is JSON format, converting it in UTC leads us to correct answer. 
![Pasted image 20261102161708.png](images/Pasted%20image%2020261102161708.png)


> 🟩 **Answer:** 2027-06-29 03:29:09




9. Identify the name of the thread responsible for malicious activity in the container.
In CRIU format every "core-X.img" file represents 1 thread, which means in every core file 1 thread name can be found. I ran this command to find all thread names of all core files.
![Pasted image 20261102163435.png](images/Pasted%20image%2020261102163435.png)

![Pasted image 20261102163456.png](images/Pasted%20image%2020261102163456.png)

By inspecting a lot of these thread names through strings and files, I found something interesting.
Running this command,
![Pasted image 20261102170117.png](images/Pasted%20image%2020261102170117.png)

in the output I found this.
![Pasted image 20261102170150.png](images/Pasted%20image%2020261102170150.png)

Timestamp match perfectly our case, and RPT-1057 is name of the file that was created through trusted workflow(question 7).

> 🟩 **Answer:** C2 CompilerThread9




**10. Thankfully, the attacker did not get the chance to steal critical data from the pod, only basic information. What is the full path of the archive file that the attacker created for potential exfiltration?**
I assumed that path is correlated to "/tmp/.x" file so I ran this command.
![Pasted image 20261102172542.png](images/Pasted%20image%2020261102172542.png)

Attacker created tar archive (tar -czf).
![Pasted image 20261102172751.png](images/Pasted%20image%2020261102172751.png)


> 🟩 **Answer:** /dev/shm/.X11-unix/.cache.tar.gz
