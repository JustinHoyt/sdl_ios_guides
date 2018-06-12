## Video Streaming
To stream video from a SDL app use the `SDLStreamingMediaManager` class. A reference to this class is available from the `SDLManager`. You can choose to create your own video streaming manager, or you can use the `CarWindow` API to easily stream video to the head unit. 

!!! NOTE
Due to an iOS limitation, video can not be streamed when the app on the device is backgrounded or when the device is sleeping/locked.
!!!

### CarWindow
`CarWindow` is a system to automatically video stream a view controller screen to the head unit. When you set the view controller, `CarWindow` will resize the view controller's frame to match the head unit's screen dimensions. Then, when the video service setup has completed, it will capture the screen and and send it to the head unit.

To start, you will have to set a `rootViewController`, which can easily be set using one of the convenience initializers:
* `autostreamingInsecureConfigurationWithInitialViewController:`  
* `autostreamingSecureConfigurationWithSecurityManagers:initialViewController:`

There are several customizations you can make to `CarWindow` to optimize it for your video streaming needs:

1. Choose how `CarWindow` captures and renders the screen using the `carWindowRenderingType` enum. 
1. By default, when using `CarWindow`, the `SDLTouchManager` will sync it's touch updates to the framerate. To disable this feature, set `SDLTouchManager.enableSyncedPanning` to `NO`.
1. `CarWindow` hard-dictates the framerate of the app. To change the framerate and other parameters, update `SDLStreamingMediaConfiguration.customVideoEncoderSettings`.

    Below are the video encoder defaults:

        @{
            (__bridge NSString *)kVTCompressionPropertyKey_ProfileLevel: (__bridge NSString *)kVTProfileLevel_H264_Baseline_AutoLevel,
            (__bridge NSString *)kVTCompressionPropertyKey_RealTime: @YES,
            (__bridge NSString *)kVTCompressionPropertyKey_ExpectedFrameRate: @15,
            (__bridge NSString *)kVTCompressionPropertyKey_AverageBitRate: @600000
        };


#### Showing a New View Controller
Simply update `self.sdlManager.streamManager.rootViewController` to the new view controller. This will also update the haptic parser.

#### Mirroring the Device Screen vs. Off-Screen UI
It is recommended that you set the `rootViewController` to a not-on-device-screen view controller, i.e. you should instantiate a new `UIViewController` class and use it to set the `rootViewController`. This view controller will appear on-screen in the car, while remaining off-screen on the device. It is also possible, but not recommended, to display your on-device-screen UI to the car screen by setting the `rootViewController` to `UIApplication.sharedApplication.keyWindow.rootViewController`. However, if you mirror your device's screen, your app's UI will resize to match the head unit's screen size, thus making most of the app's UI off-screen.

!!! NOTE
If mirroring your device's screen, the `rootViewController` should only be set after `viewDidAppear:animated` is called. Setting the `rootViewController` in `viewDidLoad` or `viewWillAppear:animated` can cause weird behavior when setting the new frame.
!!!

!!! NOTE
If setting the `rootViewController` when the app returns to the foreground, the app should register for the `UIApplicationDidBecomeActive` notification and not the `UIApplicationWillEnterForeground` notification. Setting the frame after a notification from the latter can also cause weird behavior when setting the new frame.
!!!

### Create a Custom Video Streaming Manager
If you decide to create a custom video streaming manager you must maintain the lifecycle of the video stream because there are several situations when video can not stream. Whether or not video can stream depends on the app's HMI state on the head unit and the app's application state on the device. Due to an iOS limitation, video cannot be streamed when the app on the device is no longer in the foreground. 

The lifecycle of the video stream is maintained by the SDL library. The `SDLManager.streamingMediaManager` can be accessed once the `start` method of `SDLManager` is called. The `SDLStreamingMediaManager` automatically takes care of determining screen size and encoding to the correct video format.

!!! NOTE
It is not recommended to alter the default video format and resolution behavior as it can result in distorted video or the video not showing up at all on the head unit. However, that option is available to you by implementing `SDLStreamingMediaConfiguration.dataSource`.
!!!

#### Sending Video Data
To check whether or not you can start sending data to the video stream, watch for the `SDLVideoStreamDidStartNotification`, `SDLVideoStreamDidStopNotification`, and `SDLVideoStreamSuspendedNotification` notifications. When you receive the start notification, start sending video data; stop when you receive the suspended or stop notifications. You will receive a video stream suspended notification when the app on the device is backgrounded. There are parallel start and stop notifications for audio streaming.

Video data must be provided to the `SDLStreamingMediaManager` as a `CVImageBufferRef` (Apple documentation [here](https://developer.apple.com/library/mac/documentation/QuartzCore/Reference/CVImageBufferRef/)). Once the video stream has started, you will not see video appear until Core has received a few frames. Refer to the code sample below for an example of how to send a video frame:

###### Objective-C
```objective-c
CVPixelBufferRef imageBuffer = <#Acquire Image Buffer#>;

if ([self.sdlManager.streamManager sendVideoData:imageBuffer] == NO) {
  NSLog(@"Could not send Video Data");
}
```

###### Swift
```swift
let imageBuffer = <#Acquire Image Buffer#>

guard let streamManager = self.sdlManager.streamManager, !streamManager.isVideoStreamingPaused else {
    return
}

if !streamManager.sendVideoData(imageBuffer) {
    print("Could not send Video Data")
}
```

#### Best Practices
* A constant stream of map frames is not necessary to maintain an image on the screen. Because of this, we advise that a batch of frames are only sent on map movement or location movement. This will keep the application's memory consumption lower.
* For an ideal user experience, we recommend sending 30 frames per second.
