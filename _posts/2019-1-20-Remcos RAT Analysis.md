I thought this would be a good starting point to share some simple behavioural malware analysis. The sample in this post came from a phishing email that contained a malicious Microsoft Excel file

Before opening the document I had the following tools running in my virtual environment:

1. [Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon)
2. [Process Hacker](https://processhacker.sourceforge.io/downloads.php)
3. [Burp Suite](https://portswigger.net/burp/communitydownload)
4. [Autoruns](https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns)

A common tactic used by attackers to compromise a device is to send a phishing email which contains a malicious office document. The documents often contain macros which have been configured to download the malware from a compromised website.

However in this example when opening the document there is no prompt to enable macros. Instead the document launches a process called 'EQNEDT32.EXE':

![Procmon](/images/remcos/remcos_ph.png)

This is evidence of the malicious document exploiting a vulnerability in Microsoft’s Equation Editor (CVE-2017-11882).

From using Burp to proxy the traffic we can see that the exploit downloads the payload from the following location:

_hxxp://aervoes.com/css/viccx[.]exe_

![Burp](/images/remcos/burp.png)

We can also see from the URL the original filename is 'viccx.exe'. This is then renamed to 'swsx-audio.exe' and a new process is created as the payload is launched:

![Process Tree](/images/remcos/processtree.png)

Using Process Hacker we are able to view the strings of the process running in memory and extract some useful information:

![Strings](/images/remcos/remcos.png)

Here the type of malware is identified as Remcos by the filepath. The filepath references a 'logs.dat' and there is also reference to a keylogger along with a couple of C2’s.

By navigating to the filepath location in Windows Explorer and opening the logs.dat file with Notepad we can confirm that the malware is logging the users keystrokes. The C2's will be where the recorded log file is uploaded to by the malware:

![Keylogger](/images/remcos/log.png)

In order for the malware to survive a reboot of the machine it needs to somehow remain persistent on the device it has infected.
By using Autoruns it is possible to identify what modifications had been made to the device by the malware:
 
![Persistence](/images/remcos/persistence.png)

The above output shows a vbs script called datemanger.vbs has been created and set to launch at startup. By navigating to the scripts location and opening the file in notepad we can see that the script launches swsx-audio.exe:

![Script](/images/remcos/script.png)
  
**File Hashes:**

RO#12013.xlsx - b9acbb90c6b816d574f489c388c356b1

C:\Users\Admin\AppData\Roaming\swsx-audio.exe - f064826cb414957032c0fbba66a26fb5

**C2's:**

hxxp://aervoes.com/css/viccx[.]exe

194.5.99.119

185.148.241.49:1949
