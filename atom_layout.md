#  Situation on international keyboard support for Google Blink

<p align="justify">
&nbsp; Being able to type in and use an application correctly in the keyboard layout of your native language is a simple but obvious need. If this kind of issues can be worked around/overcomed in a browser, when it comes to <strike>a text editor</strike> <a href=https://atom.io> the text editor</a> it's priority dramatically increases.
</p>


<p align="justify">
&nbsp; Blink is a open source rendering engine used by Chromium. It cannot run by itself, relying on the Chrome content module, in order to function. In this paper we will cover the current situation with international keyboard support in Blink, as well as provide extensive details about an upcoming fix. Most tests and issues covered are studied in <a href=https://atom.io/> Atom </a> or the official Chrome repository.
</p>


## Keyboard event in Atom
<p align="justify">
So, what happens when we press a key in Atom? At a simplified level, the (physical) keyboard generates an event. The event gets to the operating system that further assigns it to your focused application. A focused application is your main tab, "in which you currently are". We use <a href=http://www.chromium.org/blink> Blink </a>, the layout engine. This layout engine "feeds" Atom an object called <b> KeyboardEvent </b>. KeyboardEvent has fields(.key, .keydown, .keyup) that contain information about what key was pressed, wether it's a modifier or not etc.
</p>

## What goes wrong

<p align="justify">
There's a saying about what hapens when you chase two rabbits, so we decided to single out a keyboard and focus on that to get a firm grip of problem's nature.Therefore, I focused solely on <b> German </b> keyboard. The following three issues were identifiable and reproductible:
</p>

- alt gr + q should be "@" but no character is printed.
- \# is correctly written in the editor window, but read as \
- ctrl-shift-7 is read as ctrl-? instead of ctrl-/

<p align="justify"> From left to right, we have : alt pressed on a US keyboard layout, alt gr pressed on a German keyboard and (alt gr + q) on a German keyboard</p>
<img src=http://i.imgur.com/jentXet.png>



<p align="justify"> Based on the images above we can notice a few issues </p>

- alt gr is not recognised and has a wrong keycode (225 is the keycode for sharp s - ß)
- the .key member for alt gr is empty.

<p align="justify"> Similarly, the # gets printed correctly but is recognised as a \ . </p>

## Why it goes wrong

### Wrong keycode/charcode on some events (#, alt gr)

<p align="justify"> Still under investigation, to be updated on further notice. </p>

### Why .key is empty

<p align="justify"> Looking <a href=https://chromium.googlesource.com/chromium/blink/+/master/Source/core/events/KeyboardEvent.cpp>here</a> we can see that alt gr .key is not yet implemented/supported in Chrome/Blink, and that's the cause for which the field is empty</p>

### Solution

<p align="justify"> After investigating  Chrome bug reports and patches, an interesting code review came up. <a href=https://codereview.chromium.org/929053004>This</a> is a set of patches that plans to replace the current KeyboardEvent.KeyIdentifier with KeyboardEvent.key as well as add another code field. The code field will provide the physical key value, whereas the key field with provide the real value. <br><br>
The first version of Chrome on which this fix is applicable is 45.0.2454.12. <a href=https://omahaproxy.appspot.com> This</a> is a helpful website in order to learn more details related to a certain version, details such as Blink version from Chrome version. <br><br>
In order to get key working you have to apply <a href=https://codereview.chromium.org/929053004> issue 929053004 </a>  and then apply ( manually, if it doesn't auto apply) the issue <a href=https://codereview.chromium.org/1146173006> issue 1146173006 </a>. After that, you will no longer be required to enable the experimental flag. It will work as is. </p>

To check if the fix is already upstream, go to <a href=https://codereview.chromium.org/1146173006> issue 1146173006 </a> and see if there's a commit number attached to it. Then use <a href=https://omahaproxy.appspot.com> this</a> to identify the Chrome version.

### Solution on older versions

<p align="justify">
As this feature/fix is "bleeding edge", there's a very high chance of it not working at all.  Below, however, is the theoretical approach on how  to apply the patches for .key and .code features. The two patch sets mentioned above have a prerequisite, <a href=https://codereview.chromium.org/1103263004>issue 1103263004 </a>. If  this patch is backported to the current version, then <a href=https://codereview.chromium.org/929053004> issue 929053004 </a> and <a href=https://codereview.chromium.org/1146173006> issue 1146173006 </a> should apply smoothly. </p>


## Background details & relevant information

<p align="justify">
Tests were performed on the following setup: <br><br>
Operating System: Ubuntu 14.04<br>
Platform: x64<br>
Chrome version: Version 45.0.2441.0 (64-bit) built from source <br>
Atom version: 0.208.0-cecd233 built from source. <br> <br>
</p>
<p align="justify"><b> Note:</b>  All aforementioned key combinations (on their specific layouts) were tested in the URL bar(of Chrome) as well as in a terminal. The keyboard events issue is not caused by the operating system. Further proof is that the issues manifest differently on different operating systems. </p>

