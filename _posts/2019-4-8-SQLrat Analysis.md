SQLrat is a clever piece of malware which is dropped onto the compromised machine using a malicious Word Document. The document contains macros written in Visual Basic which drop a number of files to disk which run malicious code and will also create scheduled tasks so the malware can persist on disk.
One file is obfuscated and uses SQL commands in order to connect to the attackers C2 infrastructure. 

By using SQL the attacker is able to leave minimal artefacts on disk in comparison to typical malware.

## Bank Document.doc analysis

``Bank Statement.doc``  
``MD5 - 94e1f07a34ad040bd1a00419dc7ba971``  

Word Document contains an image prompting the user to double click to open the document. This then launches VBA code.

![Bank Document.doc](/images/SQLrat/popup.PNG)

To analyse what the document is doing the code within the document needs to be analysed. The malware authors however password protect the files:

![Password](/images/SQLrat/password.png)

The password can be overwritten by putting the document into a hex editor and searching for where this password is stored. Searching for ‘dpb’ will identify where the encrypted password is located. By changing this value to ‘dpx’ the password can be bypassed:

![dpb](/images/SQLrat/dpx.png)

The string "4B49E7B92ED62ED6D12A2FD6EE17006F301EB3709E64BDD698D4C61FA3EA5BBD76C1F5A3EF" is the encrypted password.
Once the file is saved and the document is reopened a new password can be set and the code can now be viewed.

Within the code of the VBA the filenames of three documents are declared:

![vba](/images/SQLrat/vb1.PNG)

The above image shows the files being created but they currently have no content. Next three variables are declared which have been highlighted in the image below, these are used to generate content for the newly created empty files. The content of the files is hidden in the textframe of hidden shapes within the document.

![vba](/images/SQLrat/vb5.png)

The contents of the hidden textboxes can be found using SSView:

![ssview](/images/SQLrat/ssview2.png)

The location on disk for the files is then created ‘%APPDATA%\Microsoft\Templates’:

![vba](/images/SQLrat/vb2.PNG)

The following code shows scheduled tasks being created so the malware can remain persistent:

![vba](/images/SQLrat/vb3.png)

The windows tool ‘wscript.exe’ is setup so that it can be used to invoke JavaScript to launch the malware:

![vba](/images/SQLrat/vb4.png)

The scheduled tasks for ‘init.doc’ and ‘second.dot’ are given the following names:

![vba](/images/SQLrat/vb6.png)

When the malicious Word document is opened and the VBA code is launched the following files are dropped by the document, which matches the previous static analysis of the document:

``C:\Users\Admin\AppData\Roaming\Microsoft\Templates\dir.nfo``  
``MD5 - e2e80557fabc309e94103186edfde664``  

The file ‘dir.nfo’ file was created to store a unique victim string consisting of the victim MAC address along with the volume serial number.

``C:\Users\Admin\AppData\Roaming\Microsoft\Templates\init.dot``  
``MD5 - 07523fe39e54a679f20e06c8389dd03a``  

When ‘init.dot’ is put into a hex editor the JavaScript content is visible:

![init.dot](/images/SQLrat/init_hex.PNG)

Analysis of the JavaScript can be found later in the report.

``C:\Users\Admin\AppData\Roaming\Microsoft\Templates\mspromo.dot``  
``MD5 - 58df212985d08407e7a15bd543a774d3``  

This file contains obfuscated Unicode content: 

![vba](/images/SQLrat/mspromo_hex.png)

When ‘mspromo.dot’ is opened it forces Word to open the document Unicode which obfuscates the content:

![mspromo.dot](/images/SQLrat/mspromo_chinese.png)

``C:\Users\Admin\AppData\Roaming\Microsoft\Templates\second.dot``  
``MD5 - bddc874f7602d5876fe9c4559f61f924``  

## init.dot analysis

![init.dot](/images/SQLrat/shl.PNG)

Line 1 shows a new active object being created and it looks it is being assigned the name “ÿpshl”.
However, when the file is put in a hex editor we can see that the first two values ‘ÿp’ are FF FE in hexadecimal. This is telling Word to treat the characters within the document as Unicode rather than ASCII. These two values can be removed from the script being analysed.

![init.dot](/images/SQLrat/fffe.png)

The string “Wscript\2xeShell” is using hex values in place of ASCII characters to obfuscate the text. “\x2e” in hex is ‘.’ So the true string is ‘Wscript.Shell’.

![init.dot](/images/SQLrat/script1.png)

The script also references two string prototypes on line 3 and 10:

``•	String.prototype.ouPr``  
``•	String.prototype.ouCn``  

These are then referenced on line 18 with ‘eval’. ‘eval’ will execute the functions and will run ‘String.Prototype.ouPr’ first.

![init.dot](/images/SQLrat/ouPr.PNG)

Within this function on line 11, ‘OpenTextFile’ is called and hex characters are again being used to obfuscate the file name.
The hex values converted to ASCII:

•	x6d = m  
•	x73 = s  
•	x6f = o  
•	x2e = .  

The image below shows that the de-obfuscated file that is being opened is ‘mspromo.dot’, one of the files which has been dropped by the Word document. The values 1, 0, and -1 relate to the [‘OpenTextFile’](https://www.w3schools.com/asp/met_opentextfile.asp) function in JavaScript:

•	-1 – Opens the file as Unicode  
•	0 – Opens the file as ASCII  
•	1 – Opens file for reading  

The opening of the file is assigned the variable ‘tx’ in line 11 and in line 12 the file contents are read and stored in the variable ‘ret’. The document is then closed.

![init.dot](/images/SQLrat/ouPr_deob.PNG)

Contents of String.prototype.ouCn:

![init.dot](/images/SQLrat/ouCn.png)

The value of ‘ret’ on line 4 is blank, when the function returns it will contain the de-obfuscated contents from mspromo.dot.
In line 5 a ‘for’ loop is declared which is going to iterate over the content of the variable ‘ret’. ‘this.length’ refers to the length of the content stored in ‘ret’ which is the content of ‘mspromo.dot’.
So, it iterates over the obfuscated content and then invokes the string prototype functions to perform the de-obfuscation.

The code in line 6 is broken down as follows:

``‘(this.substr(i, 1)’`` – This is going to take each Unicode character from ‘mspromo.dot’ then apply the string prototype function ‘charCodeAt’.

``‘charCodeAt’`` – This function will take each Unicode character and return its ASCII value.
  
``‘toString(16)’`` – Takes the ASCII value and returns the hex value.

``‘slice(-2)`` – Removes the prepended byte 0x34 in order to de-obfuscate the text, yielding the plaintext hexadecimal byte value.

``‘parseInt’ & ‘16’`` – These values are wrapped around the ‘for’ loop and treats the output as hex instead of a string.

``‘String.fromCharCode’`` – Converts to plaintext ASCII.

In line 8 the output is then returned to the variable ‘ret’.

The variable ‘ret’ now contains the deobfuscated content of ‘mspromo.dot’. This contains the SQL queries the malware will use to contact the C2.

## mspromo.dot analysis

In a hex editor ‘mspromo.dot’ also contains the values FF FE in the header forcing Microsoft Word to open the file in Unicode. The malware author has used a method of prepending 0x34 to the plaintext characters so that this moves along the character table. The result is when the document is opened the text is displayed in another language.

![mspromo.dot](/images/SQLrat/mspromo_hex.png)

 The above image shows the additional inserted values of 34. 34 is 4 in ASCII so by removing the value ‘4’ from the output the obfuscated SQL commands implemented by SQLrat can now be identified:

![cyber chef](/images/SQLrat/cyberchef.PNG)

The SQL commands used by the malware to connect between the compromised device and the c2 server are now visible.

At the top of the script two objects are defined:

![mspromo.dot](/images/SQLrat/dbo_connections.PNG)

``ADODB.Connection`` – This is used to create an open connection to the attackers C2 server.  
``ADODB.Recordset`` – This is used to hold the data/records being collected by the SQL queries.  

The first function called is ‘InitD’ which in the image below references another file that was dropped by the initial Word document, ‘dir.nfo’:

![mspromo.dot](/images/SQLrat/mspromo_initd.PNG)

On line 20 ‘dir.nfo’ is referenced and the following ‘if’ statement is asking that if the file does not exist to call the function ‘GetSid’. This stores the SID of the user in the variable ‘msid’ and records this information to a text file declared as the variable ‘tx’
When ‘dir.nfo’ is opened in a hex editor it shows a MAC address of the compromised machine along with some additional data.

![mspromo.dot](/images/SQLrat/mac.png)

The function ‘GetSid’ besides collecting the MAC address it also collects part of the serial number of the machine which is the value ‘12FF’ in the above image. 
The image below shows how this information is captured. The script gathers the network adapter configuration on line 63 and stores this information in the variable ‘mac’. The string is then tidied up and stored in ‘mac_adress’.
On line 73 the script begins collecting evidence of the logical disk and enumerates the serial number of the device. This is then stored along with the MAC address in ‘dir.nfo’ as seen above,

![mspromo.dot](/images/SQLrat/mac_serial.png)

The C2 communication for the malware is setup via SQL commands on line 33:

![mspromo.dot](/images/SQLrat/c2.png)

## Process activity

![process](/images/SQLrat/proc.PNG)

The process ‘taskeng.exe’ is spawned by svchost to create the persistence on disk:

``PID: 4056, Command line: taskeng.exe {F63A67D1-54F9-46E5-A360-EEF18D1C0789} S-1-5-21-969390983-3881041502-3764434786-1000:WIN-5STSPIKEHRF\Admin:Interactive:LUA[1]``

wscript.exe launching ‘init.doc’:

``PID: 1760, Command line: wscript.exe /b /e:jscript "C:\Users\Admin\AppData\Roaming\Microsoft\Templates\init.dot"``

wscript.exe launching ‘second.dot’:

``PID: 3420, Command line: wscript.exe /b /e:jscript "C:\Users\Admin\AppData\Roaming\Microsoft\Templates\second.dot"``

## Persistence

![persistence](/images/SQLrat/persistence.png)

Two tasks created set to invoke ‘wscript.exe’ that will launch ‘init.doc’ and ‘second.dot’
The task names identified in the original VBA macro code are in use ‘Micriosoft update service’ and ‘System stability service’. 
These tasks are then set to run at 9:00 everyday, after this has been triggered they will then run every three minutes. The trigger expires on 01/02/2026 09:10:00.

## C2

31[.]18[.]219[.]133
