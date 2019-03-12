# Creating an App Service
App services are a powerful feature enabling both a new kind of vehicle <-> app communication, and app <-> app communication via SDL.

Vehicle head units may use these services in various ways. One app service for each type will be the "active" service to the module. For media, for example, this will be the media app that the user is currently using or listening to. For navigation, it would be a navigation app that the user is using to navigate. For weather, it may be the last used weather app, or a user-selected default. The system may then use that service's data to perform various actions (such as navigating to an address with the active service or to display the temperature as provided from the active weather service).

Currently, there is no high-level API support for publishing an app service, so you will have to use raw RPCs for all app service related APIs.

## App Service Types
Apps are able to declare that they provide an app service for various app service types by publishing an app service manifest. Three types of app services are currently available, and more will be made available over time. The currently available types are: Media, Navigation, and Weather, and these will be discussed below.

## Publishing an App Service
Publishing a service is a two step process. Create your app service manifest, and then publish your app service using your manifest.

### Creating an App Service Manifest
The first step to publishing an app service is to create an `SDLAppServiceManifest` object. There is a set of generic parameters you will need to fill out, and then service type specific parameters based on the app service type you are creating.

##### Objective-C
```objc
SDLAppServiceManifest *manifest = [[SDLAppServiceManifest alloc] initWithServiceType:SDLAppServiceTypeMedia];
manifest.serviceName = @"My Media App"; // Must be unique across app services.
manifest.serviceIcon = [[SDLImage alloc] initWithName:@"Service Icon Name" isTemplate:NO]; // Previously uploaded service icon. This could be the same as your app icon.
manifest.allowAppConsumers = @YES; // Whether or not other apps can view your data in addition to the head unit.
manifest.rpcSpecVersion = [[SDLSyncMsgVersion alloc] initWithMajorVersion:5 minorVersion:0 patchVersion:0]; // An *optional* parameter that limits the RPC spec versions you can understand to the provided version *or below*.
manifest.handledRPCs = @[]; // If you add function ids to this *optional* parameter, you can support newer RPCs on older head units (that don't support those RPCs natively) when those RPCs are sent from other connected applications.
manifest.mediaServiceManifest = <#Code#> // Covered below
```

##### Swift
```swift
let manifest = SDLAppServiceManifest(serviceType: .media)
manifest.serviceIcon = SDLImage(name:"Service Icon Name", isTemplate: false) // Previously uploaded service icon. This could be the same as your app icon.
manifest.allowAppConsumers = true; // Whether or not other apps can view your data in addition to the head unit.
manifest.rpcSpecVersion = SDLSyncMsgVersion(majorVersion: 5, minorVersion: 0, patchVersion: 0) // An *optional* parameter that limits the RPC spec versions you can understand to the provided version *or below*.
manifest.handledRPCs = []; // If you add function ids to this *optional* parameter, you can support newer RPCs on older head units (that don't support those RPCs natively) when those RPCs are sent from other connected applications.
manifest.mediaServiceManifest = <#Code#> // Covered below
```

#### Creating a Media Service Manifest
Well, actually there's currently no media service manifest! You'll just have to create an empty media service manifest and set it into your general app service manifest.

##### Objective-C
```objc
SDLMediaServiceManifest *mediaManifest = [[SDLMediaServiceManifest alloc] init];
manifest.mediaServiceManifest = mediaManifest;
```

##### Swift
```swift
let mediaManifest = SDLMediaServiceManifest()
manifest.mediaServiceManifest = mediaManifest
```

#### Creating a Navigation Service Manifest
You will need to create a navigation manifest if you want to publish a navigation service. You will declare whether or not your navigation app will accept waypoints. That is, if your app will support receiving _multiple_ points of navigation (e.g. go to this McDonalds, then this Walmart, then home).

##### Objective-C
```objc
SDLNavigationServiceManifest *navigationManifest = [[SDLNavigationServiceManifest alloc] initWithAcceptsWayPoints:YES];
manifest.navigationServiceManifest = navigationManifest;
```

##### Swift
```swift
let navigationManifest = SDLNavigationServiceManifest(acceptsWayPoints: true)
manifest.navigationServiceManifest = navigationManifest
```

#### Creating a Weather Service Manifest
You will need to create a weather service manifest if you want to publish a weather service. You will declare they types of data your service provides in its `SDLWeatherServiceData`.

##### Objective-C
```objc
SDLWeatherServiceManifest *weatherManifest = [[SDLWeatherServiceManifest alloc] initWithCurrentForecastSupported:YES maxMultidayForecastAmount:10 maxHourlyForecastAmount:24 maxMinutelyForecastAmount:60 weatherForLocationSupported:YES];
manifest.weatherServiceManifest = weatherManifest;
```

##### Swift

```swift
let weatherManifest = SDLWeatherServiceManifest(currentForecastSupported: true, maxMultidayForecastAmount: 10, maxHourlyForecastAmount: 24, maxMinutelyForecastAmount: 60, weatherForLocationSupported: true)
manifest.weatherServiceManifest = weatherManifest
```

### Publishing Your Service
Once you have your service manifest, publishing your service is simple.

##### Objective-C
```objc
SDLPublishAppService *publishServiceRequest = [[SDLPublishAppService alloc] initWithAppServiceManifest:<#Manifest Object#>];
[[self.sdlManager sendRequest:publishServiceRequest withResponseHandler:];]([self.sdlManager sendRequest:publishService withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil || !response.success) { return; }

    SDLPublishAppServiceResponse *publishServiceResponse = (SDLPublishAppServiceResponse *)response;
    SDLAppServiceRecord *serviceRecord = publishServiceResponse.appServiceRecord
    <#Use the response#>
}];
```

##### Swift
```swift
let publishServiceRequest = SDLPublishAppService(appServiceManifest: <#Manifest Object#>)
sdlManager.send(request: publishServiceRequest) { (req, res, err) in
    guard let response = res as? SDLPublishAppServiceResponse, response.success.boolValue == true, err == nil else { return }

    let serviceRecord = response.appServiceRecord
    <#Use the response#>
}
```

Once you have your publish app service response, you will need to store the information provided in its `appServiceRecord` property. You will need the information later when you want to update your service's data.

#### Watching for App Record Updates
As noted in the introduction to this guide, one service for each type may become the "active" service. If your service is the active service, your `SDLAppServiceRecord` parameter `serviceActive` will be updated to note that you are now the active service.

After the initial app record is passed to you in the `SDLPublishAppServiceResponse`, you will need to be notified of changes in order to observe whether or not you have become the active service. To do so, you will have to observe the new `SDLSystemCapabilityTypeAppServices` using `GetSystemCapability` and `OnSystemCapability`.

For more information, see (the Using App Services guide)[Other SDL Features/Using App Services] and see the "Getting and Subscribing to Services" section.

### Updating Your Service's Data
After your service is published, it's time to update your service data. First, you must send an `onAppServiceData` RPC notification with your updated service data. RPC notifications are different than RPC requests in that they will not receive a response from the connected head unit, and must use a different `SDLManager` method call to send.

First, you will have to create an `SDLMediaServiceData`, `SDLNavigationServiceData` or `SDLWeatherServiceData` object with your service's data. Then, add that service-specific data object to an `SDLAppServiceData` object. Finally, create an `SDLOnAppServiceData` notification, append your `SDLAppServiceData` object, and send it.

##### Objective-C
```objc
SDLMediaServiceData *mediaData = [[SDLMediaServiceData alloc] initWithMediaType:SDLMediaTypeMusic mediaTitle:@"Some media title" mediaArtist:@"Some media artist" mediaAlbum:@"Some album" playlistName:@"Some playlist" isExplicit:YES trackPlaybackProgress:45 trackPlaybackDuration:90 queuePlaybackProgress:45 queuePlaybackDuration:150 queueCurrentTrackNumber:2 queueTotalTrackCount:3];
SDLAppServiceData *appData = [[SDLAppServiceData alloc] initWithMediaServiceData:mediaData serviceId:myServiceId];

SDLOnAppServiceData *onAppData = [[SDLOnAppServiceData alloc] initWithServiceData:appData];
[self.sdlManager sendRPC:onAppData];
```

##### Swift
```swift
let mediaData = SDLMediaServiceData(mediaType: .music, mediaTitle: "Some media title", mediaArtist: "Some artist", mediaAlbum: "Some album", playlistName: "Some playlist", isExplicit: true, trackPlaybackProgress: 45, trackPlaybackDuration: 90, queuePlaybackProgress: 45, queuePlaybackDuration: 150, queueCurrentTrackNumber: 2, queueTotalTrackCount: 3)
let appMediaData = SDLAppServiceData(mediaServiceData: mediaData, serviceId: serviceId)

let onAppData = SDLOnAppServiceData(serviceData: appMediaData)
sdlManager.sendRPC(onAppData)
```

## Supporting App Actions
App actions are the ability for app consumers to use the SDL services system to send URIs to app providers in order to activate actions on the provider. Service actions are *schema-less*, i.e. there is no way to define the appropriate URIs through SDL. If you provide already provide actions or wish to start providing them, you will have to document your available actions elsewhere (such as your website).

If you're wondering how to get started with actions and routing, this is a very common problem in iOS! Many apps support the [x-callback-URL](http://x-callback-url.com) format as a common inter-app communication method. There are also [many](https://github.com/devxoul/URLNavigator) [libraries](https://github.com/joeldev/JLRoutes) [available](https://github.com/skyline75489/SwiftRouter) for the purpose of supporting URL routing.

In order to support actions through SDL services, you will need to observe and respond to the `PerformAppServiceInteraction` RPC request.

##### Objective-C

```objc
// Subscribe to PerformAppServiceInteraction requests
[NSNotificationCenter.defaultCenter addObserver:self selector:@selector(performAppServiceInteractionRequestReceived:) name:SDLDidReceivePerformAppServiceInteractionRequest object:nil];

- (void)performAppServiceInteractionRequestReceived:(SDLRPCRequestNotification *)notification {
    SDLPerformAppServiceInteraction *interactionRequest = notification.request;

    // If you have multiple services, this will let you know which of your services is being addressed
    NSString *serviceID = interactionRequest.serviceID;

    // The app id of the service consumer app that sent you this message
    NSString *originAppId = interactionRequest.originApp;

    // The URL sent by the consumer. This must be something you understand, e.g. a URL scheme call. For example, if you were Youtube, it could be a URL to play a specific video. If you were a music app, it could be a URL to play a specific song, activate shuffle / repeat, etc.
    NSURLComponents *interactionURLComponents = [NSURLComponents componentsWithString:interactionRequest.serviceUri];

    // A result you want to send to the consumer app.
    NSString *result = @"Uh oh";
    SDLPerformAppServiceInteractionResponse *response = [[SDLPerformAppServiceInteractionResponse alloc] initWithServiceSpecificResult:result];

    // These are very important, your response won't work properly without them.
    response.success = @NO;
    response.resultCode = SDLResultGenericError;

    [self.sdlManager sendRPC:response];
}
```

##### Swift
```swift
// Subscribe to PerformAppServiceInteraction requests
NotificationCenter.default.addObserver(self, selector: #selector(performAppServiceInteractionRequestReceived(_:)), name: SDLDidReceivePerformAppServiceInteractionRequest, object: nil)

@objc private func performAppServiceInteractionRequestReceived(_ notification: SDLRPCRequestNotification) {
    guard let interactionRequest = notification.request as? SDLPerformAppServiceInteraction else { return }

    // If you have multiple services, this will let you know which of your services is being addressed
    let serviceID = interactionRequest.serviceID

    // The app id of the service consumer app that sent you this message
    let originAppId = interactionRequest.originApp

    // The URL sent by the consumer. This must be something you understand, e.g. a URL scheme call. For example, if you were Youtube, it could be a URL to play a specific video. If you were a music app, it could be a URL to play a specific song, activate shuffle / repeat, etc.
    let interactionURLComponents = URLComponents(string: interactionRequest.serviceUri)

    // A result you want to send to the consumer app.
    let result = "Uh oh"
    let response = SDLPerformAppServiceInteractionResponse(serviceSpecificResult: result)

    // These are very important, your response won't work properly without them.
    response.success = true
    response.resultCode = .success

    sdlManager.sendRPC(response)
}
```