Sherlock Scenario

You recently received an email from someone trying to impersonate a company, your job is to analyze the email to see if it is suspicious.

**Email:** C:\Users\LetsDefend\Desktop\Files\BusinessEmail.zip  
**Password:** infected  

**Attachment:** C:\Users\LetsDefend\Desktop\Files\united_scientific_equipent.zip  
**Password:** infected

Tasks:


**1. What is the sending email address?**

In "C:\Users\LetsDefend\Desktop\Files" folder I was provided with 2 .zip files, "united_scientific_equipent.zip" and "BusinessEmail.zip". After unzipping it with provided password "infected" I got few new files.
![Pasted image 20261031213056.png](images/Pasted%20image%2020261031213056.png)

For 1st question I opened "BusinessEmail.eml" file in Thunderbird and found an answer.
![Pasted image 20261031213225.png](images/Pasted%20image%2020261031213225.png)


> 🟩 **Answer:** yanting@united.com.sg




**2. What is the email address of the recipient?**

Answer can be observed from same picture like in question 1.
![Pasted image 20261031213349.png](images/Pasted%20image%2020261031213349.png)


> 🟩 **Answer:** admin@malware-traffic-analysis.net




**3. What is the subject line of the email?**
From same picture like in question 1 and 2
![Pasted image 20261031213534.png](images/Pasted%20image%2020261031213534.png)


> 🟩 **Answer:** united scientific equipment




**4. What date was the Email sent?**


Date format: MM/DD/YYYY
By inspecting source of file in Thunderbird app, answer can be observed.
![Pasted image 20261031214400.png](images/Pasted%20image%2020261031214400.png)


> 🟩 **Answer:** 02/08/2021




**5. What is the originating IP?**

From same picture like in question 4, answer can be observed.
![Pasted image 20261031214602.png](images/Pasted%20image%2020261031214602.png)


> 🟩 **Answer:** 71.19.248.52




**6. What country is the ip address from?**
By importing 71.19.248.52 ip address in **VirusTotal** answer can be seen.
![Pasted image 20261031214812.png](images/Pasted%20image%2020261031214812.png)


> 🟩 **Answer:** Canada




**7. What is the name of the attachment when you unzip it?**

(with extension)
Answer can be observed from picture from question 1. 
![Pasted image 20261031215016.png](images/Pasted%20image%2020261031215016.png)


> 🟩 **Answer:** united scientific equipent.exe




**8. What is the sha256 hash of the File?**

![Pasted image 20261031215537.png](images/Pasted%20image%2020261031215537.png)

Because clipboard is not working on RDP machine I got the hash on pwn box like this.
![Pasted image 20261031220406.png](images/Pasted%20image%2020261031220406.png)


> 🟩 **Answer:** 9909753bfb0ac8ab165bab3555233d03b01a9274a92e57c022f87ccbe51ca415




**9. Is the email attachment malicious?**

Yes/No
By importing hash from previous question in VirusTotal, we can see that out exe is most likely trojan.
![Pasted image 20261031220617.png](images/Pasted%20image%2020261031220617.png)

![Pasted image 20261031220636.png](images/Pasted%20image%2020261031220636.png)


> 🟩 **Answer:** Yes
