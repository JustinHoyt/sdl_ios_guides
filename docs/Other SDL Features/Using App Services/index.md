# Using App Services

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