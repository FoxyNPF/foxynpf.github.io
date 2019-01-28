Here is some analysis I completed on an email I came across that contained a malicious RTF file that dropped Lokibot malware.

**Email Subject:**

##HS-6945689-N## Order Inquiry

**Sender:**

foreigntrade@yazdrollingmill[.]com

**Content:**

![Content](/images/loki/15.png)

**Attachments:**

Inquiry.doc - d736b5858f5d3e3630e160cd314d88d1

Our Company Introduction.doc - d736b5858f5d3e3630e160cd314d88d1

**RTF Analysis:**

The email in this example contained what appeared to be two different Word Documents. However when the files are hashed they have the same MD5 so for whatever reason the attacker has included the same file twice but with different names.

In the following analysis I used 'Inquiry.doc' and put it into Hex editor:

![Header](/images/loki/1.png)

From looking at the header we can see that this is an RTF file.

The fact that this is an RTF file rather then a typical Word Doc should raise some red flags regarding the attachment. RTF (Rich Text Format) allows other files to be embedded in the file itself and are often used by attackers to embed malware.
It is possible to dig a little deeper into this file using a Linux Distro called Remnux which is loaded with a number of tools for malware analysis.

For RTF analysis I use rtfdump which was written by Didier Stevens.

The following command can check for the prescence of any embedded [OLE files](https://en.wikipedia.org/wiki/Object_Linking_and_Embedding):

``rtfdump.py -f O Inquiry.doc``

![rtfdump](/images/loki/2.png)

The above image shows that three streams have been identified – object, objdata and datastore which have been numbered 229, 230 and 236 by the rtfdump tool.

To take a look at each stream use the '-s' argument and the corresponding number rtfdump has assigned it.

``rtfdump.py -s 230 Inquiry.doc``

![rtfdump](/images/loki/3.png)

In the above output we can see the hex values begin 01050000 02000000 which indicates an OLE 1.0 object. We can convert this to binary using the ‘-H’ argument:

``rtfdump.py -s 230 -H Inquiry.doc``

![rtfdump](/images/loki/4.png)

The above output contains the string ``'Equation.3'``, evidence that an Equation Editor vulnerability is being exploited. 

The next set of strings contain: 

``cmd /c %tmp%\A.R cmd /c calc cmd /c %tmp%\A.RA`` 

This looks quite strange at first as three different commands are referenced. However the second command references the Windows calculator so the POC code has probably been reused and not deleted afterwards by the author.

By appending '-i' we can gather more information about the object:

![rtfdump](/images/loki/5.png)

The output for stream 229 is almost identical to 230, however output for object 236 is different. Here is the output converted to binary:

``rtfdump.py -s 236 -H Inquiry.doc``

![rtfdump](/images/loki/9.png)

We can see reference to XML and with the '-i' argument we can see that the magic number for this object is 'd0cf1le':

![rtfdump](/images/loki/10.png)

This indicates that there is an embedded file within this object.

**Behavioural Analysis:**

From the static analysis completed on the RTF file it is likely going to invoke a Microsoft Equation Editor exploit, contain a piece of malware called "A.R" and launch the malware from the Temp directory.

Running the document in a VM should now display this activity.

When the document is opened the following contents are displayed:

![Document](/images/loki/16.png)

We can see some text, an embedded file called "A.R" and an equation object.

The following process activity is generated when the document is opened:

![Process Tree](/images/loki/14.png)

Microsoft Equation Editor is launched (EQNEDT32.EXE), the file A.R is dropped onto the machine and a command prompt is created to launch the "A.R" file:

``_PID: 3824, Command line: cmd /c %tmp%\A.R_``

This is the same command identified in rtfdump earlier.

File ‘A.R’ is launched:

``PID: 332, Command line: C:\Users\Admin\AppData\Local\Temp\A.R``

This then launches another ‘A.R’ process:

``PID: 2044, Command line: C:\Users\Admin\AppData\Local\Temp\A.R``

And the original process (332) exits.

**Traffic Analysis:**

The captured traffic in Wireshark shows a POST request to an IP address (103.214.6[.]190) which resolves to a domain (doctkry.pw).

![Wireshark](/images/loki/12.png)

The POST request ending “fre.php” and user-agent string “Charon, Inferno” are IOC’s of Lokibot and what I used to ID the malware.

From Process Hacker I also obtained the following strings running in memory which contain the C2 and the user-agent:

![Strings](/images/loki/11.png)

**Payload:**

9d8bd584c6f2e9c9e3e6ba3a82e2f43d

**C2:**

hxxp://doctkry[.]pw/gate/sote/fre.php

**Additional Notes:**

The exploit used in this document is [CVE-2017-0802](https://github.com/rxwx/CVE-2018-0802), the exploit uses the Packager OLE object to drop an embedded payload into the %TMP% directory, and then executes the file using a short command via a WinExec call.

In the past I have also implemented pro-active measures in proxy logs to check for any traffic which either contains the string "Charon; Inferno" in the user-agent or "/fre.php" to identify potential Loki infections.

Thanks for reading, all feedback welcome via Twitter or Email.

Neil



