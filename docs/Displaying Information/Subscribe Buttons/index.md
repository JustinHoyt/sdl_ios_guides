## Subscribe Buttons
Subscribe buttons are used to detect changes to hard buttons. You can subscribe to the following hard buttons:

| Button  | Template | Button Type |
| ------------- | ------------- | ------------- |
| Ok (play/pause) | media template only | soft button and hard button |
| Seek left | media template only | soft button and hard button |
| Seek right | media template only | soft button and hard button |
| Tune up | media template only | hard button |
| Tune down | media template only | hard button |
| Preset 0-9 | any template | hard button |
| Search | any template | hard button |

### Customizing Subscribe Buttons
There is no way to customize a subscribe button's image or text.

### Music Player Subscribe Buttons
The play/pause, seek left, and seek right, tune up, and tune down subscribe buttons can only be used in the `MEDIA` template. The ok, seek left, and seek right buttons will show up on the app's SDL UI as a soft button in a predefined location dictated by the media template. The tune up and tune down subscribe buttons are triggered by hard buttons in the center console or the car's steering wheel. You will automatically be assigned the media template if you set your app's configuration `appType` to `MEDIA`.

#### Objective-C
```objc
SDLSubscribeButton *subscribeButton = [[SDLSubscribeButton alloc] initWithButtonName:SDLButtonNameOk handler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent) {
    <#subscribe button selected#>
}];
[manager sendRequest:subscribeButton withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil) { return; }
    <#subscribe button sent successfully#>
}];
```

#### Swift
```swift
let subscribeButton = SDLSubscribeButton(buttonName: .ok) { (buttonPress, buttonEvent) in
    <#subscribe button selected#>
}
sdlManager.send(request: subscribeButton) { (request, response, error) in
    guard error == nil else { return }
    <#subscribe button sent successfully#>
}
```

