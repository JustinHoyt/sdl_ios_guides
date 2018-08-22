# Media Clock
The media clock is used by media apps to present the current timing information of a playing media item – such as a song, podcast, or audiobook. It is controlled via the `SetMediaClockTimer` RPC.

!!! NOTE
Ensure your app is a media type app and you are on the media template before using this feature.
!!!

The media timer works like a timer. You set the start and end time and you set the timer to start ticking automatically either up or down.

## Counting Up
In order to count up using the timer, the "bottom end" of the time will be 0:00, you can set the progress "start time", and the "top end". For example, if you are starting a song at `0:00` and it will end at `4:13`, you will end with a timer starting at `0:00`, ending at `0:00` with zero progress. This timer will automatically increment every second.

!!! NOTE
The end time *must* be larger than the start time, or the request will be rejected
!!!

##### Objective-C
```objc
SDLSetMediaClockTimer *mediaClock = [[SDLSetMediaClockTimer alloc] initWithUpdateMode:SDLUpdateModeCountUp];mediaClock.startTime = [[SDLStartTime alloc] initWithHours:0 minutes:0 seconds:0];
mediaClock.endTime = [[SDLStartTime alloc] initWithHours:0 minutes:4 seconds:13];
[self.sdlManager sendRequest:mediaClock];
```

##### Swift
```swift
let mediaClock = SDLSetMediaClockTimer(updateMode: .countUp)
mediaClock.startTime = SDLStartTime(hours: 0, minutes: 0, seconds: 0)
mediaClock.endTime = SDLStartTime(hours: 0, minutes: 4, seconds: 13)
sdlManager.send(mediaClock)
```

The following code will create a timer starting at `0:00`, ending at `4:13` and already at `2:20` of progress.

##### Objective-C
```objc
SDLSetMediaClockTimer *mediaClock = [[SDLSetMediaClockTimer alloc] initWithUpdateMode:SDLUpdateModeCountUp];
mediaClock.startTime = [[SDLStartTime alloc] initWithHours:0 minutes:2 seconds:20];
mediaClock.endTime = [[SDLStartTime alloc] initWithHours:0 minutes:4 seconds:13];
[self.sdlManager sendRequest:mediaClock];
```

##### Swift
```swift
```swift
let mediaClock = SDLSetMediaClockTimer(updateMode: .countUp)
mediaClock.startTime = SDLStartTime(hours: 0, minutes: 2, seconds: 20)
mediaClock.endTime = SDLStartTime(hours: 0, minutes: 4, seconds: 13)
sdlManager.send(mediaClock)
```

## Counting Down
Counting down is the opposite of counting up (I know, right?). The timer bar moves from right to left and the timer will count down. For example, if you're counting down from `10:00` to `0:00`, the timer will auto-update every second.

!!! NOTE
The end time *must* be larger than the start time, or the request will be rejected
!!!

##### Objective-C
```objc
SDLSetMediaClockTimer *mediaClock = [[SDLSetMediaClockTimer alloc] initWithUpdateMode:SDLUpdateModeCountDown];
mediaClock.startTime = [[SDLStartTime alloc] initWithHours:0 minutes:10 seconds:0];
mediaClock.endTime = [[SDLStartTime alloc] initWithHours:0 minutes:0 seconds:0];
[self.sdlManager sendRequest:mediaClock];
```

##### Swift
```swift
let mediaClock = SDLSetMediaClockTimer(updateMode: .countDown)
mediaClock.startTime = SDLStartTime(hours: 0, minutes: 10, seconds: 0)
mediaClock.endTime = SDLStartTime(hours: 0, minutes: 0, seconds: 0)
sdlManager.send(mediaClock)
```

## Pausing & Resuming
When pausing the timer, it will stop the timer as soon as the request is received and processed. When a resume request is sent, the timer begins again at the paused time as soon as the request is processed. You can update the start and end times using a pause command to change the timer while remaining paused.

##### Objective-C
```objc
SDLSetMediaClockTimer *mediaClock = [[SDLSetMediaClockTimer alloc] initWithUpdateMode:SDLUpdateModePause];
[self.sdlManager sendRequest:mediaClock];
```

```objc
SDLSetMediaClockTimer *mediaClock = [[SDLSetMediaClockTimer alloc] initWithUpdateMode:SDLUpdateModeResume];
[self.sdlManager sendRequest:mediaClock];
```

##### Swift
```swift
let mediaClock = SDLSetMediaClockTimer(updateMode: .pause)
sdlManager.send(mediaClock)
```

```swift
let mediaClock = SDLSetMediaClockTimer(updateMode: .resume)
sdlManager.send(mediaClock)
```

## Clearing the Timer
Clearing the timer removes it from the screen.

##### Objective-C
```objc
SDLSetMediaClockTimer *mediaClock = [[SDLSetMediaClockTimer alloc] initWithUpdateMode:SDLUpdateModeClear];
[self.sdlManager sendRequest:mediaClock];
```

##### Swift
```swift
let mediaClock = SDLSetMediaClockTimer(updateMode: .clear)
sdlManager.send(mediaClock)
```

### Updating the Audio Indicator
The audio indicator is, essentially, the play / pause button. As of SDL v6.1, when connected to an SDL v5.0+ head unit, you can tell the system what icon to display on the play / pause button to correspond with how your app works.

For example, a radio app will probably want two button states: play and stop. A music app, in contrast, will probably want a play and pause button. If you don't send any audio indicator information, a play / pause button will be displayed.

The code below will pause the media clock and set the audio indicator to show a play symbol.

##### Objective-C
```objc
SDLSetMediaClockTimer *mediaClock = [[SDLSetMediaClockTimer alloc] initWithUpdateMode:SDLUpdateModePause];
mediaClock.audioStreamingIndicator = SDLAudioStreamingIndicatorPlay;
[self.sdlManager sendRequest:mediaClock];
```

##### Swift
```swift
let mediaClock = SDLSetMediaClockTimer(updateMode: .pause)
mediaClock.audioStreamingIndicator = .play
sdlManager.send(mediaClock)
```