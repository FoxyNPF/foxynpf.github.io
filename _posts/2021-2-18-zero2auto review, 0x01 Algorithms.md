I’m always on the lookout for any decent books or online videos that can help me level up my malware analysis skills and for the past couple of months, I’ve had my eye on the Zero2Auto Course.

Zero2Auto is the second malware analysis course created by [0verfl0w_](https://twitter.com/0verfl0w_) and [Vitali Kremez](https://twitter.com/VK_Intel) who are both well-known malware analysts in the infosec community. The first course Zero2Hero is aimed at people who are new to malware analysis whereas Zero2Auto is their advanced course.

The price of the course is £150 which compared to the price of the SANS 610 is a drop in the ocean, also what I really like is you have lifetime access to the course material. 

The course is made up of an introduction and 10 chapters:

* Algorithms
* Initial Stagers
* Evasion
* Malware Internals
* In-depth Analysis
* Exploitation
* “Decompileable2src” Malware
* Threat Intelligence
* Shellcode Analysis
* Rootkits & Bootkit

So based on the price, syllabus, and the guys behind the course I decided that this would be worth my while. While I complete this course I’m going to write up a review of each chapter and cover things I have learned and enjoyed which I thought may be of interest to anybody who is considering taking this course. I won’t be sharing any content of the course. So with that let’s get started.

## 0x01 Algorithms Review ##

The first chapter of the course is all about algorithms, this is something I was interested in as although I have looked at decryption routines in x64dbg I’m certainly not at the level where I can look at assembly code and recognise specific algorithms. Before this section my knowledge of encryption algorithms was pretty much the names of various algorithms and whether they were symmetric or asymmetric from my days way back when studying for the CompTIA S+.

This section contains a single video that comes in at just under 50 minutes in length, a PDF on recognising common crypto routines, and a zip file with some malware samples in.

![0x01](/images/algorithms/chapter1.PNG)

The video is an overview of common and some ‘exotic’ encryption,  compression, and hashing routines. Here the structure of the algorithms is explained and we are given real-world examples of what these look like within some actual malware samples.

What I found useful is that they explain how to locate these algorithms when reverse-engineering malware. These include API that would be of interest but what I was really after was how they are implemented without an API and what to keep a lookout for. These included tips such as constants, magic numbers, etc.

The next part of this section is the PDF file “Recognizing Common Cryptographic Algorithms - Encryption”.

![pdf](/images/algorithms/pdf.PNG)

I thought this part was excellent and despite this being written material this was where I began to take notes.

The PDF gives an overview of each encryption algorithm that was covered in the video, even breaking each phase of the routine down which I found useful. Alongside the explanation again is examples of what this may look like in a malware sample and evidence of the constants that they had told me to look out for in tools such as IDA. 

I then went back and watched the video a second time after going through the PDF then began loading the samples into my malware analysis lab to see if I could detect the algorithms samples using Ghidra.

I remembered on Twitter that [@ConnorSecurity](https://twitter.com/ConnorSecurity) had shared with me some cool [Ghidra resources](https://github.com/AllsafeCyberSecurity/awesome-ghidra) and decided to check them out. Quite quickly I found a plugin for Ghidra called [FindCrypt](https://github.com/d3v1l401/FindCrypt-Ghidra) that will find references to cryptography instructions, this had also been mentioned in the PDF file. I installed the plugin and within seconds I knew I had found evidence of the Serpent algorithm, result! 

![pdf](/images/algorithms/findcrypt.png)

This is the shortest chapter of the Zero2Auto course but already I have learned some new skills and picked up some knowledge. The next chapter is about ‘Initial Stagers’ which covers unpacking samples, reversing loaders, and writing some automated config extractors.

Once I have completed that chapter I will publish a similar review.
