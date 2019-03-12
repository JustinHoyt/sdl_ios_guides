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

For more information, see `Getting and Subscribing to Services` below.

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

## Getting and Subscribing to Services

Now that we have covered service providers and how they expose their service data to the SDL ecosystem of the head unit and all available SDL apps, we need to discuss how that wider ecosystem is notified of available service providers and how they can then access their data.

### Getting and Subscribing to Available Services

To get information on all services published on the system, as well as on changes to published services, you will use the `GetSystemCapability` request / response as well as the `OnSystemCapabilityUpdated` notification.

##### Objective-C

```objc
/**
 * Notification selector for when the `OnSystemCapabilityUpdated` RPC notification comes in
 */
- (void)systemCapabilityDidUpdate:(SDLRPCNotificationNotification *)notification {
    SDLOnSystemCapabilityUpdated *updateNotification = notification.notification;
    SDLLogD(@"On System Capability updated: %@", updateNotification);
}

- (void)setupAppServicesCapability {
    [NSNotificationCenter.defaultCenter addObserver:self selector:@selector(systemCapabilityDidUpdate:) name:SDLDidReceiveSystemCapabilityUpdatedNotification object:nil];

    SDLGetSystemCapability *getAppServices = [[SDLGetSystemCapability alloc] initWithType:SDLSystemCapabilityTypeAppServices subscribe:YES];
    [self.sdlManager sendRequest:getAppServices withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
        if (!response || !response.success) {
            SDLLogE(@"Error sending get system capability: Req %@, Res %@, err %@", request, response, error);
            return;
        }

        SDLAppServicesCapabilities *serviceRecord = response.systemCapability.appServicesCapabilities
        SDLLogD(@"Get system capability app service response: %@, serviceRecord %@", response, serviceRecord);
        <#Code#>
    }];
}
```

##### Swift

```swift
private func setupAppServicesCapability() {
    Notification.default.addObserver(self, selector: #selector(systemCapabilityDidUpdate(_:)), name: .SDLDidReceiveSystemCapabilityUpdated, object: nil)

    let getAppServices = SDLGetSystemCapability(type: .appServices, subscribe: true)
    sdlManager.send(request: getAppServices) { (req, res, err) in
        guard let response = res as? SDLGetSystemCapabilityResponse, let serviceRecord = response.systemCapability.appServicesCapabilities, response.success.boolValue == true, err == nil else { return }

        <#Use the service record#>
    }
}

@objc private func systemCapabilityDidUpdate(_ notification: SDLRPCNotificationNotification) {
    guard let capabilityNotification = notification.notification as? SDLOnSystemCapabilityUpdated else { return }

    SDLLog.d("OnSystemCapabilityUpdated: \(capabilityNotification)")
    <#Code#>
}
```

#### Checking the App Service Capability

Once you've retrieved the initial list of app service capabilities (in the `GetSystemCapability` response), or an updated list of app service capabilities (from the `OnSystemCapabilityUpdated` notification), you may want to inspect the data to find what you are looking for. Below is example code with comments explaining what each part of the app service capability is used for.

##### Objective-C

```objc
// From GetSystemCapabilityResponse
SDLGetSystemCapabilityResponse *getResponse = <#From whereever you got it#>;
SDLAppServicesCapabilities *capabilities = getResponse.systemCapability.appServicesCapabilities;

// This array contains all currently available app services on the system
NSArray<SDLAppServiceCapability *> *appServices = capabilities.appServices;
SDLAppServiceCapability *aCapability = appServices.first;

// This will be nil since it's the first update
SDLServiceUpdateReason *capabilityReason = aCapability.updateReason;

// The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (it always will be here), and whether or not the service is the active service for its service type (only one service can be active for each type)
SDLAppServiceRecord *serviceRecord = aCapability.updatedAppServiceRecord;


// From OnSystemCapabilityUpdated
SDLOnSystemCapabilityUpdated *serviceNotification = <#From whereever you got it#>;
SDLAppServicesCapabilities *capabilities = serviceNotification.systemCapability.appServicesCapabilities;

// This array contains all recently updated services
NSArray<SDLAppServiceCapability *> *appServices = capabilities.appServices;
SDLAppServiceCapability *aCapability = appServices.first;

// This won't be nil. It will tell you why a service is in the list of updates
SDLServiceUpdateReason *capabilityReason = aCapability.updateReason;

// The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (if it's not, it was just removed and should not be addressed), and whether or not the service is the active service for its service type (only one service can be active for each type)
SDLAppServiceRecord *serviceRecord = aCapability.updatedAppServiceRecord;
```

##### Swift

```swift

```

#### Using the System Capability Manager

In the current release, while the system capability manager supports getting the app services capability, it does not support automatically updating itself when the app services change. It must currently be polled through the `updateCapabilityType:completionHandler:` method. Therefore, until the `SystemCapabilityManager` is updated with support for auto-updating, we currently recommend using the raw `SDLGetSystemCapability` RPC and subscribing.

### Getting and Subscribing to a Service's Data

## Interacting with a Service Provider

### Send RPCs to a Service Provider

### Sending an Action to a Service Provider