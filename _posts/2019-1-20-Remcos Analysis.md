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
