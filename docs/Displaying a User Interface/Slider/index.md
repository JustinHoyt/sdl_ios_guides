# Slider
A `SDLSlider` creates a full screen or pop-up overlay depepednig on platform, that a user can control. HMILevel must be `FULL` in order to send a `SDLSlider` request.


##### Objective-C
```objc
SDLSlider *sdlSlider = [[SDLSlider alloc] initWithNumTicks:5 position:1 sliderHeader:@"This is a header" sliderFooter:@"This is a footer" timeout:30000];

[manager sendRequest:sdlSlider withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
if (!response || !response.success.boolValue) {
SDLLogE(@"Error getting the SDLSlider response");
return;
}

SDLSliderResponse *sdlSliderResponse = (SDLSliderResponse *)response;
NSString *position = [sdlSliderResponse sliderPosition];
<#Use the slider position#>
}];
```

##### Swift
```swift
let slider =  SDLSlider(numTicks: 5, position: 1, sliderHeader: "This is a header", sliderFooter: "This is a footer", timeout: 30000)
manager.send(request: slider, responseHandler: { (req, res, err) in
guard let response = res as? SDLSliderResponse, res?.resultCode == .success, err == nil, let position = response.sliderPosition else { return }
<#Use the slider position#>
})
```
