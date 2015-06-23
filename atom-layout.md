# Atom key-map & keyboard layout issues

## Current situation
<p align="justify">
&nbsp; Various issues regaring keyboard layout exist. So far, I have mostly focused on german keyboard layout, as it's likely that the cause is similar for other layouts.
</p>

On an english keyboard:
```
Alt
keyboardEvent : { altKey: true, bubbles: true, cancelBubble: false, cancelable: true,
charCode: 0, ctrlKey: false, currentTarget: null, defaultPrevented: false, detail: 0, eventPhase: 0,
keyCode: 18, keyIdentifier: "U+00A4", keyLocation: 1, layerX: 0, layerY: 0, location: 1, metaKey: false, pageX: 0, pageY: 0, Path: NodeList[15], repeat: false, returnValue: true, shiftKey:false, which: 18}

q
keyboardEvent : { altKey: true, bubbles: true, cancelBubble: false, cancelable: true,
charCode: 0, ctrlKey: false, currentTarget: null, defaultPrevented: false, detail: 0, eventPhase: 0,
keyCode: 81, keyIdentifier: "U+0051", keyLocation: 0, layerX: 0, layerY: 0, location: 1, metaKey: false, pageX: 0, pageY: 0, Path: NodeList[15], repeat: false, returnValue: true, shiftKey:false, which: 81}

```
<p align="justify">
&nbsp;On a german keyboard: </p>

```
Alt
keyboardEvent : { altKey: false, bubbles: true, cancelBubble: false, cancelable: true,
charCode: 0, ctrlKey: false, currentTarget: null, defaultPrevented: false, detail: 0, eventPhase: 0,
keyCode: 225, keyIdentifier: "U+00E1", keyLocation: 1, layerX: 0, layerY: 0, location: , metaKey: false, pageX: 0, pageY: 0, Path: NodeList[15], repeat: false, returnValue: true, shiftKey:false, which: 225}


keyboardEvent : { altKey: false, bubbles: true, cancelBubble: false, cancelable: true,
charCode: 0, ctrlKey: false, currentTarget: null, defaultPrevented: false, detail: 0, eventPhase: 0,
keyCode: 144, keyIdentifier: "U+0090", keyLocation: 0, layerX: 0, layerY: 0, location: 1, metaKey: false, pageX: 0, pageY: 0, Path: NodeList[15], repeat: false, returnValue: true, shiftKey:false, which: 144}

q
keyboardEvent : { altKey: true, bubbles: true, cancelBubble: false, cancelable: true,
charCode: 0, ctrlKey: false, currentTarget: null, defaultPrevented: false, detail: 0, eventPhase: 0,
keyCode: 81, keyIdentifier: "U+0051", keyLocation: 0, layerX: 0, layerY: 0, location: 1, metaKey: false, pageX: 0, pageY: 0, Path: NodeList[15], repeat: false, returnValue: true, shiftKey:false, which: 81}

```
<p align="justify">
&nbsp; We notice three issues right ahead: firstly, the alt key is incorrectly displayed as "false", secondly, there's a second event which has the keyCode 144 without any key pressed and the alt key keyCode is wrong, the alt key is recognised as an á.
Another issue is the " <b>#</b> " char, the key-map missinterprets it but somehow it gets printed ok.
</p>

```
"#" on german keyboard
keyboardEvent : { altKey: false, bubbles: true, cancelBubble: false, cancelable: true,
charCode: 0, ctrlKey: false, currentTarget: null, defaultPrevented: false, detail: 0, eventPhase: 0,
keyCode: 191, keyIdentifier: "U+00BF", keyLocation: 0, layerX: 0, layerY: 0, location: 1, metaKey: false, pageX: 0, pageY: 0, Path: NodeList[15], repeat: false, returnValue: true, shiftKey:false, which: 191}
```
<p align="justify">
&nbsp; As you can see, the given keyCode is 191, which is "/ ".
</p>
## Probable cause
<p align="justify">
&nbsp; KeyboardEvent(shortened event from here) has a member called keyIdentifier. Event.keyIdentifer is supposed to be a hash
that helps identifying  keys. But there's an issue with that, there are several key identifiers in webkit that are wrong. See [here](https://bugs.webkit.org/show_bug.cgi?id=19906).
Moreover, there's [this](https://code.google.com/p/chromium/issues/detail?id=263724) well known bug in chromium.

Event.keyIdentifier is <b>not</b> the same as Event.key described [in DOM 3 events keys](http://www.w3.org/TR/DOM-Level-3-Events-key/#key-value-tables) . Key is not implemented in Chrome/Webkit.
</p>
## Solution

### 1 Chromium path
<p align="justify">
&nbsp; I'm  absolutely open to any suggestion, plan or assignment. Below, I'll lay out my  " evaluation" and plan.
Most issues are derived from Chromium and we could lobby or I could try to fix the issue myself. The biggest downside of this path is that I'm not entirely sure that we can succeed (in the given time).
</p>
### 2 Hacking/patching Atom-Keymap
<p align="justify">
&nbsp; There's a function, namely ```translateKeyIdentifierForWindowsAndLinuxChromiumBug``` that fixes some of the mistakes
from chromium, I could fine tune/extend it. This step doesn't seem avoidable but have a look at 3 first.
</p>
### 3 Atom way
<p align="justify">
&nbsp; Since Atom is the hackable text editor, I find it only natural  to have an easy-to-use, simple, effective way of changing key layout / key mapping as you wish.  There's a lot to be discussed here, I certainly don't want to push my idea forward, but the way it is now it looks like the keyboard issues will keep popping up and take the focus from something important.

The biggest issue is preservation(between installs, OS  reinstalls, new computers). I figured that if we'd (prompt the user to) store the data on a server or so, the keyboard issue really dissapears. Also, being public, someone else could use them.
</p>
