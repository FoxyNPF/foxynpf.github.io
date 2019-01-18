I recently came across the following sample and thought I would share the steps taken to extract the IOC's.

Before opening the document I had the following programs running:

Process Monitor – Filter set to show “Process Create”
Process Hacker
Burp Suite

When I open the document I can see from procmon that the process EQNEDT32.EXE has been launched:

![Procmon](/images/remcos/remcos_ph.png)

As there is no macros to enable this indicates that the malicious Excel document is exploiting the vulnerability in Microsoft’s Equation Editor (CVE-2017-11882) in order to download the binary.

The process associated with Equation editor is then creating a new process:

PID: 3180, Command line: C:\Users\Admin\AppData\Roaming\swsx-audio.exe

I have used Burp Suite to proxy the traffic and can see the C2 that was called by the document:

hxxp://aervoes.com/css/viccx[.]exe

![Burp](/images/remcos/burp.png)

The payload is downloaded to the victim machine and from process hacker or process monitor we can see the original name of viccx.exe from burp  has been renamed to swsx-audio.exe.

From process hacker we can now right click the process we have identified and open the location where it has been written to disk:

C:\Users\Admin\AppData\Roaming\swsx-audio.exe
MD5 Hash of file - f064826cb414957032c0fbba66a26fb5

In process hacker we can also view the strings of the process running in memory.

![Strings](/images/remcos/remcos.png)

Here we can see the type of malware has now been identified as Remcos. There is also a filepath referencing a logs.dat along, reference to a keylogger along with a couple of C2’s.

By navigating to the location in Windows Explorer and opening the file with Notepad we can confirm that the malware is logging the users keystrokes:

![Keylogger](/images/remcos/log.png)

In order for the malware to remain persistent on the device I ran Autoruns to see what modifications had been made to the device by the malware:

![Persistence](/images/remcos/persistence.png)

The above output shows a vbs script has been created and set to launch at startup. By navigating to the scripts location and opening the file in notepad we can see that the script invokes the previously identified swsx-audio.exe:

set ZntmdjU = CReateObjEct("WscrIPt.Shell")
ZNtMDJU.run """C:\Users\Admin\Desktop\swsx-audio.exe"""

