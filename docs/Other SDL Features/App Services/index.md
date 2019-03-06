# Creating an App Service

App services are a powerful feature enabling both a new kind of vehicle <-> app communication, and app <-> app communication via SDL.

Currently, there is no high-level support for publishing an app service, so you will have to use raw RPCs for all app service related APIs.

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

### Publishing Your Service

Once you have your service manifest, publishing your service is simple.

##### Objective-C

```objc
SDLPublishAppService *publishServiceRequest = [[SDLPublishAppService alloc] initWithAppServiceManifest:<#Manifest Object#>];
```

##### Swift

```swift
let publishServiceRequest = SDLPublishAppService(appServiceManifest: <#Manifest Object#>)
```

### Updating With Service Data

To update your service data, you must 

## Getting and Subscribing to Service Data

## Interacting with a Service Provider