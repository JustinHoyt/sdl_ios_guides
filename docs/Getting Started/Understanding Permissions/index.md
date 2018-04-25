## Understanding Permissions
While you are creating your SDL app, you must remember that just because your app is connected to a head unit, it doesn't necessary mean that your app has the required permissions to send RPCs to the head unit. There are two important things to remember regarding permissions:
1. You might not be able to send an RPC when the SDL app is closed or when the main screen is obscured by a menu or alert. Each RPC has a set of `hmiLevel`s that restricts when it can be sent.
1. Some RPCs need special permissions from the OEM in order to be sent. Each OEM decides which RPCs it will restrict so it is up to the developer to check if they have the correct permissions. 


### HMI Levels
Once your app is connected to the head unit, you will receive notifications when the SDL app's HMI status changes. Your SDL app can be in one of four different `hmiLevel`s:

HMI Level   | The Head Unit's UI Status
------------|------------------------------------------------------------
NONE        | The user has not been opened your app, or it has been Exited via the "Menu" button.
BACKGROUND  | The user has opened your app, but is currently in another part of the Head Unit. If you have a Media app, this means that another Media app has been selected.
LIMITED     | For Media apps, this means that a user has opened your app, but is in another part of the Head Unit.
FULL        | Your app is currently in focus on the screen.

Be careful with sending UI related RPCs in the `NONE` and `BACKGROUND` states; some head units may reject RPCs sent in those states. We recommended that you wait until your app's `hmiLevel` enters `FULL` to set up your app's UI.

### Monitoring the HMI Level
The easiest way to monitor the `hmiLevel` of your SDL app is through a required delegate callback of `SDLManagerDelegate`. The function `hmiLevel:didChangeToLevel:` is called every time your app's `hmiLevel` changes.

#### Objective-C
```objc
- (void)hmiLevel:(SDLHMILevel)oldLevel didChangeToLevel:(SDLHMILevel)newLevel {
    if (![newLevel isEqualToEnum:SDLHMILevelNone] && (self.firstHMILevel == SDLHMIFirstStateNone)) {
        // This is our first time in a non-`NONE` state
        self.firstHMILevel = newLevel;
        <#Send static menu RPCs#>
    }

    if ([newLevel isEqualToEnum:SDLHMILevelFull]) {
        <#Send user interface RPCs#>
    } else if ([newLevel isEqualToEnum:SDLHMILevelLimited]) {
        <#Code#>
    } else if ([newLevel isEqualToEnum:SDLHMILevelBackground]) {
        <#Code#>
    } else if ([newLevel isEqualToEnum:SDLHMILevelNone]) {
        <#Code#>
    }
}
```

#### Swift
```swift
fileprivate var firstHMILevel: SDLHMILevel = .none
func hmiLevel(_ oldLevel: SDLHMILevel, didChangeToLevel newLevel: SDLHMILevel) {
    if newLevel != .none && firstHMILevel == .none {
        // This is our first time in a non-`NONE` state
        firstHMILevel = newLevel
        <#Send static menu RPCs#>
    }

    switch newLevel {
    case .full:
        <#Send user interface RPCs#>
    case .limited: break 
    case .background: break
    case .none: break
    default: break
    }
}
```

### Permission Manager
In addition to monitoring the `hmiLevel`, you can check the current permission status of a specific RPC or group of RPCs. If desired, you may also subscribe to get notifications when the permission status changes for a RPC or group of RPCs. 

#### Objective-C
```objc
```

#### Swift
```swift
```

### More Detailed HMI Information
When an interaction occurs relating to your application, there is some additional pieces of information that can be observed that help figure out a more descriptive picture of what is going on with the Head Unit.

#### Audio Streaming State
The Audio Streaming State informs your app whether any currently streaming audio is audible to user (AUDIBLE) or not (NOT_AUDIBLE). A value of NOT_AUDIBLE means that either the application's audio will not be audible to the user, or that the application's audio should not be audible to the user (i.e. some other application on the mobile device may be streaming audio and the application's audio would be blended with that other audio).

You will see this come in for things such as Alert, PerformAudioPassThru, Speaks, etc.

Audio Streaming State   | What does this mean?
------------------------|------------------------------------------------------------
AUDIBLE     			| Any audio you are streaming will be audible to the user. 
ATTENUATED  			| Some kind of audio mixing is occuring between what you are streaming, if anything, and some system level sound. This can be visible is displaying an Alert with `playTone` set to `true`.
NOT_AUDIBLE 			| Your streaming audio is not audible. This could occur during a VRSESSSION System Context.

#### Objective-C
```objc
- (void)audioStreamingState:(nullable SDLAudioStreamingState)oldState didChangeToState:(SDLAudioStreamingState)newState {
    <# code #>
}
```

#### Swift
```swift
func audioStreamingState(_ oldState: SDLAudioStreamingState?, didChangeToState newState: SDLAudioStreamingState) {
    <# code #>
}
```

#### System Context
System Context informs your app if there is potentially a blocking HMI component while your app is still visible. An example of this would be if your application is open, and you display an Alert. Your app will receive a System Context of ALERT while it is presented on the screen, followed by MAIN when it is dismissed.

System Context State   | What does this mean?
-----------------------|------------------------------------------------------------
MAIN        		   | No user interaction is in progress that could be blocking your app's visibility.
VRSESSION  			   | Voice Recognition is currently in progress.
MENU     			   | A menu interaction is currently in-progress. 
HMI_OBSCURED    	   | The app's display HMI is being blocked by either a system or other app's overlay (another app's Alert, for instance).
ALERT 				   | An alert that you have sent is currently visible (Other apps will not receive this).

#### Objective-C
```objc
- (void)systemContext:(nullable SDLSystemContext)oldContext didChangeToContext:(SDLSystemContext)newContext {
    <# code #>
}
```

#### Swift
```swift
func systemContext(_ oldContext: SDLSystemContext?, didChangeToContext newContext: SDLSystemContext) {
    <# code #>
}
```