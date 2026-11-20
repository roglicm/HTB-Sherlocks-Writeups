Sherlock Scenario

A major real estate and development corporation is under investigation for large-scale financial fraud. The company allegedly inflated revenues and concealed massive debt in order to secure bank loans and push through a stock exchange listing. Public reports painted a picture of strong growth, but the truth tells otherwise. Law enforcement has seized a corporate server believed to store critical documents. However, investigators suspect that the original financial statements reflecting the company’s real losses were deleted in an attempt to cover up the fraud. As a digital forensics specialist, your mission is to recover the missing file and bring the hidden truth to light.

Tasks: 


**1. When was the Windows Data Deduplication feature first installed on the system?**

I was given zip file that I extracted. After that I was provided with another .ad1 file that I analyzed and saw it is "ADSEGMENTEDFILE" which confirms that file is .ad1.

![Pasted image 20261110184023.png](images/Pasted%20image%2020261110184023.png)

By knowing that I decided to import it in FTK Imager because .ad1 files are readable format for FTK Imager. 

![Pasted image 20261110184530.png](images/Pasted%20image%2020261110184530.png)

There we can see FTK Imager successfully loaded .ad1 file and show its contents. Two partitions are recognized by FTK Imager. I exported files from both of them to new file called "stonks_export". After that I opened Autopsy and load in artifacts from "stonks_exports" folder.

![Pasted image 20261110185040.png](images/Pasted%20image%2020261110185040.png)

In Autopsy I searched for keyword "deduplication" and show results by "Created time" column descending. I inspected the oldest results.

![Pasted image 20261110200354.png](images/Pasted%20image%2020261110200354.png)

There it can be seen that deduplication is in "Microsoft-Windows-Deduplication%4Operational.evtx" log. So, I run gpake.exe and create csv files with "!EZParser" tool. I saved output in folder named "EZ_output".

![Pasted image 20261110201035.png](images/Pasted%20image%2020261110201035.png)

I opened EZ_output folder

![Pasted image 20261110201143.png](images/Pasted%20image%2020261110201143.png)

 and moved to event logs.

![Pasted image 20261110201209.png](images/Pasted%20image%2020261110201209.png)

There we can see "!EZParser" made one huge csv file (30,269kb) that contains all event logs. I opened it and search for "deduplication install" keyword.

![Pasted image 20261110201645.png](images/Pasted%20image%2020261110201645.png)

We can see in "Map Description" column that this event is the moment where deduplication feature was installed on the system. Since both of these 2 services are installed in range of 1 second we can be sure that this is the answer.

![Pasted image 20261110202405.png](images/Pasted%20image%2020261110202405.png)

> 🟩 **Answer:** 2025-09-18 09:56:40



**2. How many bytes of disk space were saved after the first optimization job was completed by Data Deduplication?**

In Timeline Explorer I search for saved space keyword and showed results by time ascending. 

![Pasted image 20261111032734.png](images/Pasted%20image%2020261111032734.png)

First one event occured after deduplication feature was installed on system, since timestamps are shown as time ascending that means that first event is the earliest one. In payload column of first event, answer can be obtained from "SavedSpace" field.

![Pasted image 20261111033013.png](images/Pasted%20image%2020261111033013.png)

> 🟩 **Answer:** 6805707



**3. When was Data Deduplication most recently re-enabled on volume D:?**

 For this question, I need to read data from sys vol info. 

![Pasted image 20261112191346.png](images/Pasted%20image%2020261112191346.png)

In settings folder there are .01.xml and .02.xml files. These files contains data about deduplications settings in xml format. Since "dedupConfig.01.xml" file contains data about older configuration state, we need to look inside "dedupConfig.02.xml" which contains data about later state created when deduplication was re-enabled or reinitialized. 

![Pasted image 20261113172724.png](images/Pasted%20image%2020261113172724.png)

Since timestamps in these xml files are in windows epoch format. By using converter we can convert this timestamp "134028128759740533" to UTC.
> 🟩 **Answer:** 2025-09-20 03:34:35



**4. List all file extensions that are not optimized by Data Deduplication in alphabetical order.**

In same "dedupConfig.02.xml" file there are 2 fields "excludeFileExtensions" and "excludeFileExtensionsDefault". By collecting extensions from those 2 fields and putting it in alphabetical order we can get an answer.

![Pasted image 20261113171840.png](images/Pasted%20image%2020261113171840.png)

> 🟩 **Answer:** avi, bak, edb, iso, jrs, mp4, tif



**5. How many files have been optimized by Data Deduplication?**

In "System Volume Information/Dedup/ChunkStore/State" folder there is file called "dedupStatistics.xml". By examining it, answer can be found under "optimizedFileCount" field.

![Pasted image 20261113180232.png](images/Pasted%20image%2020261113180232.png)

> 🟩 **Answer:** 168



**6. When is the next Throughput Optimisation job scheduled to run?**

For this question I need "C:\Windows\System32\Tasks\Microsoft\Windows\Deduplication" folder to examine throughput optimization xml file. I exported all files from "Deduplication" folder for easier analyzing.

![Pasted image 20261114160202.png](images/Pasted%20image%2020261114160202.png)

I got few files, but the one needed to answer this question is "ThroughputOptimization".

![Pasted image 20261114160306.png](images/Pasted%20image%2020261114160306.png)

I opened it in notebook.

![Pasted image 20261114160459.png](images/Pasted%20image%2020261114160459.png)

There are few important fields that explains how scheduled job is being executed. "StartBoundary" field is not necessary timestamp when scheduled job is executed, it is just saying that scheduled job is "ready to be executed" when other parameters require it. The actual execution time is determined by the other scheduling parameters, such as "DaysOfWeek" and "WeeksInterval". Since parameters "DaysOfWeek" and "WeeksInterval" says that task is configured to run every Thursday once a week. Therefore, its first scheduled execution is on September 25, 2025, at 06:50:00.
> 🟩 **Answer:** 2025-09-25 06:50:00



**7. What is the name of the financial report that was recently deleted in an attempt to cover up criminal activity?**

In "root\Finance\Reports\Annual\2024" folder there is one deleted file which confirms "X" mark over it. 

![Pasted image 20261114163851.png](images/Pasted%20image%2020261114163851.png)

I opened USN Journal $J csv file in Timeline Explorer to confirm this. 

![Pasted image 20261114164040.png](images/Pasted%20image%2020261114164040.png)

> 🟩 **Answer:** Internal Consolidated Financial Statements 2024.docx



**8. What is the MFT entry number of that report file?**

I searched in csv $MFT file for keyword "Internal Consolidated Financial Statements 2024.docx" and found an answer.

![Pasted image 20261114223719.png](images/Pasted%20image%2020261114223719.png)

> 🟩 **Answer:** 46



**9. List all the FILE_ATTRIBUTE flags that are set on the report file. Submit answer in ascending order based on the hex value of each flag.**

For this question I need combining entry number, raw $MFT file and MFTECmdx.exe with "--de". Since exporting 2 disk partitions from FTK Imager was not successful, because I obtain these 2 folders without any subfolders, they are empty.

![Pasted image 20261118113552.png](images/Pasted%20image%2020261118113552.png)

I decided to find $MFT through powershell. 
First of all, I run gci alias and recursively parse through current "stonks_export" folder.

![Pasted image 20261118114249.png](images/Pasted%20image%2020261118114249.png)

I found 2 different ones, one for each disk partition. One needed to answer this question is second one (from disk partition 3). Because that is where "Internal Consolidated Financial Statements 2024.docx" is located. Also entry number "46" that I will use for this question is collected from csv file that was created with kape (!EZParser) from raw $MFT file that is located on disk partition 3. 
By knowing this info, I can successfully parse report file through entry number.

![Pasted image 20261118115618.png](images/Pasted%20image%2020261118115618.png)

I obtain 3 flags Archive, SparseFile and ReparsePoint. Their hexadecimal values are Archive = 0x20, SparseFile = 0x200, ReparsePoint = 0x400. Putting it in ascending order gives us an correct answer.
> 🟩 **Answer:** FILE_ATTRIBUTE_ARCHIVE, FILE_ATTRIBUTE_SPARSE_FILE, FILE_ATTRIBUTE_REPARSE_POINT



**10. At what offset in the $MFT file does the $REPARSE_POINT attribute of the report begin?
(Answer as an 8-digit hexadecimal offset, prefixed with 0x).**

For this question I tried extracting raw $MFT (to import it in HxD) from disk partition 3 but unfortunately wasn't able to to it. It is possible to calculate it from output from previous question.

![Pasted image 20261118122056.png](images/Pasted%20image%2020261118122056.png)

To answer this question, it is needed to add up these fields. Also MFT record header is 0x38 bytes which is standard offset inside NTFS MFT record. By knowing this I need to add up '0x38 + 0x60 + 0xC8 + 0x50'.
I will do with calculator in HEX mode. It is also possible to do it through powershell. 

![Pasted image 20261118122844.png](images/Pasted%20image%2020261118122844.png)

So size of standard info, file name and data is '1B0' in hex value. Now it is needed to add up MFT record absolute offset which is 0xB800 (it can be seen from first picture in this question). 

![Pasted image 20261118123318.png](images/Pasted%20image%2020261118123318.png)

> 🟩 **Answer:** 0x0000B9B0



**11. After the report was optimized by Data Deduplication, into how many chunks was its data divided?**

For this question I need to use DedupInspector. I already built in past DedupExplorer.exe through Visual Studio. Dedup Explorer requires raw $MFT file, stream file and chunk file for full recovery of original files and scanning.
I obtain raw $MFT like this.

![Pasted image 20261118133102.png](images/Pasted%20image%2020261118133102.png)

Stream file is .ccc file in 'Stream' folder in sys vol info. I exported it into folder named 'chunkstore_stream'.

![Pasted image 20261118133332.png](images/Pasted%20image%2020261118133332.png)

And lastly chunk file is also .ccc file in 'Data' folder in sys vol info. I exported it into folder named 'chunkstore_data'.

![Pasted image 20261118133546.png](images/Pasted%20image%2020261118133546.png)

Now I have all files required for recovering original 'Internal Consolidated Financial Statements 2024.docx'. I startedDedupExplorer.exe and imported obtained files from previous steps. 

![Pasted image 20261118133924.png](images/Pasted%20image%2020261118133924.png)

Now it is ready for scanning.

![Pasted image 20261118134114.png](images/Pasted%20image%2020261118134114.png)

After scanning, our file can be seen at the top of the list. Answer can be found under 'Chunks' column. 
> 🟩 **Answer:** 2



**12. What is the data length in bytes of the first deduplicated chunk belonging to the report file under investigation?**

For this question I need to open .ccc file from 'chunkstore_stream' folder into HxD and calculate bytes for first chunk. Since exported .ccc file in 'chunkstore_stream' contains data about every deduplicated chunk I need to identify its stream-header first. I opened raw $MFT in HxD and search for 'B9B0' offset beacause that is where reparse point begins (obtained in Q10). 

![Pasted image 20261118145500.png](images/Pasted%20image%2020261118145500.png)

Now I need to locate stream-header identifier. 

![Pasted image 20261118145827.png](images/Pasted%20image%2020261118145827.png)

This means I need to add up start of reparse point and 0x78 which is relative offset that marks the start of the 30-byte stream-header identifier in .ccc stream files. 

![Pasted image 20261118150023.png](images/Pasted%20image%2020261118150023.png)

Now I search for 'BA28' in opened raw $MFT in HxD to find stream-header.

![Pasted image 20261118150250.png](images/Pasted%20image%2020261118150250.png)

After searching for 'BA28' offset I observed 'F0 97 3C BB 3C 2B 95 86' string. 
Now I imported .ccc file from 'chunkstore_stream' folder into HxD and search for 'F0 97 3C BB 3C 2B 95 86' string in Hex-values.

![Pasted image 20261118150859.png](images/Pasted%20image%2020261118150859.png)

After searching:

![Pasted image 20261118151210.png](images/Pasted%20image%2020261118151210.png)

Offset '0x5260' is where our file starts. Since it is known that '0xA8' is the relative offset where the length field is located inside 'Ckhr' record, I need to add up '0x5260' and '0xA8'.

![Pasted image 20261118152123.png](images/Pasted%20image%2020261118152123.png)

Now I know that length of the file is located on '0x5308' offset in HxD. 

![Pasted image 20261118152407.png](images/Pasted%20image%2020261118152407.png)

After searching for '0x5308' in HxD I got these results. Since windows stores this value in little-endian order I need to reverse it. That gives offset '0xEEA7'. I will use hex to binary converter.

![Pasted image 20261118153032.png](images/Pasted%20image%2020261118153032.png)

> 🟩 **Answer:** 61095



**13. After recovering the file, provide the SHA-256 hash of the recovered financial report.**

I used Dedup Explorer for this question. 

![Pasted image 20261118153423.png](images/Pasted%20image%2020261118153423.png)

I recover 'Internal Consolidated Financial Statements 2024.docx' file like this.
In powershell I run 'Get-FileHash' and observed an answer.

![Pasted image 20261118153714.png](images/Pasted%20image%2020261118153714.png)

> 🟩 **Answer:** e51e773e5404f29cc2816cff3fbdcc8b2c28f9e8040a6e7fe926b21336b66513



**14. The person who drafted this report must also be held legally accountable. What is their Full name?**

For this one, I examined contents of the file but found nothing useful. So I decided to create zip file and inspect document metadata to find 'Author/Creator'. 

![Pasted image 20261118235201.png](images/Pasted%20image%2020261118235201.png)

DocProps is the one that will most likely contains data about author/creator.

![Pasted image 20261118235256.png](images/Pasted%20image%2020261118235256.png)

In 'DocProps' folder 2 files can be seen. Core.xml is most likely one that will contain author/creator info so I opened it. 

![Pasted image 20261118235448.png](images/Pasted%20image%2020261118235448.png)

> 🟩 **Answer:** Elaine Chua



**15. How much was the profit for the year overstated in the audited report compared to the actual one?**

For this question I run Dedup Explorer and recover 'Audited Consolidated Financial Statements 2024.docx' like this.

![Pasted image 20261119000726.png](images/Pasted%20image%2020261119000726.png)

I saved it in folder named 'report_recovered'. Now I have both files recovered and can start examining. 
Audited annual income (Audited Consolidated Financial Statements 2024.docx):

![Pasted image 20261119000901.png](images/Pasted%20image%2020261119000901.png)

Internal (actual) annual income (Internal Consolidated Financial Statements 2024.docx):

![Pasted image 20261119001210.png](images/Pasted%20image%2020261119001210.png)

The audited report showed profit for the year of $2,672,196,000, while the actual report showed a loss of $1,647,555,000. Adding up those two numbers is correct answer.

![Pasted image 20261119001530.png](images/Pasted%20image%2020261119001530.png)

> 🟩 **Answer:** 4319751
