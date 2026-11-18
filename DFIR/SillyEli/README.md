Sherlock Scenario

Eli, a recent addition to the IT team, is eager to settle into his new role. Due to the company's flexible "Bring Your Own Device" (BYOD) policy, which applies to everyone except IT administrators, Eli receives a newly prepared PC from Metushelah, the senior IT administrator, as part of the IT team's secure device protocol. Eli's first priority is to install essential applications that will enable him to connect seamlessly with the rest of the team. He begins setting up the system and downloading the necessary software. After a few days on the job, Eli starts noticing something unusual: every so often, a PowerShell window flashes on his screen, disappearing almost instantly, too quickly to read any of its contents. Assuming it's just a minor glitch, Eli continues with his work. However, as the unexpected PowerShell pop-ups persist, Eli decides to consult Metushelah. Metushelah suspects these PowerShell flashes could indicate malicious activity. To ensure there are no security breaches, he conducts a forensic acquisition of Eli's PC and forwards it to the Incident Response team for further analysis. Now, as a member of the IR team, your mission is to investigate Eli's workstation.

Tasks: 


1. Identify the specific time the malware was downloaded in UTC.
I loaded everything into Autopsy and found an answer in "Web Downloads" sequence
![Pasted image 20261006155518.png](images/Pasted%20image%2020261006155518.png)

this one download was suspicious because URL was weird and abnormaly long, I converted 2024-11-06 02:57:54 CET to UTC time and it is correct answer

> 🟩 **Answer:** 2024-11-06 01:57:54




**2. Which domain name was the file downloaded from?**

I opened properties of sus executable and domain can be observed
![Pasted image 20261006155914.png](images/Pasted%20image%2020261006155914.png)


> 🟩 **Answer:** filesuneedtodownload.s3.eu-north-1.amazonaws.com




3. Identify the SHA-1 hash of the malware.
I tried to manually get hash from terminal but it didnt work so I found amcache.hve file and convert it to csv and opened it in TimelineExplorer
After that I search for malicious file found in Autopsy "TeamsSetup.exe"
![Pasted image 20261006171205.png](images/Pasted%20image%2020261006171205.png)


> 🟩 **Answer:** de733beed85f39605fcdea42dddee190506424f2




4. It looks like malicious tool attempts to download a secondary tool during execution — find and provide the full URL.
In the "Acquisition\C\ProgramData\Microsoft\Windows
Defender\Support\MPDetection-20241029-115405.log", I search for keyword "certutil" to find malicious command that malware executed 
![Pasted image 20261026141109.png](images/Pasted%20image%2020261026141109.png)


> 🟩 **Answer:** https://github.com/gentilkiwi/mimikatz/archive/refs/heads/master.zip





**5. What is the name of the batch file created on the system as part of the malware execution?**

I locate $J file, converted it to csv and open it in TimelineExplorer. Since the question asks for batch file it is most likely to contain .bat extension. In TimelineExplorer I search for keyword .bat and obtain result.
![Pasted image 20261026142458.png](images/Pasted%20image%2020261026142458.png)


> 🟩 **Answer:** TeamsUpdater.bat




**6. What was the runtime duration of the malware?**


(Duration in Ms)?
First thing I did is locate SRUDB.dat and create csv file so I can inspect it in TimelineExplorer
![Pasted image 20261026143649.png](images/Pasted%20image%2020261026143649.png)

I got bunch of files in "srudbdat-output" folder but the one containing "AppTimelineProvider_Output" is one that I am looking for
![Pasted image 20261026143923.png](images/Pasted%20image%2020261026143923.png)

I opened the file and search for "teamssetup.exe" keyword, answer can be seen under "Duration Ms" column
![Pasted image 20261026144223.png](images/Pasted%20image%2020261026144223.png)


> 🟩 **Answer:** 60003




7. Locate the local URL cache and identify the full URL the threat actor used to download an additional tool during the attack. Include the complete URL with filename and extension.
In task 4 I saw that malware used certutil.exe to download Mimikatz, but was blocked by the Defender. This suggests that attacker mabe used certutil again to fetch other tools.
I have ckecked Acquisition\C\Users\Eli\AppData\LocalLow\Microsoft\CryptnetUrlCache folder and found something interesting.

![Pasted image 20261027213412.png](images/Pasted%20image%2020261027213412.png)

Almost evenr file contains 1KB of size while this one contains 136,281 KB which is good trace. I found same hash name file in metadata folder and inspect what is inside. Result can be seen immediately.
![Pasted image 20261027213634.png](images/Pasted%20image%2020261027213634.png)


> 🟩 **Answer:** https://filebin.net/archive/0rkhisv2iq4slveo/kape.zip




**8. What is the name of the scheduled task that the attacker created for persistence?**

Since scheduled tasks and their metadata are stored in SOFTWARE hive under Schedule\TaskCache,
I loaded clean SOFTWARE registry hive (with transactions logs SOFTWARE.LOG1 and SOFTWARE.LOG2) in RegistryExplorer. I was looking under SOFTWARE_clean\Microsoft\Windows
NT\CurrentVersion\Schedule\TaskCache\Tree. TeamsUpdater immediately stands out since it is not part of any known legitimate application process. Also date/time "2024-11-06 01:58:15" perfectly match this case scenario. 
![Pasted image 20261026194110.png](images/Pasted%20image%2020261026194110.png)


> 🟩 **Answer:** TeamsUpdater




**9. What is the unique GUID associated with the malicious scheduled task?**
Answer can be obtained from opened registry hive from last question
![Pasted image 20261026195211.png](images/Pasted%20image%2020261026195211.png)


> 🟩 **Answer:** {A4F7ED3E-E7CD-4539-8E79-D73B568FC298}




10. Identify the process or command executed by the malicious scheduled task.
I was looking around usnjhournal, $**MFT** and Amcache.hve in TimelineExplorer to find something suspicious around 2024-11-06 01:58:15, since that is exact time when persistance mechanism occured. In $**MFT** I found "Teams.ps1" 1 second before persistance mechanism occured which was super suspicious.
![Pasted image 20261026200430.png](images/Pasted%20image%2020261026200430.png)

Then, in Autopsy I searched for "Teams.ps1" keyword and found result
![Pasted image 20261026200611.png](images/Pasted%20image%2020261026200611.png)

Also it was possible and maybe easier to use GUID obtained in question 9 ({A4F7ED3E-E7CD-4539-8E79-D73B568FC298}) and search it in RegistryExplorer, answer can be obtained by selecting "Actions" row
![Pasted image 20261026201749.png](images/Pasted%20image%2020261026201749.png)

Decoded text on right side is command that was executed by malicious scheduled task, or we can copy hex bytes in decoder like CyberChef and obtain result like this
![Pasted image 20261026202510.png](images/Pasted%20image%2020261026202510.png)


> 🟩 **Answer:** powershell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -File 

C:\Users\Eli\AppData\Local\Temp\Teams.ps1



11. Identify the attacker's IP address and port.
I wanted to inspect Teams.ps1 metadata so I observed entry number from $MFT csv file from **Timeline Explorer**
![Pasted image 20261026204607.png](images/Pasted%20image%2020261026204607.png)

I multiply entry number 315567 by 1024 and got 323140608, then I convert it to hexadecimal value like this
![Pasted image 20261026204745.png](images/Pasted%20image%2020261026204745.png)

then I import raw $MFT file in HxD and search 1342BC00 offset
![Pasted image 20261026205030.png](images/Pasted%20image%2020261026205030.png)

I got what I wanted but problem is I can't copy any of these decoded text, I didn't want to waste time so I decided to do it with MFTECmd.exe tool with --de flag and entry number 315567
![Pasted image 20261026210134.png](images/Pasted%20image%2020261026210134.png)

then I copied base64 string and decompressed it in one query
![Pasted image 20261026211124.png](images/Pasted%20image%2020261026211124.png)


> 🟩 **Answer:** 192.168.137.132:443



**12. What is the name of the sensitive file exfiltrated by the threat actor?**

I opened "Acquisition\C\Users\Eli\AppData\Roaming\Microsoft\Windows\PowerShell\PSR
eadLine\ConsoleHost_history.txt" file and obtained an answer
![Pasted image 20261027154928.png](images/Pasted%20image%2020261027154928.png)


> 🟩 **Answer:** Employees_Contact_Info.pdf




**13. Based on the contents of the document, what is Metushelah’s email address?**

By using sidr.exe I created 3 new csv files that contains metadata about indexed files.
![Pasted image 20261027214413.png](images/Pasted%20image%2020261027214413.png)

Opening ELI-PC_File_Report in Timeline Explorer and typing "pdf" in search we can find and answer in column.
![Pasted image 20261027215152.png](images/Pasted%20image%2020261027215152.png)


> 🟩 **Answer:** balikvar@laffa.com




**14. At what time did the attacker clear the Windows event logs?**


(in UTC)
I opened security log in Event Viewer and filter event ID 1102, which stands for clearing logs. 
I got just 1 event, in XML details answer can be observed.
![Pasted image 20261027215741.png](images/Pasted%20image%2020261027215741.png)


> 🟩 **Answer:** 2024-11-06 02:16:50




**15. What is the name of the legitimate tool used by the threat actor to acquire the SAM and SYSTEM hives?**
