Sherlock Scenario

With help from D.I. Lestrade, Holmes acquires logs from a compromised MSP connected to the city’s financial core. The MSP’s AI helpdesk bot looks to have been manipulated into leaking remote access keys - an old trick of Moriarty’s.

Tasks:


**1. What was the IP address of the decommissioned machine used by the attacker to start a chat session with MSP-HELPDESK-AI?**

First, I unpacked zip file and inspect its content.
![Pasted image 20261029154821.png](images/Pasted%20image%2020261029154821.png)

I saw .pcapng file that can probably help for first question. After loading it in **Wireshark** I check for ip addresses that has the most connections. 
Statistics -->Conversations --> TCP
![Pasted image 20261029155138.png](images/Pasted%20image%2020261029155138.png)

We can see a lot of connections from 10.0.69.45 ip to port 1337.

> 🟩 **Answer:** 10.0.69.45




**2. What was the hostname of the decommissioned machine?**

In wireshark I filter out 10.0.69.45 ip address and answer appeared in first packet.
![Pasted image 20261029155914.png](images/Pasted%20image%2020261029155914.png)




**3. What was the first message the attacker sent to the AI chatbot?**

While inspecting packets I saw many communication between 10.32.43.31 ip address and chatbot ip address 10.128.0.3. By filtering it and inspecting I didn't see anything suspicious just legitimate talk between admin and chatbot.
![Pasted image 20261029162609.png](images/Pasted%20image%2020261029162609.png)

So I decided to exclude admins ip to see what else ip addresses was talking to chatbot.
![Pasted image 20261029163723.png](images/Pasted%20image%2020261029163723.png)

After going through packets I found first message.

> 🟩 **Answer:** Hello Old Friend




**4. When did the attacker's prompt injection attack make MSP-HELPDESK-AI leak remote management tool info?**

Here, WATSON was asking for RMM troubleshooting steps with username and password.
![Pasted image 20261029164948.png](images/Pasted%20image%2020261029164948.png)

Chatbot answered him:
![Pasted image 20261029165312.png](images/Pasted%20image%2020261029165312.png)

So timestamps of this chatbot answer is correct answer for this question because chatbot leaked credentials.
![Pasted image 20261029165443.png](images/Pasted%20image%2020261029165443.png)


> 🟩 **Answer:** 2025-08-19 12:02:06




**5. What is the Remote management tool Device ID and password?**
Following **HTTP** stream I found leaked creds
![Pasted image 20261029170002.png](images/Pasted%20image%2020261029170002.png)


> 🟩 **Answer:** 565963039:CogWork_Central_97&65




**6. What was the last message the attacker sent to MSP-HELPDESK-AI?**
I set the time to descending and inspect last packet.
![Pasted image 20261029170719.png](images/Pasted%20image%2020261029170719.png)


> 🟩 **Answer:** JM WILL BE BACK




**7. When did the attacker remotely access Cogwork Central Workstation?**
I found file containing incoming connections, just by looking at timestamps we know the answer is last connection because chatbot leaked creds just 1 day before that connection. It wasn't possible for WATSON to connect on 13th or 15th of August because he didn't have creds yet.

![Pasted image 20261029171410.png](images/Pasted%20image%2020261029171410.png)


> 🟩 **Answer:** 2025-08-20 09:58:25




**8. What was the RMM Account name used by the attacker?**
Name can be observed from picture from question 7.

> 🟩 **Answer:** James Moriarty




**9. What was the machine's internal IP address from which the attacker connected?**
In Teamviewer15_Logfile I saw that timestamp is in UTC +1. From question 7 we know that attacker connected Cogwork Central Workstation at 2025-08-20 09:58:25, which means we need to find something after 2025-08-20 10:58:25. 
![Pasted image 20261030162728.png](images/Pasted%20image%2020261030162728.png)


> 🟩 **Answer:** 192.168.69.213




**10. The attacker brought some tools to the compromised workstation to achieve its objectives. Under which path were these tools staged?**
In same file I search for keyword "C:" after 2025-08-20 10:58:36.
![Pasted image 20261030163558.png](images/Pasted%20image%2020261030163558.png)


> 🟩 **Answer:** C:\Windows\temp\safe\




**11. The attacker staged a browser credential harvesting tool on the compromised system. How long did this tool run before it was terminated?**

(Provide your answer in milliseconds, rounded to the nearest thousand)
After Kape processing, EZ parser made few files. In UserAssist shows for how long did GUI apps has been used.
![Pasted image 20261030164337.png](images/Pasted%20image%2020261030164337.png)


> 🟩 **Answer:** 8000




**12. The attacker executed a OS Credential dumping tool on the system. When was the tool executed?**
![Pasted image 20261030164443.png](images/Pasted%20image%2020261030164443.png)

In Teamviewer15_Logfile we can see that attacker downloaded mimikatz.exe, for correct timestamp the best way is to check USNJournal that EZ parser created already.
![Pasted image 20261030165043.png](images/Pasted%20image%2020261030165043.png)

In USNJournal csv file we can see that pf files which stands for executing the file. 

> 🟩 **Answer:** 2025-08-20 10:07:08




**13. The attacker exfiltrated multiple sensitive files. When did the exfiltration start?**

(UTC)
Going through Teamviewer15_Logfile I found the answer, since timestamp is UTC +1 I subtracted one hour.
![Pasted image 20261030165916.png](images/Pasted%20image%2020261030165916.png)


> 🟩 **Answer:** 2025-08-20 10:12:07




**14. Before exfiltration, several files were moved to the staged folder. When was the Heisen-9 facility backup database moved to the staged folder for exfiltration?**
First I found it in Teamviewer15_Logfile b searching for keyword "Heisen-9",
![Pasted image 20261030170351.png](images/Pasted%20image%2020261030170351.png)

then in USNJorunal csv file I searched for snapshot.kdbx
![Pasted image 20261030170741.png](images/Pasted%20image%2020261030170741.png)


> 🟩 **Answer:** 2025-08-20 10:11:09




**15. When did the attacker access and read a txt file, which was probably the output of one of the tools they brought, due to the naming convention of the file?**

By inspecting Kape Ez parser output we can see in 'FileFolderAcces' folder, AutomaticDestinations csv file that probably contains our answer.

![Pasted image 20261030171303.png](images/Pasted%20image%2020261030171303.png)

Before in Teamviewer15_Logfile we saw that attacker created "dump.txt" file which is probably txt file that question is asking for.
![Pasted image 20261030171602.png](images/Pasted%20image%2020261030171602.png)

having that in mind, opening AutomaticDestinations csv file in **Timeline Explorer** and searching for keyword dump.txt we can observe an answer.
![Pasted image 20261030172158.png](images/Pasted%20image%2020261030172158.png)


> 🟩 **Answer:** 2025-08-20 10:08:06




**16. The attacker created a persistence mechanism on the workstation. When was the persistence setup?**
I loaded cleaned SOFTWARE hive into **Registry Explorer**. I found that attacker abused JM.exe, by importing it in Userinit subkey JM.exe is executing every time the system boots up. 
![Pasted image 20261030173652.png](images/Pasted%20image%2020261030173652.png)


> 🟩 **Answer:** 2025-08-20 10:13:57




**17. What is the MITRE ID of the persistence subtechnique?**
![Pasted image 20261030174125.png](images/Pasted%20image%2020261030174125.png)


> 🟩 **Answer:** T1547.004




**18. When did the malicious RMM session end?**
In Connections_incoming.txt file we can see the asnwer
![Pasted image 20261030174605.png](images/Pasted%20image%2020261030174605.png)


> 🟩 **Answer:** 2025-08-20 10:14:27




**19. The attacker found a password from exfiltrated files, allowing him to move laterally further into CogWork-1 infrastructure. What are the credentials for Heisen-9-WS-6?**

I cracked the hash with this command. Password was "cutiepie14".
```
./keepass4brute.sh "../acquired file (critical).kdbx" /usr/share/wordlists/rockyou.txt
```
After that I logged in I copied password from Heisen-9-WS-6. User Name can be seen instantly.
![Pasted image 20261030182137.png](images/Pasted%20image%2020261030182137.png)


> 🟩 **Answer:** Werni:Quantum1!
