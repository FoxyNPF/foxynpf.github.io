I’ve put this post together to try and demonstrate how to reverse engineer heavily obfuscated malicious code. Attackers will obfuscate their code as they obviously don’t want security analysts to see what they are trying to achieve. This is done by declaring random variables and function names, adding functions that don’t do anything, adding functions that perform tasks that in the end do nothing all in an attempt to make their code unreadable. In this example I will attempt to try and explain how I have pulled apart a piece of malicious obfuscated code.
I’m no expert in coding and have only recently started attempting to analyse malicious scripts. 

The simple tricks I use are to firstly identify where the first function call is made and then work my way through the code from this starting point. When looking at functions we know that they are going to return a value, by looking at the code is it obvious that something useful will be returned? Looking for loops or any data manipulation that jumps out. A quick look at the end of a function should give an indication if it is doing anything useful or not.

I’m no expert in coding and have only recently started attempting to do any real analysis of malicious scripts. The simple tricks I use to begin are to firstly identify where the first function call is made. When looking at functions we know that they are going to return a value, so by looking at the code ask yourself is it obvious that something useful will be returned? Looking for key identifiers such loops or any data manipulation will help identify anything of interest. A quick look at the end of a function should give an indication if it is doing anything useful or not.

So with that in mind, here we go.

## Creating the key

``sage.vbs``  
``md5 - 99d179405a3faa877788f5ca2e8a0da8``  

The first call in the script is to a subroutine called ‘fAsFIBEQZYVHNXelNlGhzuGxCPM’. From this screenshot it quickly becomes clear how the code is obfuscated as there is a lot of noise.

![script](/images/VBS/vbs1.PNG)

The subroutine starts on line 262 and is heavily obfuscated with a number variables and calls declared.

The same call is declared twice on line 273 and line 281, by searching for this string in the script we can identify the function that is being called.

![script](/images/VBS/vbs2.PNG)

Line 259 shows that all this function does is return the value ‘1’, so this function is noise and can be deleted from the script.
Two more functions are called on line 287 and 290. By using the same analysis technique of searching for the function names we can determine that the only function call that is off interest is ‘OTqJklYWXIQwDDSKCERkCxt’ on line 287.

![script](/images/VBS/vbs3.PNG)

The original subroutine with the variables and calls that do nothing removed now looks like this:

![script](/images/VBS/vbs4.PNG)

A simple call to a function!
The function ‘OTqJklYWXIQwDDSKCERkCxt()’ is now going to return a value which needs to be identified from the code.

![script](/images/VBS/vbs5.PNG)

I’ve cleaned up the function slightly and can see that the variable ‘ttREdlHQQlQIRVQsoEGyNKpjspQ’ on line 272 is being passed the value ‘120’. On line 273 an ‘if’ statement is being declared that is checking if 120 is less than 250. It clearly is so the variable ‘kWbmbqdHQahsHVbfpvnQHfLl’ is passed the value 339 on line 274. 

The function on line 279 is then called and passed two values, ‘pblGCGjoaScnLROrCHpKMyPlUyfp’ and ‘kWbmbqdHQahsHVbfpvnQHfLl’.
We know the value of ‘kWbmbqdHQahsHVbfpvnQHfLl’ is 339 however on line 276 ‘pblGCGjoaScnLROrCHpKMyPlUyfp’ will be passed the value of ‘hWAmtsdubgGuvyxEQHIRkufiiRrswY’.
We haven’t seen ‘hWAmtsdubgGuvyxEQHIRkufiiRrswY’ yet so need to search for this in the code and identify what the value of this will be.
After searching for this function I have highlighted what initially stands out to me:

![script](/images/VBS/vbs6.PNG)

The value 14 being declared on line 573 as ‘fuQMlllGRSdERpUIpaADMZnZ’.

The value 12 being declared on line 576 as ‘PyGWivKMXYQtZonUATvXxRq’.

A for loop being declared on line 582 and 583.

We can see on line 583 that the output of the for loop is going to have the value 14 added to it and then 12 subtracted from it. This is just another technique by the attacker to obfuscate the code by adding unnecessary addition and subtractions, as all this is going to do is add ‘2’ to the final output.

Let’s break down what the ‘for loop’ is going to do.

‘i = 0’ – The counter for the loop is set to zero.

‘To 2387408’ – The loop is going to increment the counter to 2387408.

‘Step 1’- The value assigned with ‘step’, in this case ‘1’, is the amount by which the counter will be incremented each time through the loop. This is the key here as by the time the loop has completed the value will have doubled to 4774816. This is just a convoluted way the attacker has multiplied the eventual output by 2. 

The value 14 is added and then 12 subtracted to then change this number to 4774818.

On line 589 this value is now passed to ‘hWAmtsdubgGuvyxEQHIRkufiiRrswY’.

Lets go back to the previous function, ‘OTqJklYWXIQwDDSKCERkCxt()’, where we first saw this declared now it has the value 4774818 returned to it:

![script](/images/VBS/vbs5.PNG)

On line 276 ‘hWAmtsdubgGuvyxEQHIRkufiiRrswY’ which contains the value ‘4774818’, is passed straight to ‘pblGCGjoaScnLROrCHpKMyPlUyfp’.

Cleaned up function:

![script](/images/VBS/vbs45.PNG)

Let’s now take a look at a new function being called, ‘rtyuIzOvvJBUuAyVbQHLoVvPA’, which is going to be passed the values 4774818, and 339.

Function ‘rtyuIzOvvJBUuAyVbQHLoVvPA’:

![script](/images/VBS/vbs7.PNG)

The arguments in this function now contain the values passed from the previous function.

zTWyTPGHjnAcJgCmaqVteoYPFdOfBsB = 4774818

UeArMgNHJKdelagHRBfWZ = 339

A search of ‘UeArMgNHJKdelagHRBfWZ’ shows this isn’t even used, its just created as noise!

A search of 'zTWyTPGHjnAcJgCmaqVteoYPFdOfBsB’ shows it being converted to a string using the ‘CStr’ function, the string is passed to ‘uBJkrfLVOGOKMLKWyqKJPL’ and this is then referenced again on line 408.

![script](/images/VBS/vbs8.PNG)

Line 408 shows the Mid function being declared which is used to determine a string length. The arguments ‘5’ and ‘2’ which are being passed are stating that from the fifth position of the string, return only 2 characters.

From using the Mid function on the string 4774**81**8, 81 is now recorded and passed to ‘TZRnlPZMNWsTcloXCAtMsSrgTIHol’

On line 410 ‘TZRnlPZMNWsTcloXCAtMsSrgTIHol’ is made an integer using the ‘CInt’ function and is now passed to yet another new variable ‘XcEmqDWSkHkwbbRGgtsoi’.

![script](/images/VBS/vbs9.PNG)

Line 411 shows the Mid function being used again, this time being passed ‘XcEmqDWSkHkwbbRGgtsoi’ and a new variable ‘kMFwDkfZerPAnZcoKwrvZISTTZ’.

From looking at the Mid function we know it is going to look for the 81st character and is only going to record a single character of the string associated with ‘kMFwDkfZerPAnZcoKwrvZISTTZ’.

A search of ‘kMFwDkfZerPAnZcoKwrvZISTTZ’ shows it is assigned a long string and the 81st character is ‘A’.

![script](/images/VBS/vbs10.PNG)

On line 411 ‘A’ is then converted to its [ASCII](http://www.asciitable.com/) value using the ‘Asc’ function which is ‘65’. 65 is then assigned to ‘vKgEOJdwnvJSHhpHWuhopbQ’. This is an incredibly convoluted process to simply return the value 65. We will park this for now as the same function is also performing another task.

## Base64 content decoded

On line 399 a large string is passed to ‘anbhhkXxyBxMfsCYVVdRuzR’ and from line 400 – 418 the content of this variable is being added to with additional strings which will create one large string.

![script](/images/VBS/vbs11.PNG)

These strings can be concatenated by copying the output to a new file. At the bottom of the file add ‘WScript.echo (anbhhkXxyBxMfsCYVVdRuzR) and save as a .vbs file.

From cmd the file can be launched using cscript and passed to a text file i.e. cscript base64.vbs > output.txt.

The output is base64 and the image below shows it decoded in Cyber Chef:

![cyber chef](/images/VBS/cyberchef.PNG)

The decoded output is a set of numbers split by the ‘@’ symbol.

We know now that the VBA code now has to perform the same function.

Before we go looking for the routine in the obfuscated code here is an example of what a base64 routine in VBA would look like.
Using this knowledge we can look for similarities in the obfuscated code:

![base64](/images/VBS/base64routine.PNG)

The below image shows the base64 (anbhhkXxyBxMfsCYVVdRuzR) being passed to a new function which we can assume is going to be a function that will return the de-obfuscated base64.

![script](/images/VBS/vbs12.PNG)

Looking at this function there is some interesting code on lines 20, 28, 38 and 46. This is converting numbers into their ASCII values using the ‘chr’ function.

![script](/images/VBS/vbs13.PNG)

Using Cyber Chef again we can capture just the numeric values using the regex ‘\d+’ and then convert these values to base10.

The output of the line 20 can be seen below:

![cyber chef](/images/VBS/cyberchef1.PNG)

By repeating this process for each line of code the following strings are returned:

``OcmhRYUSRJWsyhJBYRDSlDSOfTE = Msxml2.DOM``  
``ITaLcCXIWUdGZAoWfDTafYmS = Document.3.0``  
``oVumZCnNCHzhIBskqkaepmqVcB = base64``  
``xzkMvzQcPcKWDMSdEyqUsWTnov = bTin.TbasTe64T``  

The first 2 strings are then combined to create **‘Msxml2.DOMDocument.3.0’** and passed to ‘ITaLcCXIWUdGZAoWfDTafYmS’.

The below image shows that the script is starting to create a new activeX object, which will be **‘CreateObject(Msxml2.DOMDocument.3.0)’**, and then passes this value to ‘PybXfddAiIvPkEtEfrMDoTN’.

![script](/images/VBS/vbs14.PNG)

The string ‘base64’ is passed to ‘oVumZCnNCHzhIBskqkaepmqVcB’:

![script](/images/VBS/vbs15.PNG)

Cleaned up version of same line of code:

![script](/images/VBS/vbs44.PNG)

The last string that was decoded, ‘bTin.TbasTe64T’ is passed to another function called ‘vzaikHALFcaFckCQUbxEHckpGaipWYqafYF’.

![script](/images/VBS/vbs16.PNG)

Let’s take a look at what this function is going to do with this string.

![script](/images/VBS/vbs17.PNG)

The string ‘bTin.TbasTe64T’ is passed to this function as the argument ‘uTsavyoQeWUuOVApZUvMMKOzOipN’. A search of this argument in the function shows the ‘T’’s from the string are being removed and then this cleaned up the string, ‘bin.base64’ is returned to the previous function.

![script](/images/VBS/vbs18.PNG)

In the below image the returned value of ‘bin.base64’ is passed to ‘oXtYNBZAiZCidtHIUDCrbunioHl’ on line 49. This is then set as the data type on line 50.

![script](/images/VBS/vbs37.PNG)

Here is the cleaned up code for this function which hopefully should make this a little clearer:

![script](/images/VBS/vbs40.PNG)

A new function is called on line 12 ‘dTsiijTSIEcymftdQtZPIKDW’ which is passed the newly created object.

A look at this function also shows numerical values being converted to ASCII characters again:

![script](/images/VBS/vbs38.PNG)

On lines 78, 90 and 97 we can see that a string is being built. Using the same tactics we used earlier in CyberChef we can see this value is ‘ADODB.Stream’.

I’ve removed all the noise but left the variables with their original names in the image below:

![script](/images/VBS/vbs42.PNG)

Once the numeric values have been converted to ASCII and concatenated they make up the string ‘ADODB.Stream’ and are stored in ‘GjfoZpFnaKNYoSprNRcJJyLMODoi’ on line 25.

On line 26 this string is passed to ‘CreateObject’ and the string ‘CreateObject(ADODB.Stream)’ is passed to ‘pmMjOCgiDLFTeCZuSHzIeQfFHK’.
Line 27 the object is set to a constant of 1, see line 21.

Line 28 the Stream is opened.

Line 29 is passed ‘(CreateObject(Msxml2.DOMDocument.3.0).CreateElement(base64).nodeTypedValue)’ which is the argument from the previous function and the activex document is written.

Line 30 sets the document to be read from the start, position 0.

Line 31 constant set to 2, see line 20.

Line 32 language set.

Line 33 contents of document read/stored and returned to previous function.

Cleaned up code for clarity:

![script](/images/VBS/vbs41.PNG)

The decoded base64 is going to be passed into this newly created document.

The split function is called which then creates an array of the numeric values which were separated by the ‘@’ symbol which is passed to ‘waxegOqEFjMuhdHBEnKadcWpLq’:

![script](/images/VBS/vbs20.PNG)

Array is then passed to ‘ghztIBHNJfcgvdiWooCLl’:

![script](/images/VBS/vbs21.PNG)

The length of the array is then calculated using the upper bound and lower bound function and result passed to ‘FVIymmhzEhanEBeulnkYxkYNN’:

![script](/images/VBS/vbs22.PNG)

The ‘split’ function has now indexed the contents of the array i.e.:

``“112@113@114@115”``  
``Result = [112,113,114,115]``  
``Result[0] = 112``  
``Result [1] = 113``  
``Result [2] = 114``  

A ‘for loop’ then iterates over each index in the array:

![script](/images/VBS/vbs23.PNG)

The i’th value, so each index in the array, is then passed through the loop one piece at a time:

![script](/images/VBS/vbs24.PNG)

![script](/images/VBS/vbs26.PNG)

‘nDyZOdmcOntIOzVZYVEyARUigAZirv’ is the sequential value of the array i.e. [0] and will change/increment as the ‘for loop’ is iterated through. So the loop here will pass the key and each value of the array to the function ‘tJaFmDCDPVOIptMdHUzxve’.

‘tJaFmDCDPVOIptMdHUzxve’ is an XOR routine and the key identifiers are highlighted below:

![script](/images/VBS/vbs25.PNG)

Cleaned up function:

![script](/images/VBS/vbs28.PNG)

In the above image on line 203 value of the array ‘AlyDAeBLVKpKIctrCTRyYgHKjYDnWXO’ and the key value 65 ‘AkwkjFlIQPlKEwQSNySvCinnwzTEqlhe’ are ‘or’ together and stored the output stored in a variable.

Line 204 shows the same values being ‘and’ together and also stored as another separate variable.

These values are then subtracted from each other on line 205 and on 206 returned to the previous function containing the for loop.

Looking at the cleaned up code below the author has created a custom [XOR](https://en.wikipedia.org/wiki/XOR_cipher) routine.

![script](/images/VBS/vbs27.PNG)

The ‘or’ function of the routine will compare the binary value of the each value indexed in the array and the binary value of value key which is 65.
Let say that the value from the array is 50, this will be 00110101 00110000 in binary and 65 will be 00110110 00110101. The ‘or’ function wil then compare each byte and return a ‘1’ if a ‘1’ is present.

00**11**0**1**0**1** 00**11**0000 = 50  
00**11**0**11**0 00**11**0**1**0**1** = 65  
00**11**0**111** 00**11**0**1**0**1** = 75  

On line 203 ‘eaXkDIzxjNrpjOXLtRCIbe’ would then have the value 75 stored here.

On line 204 the ‘and’ function will make another comparison of the binary values but return a ‘1’ if both values are ‘1’.

00**11**0**1**01 00**11**0000 = 50  
00**11**0**1**10 00**11**0101 = 65  
00**11**0**1**00 00**11**0000 = 40  

On line 204 the value 40 will be stored in ‘aSCAMjdRPaolrjhvdaWvwPVNQKBj’.

On line 205 the code is subtracting 75 – 40 which is 35 and then returning this back to the function with the for loop. This is repeated for each index value in the array.
The purpose of constructing this obfuscated XOR routine is that AV will look for XOR routines. This allows the script to bypass the AV detection.

Fully cleaned up XOR function:

![script](/images/VBS/vbs43.PNG)

Back to the code which called the XOR routine:

![script](/images/VBS/vbs26.PNG)

Cleaned up code:

![script](/images/VBS/vbs29.PNG)

The first converted value from the array which in the above example  was ‘35’ is now stored in ‘JHryhQmYodbHHpLfKGJMUAfAE’.

This is then converted from a number to an ASCII character using the ‘Chr’ function:

![script](/images/VBS/vbs30.PNG)

So 35 would be [‘#’](http://www.asciitable.com/). This is then converted to a string:

![script](/images/VBS/vbs31.PNG)

The full string is then built up as each indexed value from the array is passed through the loop performing the base64 de-obfuscation and XOR routine:

![script](/images/VBS/vbs32.PNG)

The above image shows that the last XOR’d value is passed to ‘pDyPTNKELxZiXvhnkXDd’, added to ‘XPjGFJGMJaNXrXbEjoCDY2’ and stored in ‘flDAbKJzStNuafxneMYxpuwDZcO’. As this triangle like process is completed the decoded string will be built up in ‘flDAbKJzStNuafxneMYxpuwDZcO’.
The decoded output is then stored in ‘OwBUpnMFkdXEAvgnULmxXuy’ which is then passed to a new function.

![script](/images/VBS/vbs33.PNG)

By searching for this function it is the base64 decode function which was called previously so the whole thing is now being base64 decoded again.
Once the second layer of base64 decoding is completed, the output is then passed to a new function along with a string:

![script](/images/VBS/vbs34.PNG)

The string above in speech marks is just made up by the author and doesn’t do anything of interest. In the next function we can just concentrate on the second argument which is our decoded base64.

![script](/images/VBS/vbs35.PNG)

A search of ‘NZbbJKpgPEQtuoIvazrXngG’ shows it being passed to ‘Mw’. ‘Mw’ is the code being executed using 'executegloba;'.

![script](/images/VBS/vbs36.PNG)

‘Mw’ is now the deobfuscated code and endgame of the script. We can now decode this by removing ‘eXEcuTeGLobAL Mw’ replacing this with ‘WScript.echo (Mw)’ and saving the changes. Launching the file using ‘cscript file.vbs > output.txt’ we can now see what has been decoded.

## De-obfuscated code Analysis

Let’s take a look at the first part of the code:

![script](/images/VBS/final1.PNG)

On line 4 the script is pulling all running processes from the compromised machine and storing the output in ‘ci’.

The output is then checked to see if the string ‘ImageColorInverter.exe’ is present. This is what the malware is going to be calles and the script is now checking to see if the malware is already running on the device, if it is then the script will quit on line 8.

If this string isn’t found then the script builds and stores the command **‘CreateObject(“wscript.shell”).ExpandEnvironmentStrings(“%TEMP%\ImageColorInverter.exe”)** in the variable ‘t’.

A call is then made to ‘f’ which is the next subroutine.

![script](/images/VBS/final2.PNG)

On line 17 and 18 a command is stored which is going to check ‘\\.\root\SecurityCenter2’. This is a WMI namespace which exposes information from the Windows Security Centre that includes what AV is installed on the compromised device.

Line 19 captures the AV name from the list it has enumerated and stores this in ‘avn’.

![script](/images/VBS/final3.PNG)

In the above image the script is identifying what OS the device is running and storing this in ‘osn’.

The image below shows the script creating some functionality to create a network connection on lines 39 and 40.

Line 41 shows that a GET request over http will be used and the location of where the malware is hosted can be identified on line 42.

On line 43 a custom user agent for the machine is generated using the OS and AV info that was enumerated earlier and the malware is saved to disk on line 49.

![script](/images/VBS/final4.PNG)

Same code cleaned up:

![script](/images/VBS/final5.PNG)

Once the malware is downloaded the script then launches the malware on line 51. The list of processes are enumerated on line 52 and the malware checks to see if it is has been successfully launched. If it has then the variable ‘sttt’ is set as ‘GOOD’ if it hasn’t then it is set as ‘BAD’.

![script](/images/VBS/final6.PNG)

Cleaned up code:

![script](/images/VBS/final8.PNG)

If the result was ‘BAD’ then script attempts to download the malware again.

![script](/images/VBS/final9.PNG)

## Behavioural Analysis

**C2's:**

jessicarea[.]net/memory/quota.xls  
hxxps[://]lunchrappz[.]com/index[.]htm - Post infection traffic

**Payload**

Gozi malware  
C:\Users\Administrator\AppData\Local\Temp\ImageColorInverter.exe  
md5 - 4b8ecad93a2b2a52dd837ed56f2c1350  





