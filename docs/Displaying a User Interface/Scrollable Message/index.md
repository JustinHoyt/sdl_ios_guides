# Scrollable Message
A `SDLScrollableMessage` creates a full screen overaly typically containig a large block of formatted text that can be scrolled. `SDLScrollableMessage` contains a body of text, a message timeout , and up to 8 soft buttons depending on head unit. You must check the `DisplayCapabilities` to get the max number of `SoftButtons` allowed by the head unit. 

You simply create  an  `SDLScrollableMessage` object and send a RPC request to display the Scrollable Message. 

##### Objective-C
```objc
//Create SoftButton Array
NSMutableArray<SDLSoftButton *> *softButtons = [NSMutableArray new];

//Create Message To Display
NSString *scrollableMessageString = [NSString stringWithFormat:@"Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.\n\n\nVestibulum mattis ullamcorper velit sed ullamcorper morbi tincidunt ornare. Purus in massa tempor nec feugiat nisl pretium fusce id.\n\n\nPharetra convallis posuere morbi leo urna molestie at elementum eu. Dictum sit amet justo donec enim diam."];

//Create a Timeout
int16_t scrollableMessageTimeout = 5000;

//Create SoftButtos
SDLSoftButton *scrollableSoftButton = [[SDLSoftButton alloc] initWithType:SDLSoftButtonTypeText text:@"Button 1" image:nil highlighted:NO buttonId:111 systemAction:nil handler:nil];
SDLSoftButton *scrollableSoftButton2 = [[SDLSoftButton alloc] initWithType:SDLSoftButtonTypeText text:@"Button 2" image:nil highlighted:NO buttonId:222 systemAction:nil handler:nil];
[softButtons addObject:scrollableSoftButton];
[softButtons addObject:scrollableSoftButton2];

//Create SDLScrollableMessage Object
SDLScrollableMessage *scrollableMessage = [[SDLScrollableMessage alloc] initWithMessage:scrollableMessageString timeout:scrollableMessageTimeout softButtons:softButtons];

//Finally Send A RPC Message
[sdlManager sendRequest:scrollableMessage];
```

##### Swift
```swift
//Create SoftButton Array
var softButtons = [SDLSoftButton]()

//Create Message To Display
let scrollableMessageText = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.\n\n\nVestibulum mattis ullamcorper velit sed ullamcorper morbi tincidunt ornare. Purus in massa tempor nec feugiat nisl pretium fusce id.\n\n\nPharetra convallis posuere morbi leo urna molestie at elementum eu. Dictum sit amet justo donec enim diam."

//Create a Timeout
let scrollableTimeout: UInt16 = 50000

//Create SoftButtos
let scrollableSoftButtton = SDLSoftButton(type: .text, text: "Button 1", image: nil, highlighted: false, buttonId: 111, systemAction: .defaultAction, handler: nil)
let scrollableSoftButtton2 = SDLSoftButton(type: .text, text: "Button 2", image: nil, highlighted: false, buttonId: 222, systemAction: .defaultAction, handler: nil)
softButtons.append(scrollableSoftButtton)
softButtons.append(scrollableSoftButtton2)

//Create SDLScrollableMessage Object
let scrollableMessage = SDLScrollableMessage(message: scrollableMessageText, timeout: scrollableTimeout, softButtons: softButtons)

//Finally Send A RPC Message
sdlManager.send(scrollableMessage)
```
