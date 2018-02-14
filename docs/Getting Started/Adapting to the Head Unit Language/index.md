## Adapting to the Head Unit Language

Since a car's head unit can support multiple languages, you have the option to detect which language is currently be used by the head unit as well as get notifications when the user changes the head unit's current language. If desired,  parts of the `SDLLifecycleConfiguration` can be updated to reflect the head unit's current language, including the app's name and the app's text-to-speech name.

If your app does not support the current head unit language, you should decide on a default language to use within your app. The voice-recognition (VR) system expects the user to issue VR commands in the head unit's current language. The VR system will automatically convert and listen for any VR commands created by your app in the default language so you do not need to do any extra work to handle VR commands spoken the current head unit language.

### Getting the Current Head Unit Language
After starting the `SDLManager`, you can check the `registerResponse` property for the head unit's `language` and `hmiDisplayLanguage`. The `language` property gives you the current VR system language; `hmiDisplayLanguage` the current display text.

#### Objective-C
```objc
SDLLanguage headUnitLanguage = self.sdlManager.registerResponse.language;
SDLLanguage headUnitHMIDisplayLanguage = self.sdlManager.registerResponse.hmiDisplayLanguage;
```

#### Swift
```swift
let headUnitLanguage = sdlManager.registerResponse?.language
let headUnitHMIDisplayLanguage = sdlManager.registerResponse?.hmiDisplayLanguage
```
### Updating the SDL App Name
If desired, you can customize your app name, text-to-speech app name, and VR command names, based on the head unit's current language. Since you will not know the head unit's current language until after the app connects to the head unit, the customized app name must be updated with a `SDLChangeRegistration` RPC after the connection is established.

1. Set the default `appName` and `language` in the `SDLManager`'s  `SDLLifecycleConfiguration`.
1. After a connection beween the app and the head unit has been established, get the head unit's current `language` and `hmiDisplayLanguage` from the `registerResponse`.
2. If your app supports the head unit's current `language` and `hmiDisplayLanguage` and the app's default language is different from the head unit's current language, send a `SDLChangeRegistration` RPC with the new app name.

### When to Send a `SDLChangeRegistration` RPC
| App Current Language | App Supported Languages | Head Unit Language  | What to do |
| ------------ | ------------------- |------------------- | ---------- |
| x | [x, y, z]    | x | Nothing, you already match |
| x | [x, y, z]    | a | Nothing, continue using the app's current language |
| x | [x, y, z]    | y | Switch the app to the head unit's language |

#### Objective-C
```objc
SDLChangeRegistration *change = [[SDLChangeRegistration alloc] initWithLanguage:<#Matching language#> hmiDisplayLanguage:<#Matching language#> appName:@"<#App name for new language#>" ttsName:@[<#App TTS name for language#>] ngnMediaScreenAppName:nil vrSynonyms:nil];

[self.sdlManager.sendRequest:change withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
	if (![response.resultCode isEqualToEnum:SDLResultSuccess]) {
		// The change registration failed
		return;
	}
}
```

#### Swift
```swift
let change = SDLChangeRegistration(language:<#Matching language#>, hmiDisplayLanguage:<#Matching language#>, appName:"<#App name for new language#>" ttsName:[<#App TTS name for language#>], ngnMediaScreenAppName:nil, vrSynonyms:nil)

self.sdlManager.send(request: change) { (request, response, error) in
	if response?.resultCode != .success {
		// The change registration failed
		return
	}
}
```
