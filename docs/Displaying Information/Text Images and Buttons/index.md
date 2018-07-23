## Text, Images, and Buttons
All text, images, and soft buttons must be sent as part of a `SDLShow` RPC.

!!! NOTE
You do not need to send `SDLShow` requests if you are using the `SDLScreenManager`. The screen manager makes it simple to create and update your app's UI. To find out more, please go to [Displaying Information/Creating the User Interface](Displaying Information/Creating the User Interface).
!!!

### Text
A maximum of four lines of text can be set in `SDLShow` RPC, however, some templates may only support 1, 2, 3 or even 0 lines of text. If all four lines of text are set in the `SDLShow` RPC, but the template only supports three lines of text, then the fourth line will simply be ignored.

To determine the number of lines available on a given head unit, check the `RegisterAppInterfaceResponse.displayCapabilities.textFields` for `mainField(1-4)`.

#### Objective-C
```objc
SDLShow* show = [[SDLShow alloc] initWithMainField1:@"<#Line 1 of Text#>" mainField2:@"<#Line 2 of Text#>" mainField3:@"<#Line 3 of Text#>" mainField4:@"<#Line 4 of Text#>" alignment:SDLTextAlignmentCentered];
self.sdlManager sendRequest:show withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResultSuccess]) {
        <#Text Sent Successfully#>
    }
}];
```

#### Swift
```swift
let show = SDLShow(mainField1: "<#Line 1 of Text#>", mainField2: "<#Line 2 of Text#>", mainField3: "<#Line 3 of Text#>", mainField4: "<#Line 4 of Text#>", alignment: .centered)
sdlManager.send(request: show) { (request, response, error) in
    guard let response = response else { return }
    if response.resultCode == .success {
        <#Text Sent Successfully#>
    }
}
```

#### Text Field Types
The text field type provides context as to the type of data contained in a text-field. Each OEM decides how to customize the template based on the type of data included in the text field. For example, a head-unit could display a thermometer icon next to the temperature in a weather app or bold the song title of the current media. If the head-unit does not support text field types, the provided data will simply be ignored.

| Text Field Types | Description |
|:-------------------|:--------------|
| mediaTitle | The title of the currently playing audio track |
| mediaArtist | The artist or creator of the currently playing audio track |
| mediaAlbum | The album title of the currently playing audio track |
| mediaYear | The creation year of the currently playing audio track |
| mediaGenre| The genre of the currently playing audio track |
| mediaStation | The name of the current source for the media |
| rating | A rating |
| currentTemperature | The current temperature |
| maximumTemperature | The maximum temperature for the day |
| minimumTemperature | The minimum temperature for the day |
| weatherTerm | The current weather (ex. cloudy, clear, etc.) |
| humidity | The current humidity value |

### Images
The position and size of images on the screen is determined by the currently set template. All images must first be uploaded to the remote system using the SDLManager’s [file manager](Displaying Information/Uploading Graphics) before being used in a `SDLShow` RPC. Once the image has been successfully uploaded, the app will be notified in the upload method's completion handler. For information relating to how to upload images, go to the [Uploading Graphics](Uploading Graphics) section.

!!! NOTE
Some head units you may be connected to may not support images at all. Please consult the `RegisterAppInterfaceResponse.displayCapabilities.graphicSupported` [documentation](https://smartdevicelink.com/en/docs/iOS/master/Classes/SDLDisplayCapabilities/).
!!!

#### Show the Image on a Head Unit
Once the image has been uploaded to the head unit, you can show the image on the user interface. To send the image, create a `SDLImage`, and send it using the `SDLShow` RPC. The `value` property should be set to the same string used in the `name` property of the uploaded `SDLArtwork`. Since you are uploading your own image, the `imageType` property should be set to `dynamic`.

#### Objective-C
```objc
SDLImage* image = [[SDLImage alloc] initWithName:@"<#Uploaded as Name#>"];

SDLShow* show = [[SDLShow alloc] init];
show.graphic = image;

self.sdlManager sendRequest:show withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResultSuccess]) {
      <#Image Sent Successfully#>
    }
}];
```

#### Swift
```swift
let sdlImage = SDLImage(name: "<#Uploaded As Name#>")

let show = SDLShow()
show.graphic = image

sdlManager.send(request: show) { (request, response, error) in
    guard let response = response, response.resultCode == .success else { return }
    <#Image Sent Successfully#>
}
```

### Soft Buttons
Buttons on the HMI screen are referred to as soft buttons to distinguish them from hard buttons, which are physical buttons on the head unit. Don’t confuse soft buttons with subscribe buttons, which are buttons that can detect user selection on hard buttons.

#### Soft Buttons
Soft buttons can be created with text, images or both text and images. The location, size, and number of soft buttons visible on the screen depends on the template. If the button has an image, remember to upload the image first to the head unit before setting the image in the `SDLSoftButton` instance.

!!! IMPORTANT
If a button type is set to `image` or `both` but no image is stored on the head unit, the button might not show up on the HMI screen.
!!!

#### Objective-C
```objc
SDLSoftButton* softButton = [[SDLSoftButton alloc] init];

// Button Id
softButton.softButtonID = @(<#Unique Button Id#>);

// Button handler - This is called when user presses the button
softButton.handler = ^(SDLOnButtonPress *_Nullable buttonPress,  SDLOnButtonEvent *_Nullable buttonEvent) {
    if (buttonPress != nil) {
        if ([buttonPress.buttonPressMode isEqualToEnum:SDLButtonPressModeShort]) {
            // Short button press
        } else if ([buttonPress.buttonPressMode isEqualToEnum:SDLButtonPressModeLong]) {
            // Long button press
        }
    } else if (buttonEvent != nil) {
        if ([buttonEvent.buttonEventMode isEqualToEnum:SDLButtonEventModeButtonUp]) {
            // Button up
        } else if ([buttonEvent.buttonEventMode isEqualToEnum:SDLButtonEventModeButtonDown]) {
            // Button down
        }
    }
};

// Button type can be text, image, or both text and image
softButton.type = SDLSoftButtonTypeBoth;

// Button text
softButton.text = @"<#Button Text#>";

// Button image
softButton.image = [[SDLImage alloc] initWithName:@"<#Save As Name#>"];

SDLShow *show = [[SDLShow alloc] init];

// The buttons are set as part of an array
show.softButtons = @[softButton];

// Send the request
[self.sdlManager sendRequest:show withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResultSuccess]) {
        <#Button created successfully#>
    }
}];
```

#### Swift
```swift
let softButton = SDLSoftButton()

// Button Id
softButton.softButtonID = <#Unique Button Id#>

// Button handler - This is called when user presses the button
softButton.handler = { (press, event) in
    if let buttonPress = press {
        if buttonPress.buttonPressMode == .short {
            // Short button press
        } else if buttonPress.buttonPressMode == .long {
            // Long button press
        }
    } else if let buttonEvent = event {
        if buttonEvent.buttonEventMode == .buttonUp {
            // Button up
        } else if buttonEvent.buttonEventMode == .buttonDown {
            // Button down
        }
    }
}

// Button type can be text, image, or both text and image
softButton.type = .both

// Button text
softButton.text = "<#Button Text#>"

// Button image
softButton.image = SDLImage(name: "<#Save As Name#>")

let show = SDLShow()

// The buttons are set as part of an array
show.softButtons = [softButton]

// Send the request
sdlManager.send(request: show) { (request, response, error) in
    guard let response = response else { return }
    if response.resultCode == .success {
        <#Button created successfully#>
    }
}

// OR

// Convenience Init
let softButton = SDLSoftButton(type: <#T##SDLSoftButtonType#>, text: <#T##String?#>, image: <#T##SDLImage?#>, highlighted: <#T##Bool#>, buttonId: <#T##UInt16#>, systemAction: <#T##SDLSystemAction?#>, handler: <#T##SDLRPCButtonNotificationHandler?##SDLRPCButtonNotificationHandler?##(SDLOnButtonPress?, SDLOnButtonEvent?) -> Void#>)
```
