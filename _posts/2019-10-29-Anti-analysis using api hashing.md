Malware authors are always using different tricks and techniques to try and stop malware analysts from analysing their malware. One common technique a malware analyst will do is take a look at the Import Address Table (IAT) once they have unpacked sample and see if the IAT gives any clues as to how the malware may behave.

So what is the IAT? The IAT is a list if Windows functions that the malware will import, so if it needs to create a new process it can use the Windows API CreateProcessInternalW. This means that rather than write a piece of code to create a new process, the malware author can import this function, use it and then return to their malicious code.

Windows contains various libraries known as DLL’s (Dynamic Link Library) which store these functions, or API calls as they are also known. Seeing a DLL such as crypt32.dll in a malware sample’s IAT may mean it is going to perform some sort of encryption routine. So based on this finding, a malware analyst may set a breakpoint on this API call to try and understand what information is being encrypted or decrypted.

This is obviously bad for malware authors so how do they get around this? One technique is rather than populate the IAT once the malware is unpacked, they obfuscate or hash the names of the Windows API and then deobfuscate and load them on the fly. This method of dynamically loading the API’s for when they are needed means that an unpacked malware sample will not have as much data in the IAT.

To help illustrate this I have a sample of Lokibot malware which uses this technique, this post will attempt to explain how the malware dynamically loads the API calls.

The image below is the IAT from a sample I unpacked and it only contains a small number of functions, apart from the highlighted network functions that I had to google this doesn’t really give me much information on how the malware may behave.

![IAT](/images/api_hashing/imports.png)

The loading of the API’s is done by storing a large number of preconfigured strings which are hashed names of legitimate Windows API’s in the malware code. The malware then loads the correct DLL where this function is stored and iterates through each API name within the DLL. The name of each API is then hashed and checked against the preconfigured hashed API value, when it eventually populates a hashed string that matches it is then deobfuscated and called via the EAX register.

To help explain this I will go through this routine with an example using the API value StrStrW. I have also renamed some of the malware’s functions to help illustrate what is happening.

The image below shows a hashed function name being pushed onto the stack – D6865BD4, along with the value ‘2’. The value ‘2’ relates to an array containing a list of DLL names, each indexed value in the array is assigned a numerical value beginning at 2. So if the malware needs to load shlwapi.dll the index value 2 is pushed onto the stack, if crypt.dll is needed then the numerical value ‘3’ is pushed onto the stack and so on.

These values are then passed to the function Deobfuscate_Hash where the hash is decrypted and the return value, StrStrW, is then stored in the EAX register. The function StrStrW is then called indirectly via the instruction ‘call EAX’.

![StrStrW](/images/api_hashing/strstrw.png)

The actual deobfuscation of the API name is done in sub_4031E5 (Deobfuscate_Hash) so let’s take a closer look into the contents of this function.

The same values are stored in a couple of local variables, passed to the stack and a new function sub_4030A5 is then called:

![x32dbg](/images/api_hashing/indexhash.png)

The two parameters passed to the stack are the value ‘2’ and the hashed function name ‘D6865BD4’:

![x32dbg](/images/api_hashing/4030A5_stack.png)

The next function called sub_402CA4 (Index_and_Hash), is again passed the value ‘2’ and contains a number of stack strings. Stack strings take individual pieces of data and they are eventually stacked together to form a larger string.

For example:

``push N``  
``push E``  
``push I``  
``push L``  

The above individual strings can then be put together to form a larger string, ‘NEIL’.

In this part of the deobfuscation routine the malware is pushing a number of individual hex values onto the stack, when converted to ASCII and put together create the names of the various DLL’s the malware will need to load during execution.

To help illustrate this the below image is a snippet of the function in x32dbg. The first 3 commands I have highlighted show the function prologue creating a reasonably large amount of space on the stack with the command ‘sub esp, 188’.  Further down the code you can then see a number of hex values being pushed onto the stack – 73, 68, 6C...

These hex values are then converted to ASCII and then concatenated to create the names of the DLL’s the malware will later need to call.

![x32dbg](/images/api_hashing/402CA4_stackstrings.png)

Once the stack strings have been created the array of the DLL names can then be identified.

As I mentioned earlier the array begins at index value ‘2’ so shlwapi.dll is the DLL that is loaded in this example, this is why the number ‘2’ was pushed onto the stack along with the hashed API as this is the DLL that contains StrStrW. If the malware needed to load the DLL crypt32.dll then ‘3’ would have been pushed onto the stack as this is the next index in the array.

Output of stack strings:

![x32dbg](/images/api_hashing/enumerate_dll.PNG)

Once shlwapi.dll has been identified from the array it is then passed as a parameter to 4032E1. This just contains a jump instruction to the next function ‘402C1F’, in this function we can now see another hash being pushed onto the stack – E811E8D4.

![x32dbg](/images/api_hashing/loadlibraryw.png)

E811E8D4 is the hashed value for LoadLibraryW, I will explain the relevance of this next.

Note that this time though the value ‘0’ is pushed onto the stack, this is not a valid value for the indexed DLL’s in our array which I mentioned starts at ‘2’. Instead this will branch the code off to load kernel32.dll.

Parameters from x32dbg:

![x32dbg](/images/api_hashing/loadlibparams.png)

This is then passed to sub_4031E5 (Deobfuscate_Hash), this is the decryption routine that our original hash was passed to before.

At this point the malware now has a list of DLL’s it wants to load and also what API calls it needs as this is all hardcoded by the bad guy. However how does it find, locate and import these functions from the machine it has infected? Remember these DLL’s are not part of the malicious code, they are legitimate Windows functions that are being imported.

In order to load the API functions that the malware will use it needs to locate and load the DLL kernel32. This is because kernel32 contains the API ‘LoadLibraryW’, which you can probably gather from the name is used to load the libraries the malware needs. This is why the hash of ‘LoadLibraryW’ and ‘0’ were pushed onto the stack to load kernel32.

The hash value for kernel32 is then passed to a new function sub_40317B. This function is where the malware locates the PEB.

![x32dbg](/images/api_hashing/kernel32hash.png)

So what is the PEB and what is going on now?

In order to load kernel32 the malware will need to locate it in memory on the infected machine, this is done by locating the Process Environment Block (PEB). The PEB contains information about the currently running processes including the list of DLL’s that have been loaded or mapped into the processes memory. The FS register contains the address of the data structure called the ‘Thread Information Block’ (TIB) and a pointer to the PEB can be found in the TIB at the offset value of 0x30. Based on this information a pointer to the PEB can always be found at FS:[30].

![x32dbg](/images/api_hashing/peblocated.png)

The previous image shows the location of the PEB being moved into EAX and any evidence of this code in a piece of malware should tip you off as to what the malware is doing.

Once located the malware then iterates through the name of each process it has identified from the PEB and then hashes the name of the process executable. Once a process name has been hashed its value is then compared to ‘F96AF9CE’ to see if it matches. If it doesn’t match the malware knows It hasn’t located kernel32 and moves to the next process, the next process has its name loaded from memory and then hashed. When the output of this hashing routine matches ‘F96AF9CE’ the malware knows it has found kernel32.

The below image should hopefully illustrate the methodology of locating kernel32.

![x32dbg](/images/api_hashing/kernel32location.png)

The above image shows the following:

1.	PEB located  
2.	The function <Get_Process_Name> enumerates the filepath of the first process  
3.	sub_4032EA takes the string and makes sure all characters are lowercase  
4.	Lower case process executable name is enumerated  
5.	Process name is then hashed using sub_402C38  

The hashing routine takes each letter from the filename string and stores the value in EAX.
The EDX register is used as a counter and then the first letter is XOR’d with the contents of ECX – FFFFFFFF. A 2nd loop is then instigated which iterates through 8 times and XOR’s the contents of ECX with 4358AD54. Once the 2nd loop has finished the next character from filename is loaded into EAX and the loop is repeated. This is done for each letter of the filename, when the hash routine is finished the final hash value is moved from ECX into EAX and checked to see if it matches the kernel32 hash.

![x32dbg](/images/api_hashing/hashroutine.png)

The below image shows the malware successfully locating kernel32.

![x32dbg](/images/api_hashing/kernel32.png)

The next function called is 4030C4 which is used to find LoadLibraryW in kernel32. To do this the malware iterates through each API in kernel32 and runs the same hashing routine as before. Each API name is hashed and checked against the value E811E8D4, once it has a match the deobfuscated value (LoadLibraryW) is stored in EAX. The string shlwapi.dll is then passed as a parameter to LoadLibraryW and the function is called via EAX. shlwapi.dll has now finally been loaded.

![x32dbg](/images/api_hashing/loadlibraryloaded.png)

The same process is then completed again, each API within shlwapi.dll is then enumerated, hashed and checked if it matches what the malware expects to find for StrStrW - D6865BD4. Once identified the API can finally be used by a call to EAX which takes us back to where we began:

![x32dbg](/images/api_hashing/strstrwfinal.png)

**TLDR;**

1.	Malware has preconfigured list of hashes, these hashes are obfuscated API calls which are loaded dynamically and deobfuscated on the fly by the malware.
2.	This example outlines how the malware loads StrStrW, the hashed name of this API is D6865BD4.
3.	The hash and a numerical value are pushed onto the stack and passed to sub_4031E5 (Deobfuscate_Hash) which is the function that deobfuscates the hashed API names.
4.	The numerical value pushed onto the stack is an index number which relates to an array. This array will contain a list of preconfigured DLL names. The index values in this array will start at ‘2’ which is shlwapi.dll.  
5.	sub_4030A5 (Index_and_Hash) passes the values to local variables which are then pushed onto the stack.  
6.	sub_402CA4 (DLL_Stack_Array) contains a number of stack strings which are the names of several DLL’s the malware will use. The index value from before is used to load the correct DLL.  
7.	Identified DLL passed to function 4032E1 (Jump), this contains a jump instruction to 402C1F.  
8.	402C1F pushes the hash for LoadLibraryW onto the stack and the name of the DLL to be loaded from the array.  
9.	Function sub_4031E5 (Deobfuscate_Hash) is then called again as LoadLibraryW needs to be deobfuscated. The hash for LoadLibraryW is pushed onto the stack.  
10.	sub_4030A5 (Index_and_Array) is called again, and ‘0’ pushed onto the stack.  
11.	sub_402CA4 (DLL_Stack_Array) called, this is where the malware loads a DLL from a preconfigured list. However as ‘0’ has been pushed to the stack and is not a valid index value from the DLL array the code branches and pushes the hash for kernel32 onto the stack - F96AF9CE.  
12.	40317B (PEB_Located) called and malware locates the PEB. This is so the malware can locate kernel32 on the victim machine. The reason being kernel32 contains LoadLibraryW which will give the malware the capability to load the API calls it needs.  
13.	List of current running processes enumerated from victim machine.  
14.	The filename of each running process is enumerated and then hashed using the function 402C38 (Hashing_Routine)  
15.	Once the filename from a running process has been hashed it is then compared to the value F96AF9CE. This is the hashed name of kernel32, if it has a match it knows it has located the kernel32.  
16.	Function 4030C4 then iterates through each API in kernel32.dll, the name of each function is hashed and checked against the value E811E8D4. This is the hashed value of LoadLibraryW.  
17.	LoadLibraryW is called via EAX and the value shlwapi.dll is passed as a parameter. Shlwapi.dll is now loaded.  
18.	Function 4030C4 is then called again and the malware iterates through each API in shlwapi.dll, the name of each function is hashed and checked against the value D6865BD4. This is the hashed value of StrStrW.  
19.	Deobfuscated API is finally loaded into EAX. Function can now be used by the malware by calling EAX.  


