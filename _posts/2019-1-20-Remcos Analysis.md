This post outlines the analysis taken for a phishing email that contained a malicious Excel file that downloads Remcos malware.

Before opening the document I had the following programs running in my virtual environment:

Process Monitor – Filter set to show “Process Create”
Process Hacker
Burp Suite

The most common tactic for attackers sending phishing emails is the use of macros. The user is prompted to enable macros in order to view the document and once clicked the payload is downloaded to there machines.

However in this example there is no prompt to enable macros. The document launches a process called 'EQNEDT32.EXE':

![Procmon](/images/remcos/remcos_ph.png)

As there ARE no macros configured this indicates that the malicious Excel document is exploiting the vulnerability in Microsoft’s Equation Editor (CVE-2017-11882) in order to download the binary.

The exploit downloads the payload from the following location:

hxxp://aervoes.com/css/viccx[.]exe

![Burp](/images/remcos/burp.png)

We can see from the URL the original filename is 'viccx.exe'. This is renamed to 'swsx-audio.exe' and a new process is created:

PID: 3180, Command line: C:\Users\Admin\AppData\Roaming\swsx-audio.exe

As the process for this executable has now been created we can see it running in Process Hacker. By right clicking the process it is possible to open the file location to where the payload has been copied to:

C:\Users\Admin\AppData\Roaming\swsx-audio.exe
MD5 Hash of file - f064826cb414957032c0fbba66a26fb5

Viewing the strings of the process running in memory we are able to identofy the following information.

![Strings](/images/remcos/remcos.png)

Here the type of malware is identified as Remcos by the filepath. The filepath references a logs.dat and there is also reference to a keylogger along with a couple of C2’s.

By navigating to the filepath location in Windows Explorer and opening the file with Notepad we can confirm that the malware is logging the users keystrokes. The C2's will be where the recorded log file is uploaded to by the malware in the hope that potentially some creds have been captured:

![Keylogger](/images/remcos/log.png)

In order for the malware to survive a reboot of the machine it needs to somehow remain persistent on the device it has infected.
By using Autoruns it is possible to identofy what modifications had been made to the device by the malware:

![Persistence](/images/remcos/persistence.png)

The above output shows a vbs script has been created and set to launch at startup. By navigating to the scripts location and opening the file in notepad we can see that the script invokes the previously identified swsx-audio.exe:

set ZntmdjU = CReateObjEct("WscrIPt.Shell")
ZNtMDJU.run """C:\Users\Admin\Desktop\swsx-audio.exe"""

IOC's:

File Hashes
RO#12013.xlsx - b9acbb90c6b816d574f489c388c356b1
C:\Users\Admin\AppData\Roaming\swsx-audio.exe - f064826cb414957032c0fbba66a26fb5

Keylogger output
C:\Users\Admin\AppData\Roaming\remcos\logs.dat

C2's
hxxp://aervoes.com/css/viccx[.]exe
194.5.99.119
185.148.241.49:1949
