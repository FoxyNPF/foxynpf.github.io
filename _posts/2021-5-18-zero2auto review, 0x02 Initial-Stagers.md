My schedule has been crazy busy lately so apologies for only just getting round to the chapter 2 review of the Zero2Auto course. Part one which covers Algorithsm can be found [here](https://neil-fox.github.io/zero2auto-review,-0x01-Algorithms/).

## 0x01 Initial Stagers Review ##

This chapter is called ‘Initial Stagers’ and focuses on the stages used by malware to infect a host and how to analyse each stage, this part of the course is broken down into 5 videos:

* Unpacking Malware Samples
* Diving into 1st Stage Loaders
* Reversing Second Stage Loaders - IcedID
* Reversing Second Stage Loaders - Zloader
* Writing Automated Config Extractors and Emulators

### Unpacking Malware Samples ###

This video is just shy of an hour in length and begins with breaking down what a packer is and the types of packers you will come across.

We then move into how you can detect packed malware as part of your investigation using some malware analysis tools such as PEID and also touch on some useful techniques that should be of interest for people new to malware analysis.

How malware is packed and unpacked is then covered along with methods of unpacking common packers using static, dynamic and automated methods such as unpac.me.

For anybody who doesn’t know how to unpack malware using a tool such as x64dbg then this part of the video should prove useful as the video covers how to unpack 4 different samples of malware:

* Dridex - x32dbg used with breakpoints being set to identify virtual memory being allocated for unpacked malware, the unpacked malware is then dumped from memory using Process Hacker and rebuilt with PE Bear.

* Ramnit - Dumped using Process Hacker. CFF Explorer used to unpack UPX.

* Remcos - .NET packer, unpacked using x32 and dnSpy.

* Zloader - DLL

### Diving into 1st Stage Loaders ###

This video is 1 hour 10 mins in length and covers how Word and Excel documents along with PDF’s are used to download malware.

As part of the course content you are provided with 3 documents to analyse.

The first is a macro enabled Word Doc that delivers Urnsif. Macros are obfuscated which we would expect to see, the video then takes you through how to deobfuscate the macros. The video begins with some nice tips on how to remove junk code which is there to create noise within the macro script and make analysis difficult.

Using Visual Basic you are also shown some nice tips for safely debugging the macros in order to pull out useful information that is being returned by the custom built functions.

The video then demonstrates how to write some Python to fully deobfuscate the strings. This is something I enjoyed as i suck at writing code and also lose interest in hello word tutorials that don’t relate to malware analysis.

Next we move onto an Excel document that delivers Zloader and perform some analysis using olebrowse and olevba to identify malicious indicators. Again this is quite technical and x32dbg is used to actually debug the Excel document which is something I have never done and found quite interesting.

Finally we have another Word Document which makes use of the Equation Editor exploit and we are shown how to use rtfdump to analyse the file.

### Reversing 2nd Stage Loaders - IcedID ###

The next video focuses on a sample of IcedID malware and is just under 45 mins in length. This is where we are analysing malware which has been downloaded by the 1st stage malicious document, this is a common technique and was the main attack vector of Emotet.

This was a video I really enjoyed as it covers how to automatically extract the config from a piece of malware which contains information such as the bad guys c2’s.

We begin with some static analysis of the sample using PEStudio and unpacking the malware using x64dbg and some nice pointers are given for manually unpacking malware. I always find it useful to see other peoples methodology when analysing or unpacking malware so even for somebody who has completed the SANS 610 it was nice to pick up a few little pointers. The video also begins with some very informative slides covering 0verfl0w_’s methodology for analysing a loader and developing a config extractor.

What I also liked was that a different technique was demonstrated for dumping the unpacked file to the desktop than I tend to use which involved using PEBear. This is a tool that is widely used but something I have never really used myself so again I found this useful.

In the video the unpacked malware is then loaded into IDA to begin static analysis. I actually used Ghidra at first but found that it wasnt resolving the Windows API function parameters, this basically rendered it useless for this part of the exercise. This was despite me having Ghidra configured to display this information.

![ghidra](/images/initial_stagers_images/params.PNG)

Assembly code in Ghidra with missing parameter labels:

![ghidra](/images/initial_stagers_images/params1.PNG)

Assembly code in IDA with parameters:

![ida](/images/initial_stagers_images/ida_params.PNG)

So with that I installed IDA free and reloaded the malware. As part of the demo we identify an RC4 encryption routine, this is great as we are building on knowledge from the previous chapter. I actually went back to my notes on the previous chapter to fully understand what an S-box does, I can now recognise when data is being encrypted due to the XOR and relevant S-box so again another nice win.

What I enjoyed about this video is I don't tend to use tools such as Ghidra and IDA, my go to tool is x64dbg for analysing malware, what's nice is that using a tool such as IDA allows me to identify key features of the malware and map them out without running the sample.

My first minor complaint of the course was at the next stage of the video where the demo flips to using the pseudo-code feature of IDA. Being an IDA noob I wondered why I was getting an error message. The reason is this feature is only available in IDA Pro, so I was now unable to exactly follow along with the video.

I then had to flip back to Ghidra to use the pseudo-code feature within that tool however it was then decompiled slightly differently to IDA which wasn't ideal.

Once the static analysis is complete we then move into x64dbg to analyse some of the functions we have identified in IDA/Ghidra. What I found really useful is that from identifying our RC4 decryption routine we can now locate it in x64dbg and look at what is being passed to the function on the stack and identify what is being decrypted, result! Again we are provided with some really useful information such as how to identify the RC4 key and the data to be decrypted.

### Reversing 2nd Stage Loaders - Zloader ###

This video is just under 35 mins in length and some pre-work had been done by the instructor with a function already defined and we are advised how to find the offset so we can then follow along.

We jump to pseudo-code so again so we would need a paid version of IDA or load the sample into Ghidra which wont exactly match the decompiled code in the video.

I gave up on this video on the 2nd time of watching, probably laziness on my part but I didn’t want to mess about with a sample and start looking for offsets to then import it into IDA where I don't have the pseudocode capability.

### Writing Automated Config Extractors and Emulators ###

This was another video I really enjoyed and is 1 hour in length. In this lesson we have the goal of automatically identifying c2’s and encryption keys from the payload.

From the analysis covered in previous videos I now had some functions defined in Ghidra, however when you cover this chapter I would recommend having Ghidra and IDA open.

![ghidra](/images/initial_stagers_images/icedid_functions.PNG)

The walkthrough of building and creating the config extractor was excellent, i’m not a coder but understand the fundamentals and concepts so this was really useful. By the end I was able to start pulling out c2’s automatically by following along with the video.

![config](/images/initial_stagers_images/config_extracted_py.PNG)
