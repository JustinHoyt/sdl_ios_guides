## Custom Menus
![Perform Interaction Layout](assets/PerformInteractionListOnly.png)
Custom menus, called **perform interactions**, are one level deep, however, you can create submenus by triggering another perform interaction when the user selects a row in a menu. Perform interactions can be set up to recognize speech, so a user can select an item in the menu by speaking their preference rather than physically selecting the item.

Perform interactions are created by sending two different RPCs. First a `SDLCreateInteractionChoiceSet` RPC must be sent. This RPC sends a list of items that will show up in the menu. When the request has been registered successfully, then a `SDLPerformInteraction` RPC is sent. The `SDLPerformInteraction` RPC sends the formatting requirements, the voice-recognition commands, and a timeout command.

#### Create a Set of Custom Menu Items
Each menu item choice defined in `SDLChoice` should be assigned a unique id. The choice set in `SDLCreateInteractionChoiceSet` should also have its own unique id.

#### Objective-C
```objc
SDLChoice* choice = [[SDLChoice alloc] initWithId:<#Unique Id#> menuName:@"<#Menu Title#>" vrCommands:@[@"<#Menu Commands#>"]];

SDLCreateInteractionChoiceSet* createRequest = [[SDLCreateInteractionChoiceSet alloc] initWithId:<#Unique Id#> choiceSet:@[choice]];

[self.sdlManager sendRequest:createRequest withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResultSuccess]) {
      // The request was successful, now send the SDLPerformInteraction RPC
    }
}];
```

#### Swift
```swift
let choice = SDLChoice(id: <#Unique Id#>, menuName: "<#Menu Title#>", vrCommands: ["<#Menu Command#>"])

let createRequest = SDLCreateInteractionChoiceSet(id: <#Unique Id#>, choiceSet: [choice])

sdlManager.send(request: createRequest) { (request, response, error) in
    if response?.resultCode == .success {
    		// The request was successful, now send the SDLPerformInteraction RPC
    }
}
```

#### Format the Set of Custom Menu Items
Once the set of menu items has been sent to SDL Core, send a `SDLPerformInteraction` RPC to get the items to show up on the HMI screen.

#### Objective-C
```objc
SDLPerformInteraction* performInteraction = [[SDLPerformInteraction alloc] initWithInitialPrompt:@"<#Prompt spoken when shown#>" initialText:@"<#Text Displayed When Shown#>" interactionChoiceSetID:<#SDLCreateInteractionChoiceSet id#>];
```

#### Swift
```swift
let performInteraction = SDLPerformInteraction(initialPrompt: "<#Prompt spoken when shown#>", initialText: "<#Text Displayed When Shown#>", interactionChoiceSetID: <#SDLCreateInteractionChoiceSet id#>)
```

#### Interaction Mode
The interaction mode specifies the way the user is prompted to make a selection and the way in which the user’s selection is recorded. You may wish to use "Both" if the user activates the menu with voice, and "Manual Only" if the user activates the menu with touch.

| Interaction Mode  | Description |
| ----------------- | ----------- |
| Manual only       | Interactions occur only through the display |
| VR only           | Interactions occur only through text-to-speech and voice recognition |
| Both              | Interactions can occur both manually or through VR |

#### Objective-C
```objc
performInteraction.interactionMode = SDLInteractionModeManualOnly;
```

#### Swift
```swift
performInteraction.interactionMode = .manualOnly
```

#### VR Interaction Mode
##### Ford HMI
![VR Perform Interaction](assets/PerformInteractionVROnly.png)

#### Manual Interaction Mode
##### Ford HMI
![Manual Perform Interaction](assets/PerformInteractionManualOnly.png)

#### Interaction Layout
The items in the perform interaction can be shown as a grid of buttons (with optional images) or as a list of choices.

| Layout Mode      | Formatting Description |
| ---------------- | ---------------------- |
| Icon only        | A grid of buttons with images |
| Icon with search | A grid of buttons with images along with a search field in the HMI |
| List only        | A vertical list  of text |
| List with search | A vertical list of text with a search field in the HMI |
| Keyboard         | A keyboard shows up immediately in the HMI |

!!! NOTE
Keyboard is currently only supported for the navigation app type and its use is slightly different. See [mobile navigation](Mobile Navigation/Keyboard Input) for more details.
!!!

#### Objective-C
```objc
performInteraction.interactionLayout = SDLLayoutModeListOnly;
```

#### Swift
```swift
performInteraction.interactionLayout = .listOnly
```

#### Icon Only Interaction Layout
##### Ford HMI
![Icon Only Interaction Layout](assets/PerformInteractionIconOnly.png)

#### List Only Interaction Layout
##### Ford HMI
![List Only Interaction Layout](assets/PerformInteractionListOnly.png)

#### List with Search Interaction Layout
##### Ford HMI
![List with Search Interaction Layout](assets/PerformInteractionListwithSearch.png)

#### Text-to-Speech (TTS)
A text-to-speech chunk is a text phrase or prerecorded sound that will be spoken by the head unit. The text parameter specifies the text to be spoken or the name of the pre-recorded sound. Use the type parameter to define the type of information in the text parameter. The `SDLPerformInteraction` request can have a initial, timeout, and a help prompt.

#### Objective-C
```objc
SDLTTSChunk* ttsChunk = [[SDLTTSChunk alloc] initWithText:@"<#Text to Speak#>" type:SDLSpeechCapabilitiesText];
performInteraction.initialPrompt = @[ttsChunk];

// or - more easily

performInteraction.initialPrompt = [SDLTTSChunk textChunksFromString:@"<#Text to Speak#>"];
```

#### Swift
```swift
let ttsChunk = SDLTTSChunk(text: "<#Text to Speak#>", type: .text)
performInteraction.initialPrompt = [prompt]

// or - more easily

performInteraction.initialPrompt = SDLTTSChunk.textChunks(from: "<#Text to Speak#>")
```

#### Timeout
The timeout parameter defines the amount of time the menu will appear on the screen before the menu is dismissed automatically by the HMI.

#### Objective-C
```objc
performInteraction.timeout = @30000; // 30 seconds
```

#### Swift
```swift
performInteraction.timeout = 30000 // 30 seconds
```

#### Send the Request
#### Objective-C
```objc
[self.sdlManager sendRequest:performInteraction withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (![response isKindOfClass:SDLPerformInteractionResponse.class]) {
        return;
    }

    SDLPerformInteractionResponse* performInteractionResponse = (SDLPerformInteractionResponse*)response;

    if ([performInteractionResponse.resultCode isEqualToEnum:SDLResultTimedOut]) {
        // The custom menu timed out before the user could select an item
    } else if ([performInteractionResponse.resultCode isEqualToEnum:SDLResultSuccess]) {
        NSNumber* choiceId = performInteractionResponse.choiceID;
        // The user selected an item in the custom menu
    }
}];

```

#### Swift
```swift
sdlManager.send(request: performInteraction) { (request, response, error) in
    guard let performInteractionResponse = response as? SDLPerformInteractionResponse else {
        return
    }

    // Wait for user's selection or for timeout
    if performInteractionResponse.resultCode == .timedOut {
        // The custom menu timed out before the user could select an item
    } else if performInteractionResponse.resultCode == .success {
        let choiceId = performInteractionResponse.choiceID
        // The user selected an item in the custom menu
    }
}
```

#### Delete the Custom Menu
If the information in the menu is dynamic, then the old interaction choice set needs to be deleted with a `SDLDeleteInteractionChoiceSet` RPC before the new information can be added to the menu. Use the interaction choice set id to delete the menu.

#### Objective-C
```objc
SDLDeleteInteractionChoiceSet* deleteRequest = [[SDLDeleteInteractionChoiceSet alloc] initWithId:<#SDLCreateInteractionChoiceSet id#>];

[self.sdlManager sendRequest:deleteRequest withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResultSuccess]) {
      // The custom menu was deleted successfully
    }
}];
```

#### Swift
```swift
let deleteRequest = SDLDeleteInteractionChoiceSet(id: <#SDLCreateInteractionChoiceSet id#>)

sdlManager.send(request: deleteRequest) { (request, response, error) in
    if response?.resultCode == .success {
     	// The custom menu was deleted successfully
    }
}
```
