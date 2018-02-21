## Adapting to the Head Unit Language

Since a head unit can support multiple languages, you may want to add support for more than one language to your SDL app. The SDL library allows you to check which language is currently be used by the head unit and get notifications when the user changes the head unit's current language. If desired, the app's name and the app's text-to-speech (TTS) name can be customized to reflect the head unit's current language. If your app name is not part of the current lexicon, you should tell the VR system how a native speaker will pronounce your app name by setting the TTS name using [https://en.wikipedia.org/wiki/Phoneme](phonemes) from either the Microsoft SAPI phoneme set or from the LHPLUS phoneme set.

The initial configuration of the `SDLManager` requires a default language when setting the `SDLLifecycleConfiguration`. If not set, the SDL library uses American English (*EN_US*) as the default language. The connection will fail if the head unit does not support the language set in the `SDLLifecycleConfiguration`. The `RegisterAppInterfaceResponse` RPC will return `INVALID_DATA` as the reason for rejecting the `RegisterAppInterfaceRequest` RPC.

If your app does not support the current head unit language, you should decide on a default language to use in your app. All text should be created using this default language. Unfortunately, your VR commands will probably not work as the VR system will not recogize your users' pronunciation.

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
If desired, you can customize your app name, and text-to-speech app name, based on the head unit's current language. Since you will not know the head unit's current language until after the app connects to the head unit, the customized app name must be updated with a `SDLChangeRegistration` RPC after a connection is established.

1. Set the default `appName` and `language` in the `SDLManager`'s  `SDLLifecycleConfiguration`.
1. After a connection between the app and the head unit has been established, get the head unit's current `language` and `hmiDisplayLanguage` from the `registerResponse`.
2. If your app supports the head unit's current `language` and `hmiDisplayLanguage` and the app's default language is different from the head unit's current language, send a `SDLChangeRegistration` RPC with the new app name.

#### Objective-C
```objc
SDLChangeRegistration *changeRegistration = [[SDLChangeRegistration alloc] initWithLanguage:<#Matching language#> hmiDisplayLanguage:<#Matching language#> appName:@"<#App name for new language#>" ttsName:@[<#App TTS name for language#>] ngnMediaScreenAppName:<#NGN media screen app name#> vrSynonyms:<#VR synonyms#>];

[self.sdlManager.sendRequest:changeRegistration withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
	if (![response.resultCode isEqualToEnum:SDLResultSuccess]) {
		// The change registration failed
		return;
	}
}
```

#### Swift
```swift
let changeRegistration = SDLChangeRegistration(language:<#Matching language#>, hmiDisplayLanguage:<#Matching language#>, appName:"<#App name for new language#>" ttsName:[<#App TTS name for language#>], ngnMediaScreenAppName:nil, vrSynonyms:nil)

sdlManager.send(request: changeRegistration) { (request, response, error) in
    guard error == nil, let response = response, response.resultCode == .success else {
        // The change registration failed
        return
    }
}
```

### Pseudo Code for Handling Language Updates
The pseudo code below shows you the project flow for monitoring the head unit's current language and for determining when to send a `changeRegistration` request.

```swift
class ProxyManager {
    static let supportedLanguages: [SDLLanguage] = [‘<#Your Supported Language#>’, ‘<#Your Supported Language#>’]
    static let defaultLanguage: SDLLanguage = ProxyManager.supportedLanguages[0]

    private var currentLanguage: SDLLanguage = ProxyManager.defaultLanguage
    private var currentHMIDisplayLanguage: SDLLanguage = ProxyManager.defaultLanguage

    func start() {
        registerForNotifications()
        sdlManager.start { (success, error)
            guard success else { return }
            // This is called every time the app reconnects to the head unit
            checkHeadUnitLanguage()
        }
    }

    func checkHeadUnitLanguage() {
        // 1. Get the current head unit `language` and `hmiDisplayLanguage` from the `registerAppInterfaceResponse`

        // 2. Check if the app’s `currentLanguage` and `currentHMIDisplayLanguage` matches the head unit’s current language and `hmiDisplayLanguage`. If so, no need to send a `changeRegistration` request.

        // 3. Check if the head unit’s current language is supported by the app. If it is not, set the app’s `currentLanguage` and `currentHMIDisplayLanguage` to the app’s default supported language. No need to send a `changeRegistration` request.

        // 5. If the app supports the head unit’s current language, send a `changeRegistration` request. Update the `currentLanguage` and `currentHMIDisplayLanguage` vars with the head unit's language.
    }
}
```
