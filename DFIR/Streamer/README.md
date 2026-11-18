Sherlock Scenario

Simon Stark is a dev at forela who recently planned to stream some coding sessions with colleagues on which he received appreciation from CEO and other colleagues too. He unknowingly installed a well known streaming software which he found by google search and was one of the top URL being promoted by google ads. Unfortunately things took a wrong turn and a security incident took place. Analyze the triaged artifacts provided to find out what happened exactly.

Tasks:


**1. What's the original name of the malicious zip file which the user downloaded thinking it was a legit copy of the software?**

I transformed $**MFT** file to csv and imported it into **Timeline Explorer**. Then I filter results by.zip file and found an answer
![Pasted image 20260917165837.png](images/Pasted%20image%2020260917165837.png)


> 🟩 **Answer:** OBS-Studio-28.1.2-Full-Installer-x64.zip




**2. Simon Stark renamed the downloaded zip file to something else. What's the renamed Name of the file alongside the full path?**

Answer can be obtained from Timeline Explorer like in previous question
![Pasted image 20260917170243.png](images/Pasted%20image%2020260917170243.png)


> 🟩 **Answer:** C:\Users\Simon.stark\Documents\Streaming Software\Obs Streaming Software.zip




**3. What's the timestamp when the file was renamed?**
Answer can be obtained from Timeline Explorer like in last question
![Pasted image 20260917170943.png](images/Pasted%20image%2020260917170943.png)


> 🟩 **Answer:** 2023-05-05 10:22:23




**4. What's the Full URL from where the software was downloaded?**

Answer can be obtained from Zone Id Contents column 
![Pasted image 20260917171758.png](images/Pasted%20image%2020260917171758.png)

Our answer can be shown in ReferrerUrl field

> 🟩 **Answer:** **HTTP**://obsproicet.net/download/v28_23/OBS-Studio-28.1.2-Full-Installer-x64.zip




5. Dig down deeper and find the IP Address on which the malicious domain was being hosted.
Find **DNS** logs and trasnform it to csv file, then load it in Timeline Explorer and filter with domain from previous question
![Pasted image 20260917190736.png](images/Pasted%20image%2020260917190736.png)


> 🟩 **Answer:** 13.232.96.186




6. Multiple Source ports connected to communicate and download the malicious file from the malicious website. Answer the highest source port number from which the machine connected to the malicious website.
I opened firewall logs and filter with ipv6 address, I got few ports but this is highest number port
![Pasted image 20260917191123.png](images/Pasted%20image%2020260917191123.png)


> 🟩 **Answer:** 50045




7. The zip file had a malicious setup file in it which would install a piece of malware and a legit instance of OBS studio software so the user has no idea they got compromised. Find the hash of the setup file.
First I found amcache files and parse amcache entry files into csv. After that I opened csv file in Timeline Explorer and filter out with obs. I got this
![Pasted image 20260917193648.png](images/Pasted%20image%2020260917193648.png)

Paths of first 2 results are indicating that I am on the right path
![Pasted image 20260917193742.png](images/Pasted%20image%2020260917193742.png)

As we can see product name is telling us that 1 product is obs as Simon wanted to install and it is legit, but there is also second named web compilate which is suspicious. I took the SHA1 hash of the suspicious product name

> 🟩 **Answer:** 35e3582a9ed14f8a4bb81fd6aca3f0009c78a3a1




**8. The malicious software automatically installed a backdoor on the victim's workstation. What's the name and filepath of the backdoor?**
Since in question it mentions "backdoor" which usually means suspicious exe file, I filtered $MFT data csv file Created0x10  column and looked what happened after 2023-05-05 10:22:23 
![Pasted image 20260917200229.png](images/Pasted%20image%2020260917200229.png)

This was extremely sus so that was my final answer

> 🟩 **Answer:** C:\Users\Simon.stark\Miloyeki ker konoyogi\lat takewode libigax weloj jihi quimodo datex dob cijoyi mawiropo.exe




9. Find the prefetch hash of the backdoor.
I used this command to find file from previous question in prefetch folder
![Pasted image 20260917201951.png](images/Pasted%20image%2020260917201951.png)

Since files in prefetch folder on windows contains hashes in names, we can obtain an answer

> 🟩 **Answer:** D8A6D943




**10. The backdoor is also used as a persistence mechanism in a stealthy manner to blend in the environment. What's the name used for persistence mechanism to make it look legit?**
First I was looking for task that contains malicious .exe file
![Pasted image 20260917203219.png](images/Pasted%20image%2020260917203219.png)

There we can see that malicious file is using legitimate COMSurrogate task for persistence

> 🟩 **Answer:** COMSurrogate



**11. What's the bogus/invalid randomly named domain which the malware tried to reach?**

I opened dns logs and filtered to Event Id 3006 and looket around 10:20
![Pasted image 20260917205527.png](images/Pasted%20image%2020260917205527.png)


> 🟩 **Answer:** oaueeewy3pdy31g3kpqorpc4e.qopgwwytep




**12. The malware tried exfiltrating the data to a s3 bucket. What's the url of s3 bucket?**
In same place, I filter with s3 and obtained an answer

> 🟩 **Answer:** bbuseruploads.s3.amazonaws.com




**13. What topic was simon going to stream about in week 1?**


Find a note or something similar and recover its content to answer the question.
I found week 1 plan file
![Pasted image 20260917214749.png](images/Pasted%20image%2020260917214749.png)

then I obtained its Entry number
![Pasted image 20260917214822.png](images/Pasted%20image%2020260917214822.png)

I multiple it by 1024 and convert it to hexadecimal value 
![Pasted image 20260917215033.png](images/Pasted%20image%2020260917215033.png)

after it I loaded $MFT raw file in HxD and search for hexadecimal value above
![Pasted image 20260917215522.png](images/Pasted%20image%2020260917215522.png)

answer can be found in decoded text

> 🟩 **Answer:** Filesystem Security




**14. What's the name of Security Analyst who triaged the infected workstation?**

I converted NTUSER.DAT file from Simon.Stark to csv file with this cmd
```
.\RECmd.exe -f "C:\Users\Marin\Desktop\Streamer(1)\Streamer\Acquisition\C\Users\Simon.stark\NTUSER.DAT" --bn ".\BatchExamples\DFIRBatch.reb" --csv "C:\Users\Marin\Desktop\1"
```
then opened it in Timeline Explorer and filter with Triage
![Pasted image 20260917221502.png](images/Pasted%20image%2020260917221502.png)


> 🟩 **Answer:** CyberJunkie




**15. What's the network path from where acquisition tools were run?**
![Pasted image 20260917222039.png](images/Pasted%20image%2020260917222039.png)


> 🟩 **Answer:** \\DESKTOP-887GK2L\Users\CyberJunkie\Desktop\Forela-Triage-Workstation\Acquisiton and Triage tools
