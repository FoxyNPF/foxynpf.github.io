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

This now contains another level of Base64 so copy and paste this into Cyber Chef as a new recipe.

``XZFvb5swEIe/Ci+QnCgLHi1royKkHrAwpP1DGxuLJlWOY8DF2IyYQBvlu8/a1qbaSX7h0+PHp9/ZBZiakgClHoURkG+HADncQSDZuFTbe0a19ZFp5zvbRoIzqX37sTBMMkGAaq27/Q3GFWnZfmgcqlosBelGVV56Db6KyosP+PaJkj0RrCLiDzZ2S6qkNkL8jn56+5n95Qzmrhx3ZY577biX3ktQrmN51v0a+KMSquL0rCS7lkuchkV75raEO0R0NVEt23HiqL7Cq7K5bx4EJl/5NUbOl05wPUO3aO7bB4DCmwKUuBBGAJmJ5FsOEIIVWOjqzYW5j9BEkAdoTeMih9B0fkBkMjE9m8nDzbBnfderkgu2QD/R4p9ggRw2MeSXqmeE1jM7gnBq8gksLq3nVOdH3T8cn9bgxGqUQpHd2sjOL15Zz1+akYciVJCZfaRA3XFjtpiW1myWML1MNWtfsI5gstK1tayY5b02NbeOqTyohv1H+naaud7GOLspjpIY+VszdOOfTpRoWh9PJ3u4i+IMYAzQ+w1kHrjI/w0=``

Use the following operations:

``From Base64``  
``Raw Inflate``  
``Split – Set the delimeter to ‘@’``  
``Defang URL``

![CyberChef](/images/CC/7.png)

The above image shows the decoded and defanged URL's which can now be extracted and safely shared.
