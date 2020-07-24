Mimikatz is a tool used to dump credentials from memory and has been used by numerous APT groups including Wizard Spider, Stone Panda, APT 41, Fancy bear, Refined Kitten, Helix Kitten, Remix Kitten and Static Kitten. If not detected by AV this tool can be quite stealthy as it operates in memory and leaves few artefacts behind. Mimikatz can also perform pass the hash attacks and generate golden tickets allowing an attacker to move laterally.

This report will outline how to use some of the Mimikatz modules and what artefacts, if any, are left behind.

## Mimikatz

https://github.com/gentilkiwi/mimikatz

## Module – sekurlsa:

This module extracts passwords, keys, pin codes and tickets from the memory of ‘lsass’ (Local Security Authority Subsystem Service).

### sekurlsa::logonpasswords

Below image shows extract of output from ‘sekurlsa::logonpasswords’. The NTLM hash of the user is dumped and can then be cracked offline using a tool such as John the Ripper or Hashcat, alternatively the hash can be passed around the network using Mimikatz.

![mimikatz](/images/mimiscreens/sekurlsa_logonpass.PNG)

One event recorded in Sysmon logs:

![mimikatz](/images/mimiscreens/sekurlsa_logonpass_sys.PNG)

![mimikatz](/images/mimiscreens/sekurlsa_logonpass_sys1_2.png)

Same event in ‘Details - Friendly View’:

![mimikatz](/images/mimiscreens/sekurlsa_logonpass_sys2_2.png)

This activity can be detected by focussing on any Sysmon events that have the event ID of 10 and where the target process is listed as ‘C:\Windows\system32\lsass.exe’ and GrantedAccess is ‘0x1010’.

### sekurlsa::pth

Mimikatz can perform pass the hash attacks to run a process under another user’s credentials, this is done using the NTLM hash of the user's password.

In the following example a machine on the fox.local domain has been compromised, the account on this machine is ‘bwayne’. Mimikatz can dump the NTLM hash of any users that have previously logged into the machine since its last reboot and then pass this hash to allow an attacker to move laterally.

The image below shows that the current user ‘bwayne’ does not have access to any domain admin groups and is unable access to another device in the same domain called ‘IRONMAN’ using PsExec:

![mimikatz](/images/mimiscreens/pth_whoami.PNG)

First the attacker can dump the NTLM hashes using the command ‘sekurlsa::logonpasswords’, the image below shows the NTLM hash for the user ‘tstark’ being dumped:

![mimikatz](/images/mimiscreens/tstarkhash.PNG)

User ‘tstark’ privileges then gained from using NTLM hash.

![mimikatz](/images/mimiscreens/sekurlsa_pth.PNG)

A new command prompt is then generated using the ‘tstark’ NTLM hash, the attacker is now able to remotely access the ‘IRONMAN’ machine as this user using a tool such as PsExec:

![mimikatz](/images/mimiscreens/ironman_popped.PNG)

No logs generated for Mimikatz; however, the following logs are generated for the lateral movement using PsExec.

Two events generated in Windows Security event logs:

![mimikatz](/images/mimiscreens/pth_seclog.PNG)

Content of Event 4648 ‘A logon was attempted using explicit credentials’:

![mimikatz](/images/mimiscreens/4648.PNG)

Three events recorded in Sysmon logs:

![mimikatz](/images/mimiscreens/pth_sysmonlog.PNG)

Sysmon, Event ID 3 content:

![mimikatz](/images/mimiscreens/3sysmon.PNG)

Sysmon, Event ID 22 content:

![mimikatz](/images/mimiscreens/22sysmon.PNG)

Running processes:

![mimikatz](/images/mimiscreens/pth_ptree.PNG)

![mimikatz](/images/mimiscreens/pth_procmon.PNG)

### sekurlsa::tickets

The ‘tickets’ command is able to list Kerberos tickets of all sessions. The optional argument ‘/export’ can be used to export tickets into ‘.kirbi’ files.

![mimikatz](/images/mimiscreens/sekurlsa_ticket1.PNG)

Multiple events recorded in Sysmon logs:

![mimikatz](/images/mimiscreens/sekurlsa_ticket_sys.PNG)

Sysmon, Event ID 11 content:

![mimikatz](/images/mimiscreens/sekurlsa_ticket_sys1.PNG)

## Module – lsadump:

### lsadump::lsa /patch

This command is another method of dumping the lsa which contains usernames and the associated NTLM hashes. Hashes can be cracked offline or passed around the network.

![mimikatz](/images/mimiscreens/lsadump_lsa.PNG)

Windows Security Event logs generated:

![mimikatz](/images/mimiscreens/lsadump_lsa_sec.PNG)

### lsadump::lsa /inject /name:krbtgt

This command is used to dump the credentials of a specific account, in this example the krbtgt account has been targeted:

![mimikatz](/images/mimiscreens/lsadump_lsa_inject.PNG)

The image below shows the syntax required to create a golden ticket and initiate a pass the ticket attack:

``•	Custom user created - /User:FakeAdmin``  
``•	Domain for ticket - /domain:fox.local``  
``•	SID (Security Identifier) for domain - /sid<sid>``  
``•	krbtgt account NTLM hash - /krbtgt:<NTLM Hash>``  
``•	User id for created account, 500 specified for Admin rights - /id:500``  
``•	Initiate pass the ticket attack - /ptt``  

![mimikatz](/images/mimiscreens/goldenticket.PNG)

Command prompt generated with newly created ticket using ‘misc::cmd’.

Then able to list the directory of a remote machine using the ‘dir’ command.

![mimikatz](/images/mimiscreens/goldenticket2.PNG)

Events generated in Windows Security logs of remote machine:

![mimikatz](/images/mimiscreens/gticket_batman_sec.PNG)

Content of Event 4624 ‘An account was successfully logged on’:

![mimikatz](/images/mimiscreens/gticket_batman_sec2.PNG)

Running Processes:

![mimikatz](/images/mimiscreens/gticket_ptree.PNG)
