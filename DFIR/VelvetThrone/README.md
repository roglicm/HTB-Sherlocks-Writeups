Axiom Financial Partners, a private equity firm managing $4.2 billion in assets, has suffered a significant breach. The incident response team has been engaged after anomalous activity was detected across multiple systems. Initial triage suggests that an external threat actor gained initial access through an identity provider compromise and subsequently deployed custom malware, performed credential theft, and exfiltrated highly sensitive merger and acquisition documents totaling nearly 25GB.

The evidence package contains logs from Okta, Windows Security auditing, Sysmon, the corporate proxy, Azure AD, **DNS** resolver, network flow data, Windows Defender, SharePoint audit logs, email gateway logs, malware static analysis output, raw binary files from the compromised host, and a captured encrypted C2 session.

Your task is to reconstruct the full attack chain, identify all attacker infrastructure, reverse the custom encryption used for C2 communications, and decode the covert channel hidden in DNS traffic.

Tasks: 


**1. What is the username of the account compromised via MFA push bombing?**

I ran this command to extract MFA challenge events
```
jq -r '.[] | select(.eventType == "user.mfa.challenge") | [
  .published,
  .actor.login,
  .actor.displayName,
  .outcome.result,
  (.outcome.reason // "-"),
  .client.ipAddress,
  .client.geographicalContext.city,
  .client.geographicalContext.country,
  .client.userAgent.rawUserAgent,
  .target[0].displayName
] | @tsv' okta_system_log.json |
column -t -s $'\t'
```
output evidence: Between 08:10:00Z and 08:21:55Z, Okta records 10 MFA push challenges for Kyle Morrison

> 🟩 **Answer:** kyle.morrison




**2. What is the attacker's source IP address used for the initial MFA bombing and subsequent VPN login?**

I made okta timeline file for kyle morrison with this command
```
jq -r '.[] | select(
  (.actor.login // "" | ascii_downcase) == "kyle.morrison@axiomfp.com"
) | [
  .published,
  .eventType,
  (.outcome.result // "-"),
  (.outcome.reason // "-"),
  (.client.ipAddress // "-"),
  (.client.geographicalContext.city // "-"),
  (.client.geographicalContext.country // "-"),
  (.target[0].displayName // "-")
] | @tsv' okta_system_log.json |
sort > kyle_okta_timeline.txt
```
output:
![Pasted image 20260926190655.png](images/Pasted%20image%2020260926190655.png)


> 🟩 **Answer:** 45.33.32.156


**3. What is the filename of the backdoor/implant dropped on the compromised workstation?**
While doing answer 6. I found answer for question 3.

![Pasted image 20260927180921.png](images/Pasted%20image%2020260927180921.png)

This event ID is 4967 what determines installing new service, we can conclude an answer

> 🟩 **Answer:** ProxyHealth.exe



**4. What symmetric encryption algorithm does the malware use for C2 communications?**

![Pasted image 20260926170753.png](images/Pasted%20image%2020260926170753.png)

In pentest_report_json.log I found **MITRE ATT&CK** technique (T1573.001) that usually uses algorithms AES, DES, 3DES, Blowfish, and **RC4**, since in next line we can see that pentesters used **RC4** algorithm we can say that its high probability that attackers will use the same. To be sure I ran this command:
![Pasted image 20260926174630.png](images/Pasted%20image%2020260926174630.png)

There we can clearly see that rc4 is used as algorithm 

> 🟩 **Answer:** RC4




**5. What is the full C2 domain the implant beacons to?**

I checked sysmon events, windows security and defender alert logs with this command
```
grep -Ein -C 10 "wmic|logicaldisk|volumeserialnumber|deviceid|GetComputerName|hostname" \
```
![Pasted image 20260926180340.png](images/Pasted%20image%2020260926180340.png)


> 🟩 **Answer:** proxy-health-api.azurecloud-monitor.com


**6. Which persistence technique was used to ensure the implant remained active on the system?**


(MITRE ATT&CK TECHNIQUE ID)
![Pasted image 20260926183546.png](images/Pasted%20image%2020260926183546.png)

This event ID is 4967 which means installing new service, probably for malware execution
![Pasted image 20260926183758.png](images/Pasted%20image%2020260926183758.png)


> 🟩 **Answer:** T1543.003




**7. Which legitimate Microsoft tool did the attacker abuse to dump the **LSASS** process memory for credential theft?**
I used this command to filter out possible tools used by attacker 
![Pasted image 20260926181917.png](images/Pasted%20image%2020260926181917.png)

from output we can see "CommandLine":"**ProcDump**.exe -ma lsass.exe C:\\Windows\\Temp\\lsass.dmp" which confirms that **ProcDump** tool was used to run lsass process

> 🟩 **Answer:** ProcDump




**8. What service account credentials were harvested from the LSASS memory dump?**


(username only)
Since we know from scenario that attacker after lsass dumping use SharePoint service, also from output we can se that svc_sharepoint_farm is in 3 different logs (Windows security log, Azure AD log and over 300 events in sharepoint audit log) and name fits SharePoint service acc, we can conclude an answer
![Pasted image 20260926182713.png](images/Pasted%20image%2020260926182713.png)


> 🟩 **Answer:** svc_sharepoint_farm




**9. What is the NetBIOS hostname of the initially compromised machine?**
Since I know filename of backdoor, I filter windows security and sysmon logs with it
![Pasted image 20260926184300.png](images/Pasted%20image%2020260926184300.png)


> 🟩 **Answer:** EXEC-WKSTN-01




**10. What is the volume serial number of the C: drive on the compromised machine, as used in the RC4 key derivation?**

Malware strings shows that backdoor starts: wmic logicaldisk get volumeserialnumber,deviceid, so its good to filter out sysmon logs for WMIC i volume serial with this command
```
jq -c '.[] | select(
  [
    (.Image // ""),
    (.CommandLine // ""),
    (.ParentImage // ""),
    (.ParentCommandLine // "")
  ]
  | join(" ")
  | test("wmic|logicaldisk|volumeserialnumber|deviceid"; "i")
)' sysmon_events.json > volume_serial_hits.json
```
then we list the results
![Pasted image 20260926191341.png](images/Pasted%20image%2020260926191341.png)


> 🟩 **Answer:** 4B7A2C9E




**11. What internal IP address did the attacker pivot to after obtaining the SharePoint service account credentials?**
This output was obtained while doing task 14
![Pasted image 20260926193508.png](images/Pasted%20image%2020260926193508.png)


> 🟩 **Answer:** 10.10.20.30




**12. Decrypt the captured C2 beacon (c2_session_capture.bin) using the derived RC4 key. What command was issued to the implant?**
Since static strings do not provide a separator/formula, I made small script in AI that automatically tests real combinations: hostname + serial, serial + hostname, with/without separator and with different casing
![Pasted image 20260926192119.png](images/Pasted%20image%2020260926192119.png)

We know we are on right path because in sherlock scenario it says that attacker exfiltrated about 25GB of data, from this output we can see same amount (24999591936 bytes)

> 🟩 **Answer:** COMPRESS_AND_STAGE




**13. What is the filename of the archive created to stage the exfiltrated data?**
from same output like in previous question:
![Pasted image 20260926192305.png](images/Pasted%20image%2020260926192305.png)


> 🟩 **Answer:** axiom_q1_portfolio.7z




**14. What is the full domain name the attacker used as the exfiltration endpoint to upload the staged archive?**

![Pasted image 20260926193109.png](images/Pasted%20image%2020260926193109.png)


> 🟩 **Answer:** blob-sync-backup.s3-azure-cdn.com




**15. Decode the covert message hidden in DNS subdomain queries found in the DNS resolver log. What is the full decoded string?**

Malware strings gave us XOR key "covert_dns_xor_key=0x17", I ran this command to get all c2 DNS queries
```
jq -r '.[] | select(
  (.query_name | endswith("azurecloud-monitor.com"))
) | [
  .timestamp,
  .client_ip,
  .query_name,
  .query_type,
  .response_code
] | @tsv' dns_resolver_log.json |
sort > azurecloud_dns.txt
```
then I ran 
```
cat azurecloud_dns.txt
```
most important part of output:
![Pasted image 20260926194618.png](images/Pasted%20image%2020260926194618.png)

first column is DNS fragment which can be decoded using XOR key like this:
![Pasted image 20260926195007.png](images/Pasted%20image%2020260926195007.png)

we can also use XOR Calculator or python

> 🟩 **Answer:** UNC3944_AXIOM_PWNED
