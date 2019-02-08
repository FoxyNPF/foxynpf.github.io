I've taken a look at a couple of Emotet emails today and noticed they have tried to hide their list of C2's that are embedded within the document.

This post outlines how to get the C2's and also highlights a couple of cool tools I use when capturing command line output from malware.

http://www.kahusecurity.com/posts/cmd_watcher_updated.html - This is great tool for displaying any commands that are being run by a piece of malware and having that information recorded.

https://github.com/felixweyne/ProcessSpawnControl - This is a really useful tool in that it will suspend any newly created processes. The user then has the option to terminate the process or allow it to run.

**Email Subject:**

Your Amazon Order 184-1748378-0895603 

**Sender:**

vaalparkspar@telkomsa.net

**Attachment:**

Order 184-1748378-0895603.doc - MD5:17bc30d373d4fb1020fd5daae6e70e05

**Analysis:**

Microsoft Word opened and Macro’s enabled. This launches PowerShell which contains a bas364 encoded script.

Example of CMD Watcher capturing this information (Note I have unticked the “Kill Process” Boxes):

![CMD Watcher](/images/Emotet/cmd.png)

Same data captured in ProcessSpawnControl:

![Process Spawn Control](/images/Emotet/psc.png)

When base64 decoded the following output is displayed:

![base64 decode](/images/Emotet/decode.PNG)

The PowerShell commands are still obfuscated using ‘.’ (null bytes) to separate every character. 

Output with null bytes removed:

![null bytes removed](/images/Emotet/nullremove.PNG)

The URL’s within the script which are used to download the malware are also broken down and pieced together using ‘+’ i.e. http:'+'//'+'memtre'+'at'+'.com/TO'+'n9'+'K5'+'1QK1pJ2qI'+'_SKaebFAz.
Removing this obfuscation technique provides the C2’s that the PowerShell script uses to download the malicious payload:

``hxxp://memtreat[.]com/TOn9K51QK1pJ2qI_SKaebFAz/``

``hxxp://medongho[.]vn/SVm5yC0s``

``hxxp://otojack.co[.]id/wp-content/uploads/xvVQc2RzdDhTWswVa``

``hxxp://ptmmf.co[.]id/uNVMPELTQ_ldQ``

``hxxp://potlackariet[.]sk/bXfkJ2SeKd7g``

A this point the payload is downloaded to the following location and given a 3 digit name:

``PID: 2080, Command line: "C:\Users\Administrator\319.exe"``

Using ProcessSpawnControl I am able to capture 319.exe before it deletes itself.

![319.exe](/images/Emotet/cmd_spawn.png)

Once I have taken a copy of the binary I can allow the malware to continue running. This is such a useful feature where a piece of malware may drop multiple files and then delete some of them when it tries to clean itself up:

![319.exe suspended](/images/Emotet/319_allow.png)

319.exe launched:

``PID: 2992, Command line: "C:\Users\Administrator\319.exe"``

The file is then copied to its persistence location and deleted from “C:\Users\Administrator\”.

**Process Activity**

![Process Activity](/images/Emotet/process1.png)

![Process Activity](/images/Emotet/process2.png)

**Payload:**

slidemenus.exe - MD5: abac88c5e2220d78d2c835e138a6d78a

The same Word document opened on a 64bit machine downloaded a different payload:

issturned.exe - MD5: 477e59649a0a32de005b6947df42205a

**C2s:**

hxxp://memtreat[.]com/TOn9K51QK1pJ2qI_SKaebFAz/

hxxp://medongho[.]vn/SVm5yC0s

hxxp://otojack.co[.]id/wp-content/uploads/xvVQc2RzdDhTWswVa

hxxp://ptmmf.co[.]id/uNVMPELTQ_ldQ

hxxp://potlackariet[.]sk/bXfkJ2SeKd7g

**IP Addresses pulled from memory:**

0x398d80 (22): hxxp://98.157.215[.]153/

0x3b51ca (52): :tp://153.121.36[.]202:7080/

0x3bb82a (23): tp://71.240.202[.]13:443/

0x3bb86a (23): tp://70.164.196[.]211:20/

0x3d19b0 (14): 98.157.215.153

0x3d1c20 (13): 181.119.30.27

0x3d1c38 (14): 153.121.36.202

0x3d1c50 (14): 69.136.227.134

0x3d1c98 (14): 216.49.114.172

0x3d7a30 (44): %ttp://175.101.79.120/

0x3d7a6a (40): ttp://190.215.53.85/
