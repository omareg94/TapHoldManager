# TapHoldManager

An AutoHotkey library that allows you to map multiple actions to one key using a tap-and hold system  
Long press / Tap / Multi Tap / Tap and Hold / Multi Tap and Hold etc are all supported  

# Getting Help
Use the [TapHoldManager Discussion Thread on the AHK Forums](https://autohotkey.com/boards/viewtopic.php?f=6&t=45116)

# Normal Usage
## Setup
1. Download a zip from the releases page
2. Extract the zip to a folder of your choice
3. Run the Example script

## Usage

Include the library
```
#include Lib\TapHoldManager.ahk
```

Instantiate TapHoldManager
```
thm := new TapHoldManager()
```

Add some key(s)
```
thm.Add("1", Func("MyFunc1"))
thm.Add("2", Func("MyFunc2"))
```

Add your callback function(s)
```
MyFunc1(isHold, taps, state){
	ToolTip % "1`n" (isHold ? "HOLD" : "TAP") "`nTaps: " taps "`nState: " state
}

MyFunc2(isHold, taps, state){
	ToolTip % "2`n" (isHold ? "HOLD" : "TAP") "`nTaps: " taps "`nState: " state
}
```

`IsHold` will be true if the event was a hold. If so, `state` will holl `1` for pressed or `0` for released.  
If `IsHold` is false, the event was a tap, and `state` will be `-1`  
In either case, `taps` holds the number of taps which occurred.  
For example, if I double-tap, `IsHold` will be false, and `taps` will be `2`.  
If I double-tapped and held on the second tap, then on press the function would be fired once with `IsHold` as true, taps would as `2` and state as `1`. When the key is released, the same but `state` would be `0`  

## Syntax  
```
thm := new TapHoldManager([ <tapTime := -1>, holdTime := -1, <maxTaps := -1>, <prefix := "$"> ])
thm.Add("<keyname>", <callback (function object)>)
```

**tapTime** The amount of time after a tap occured to wait for another tap.  
Defaults to 150ms.  
**holdTime** The amount of time that you need to hold a button for it to be considered a hold.  
Defaults to the same as `tapTime`.  
**maxTaps** The maximum number of taps before the callback will be fired.  
Defaults to infinite.  
Setting this value to `1` will force the callback to be fired after every tap, whereas by default if you tapped 3 times quickly it would fire the callback once and pass it a `taps` value of `3`, it would now be fired 3 times with a `taps` value of `1`.  
If `maxTaps` is 1, then the `tapTime` setting will have no effect.  
**prefix** The prefix used for all hotkeys, default is `$`  

You can pass as many parameters as you want.  
`thm := new TapHoldManager()`  
`thm := new TapHoldManager(100, 200, 1, "$*")`  

When specifying parameters, you can use `-1` to leave that parameter at it's default.  
For example, if you only wish to alter the `prefix` (3rd) parameter, you could pass `-1` for the first three parameters.  
`thm := new TapHoldManager(-1, -1, -1, "$*")` 

When adding keys, you can also add the same parameters to the end to override the manager's default settings  
`thm.Add("2", Func("MyFunc2"), 300, 1000, 1, "~$")`  

## Logic Flowchart
(Note that the firing of the function on key release during a hold is omitted from this diagram for brevity)
![flowchart](LogicFlowchart.png)  

## Example Timelines
![timeline](Timelines.png)

## Minimizing response times  
To make taps fire as quickly as possible, set `tapTime` as low as possible.  
If you do not require multi-tap or multi-tap-and-hold, then setting `maxTaps` to 1 will make taps respond instantly upon release of the key. In this mode, the `tapTime` setting will have no effect.  
To make holds fire as quickly as possible, setting `holdTime` as low as possible will help. `maxTaps` will not affect response time of holds, as they always end a sequence.

# Integration with the Interception driver (Multiple Keyboard support)
TapHoldManager can use the [Interception driver](http://www.oblita.com/interception) to add support for per-keyboard hotkeys - you can bind TapHoldManager to keys on a second keyboard, and use them completely independently of your main keyboard.  

## Interception Setup
1. Set up my [AutoHotInterception](https://github.com/evilC/AutoHotInterception) AHK wrapper for Interception.  
Get the interception example running, so you know AHK can speak to interception OK.  
2. Download the latest release of TapHoldManager from the releases page and extract it to a folder of your choice.  
3. You need to add the files from TapHoldManager's lib folder to AutoHotInterception's lib folder.  
(Or, you can make sure the contents of both lib folders are in `My Documents\AutoHotkey\Lib`)  
4. Enter the VID / PID of your keyboard(s) into the Interception example script and run  

## AutoHotInterception modes
TapHoldManager works in both AutoHotInterception modes - "Context" and "Subscription" modes.  

### AutoHotInterception Context Mode
Just before you call `Add`, set the context for hotkeys using `hotkey, if, <context var>` which matches that which your AHK Context Callback sets.  
For example:  
```
SetKb1Context(state){
	global isKeyboard1Active
	Sleep 0		; We seem to need this for hotstrings to work, not sure why
	isKeyboard1Active := state
}

[...]

hotkey, if, isKeyboard1Active	; < Causes TapHoldManager's hotkeys to use the context
thm.Add("1", Func("MyFunc1"))
```

### AutoHotInterception Subscription Mode 
A wrapper is included which extends the TapHoldManager class and replaces the hotkey bind code with Interception bind code.  

**Instead of** including the TapHoldManager library, include the interception version:  
```
; #include Lib\TapHoldManager.ahk
#include Lib\InterceptionTapHold.ahk
```

Instantiate `InterceptionTapHold` **instead of** `TapHoldManager`  
`kb1 := new InterceptionTapHold(<VID>, <PID> [, <isMouse = 1>, <instance = 1>, <tapTime>, <block>])`  

**Required Parameters**  
`VID / PID` = The VendorID and ProductID of the device you wish to subscribe to.  
To find the VID / PID of your device, you can use the Monitor demo app from the AHI project.  

**Optional Parameters**  
`isMouse` = Set to true if the device is a Mouse, else leave on false.  
`instance` = When using multiple identical devices, this identifies which instance to use.  
	If you only have one device, leave this at 1
`block` = whether or not to block the input. Defaults to true.  

Note: Use one manager per keyboard.  
```
kb1 := new InterceptionTapHold(0x413C, 0x2107)
kb2 := new InterceptionTapHold(0x1234, 0x2107)
```
