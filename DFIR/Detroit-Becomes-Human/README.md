Sherlock Scenario

Alonzo Spire is fascinated by AI after noticing the recent uptick in usage of AI tools to help aid in daily tasks. He came across a sponsored post on social media about an AI tool by Google. The post had a massive reach, and the Page which posted had 200k + followers. Without any second thought, he downloaded the tool provided via the Post. But after installing it he could not find the tool on his system which raised his suspicions. A DFIR analyst was notified of a possible incident on Forela's sysadmin machine. You are tasked to help the analyst in analysis to find the true source of this odd incident.

Tasks:


**1. What is the full link of a social media post which is part of the malware campaign, and was unknowingly opened by Alonzo spire?**

First, I analyzed kape file like this 
![Pasted image 20260915183606.png](images/Pasted%20image%2020260915183606.png)

we know that Alonzo saw a social media post and downloaded file from it and installed it. So we need to focus on web browsing, especially browsing history. 
![Pasted image 20260915183812.png](images/Pasted%20image%2020260915183812.png)

here I got default browsing history of Alonzo in last column, three samples above are old ones and can be useful if history is deleted or changed
I ran sqlite3 command to obtain URLs from main history 
```
sqlite3 -header -column \
"./Users/alonzo.spire/AppData/Local/Microsoft/Edge/User Data/Default/History" \
"SELECT id, url, title, visit_count FROM urls ORDER BY last_visit_time DESC;"
```
in output there was answer

> 🟩 **Answer:** https://www.facebook.com/AI.ultra.new/posts/pfbid0BqpxXypMtY5dWGy2GDfpRD4cQRppdNEC9SSa72FmPVKqik9iWNa2mRkpx9xziAS1l

Also it was possible to open file in DB Browser **SQLite** and find an answer



**2. Can you confirm the timestamp in UTC when alonzo visited this post?**

```
sqlite3 -header -column \
"./Users/alonzo.spire/AppData/Local/Microsoft/Edge/User Data/Default/History" \
"SELECT urls.id, urls.url, urls.title, visits.visit_time FROM urls JOIN visits ON urls.id = visits.url WHERE urls.id = 21;"
```
with this command retrieve all visited posts, since output doesn't show us visit time we need to do this command 
```
sqlite3 -header -column \
"./Users/alonzo.spire/AppData/Local/Microsoft/Edge/User Data/Default/History" \
"SELECT datetime(visit_time/1000000 - 11644473600, 'unixepoch') AS UTC, visit_time FROM visits WHERE url = 21 ORDER BY visit_time;"
```
![Pasted image 20260915185021.png](images/Pasted%20image%2020260915185021.png)


> 🟩 **Answer:** 2024-03-19 04:30:00




**3. Alonzo downloaded a file on the system thinking it was an AI Assistant tool. What is name of the archive file downloaded?**

I went to DB Browser SQLite and import history file
Then I went to browser data and set Table filter to downloads and got something like this
![Pasted image 20260915215631.png](images/Pasted%20image%2020260915215631.png)

in question it mentions AI Asisstant tool, we can see that file on the right side where Alonzo downloaded it

> 🟩 **Answer:** AI.Gemini Ultra For PC V1.0.1.rar




**4. What was the full direct url from where the file was downloaded?**

I went to Table: downloads but could not find source URL, so I went to ExecuteSQL and executed this: 
```
SELECT
    d.id,
    d.target_path,
    d.tab_url,
    d.referrer,
    duc.url
FROM downloads d
JOIN downloads_url_chains duc
ON d.id = duc.id;
```
to find all urls for all downloads

![Pasted image 20260915222005.png](images/Pasted%20image%2020260915222005.png)

The answer is clear from this output.

> 🟩 **Answer:** https://drive.usercontent.google.com/download?id=1z-SGnYJCPE0HA_Faz6N7mD5qf0E-A76H&export=download




**5. Alonzo then proceeded to install the newly download app, thinking that its a legit AI tool. What is the true product version which was installed?**

Since we know that Alonzo downloaded an archive related to Gemini AI, we will first try to search for the keyword Gemini in the Windows logs.

We use **Chainsaw**, a very powerful tool to search and hunt through Windows logs:
```powershell
.\chainsaw.exe --no-banner search -e Gemini "C:\Users\Marin\Desktop\Triage\C\Windows\System32\winevt\Logs\Application.evtx"
```
There are two events, one of which is shown above.
Both events come from the MsiInstaller provider.
The two events, one with ID 1040 and the other with ID 1042, are related to the installation of an installer named `Google AI Gemini Ultra For PC V1.0.1.msi`. The installer starts at `2024-03-19 04:31:02` and exits at `2024-03-19 04:31:20`.

We now try to get more information about this installer by filtering entries with the provider equal to `MsiInstaller` and a timestamp equal to or newer than `2024-03-19T04:31:02`.

We build the Chainsaw query with the following options:

- `--timestamp`: Specifies which field contains the timestamp of the events.
- `-t`: The Tau expression that filters events based on the specified provider.
- `--from`: The timestamp from which we want to start searching. We need to use the same timestamp format as shown in the log entry, so the `T` between the date and time is required.

We then run the following command:
```powershell
.\chainsaw.exe --no-banner search --timestamp 'Event.System.TimeCreated_attributes.SystemTime' -t 'Event.System.Provider_attributes.Name: MsiInstaller' --from '2024-03-19T04:31:02' "C:\Users\Marin\Desktop\Triage\C\Windows\System32\winevt\Logs\Application.evtx"
```
There are a total of 4 events:
- **1040** and **1042**, which we saw earlier.
- **11707** (`Product: ... -- Installation operation completed successfully.`): This event is logged after a product is successfully installed.
- **1033** (`Windows Installer installed the product.`): This event contains information about the installed product, such as its name and version.

Now we can finally get the actual product version by printing the event from the MsiInstaller provider with ID 1033:
```powershell
.\chainsaw.exe --no-banner search --timestamp 'Event.System.TimeCreated_attributes.SystemTime' -t 'Event.System.EventID: =1033' --from '2024-03-19T04:31:02' "C:\Users\Marin\Desktop\Triage\C\Windows\System32\winevt\Logs\Application.evtx"
```

> 🟩 **Answer:** 3.23.3




**6. When was the malicious product/package successfully installed on the system?**
![Pasted image 20260916001646.png](images/Pasted%20image%2020260916001646.png)


> 🟩 **Answer:** 2024-03-19 04:31:33




**7. The malware used a legitimate location to stage its file on the endpoint. Can you find out the Directory path of this location?**

To answer this question, we will focus our attention on the $**MFT** artifact. After
converting it to a CSV file, we open it with **Timeline Explorer**.
Then we apply a time filter to select only events with FileName creation timestamp
( Created0x30 ) newer than the first installation event of the MSI file.
After sorting the entries in ascending order, we can see right away that there are a
bunch of files that are located under the C:\Program Files (x86)\Google\Install
folder.
![Pasted image 20260916004806.png](images/Pasted%20image%2020260916004806.png)


> 🟩 **Answer:** C:\Program Files (x86)\Google




**8. The malware executed a command from a file. What is name of this file?**
![Pasted image 20260916205525.png](images/Pasted%20image%2020260916205525.png)

I opened $MFT csv file in Timeline Explorer and found few suspicious file names. install.cmd seems like the right one

> 🟩 **Answer:** install.cmd




**9. What are the contents of the file from question 8?**

Remove whitespace to avoid format issues.
We can confirm our previous assumption by looking at the contents of install.cmd using MFTECmd. From the Timeline Explorer we noted the Entry Number (51471) and the Sequence Number (4) for install.cmd , then we execute: .\MFTECmd.exe -f 'C:\Users\Marin\Desktop\Triage\C\$MFT' --de 51471-4
![Pasted image 20260916212745.png](images/Pasted%20image%2020260916212745.png)


> 🟩 **Answer:** @echooffpowershellExecutionPolicyBypassFile"%~dp0nmmhkkegccagdldgiimedpic/ru.ps1"




**10. What was the command executed from this file according to the logs?**
I executed this command and obtained an answer
```
.\chainsaw.exe --no-banner search -e 'ru.ps1' 'C:\Users\Marin\Desktop\Triage\C\Windows\System32\winevt\logs'
```

> 🟩 **Answer:** powershell -ExecutionPolicy Bypass -File C:\\Program Files (x86)\\Google\\Install\\nmmhkkegccagdldgiimedpic/ru.ps1




**11. Under malware staging Directory, a js file resides which is very small in size.What is the hex offset for this file on the filesystem?**
In Timeline Explorer I saw content.js file in same directory, I obtained it Entry Number and filter it with MFTECmd.exe
![Pasted image 20260916215436.png](images/Pasted%20image%2020260916215436.png)


> 🟩 **Answer:** 3E90C00




12. Recover the contents of this js file so we can forward this to our RE/MA team for further analysis and understanding of this infection chain. To sanitize the payload, remove whitespaces.
Answer can be obtained from output from last command from previous answer

> 🟩 **Answer:** varisContentScriptExecuted=localStorage.getItem('contentScriptExecuted');if(!isContentScriptExecuted){chrome.runtime.sendMessage({action:'executeFunction'},function(response){localStorage.setItem('contentScriptExecuted',true);});}




**13. Upon seeing no AI Assistant app being run, alonzo tried searching it from file explorer. What keywords did he use to search?**
I opened NTUSER.DAT file in **Registry Explorer** and search for WordWheelQuery hive. There I found the answer
![Pasted image 20260916220903.png](images/Pasted%20image%2020260916220903.png)


> 🟩 **Answer:** Google Ai Gemini tool




**14. When did alonzo searched it?**
It is shown in previous answer

> 🟩 **Answer:** 2024-03-19 04:32:11




**15. After alonzo could not find any AI tool on the system, he became suspicious, contacted the security team and deleted the downloaded file. When was the file deleted by alonzo?**
Since we know original file was .rar extension. I loaded csv file of $MFT and filter results by .rar and found an answer
![Pasted image 20260916223509.png](images/Pasted%20image%2020260916223509.png)


> 🟩 **Answer:** 2024-03-19 04:34:16




16. Looking back at the starting point of this infection, please find the md5 hash of the malicious installer.
![Pasted image 20260916225843.png](images/Pasted%20image%2020260916225843.png)


> 🟩 **Answer:** BF17D7F8DAC7DF58B37582CEC39E609D
