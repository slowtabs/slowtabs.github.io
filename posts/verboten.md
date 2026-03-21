---
title: Verboten

---

## Verboten

> Randon, an IT employee finds a USB on his desk after recess. Unable to contain his curiosity he decides to plug it in. Suddenly the computer goes haywire and before he knows it, some windows pops open and closes on its own. With no clue of what just happened, he tries seeking help from a colleague. Even after Richard's effort to remove the malware, Randon noticed that the malware persisted after his system restarted.

> NOTE: All timestamps are in IST.



---

I started with opening the `verboten.ad1` file. On some googling of the file format, I found that it can be opened using FTK imager. Therefore I opened it in FTK imager and extracted the files into a folder.


#### Q1) What is the serial number of the sandisk usb that he plugged into the system? And when did he plug it into the system?
`Format: verboten{serial_number:YYYY-MM-DD-HH-MM-SS}`

The serial number and stuff of USBs that were plugged in usually lie in the registry, but we dont have the registry file but we do have `System32`.

We can analyse `config/SYSTEM` using registry explorer. 

In the SYSTEM hive, USB details can be found in `ROOT>ControlSet001>USBSTOR`

![image](https://hackmd.io/_uploads/BkuEqxjwZx.png)

Found the serial number and the date and time. 

Answer: verboten{4C530001090312109353&0:2024-02-16-12-01-57}

#### Q2) What is the hash of the url from which the executable in the usb downloaded the malware from?
`Format: verboten{md5(url)}`


To find the URL, first thing that came to mind was browser history. Therefore, I went into `Users/randon/AppData/Local` and analysed the History database of Chrome. 
Since we need the link from which it was downloaded, we can find the URL in the downloads table.

![image](https://hackmd.io/_uploads/Hk9Zalivbe.png)
> MD5 that and we get our answer.

`Answer: verboten{11ecc1766b893aa2835f5e185147d1d2}`


#### Q3) What is the hash of the malware that the executable in the usb downloaded which persisted even after the efforts to remove the malware?
`Format: verboten{md5{malware_executable)}`

My antivirus kinda removed the file but it was to be found in the Startup folder, as they mentioned pesistant file.

Flag: verboten{169cbd05b7095f4dc9530f35a6980a79}


#### Q4) What is the hash of the zip file and the invite address of the remote desktop that was sent through slack?
`Format: verboten{md5(zip_file):invite_address}`


Since the question mentioned invite address for remote access, we probably need to find the code for remote access. 

I searched online on where to find local storage of Slack messages and I found this article: 
https://medium.com/@jeroenverhaeghe/forensics-finding-slack-chat-artifacts-d5eeffd31b9c

I found that the data is stored in IndexedDB/ and now I need to analyse it. I found [Slack Parser](https://github.com/0xHasanM/Slack-Parser) through the same article. By using that I found the messages stored on the Slack IndexedDB and opened it using Slack Parser and found the invite code in the messages.

![image](https://hackmd.io/_uploads/SJeWnGoDZe.png)

Now to find the zip. I ran `grep -rlP "\x50\x4B\x03\x04"` in the parent repository and found `f_0000ad` as a matching file.

![image](https://hackmd.io/_uploads/ryzDh7ovbe.png)

 I extracted the file and found the MD5.
 
 Answer: verboten{b092eb225b07e17ba8a70b755ba97050:1541069606}
 
 
#### Q5) What is the hash of all the files that were synced to Google Drive before it was shredded?
`Format: verboten{md5 of each file separated by ':'}`
 
 
 For this, I remembered seeing `DriveFS` in AppData/Local/Google. There was a folder with an arbitrary number as the name and on searching a little I found that the numbers identify a google account. I went into content_cache and found a file in each of the dx/d5x folders, which happened to be the files we required. I was a little curious so I ran `file` on each of them and found that they were three word documents and two jpeg images.
 
 ![image](https://hackmd.io/_uploads/rJ7J0Fm_Zx.png)

I calculated the md5 for each of them and got the flag!

![image](https://hackmd.io/_uploads/rkCx0YmObl.png)

Flag: verboten{ae679ca994f131ea139d42b507ecf457:4a47ee64b8d91be37a279aa370753ec9:870643eec523b3f33f6f4b4758b3d14c:c143b7a7b67d488c9f9945d98c934ac6:e6e6a0a39a4b298c2034fde4b3df302a}


#### Q6) What is time of the incoming connection on AnyDesk? And what is the ID of user from which the connection is requested?
`Format: verboten{YYYY-MM-DD-HH-MM-SS:user_id}`


I went into the AppData/Roaming/Anydesk directory for this one looking for any logs. I got 4 files, 3 of them didn't give me much. But ad.trace was interesting. I searched online and found: 

![image](https://hackmd.io/_uploads/S1EWe97OZg.png)

This is exactly what we need and it can be read in a text editor so I opened it in sublime text and searched for "incoming" and found a potential match on line 507. (zoom on the image). It had the time + the connection ID of the person who connected to the device.

![image](https://hackmd.io/_uploads/Hk0Hgcmdbx.png)

Using the details on that line I tried the submission and it was right :D

Flag: verboten{2024-02-16-20-29-04:221436813}


#### Q7) When was the shredder executed?
`Format: verboten{YYYY-MM-DD-HH-MM-SS}`


For this I went to prefetch, and found BLANKANDSECURE which was the only real one that stood out as a shredder name.

![image](https://hackmd.io/_uploads/rkQMSc7d-e.png)

The windows properties time stamp happened to be the wrong time stamp for some reason, maybe something related to local time. On using the time on FTK imager, I found the correct date and time, and also the answer.

![image](https://hackmd.io/_uploads/r1cdScmuZl.png)

Flag: verboten{2024-02-16-08-31-06}

#### Q8) What are the answers of the backup questions for resetting the windows password?
`Format: verboten{answer_1:answer_2:answer_3}`

This was pretty interesting as knowing where it was itself intrigued me. I searched online on where these are saved and found that they're in the SAM hive (can't be accessed on an active system ofc). 

I went into System32/config/SAM and opened it using RegistryExplorer. Under SAM/Domains/Account/Users/000003E8 I found the questions and also our required answers :D

![image](https://hackmd.io/_uploads/rJv2vcQOWx.png)

Flag: verboten{Stuart:FutureKidsSchool:Howard}

--

couldnt get: 

Q9) What is the single use code that he copied into the clipboard and when did he copy it?
Format: verboten{single_use_code:YYYY-MM-DD-HH-MM-SS}
