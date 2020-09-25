I recently did a deep dive analysis of Emotet and thought I would share the analysis I have done. I havent spent too much time on the macros/PowerShell used to download the malware as there are already plenty of resources available that have that covered.
Using x32dbg I have broken down how the malware creates the seemingly random filenames for the malware, enumerates and encrypts the running processes, how the malware sets up it's C2 connectivity and also how to extract the config.

**MD5 of analysed sample:**  
``B2489E53BBB1C8B29E821601A5CFD907``

## Attack Vector

Example email containing malicious Word document.

![Emotet](/images/Emotet/email.png)

Word document with embedded macros, clicking ‘Enable Content’ will launch the macro content.

![Emotet](/images/Emotet/worddoc.PNG)

Macros launch encoded PowerShell command to download payload from list of compromised websites. The URL’s are often obfuscated using base64 and are relatively easy to decipher:

![Emotet](/images/Emotet/cyberchef.PNG)

Payload being downloaded from compromised website and subsequent call to attacker C2:

![Emotet](/images/Emotet/processtree.png)

Process tree listing. This shows PowerShell being used to download the malware to the User directory. Original filename is 215.exe, this is then copied to the malwares persistence location and renamed:

![Emotet](/images/Emotet/download.PNG)

## Unpacking the Malware

The malware uses a common technique of process hollowing to unpack itself in memory. Extracting the unpacked binary can be done by setting a breakpoint on VirtualAlloc in a debugger such as x32dbg. In this example the unpacked malware was then stored in a buffer [edi+54] at location 001d0115.
The header was prepended with some junk code, once the unpacked binary has been dumped this needs removing using a tool such as HXD to create a clean PE header.

![Emotet](/images/Emotet/unpacked.png)

Location of unpacked binary in memory map:

![Emotet](/images/Emotet/unpackedmemorymap.PNG)

## Hashed Functions

Statically analysing the unpacked malware shows a single import as being present - IsProcessorFeaturePresent. This is because the malware functions are hashed and each library and its associated API’s are loaded dynamically when needed. The first two function calls in this sample contain the hashes for ntdll and kernel32.

![Emotet](/images/Emotet/hashedfunctions2.png)

Once the functions sub_92ACEC9 and sub_92BE17 have been called the deobfuscated API calls then become visible in x32dbg:

![Emotet](/images/Emotet/hashedfunctions3.PNG)

A closer look at these two functions shows how the malware dynamically loads its API calls. The first function resolves ntdll.dll and its associated API calls. This is done by moving the hash values into global variables and the following parameters being pushed onto the stack.

The hash value D22E2014 is moved into ECX, this is the hash value of ntdll.dll.

![Emotet](/images/Emotet/hashparams.PNG)

A call is then made to a function which traverses the PEB. The PEB contains information about the currently running processes including the list of DLL’s that have been loaded or mapped into the process memory. The FS register contains the address of the data structure called the ‘Thread Information Block’ (TIB) and a pointer to the PEB can be found in the TIB at the offset value of 0x30. Based on this information a pointer to the PEB can always be found at FS:[30].

D22E2014 is moved to the EBX register and a loop iterates through the name of each running process that has been identified from the TIB. The loop checks that each character of the process name is a lowercase character and the hashing routine is performed. Once completed the hashed value is stored in EAX and compared to EBX. When the two values match the malware knows it has found the process it was looking for in memory, in this case ntdll.

![Emotet](/images/Emotet/peb.PNG)

The next function then performs the same process, however the hash value is different as the malware is locating kernel32. The below image shows the hash has been generated and stored in EAX, this is then compared to the hash value in EBX. The values match so the malware has successfully located kernel32.

![Emotet](/images/Emotet/kernel32located.PNG)

Once the DLL has been identified the malware needs to locate the API’s it wants to use.

Once the DLL has been identified the malware then reads the Export Address Table of the DLL and hashes each name and compares it to the hash values it has stored to check for matching hashes. Once a match has been found it has the location of the API calls it wants to use such as GetWindowsDirectoryW.

## Mutex Creation

Malware identifies root directory of filesystem by calling GetWindowsDirectoryW. 

![Emotet](/images/Emotet/getwindowsdirectory.PNG)

Directory C:\\Windows identified and then a call is made to GetVolumeInformationW. 

![Emotet](/images/Emotet/getvolumeinformationw.PNG)

This API call retrieves information about the file system and the Windows root directory. The above image shows the fouth parameter which is pushed onto the stack will retrieve the volume serial number.

Serial number of virtual machine that analysis was conducted on:

![Emotet](/images/Emotet/serial.PNG)

The returned value can be seen in the below image, however this is reversed due to endianess.

![Emotet](/images/Emotet/volumeserialnumber.PNG)

The malware then begins to create a mutex using this data. First a call is made to snwprintf which writes formatted data to a string, in the below image the format that will be used is ‘Global\\I%X’, where ‘%X’ will be the volume information.

![Emotet](/images/Emotet/createmutexw.PNG)

Once the format of the mutex name has been set, a call to CreateMutexW is then called. The newly created mutex, highlighted in green, is stored in the EAX register.

A second mutex is then created, however this one is formatted with ‘M%X’ instead of ‘I%X’.

![Emotet](/images/Emotet/createmutexw2.PNG)

The next function then calls CreateEventW using the format ‘E%X’ in conjunction with the serial number.

![Emotet](/images/Emotet/createeventw.PNG)

## File Naming

The malware makes a call to GetModuleFileNameW, this returns the current location of where the malware is running from. In the example below the malware has been manually unpacked and is running from the desktop:

![Emotet](/images/Emotet/getmodulefilenamew.PNG)

The malware decrypts a lists of strings that will be used to create a random filename for the malware.

![Emotet](/images/Emotet/decryptionroutine.PNG)

Decrypted strings:

![Emotet](/images/Emotet/strings.PNG)

``"engine,finish,magnify,resapi,query,skip,wubi,svcs,router,crypto,backup,hans,xcl,con,edition,wide,
loada,themes,syc,pink,tran,khmer,chx,excel,foot,wce,allow,play,publish,fwdr,prep,mspterm,nop,define,
chore,shlp,maker,proc,cap,top,tablet,sizes,without,pen,dasmrc,move,cmp,rebrand,pixel,after,sms,minimum,
umx,cpls,tangent,resw,class,colors,generic,license,mferror,kds,keydef,cable"``

Once the strings have been decrypted the malware then begins the process of generating a name using a combination of two strings.

A call is made to lstrlenw to calculate the length of the combined strings. This returns the hex value 177 to the EAX register and is then moved into ECX.

![Emotet](/images/Emotet/stringlength.PNG)

When 177 is converted from hex to decimal, the string length is 375 characters which can be seen in the screenshot below.

![Emotet](/images/Emotet/sublime.PNG)

The mov command highlighted below shows data being moved into EAX, this is the volume serial number of the infected machine that was captured earlier for the mutex creation – EAA53FEC.

The hex value EA A5 3F EC taken from the serial number is then divided by the string length of 375, the remainder of this equation is stored in EDX.

![Emotet](/images/Emotet/divecx.PNG)

The value in EDX is the hex value 62 which is 98 in decimal. The relevance of 98 is that the malware has now identified the first part of the filename by moving 98 places along the list of strings. The image below shows that is has landed in the string ‘loada’.

![Emotet](/images/Emotet/sublime2.PNG)

The malware then performs some checks to make sure it captures the entire string and the first part of the filename is enumerated.

The process is then repeated to get the second string which will make up the final part of the filename. In the previous routine a ‘NOT EAX’ command was called after ‘DIV ECX’, this will reverse the bits in EAX. This ensures that the string used for the second part of the filename will differ to the first. After this routine completes the second string identified is ‘tangent’.

![Emotet](/images/Emotet/stringlength.PNG)

Next the malware prepares the location of where the malware will persist. To do this a call is made to SHGetFolderPathW, the second parameter passed to this API call is the value 1C.

Microsoft’s documentation states that this parameter is the CSIDL value. A CSIDL value identifies the folder whose path is to be retrieved. The value 1C corresponds to CSIDL_LOCAL_APPDATA which relates to the filesystem location AppData\Local.

Once the folder location has been retrieved the malware then begins to setup the persistence location. The below image shows where the string loadatagent will be used as directory name and executable name. This is used in conjunction with the AppData location identified from the call to SHGetFolderPathW.

![Emotet](/images/Emotet/shgetfolderpathw.PNG)

Malware creating persistence location in filesystem with call to CreateDirectoryW:

![Emotet](/images/Emotet/createdirectoryw.PNG)

New process then created:

![Emotet](/images/Emotet/createprocess.PNG)

## System Info & Process Enumeration

The malware captures information about the currently running operating system and also information on the architecture and processor by calling RtlGetVersion and GetNativeSystemInfo..

![Emotet](/images/Emotet/rtlversion.PNG)

The PEB is then located, this allows the malware to enumerate all running processes from memory. The below image shows the PEB location being moved into EAX.

![Emotet](/images/Emotet/peb2.PNG)

A call is then made to CreateToolHelp32Snapshot, the value 2 is pushed onto the stack which relates to then value ‘TH32CS_SNAPPROCESS’. This means that all processes will be included in the snapshot.

![Emotet](/images/Emotet/createtoolhelp32snapshot.PNG)

A call is then made to Process32FirstW, this retrieves information about the first process encountered in a system snapshot. Each process is enumerated and stored in memory:

![Emotet](/images/Emotet/processlist.PNG)

## Encryption

Once the malware has captured information such as the hostname and a list of running processes. A unique identifier is created for the infected machine using the hostname and volume serial number.

![Emotet](/images/Emotet/serialprocess.PNG)

This information is the compressed using the deflate algorithm:

![Emotet](/images/Emotet/deflatecompression.PNG)

A call to CryptGenKey creates an AES128 session key, this is used to encrypt the compressed data. A hash value is also generated by calling CryptCreateHash. 

![Emotet](/images/Emotet/cryptgenkey.PNG)

Location of AES Key:

![Emotet](/images/Emotet/key.PNG)

Location of hash, this is stored below the list of plaintext process names:

![Emotet](/images/Emotet/hash.PNG)

Call made to CryptEncrypt which is passed the above key and hash. The data stored in ebp-4 highlighted below contains the deflate compressed data identified earlier.

![Emotet](/images/Emotet/cryptencrypt.PNG)

The image below shows the data is now encrypted, this is highlighted in red. The data highlighted in green in is the hash value.

![Emotet](/images/Emotet/encryptedandhash.PNG)

The AES key is then exported with a call to CryptExportKey which returns a key BLOB. The key BLOB is a secure way of sending sessionkeys over the internet and can then imported on the receiving side of the internet connection for decryption.

The image below shows the keys and BLOB type being pushed onto the stack. The parameter CRYPT_OAEP specifies that RSA encryption will be used when exporting the key BLOB.

![Emotet](/images/Emotet/cryptexportkey.PNG)

The RSA encrypted session key, hash value and encrypted data is then Base64 encoded and ready to be sent to the attacker.

![Emotet](/images/Emotet/base64location.PNG)

## C2 Setup

Next the malware begins to setup connectivity to the C2 infrastructure, this process begins with a routine that decrypts a string that is the formatting of an IP address:

![Emotet](/images/Emotet/c2format.PNG)

First C2 then loaded:

![Emotet](/images/Emotet/c2.PNG)

A list of strings is then generated which will be used to make up the full URL path.

``"teapot,pnp,tpt,splash,site,codec,health,balloon,cab,odbc,badge,dma,psec,cookies,iplk,devices,
enable,mult,prov,vermont,attrib,schema,iab,chunk,publish,prep,srvc,sess,ringin,nsip,stubs,img,add,
xian,jit,free,pdf,loadan,arizona,tlb,forced,results,symbols,report,guids,taskbar,child,cone,glitch,
entries,between,bml,usbccid,sym,enabled,merge,window,scripts,raster,acquire,json,rtm,walk,ban"``

Before these strings are implemented the formatting of the URL is setup. The below image shows where two strings will be used to make up the URL, this will be the IP address and a string from the list that was generated:

![Emotet](/images/Emotet/referersetup.PNG)

Once the formatting has been implemented the strings are populated:

![Emotet](/images/Emotet/referergenerated.PNG)

The malware then retrieves the User-Agent HTTP request header string that will be used.

![Emotet](/images/Emotet/useragent.PNG)

Internet connectivity is then setup with calls to InternetOpenW, InternetConnectW, HttpOpenRequestW and HttpSendRequestW.

![Emotet](/images/Emotet/httpopenrequestw.PNG)

![Emotet](/images/Emotet/httpsendrequestw.PNG)

The lpOptional field highlighted above in HttpSendRequestW contains a handle to the data that is going to be sent to the C2. This is base64 encoded data that was identified earlier in this report.

## Config Extraction

The malware config that contains the IP addresses and port numbers of the attacker infrastructure is stored in plain text in the unpacked binary. From the analysis conducted the IP address 190[.]117[.]206[.]153 has been identified as belonging to the attackers C2 infrastructure. This address can be located in the binary by converting the numerical values of the IP address to hex values and searching for the hex pattern in the memory map. Note when searching for the IP addresses reverse the order of IP address due to endianness.

Example:

![Emotet](/images/Emotet/hexpattern.PNG)

By right clicking on this pattern and following in the dump the malware config can be identified:

![Emotet](/images/Emotet/configextract.PNG)

In the above image the first four hex values contain the IP address and the following two values are the port to be used. Following this data in the memory map shows that the config is stored in the data section of the malware.

![Emotet](/images/Emotet/datasection.PNG)

This section can now be dumped and the C2's can easily be extracted using the following python script written by CyberCDH:

https://github.com/cybercdh/hacks/tree/master/emotet

C2:

190[.]117[.]206[.]153:443  
203[.]99[.]187[.]137:443  
200[.]55[.]168[.]82:20  
70[.]32[.]94[.]58:8080  
213[.]138[.]100[.]98:8080  
144[.]76[.]62[.]10:8080  
203[.]99[.]188[.]203:990  
201[.]196[.]15[.]79:990  
203[.]99[.]182[.]135:443  
176[.]58[.]93[.]123:80  
192[.]241[.]220[.]183:8080  
94[.]177[.]253[.]126:80  
181[.]47[.]235[.]26:993  
216[.]75[.]37[.]196:8080  
95[.]216[.]207[.]86:7080  
78[.]109[.]34[.]178:443  
113[.]52[.]135[.]33:7080  
216[.]70[.]88[.]55:8080  
138[.]197[.]140[.]163:8080  
181[.]113[.]229[.]139:990  
83[.]169[.]33[.]157:8080  
212[.]112[.]113[.]235:80  
143[.]95[.]101[.]72:8080  
190[.]13[.]146[.]47:443  
178[.]249[.]187[.]150:7080  
157[.]7[.]164[.]178:8081  
5[.]189[.]148[.]98:8080  
51[.]38[.]134[.]203:8080  
93[.]78[.]205[.]196:443  
91[.]109[.]5[.]28:8080  
173[.]249[.]157[.]58:8080  

## Persistence

Malware copied to the following location:

``C:\Windows\SysWOW64\loadatangent.exe``
