Here’s a quick post on how to use [Cyber Chef](https://gchq.github.io/CyberChef/) to pull out the obfuscated URL’s in the latest Emotet word doc i've seen.

First grab the base64 that is launched from the word doc (I’ve covered this in a previous post and also included some cool tools to grab this data).

Here’s the example I’m using in this post:

![CyberChef](/images/CC/1.PNG)

Copy the code, excluding ‘powershell -e’ and paste into Cyber Chef’s ‘Input’ section.

Drag in the following operations into the recipe:

``From Base64``  
``Remove Null Bytes``  
``Find / Replace``

Set the ‘Find / Replace’ option to find ‘+’ and set it to detect on ‘Simple String’ rather than ‘Regex’.

![CyberChef](/images/CC/8.PNG)

This should now provide you with the following output:

![CyberChef](/images/CC/3.PNG)

This now contains another level of Base64 after 'FRomBaSE64stRINg', copy and paste this into Cyber Chef as a new recipe.

Use the following operations:

``From Base64``  
``Raw Inflate``  
``Split – Set the delimeter to ‘@’``  
``Defang URL``

![CyberChef](/images/CC/7.png)

The above image shows the decoded and defanged URL's which can now be extracted and safely shared.
