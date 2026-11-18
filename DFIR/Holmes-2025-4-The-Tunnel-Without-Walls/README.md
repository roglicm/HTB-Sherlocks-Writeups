Sherlock Scenario

A memory dump from a connected Linux machine reveals covert network connections, fake services, and unusual redirects. Holmes investigates further to uncover how the attacker is manipulating the entire network!

Tasks:


**1. What is the Linux kernel version of the provided image?**

I ran volatility command for analyzing memory dumps, command was different because mem dump of this OS was Linux
![Pasted image 20260927200628.png](images/Pasted%20image%2020260927200628.png)


> 🟩 **Answer:** 5.10.0-35-amd64




**2. The attacker connected over SSH and executed initial reconnaissance commands. What is the PID of the shell they used?**

For this question and next ones, I needed to first import right symbols.
I have Linux version so I just need to download it from github like this
![Pasted image 20260927202234.png](images/Pasted%20image%2020260927202234.png)

then list the directory to see if proper version is in there
![Pasted image 20260927202350.png](images/Pasted%20image%2020260927202350.png)

it is, now we can run commands, but also we need to include this file in our command to run properly
![Pasted image 20260927203616.png](images/Pasted%20image%2020260927203616.png)

I filter out 'ssh' with grep and 2 processes before and after with 'A' and 'B' to have clearer view of output, PID 13608 is first interactive shell after SSH authentication 

> 🟩 **Answer:** 13608




3. After the initial information gathering, the attacker authenticated as a different user to escalate privileges. Identify and submit that user's credentials.
For this question I found that attacker autehnticated as user "jm"
![Pasted image 20260927211908.png](images/Pasted%20image%2020260927211908.png)

then created dump file of strings and filter out "jm:"
![Pasted image 20260927212206.png](images/Pasted%20image%2020260927212206.png)

this is md5 hash meant to be cracked with John the ripper tool, after importing rockyou.txt and cracking the hash I obtained a result
![Pasted image 20260927212400.png](images/Pasted%20image%2020260927212400.png)

also I found something else in strings
![Pasted image 20260927212616.png](images/Pasted%20image%2020260927212616.png)

The attacker used a Docker container with the host’s `/etc` directory mounted at `/mnt` to append a new entry to the host’s `/etc/passwd` file. The command created the user `jm` with UID and GID `0`, `/root` as its home directory, and `/bin/bash` as its login shell. Since UID 0 grants root-level privileges, this action established a persistent root-equivalent backdoor account

> 🟩 **Answer:** jm:WATSON0




**4. The attacker downloaded and executed code from Pastebin to install a rootkit. What is the full path of the malicious file?**

First I ran this command to find something malicious, since I dont know what exactly is installed via link that contains pastebin domain
![Pasted image 20260927214136.png](images/Pasted%20image%2020260927214136.png)

With name obtained, I started filtering every volatility command with observed name that can provide full path of the file
![Pasted image 20260927214852.png](images/Pasted%20image%2020260927214852.png)

this one gave results

> 🟩 **Answer:** /usr/lib/modules/5.10.0-35-amd64/kernel/lib/Nullincrevenge.ko




**5. What is the email account of the alleged author of the malicious file?**

From previous question we know the inode number is: 0x9b3386454a
![Pasted image 20260927220434.png](images/Pasted%20image%2020260927220434.png)

by knowing that I extract data with inode number from page cache 
![Pasted image 20260927220723.png](images/Pasted%20image%2020260927220723.png)

I obtained new file called "inode_0x9b3386454a80.dmp", by running strings and filtering it with "author=" answer can be obtained

> 🟩 **Answer:** i-am-the@network.now




6. 
![Pasted image 20260927221752.png](images/Pasted%20image%2020260927221752.png)

Here we can see that attacker installed dnsmasq, we need PID now
![Pasted image 20260927221859.png](images/Pasted%20image%2020260927221859.png)

PID can be observed through volatility by enumerating process with pstree

> 🟩 **Answer:** dnsmasq,38687




**7. Clearly, the attacker's goal is to impersonate the entire network. One workstation was already tricked and got its new malicious network configuration. What is the workstation's hostname?**

![Pasted image 20260927223815.png](images/Pasted%20image%2020260927223815.png)

after observing inode number make directory and run vol.py with page cache inode pages
![Pasted image 20260927224232.png](images/Pasted%20image%2020260927224232.png)

after that I just read the dump file
![Pasted image 20260927224346.png](images/Pasted%20image%2020260927224346.png)


> 🟩 **Answer:** Parallax-5-WS-3




**8. After receiving the new malicious network configuration, the user accessed the City of CogWork-1 internal portal from this workstation. What is their username?**

using bull extractor we can obtain an answer, using this command
```
bulk_extractor -o extracted_data memdump.mem
```
![Pasted image 20261001173446.png](images/Pasted%20image%2020261001173446.png)

When we set the filter for **HTTP** traffic, we immediately see that the victim workstation (192.168.211.52) connected to a different server (10.129.232.25) on port 8081, which hosts the CogWork-1 People Portal.
We can see that the current user is Mike.sullivan

> 🟩 **Answer:** Mike.sullivan




**9. Finally, the user updated a software to the latest version, as suggested on the internal portal, and fell victim to a supply chain attack. From which Web endpoint was the update downloaded?**
The victim user browsed two pages on the CogWork-1 People Portal, profile.php and tasks.php . The second one contains a pending task that inform the user to update a software called AetherDesk.
![Pasted image 20261001173618.png](images/Pasted%20image%2020261001173618.png)

Currently, the attacker can intercept and redirect traffic wherever they want. Moreover, the Bash history indicates that the attacker spawned a second Docker container after installing Dnsmasq and interacting with iptables. By comparing the timestamp of the docker run command with the creation timestamp of the <container_id>-json.log files, we can examine the logs from the jm_proxy container. This container is based on the nginx image and was executed with a custom default.conf.
![Pasted image 20261001173702.png](images/Pasted%20image%2020261001173702.png)

Now let's dump it with linux.pagecache.InodePages plugin and review its content
![Pasted image 20261001173711.png](images/Pasted%20image%2020261001173711.png)


> 🟩 **Answer:** /win10/update/CogSoftware/AetherDesk-v74-77.exe




10. To perform this attack, the attacker redirected the original update domain to a malicious one. Identify the original domain and the final redirect IP address and port. (domain,IP:port)
![Pasted image 20260927231614.png](images/Pasted%20image%2020260927231614.png)


> 🟩 **Answer:** updates.cogwork-1.net,13.62.49.86:7477
