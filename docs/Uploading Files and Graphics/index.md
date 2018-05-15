## Uploading Files and Graphics
You should be aware of these four things when using images in your SDL app:

1. You may be connected to a head unit that does not have the ability to display images.
1. You must upload images from your mobile device to the head unit before using them in a template.
1. Persistant images are stored on a head unit between sessions. Ephemeral images are destroyed when a sessions ends.
1. Images can not be uploaded when the app's `hmiLevel` is `NONE`. For more information about permissions, please review [Getting Started/Understanding Permissions](Getting Started/Understanding Permissions).

To learn how to use images once they are uploaded, please see [Displaying Information > Text, Images, and Buttons](Displaying Information/Text, Images, and Buttons).

### Checking if Graphics are Supported
Before uploading images to a head unit you should first check if the head unit supports graphics. If not, you should avoid uploading unneccessary image data. To check if graphics are supported look at the `SDLManager`'s `registerResponse` property once the `SDLManager` has started successfully.

#### Objective-C
```objc
__weak typeof (self) weakSelf = self;
[self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
    if (!success) {
        NSLog(@"SDL errored starting up: %@", error);
        return;
    } 

    SDLDisplayCapabilities *displayCapabilities = weakSelf.sdlManager.registerResponse.displayCapabilities;
    BOOL areGraphicsSupported = NO;
    if (displayCapabilities != nil) {
        areGraphicsSupported = displayCapabilities.graphicSupported.boolValue;
    } 
}];
```

#### Swift
```swift
sdlManager.start { [weak self] (success, error) in
    if !success {
        print("SDL errored starting up: \(error.debugDescription)")
        return
    }
    
    var areGraphicsSupported = false
    if let displayCapabilities = self?.sdlManager.registerResponse?.displayCapabilities {
        areGraphicsSupported = displayCapabilities.graphicSupported.boolValue
    }
}
```

### Uploading an Image Using SDLFileManager
The `SDLFileManager` uploads files and keeps track of all the uploaded files names during a session. To send data with the `SDLFileManager`, you need to create either a `SDLFile` or `SDLArtwork` object. `SDLFile` objects are created with a local `NSURL` or `NSData`; `SDLArtwork` a `UIImage`.

#### Objective-C
```objc
UIImage* image = [UIImage imageNamed:@"<#Image Name#>"];
if (!image) {
    <#Error Reading from Assets#>
    return;    
}

SDLArtwork* artwork = [SDLArtwork artworkWithImage:image asImageFormat:<#SDLArtworkImageFormat#>];

[self.sdlManager.fileManager uploadArtwork:artwork completionHandler:^(BOOL success, NSString * _Nonnull artworkName, NSUInteger bytesAvailable, NSError * _Nullable error) {
    if (error != nil) { return; }
    <#Image Upload Successful#>
    // To send the image as part of a show request, create a SDLImage object using the artworkName
    SDLImage *image = [[SDLImage alloc] initWithName:artworkName];
}];
```

#### Swift
```swift
guard let image = UIImage(named: "<#Image Name#>") else {
	<#Error Reading from Assets#>
	return
}
let artwork = SDLArtwork(image: image, persistent: <#Bool#>, as: <#SDLArtworkImageFormat#>)

sdlManager.fileManager.upload(artwork: artwork) { (success, artworkName, bytesAvailable, error) in
    guard error == nil else { return }
    <#Image Upload Successful#>
    // To send the image as part of a show request, create a SDLImage object using the artworkName
    let graphic = SDLImage(name: artworkName)
}
```

### Batch File Uploads
If you want to upload a group of files, you can use the `SDLFileManager`'s batch upload methods. Once all of the uploads have completed you will be notified if any of the uploads failed. If desired, you can also track the progress of each file in the group.

#### Objective-C
```objc
SDLArtwork *artwork = [SDLArtwork artworkWithImage:image asImageFormat:<#SDLArtworkImageFormat#>];
SDLArtwork *artwork2 = [SDLArtwork artworkWithImage:image asImageFormat:<#SDLArtworkImageFormat#>];

[self.sdlManager.fileManager uploadArtworks:@[artwork, artwork2] progressHandler:^BOOL(NSString * _Nonnull artworkName, float uploadPercentage, NSError * _Nullable error) {
    // A single artwork has finished uploading. Use this to check for individual errors, to use an artwork as soon as its uploaded, or to check the progress of the upload
    // The upload percentage is calculated as the total file size of all attempted artwork uploads (regardless of the successfulness of the upload) divided by the sum of the data in all the files
    // Return YES to continue sending artworks. Return NO to cancel any artworks that have not yet been sent.
} completionHandler:^(NSArray<NSString *> * _Nonnull artworkNames, NSError * _Nullable error) {
    // All artworks have completed uploading.
    // If all arworks were uploaded successfully, the error will be nil
    // The error's userInfo parameter is of type [artworkName: error message]
}];
```

#### Swift
```swift
let artwork = SDLArtwork(image: <#UIImage#>, persistent: <#Bool#>, as: <#SDLArtworkImageFormat#>)
let artwork2 = SDLArtwork(image: <#UIImage#>, persistent: <#Bool#>, as: <#SDLArtworkImageFormat#>)

sdlManager.fileManager.upload(artworks: [artwork, artwork2], progressHandler: { (artworkName, uploadPercentage, error) -> Bool in
    // A single artwork has finished uploading. Use this to check for individual errors, to use an artwork as soon as its uploaded, or to check the progress of the upload
    // The upload percentage is calculated as the total file size of all attempted artwork uploads (regardless of the successfulness of the upload) divided by the sum of the data in all the files
    // Return true to continue sending artworks. Return false to cancel any artworks that have not yet been sent.
}) { (artworkNames, error) in
    // All artworks have completed uploading.
    // If all arworks were uploaded successfully, the error will be nil
    // The error's userInfo parameter is of type [artworkName: error message]
}
```

### File Persistance
`SDLFile`, and its subclass `SDLArtwork` support uploading persistant files, i.e. images that are not deleted when the car turns off. Persistance should be used for images that will show up every time the user opens the app, such as logos and soft button icons. If the image is only displayed for short time (i.e. like an album cover that is only displayed while the song is playing) the image should not be persistant because it will take up unnecessary space on the head unit. Objects of type `SDLFile`, and its subclass `SDLArtwork` should be initialized as persistent files if need be.  You can check the persistence via:

#### Objective-C
```objc
if (artwork.isPersistent) {
    <#File was initialized as persistent#>
}
```

#### Swift
```swift
if artwork.isPersistent {
    <#File was initialized as persistent#>
}
```

!!! NOTE
Be aware that persistance will not work if space on the head unit is limited. `SDLFileManager` will always handle uploading images if they are non-existent.
!!!

### Overwrite Stored Files
If a file being uploaded has the same name as an already uploaded file, the new file will be ignored. To override this setting, set the `SDLFile`’s `overwrite` property to true.

#### Objective-C
```objc
file.overwrite = YES;
```

#### Swift
```swift
file.overwrite = true
```

### Check the Amount of File Storage
To find the amount of file storage left for your app on the head unit, use the `SDLFileManager`’s `bytesAvailable` property.

#### Objective-C
```objc
NSUInteger bytesAvailable = self.sdlManager.fileManager.bytesAvailable;
```

#### Swift
```swift
let bytesAvailable = sdlManager.fileManager.bytesAvailable
```

### Check if a File has Already Been Uploaded
You can check out if an image has already been uploaded to the head unit via the `SDLFileManager`'s `remoteFileNames` property.

#### Objective-C
```objc
BOOL isFileOnHeadUnit = [self.sdlManager.fileManager.remoteFileNames containsObject:@"<#Name Uploaded As#>"];
```

#### Swift
```swift
if let fileIsOnHeadUnit = sdlManager.fileManager.remoteFileNames.contains("<#Name Uploaded As#>") {
    if fileIsOnHeadUnit {
        <#File exists#>
    } else {
        <#File does not exist#>
    }
}
```

### Delete Stored Files
Use the file manager’s delete request to delete a file associated with a file name.

#### Objective-C
```objc
[self.sdlManager.fileManager deleteRemoteFileWithName:@"<#Save As Name#>" completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError *error) {
    if (success) {
        <#Image was deleted successfully#>
    }
}];
```

#### Swift
```swift
sdlManager.fileManager.delete(fileName: "<#Save As Name#>") { (success, bytesAvailable, error) in
    if success {
        <#Image was deleted successfully#>
    }

}
```

### Batch Delete Files
```objc
[self.sdlManager.fileManager deleteRemoteFileWithNames:@[@"<#Save As Name#>", @"<#Save As Name 2#>"] completionHandler:^(NSError *error) {
    if (error == nil) {
        <#Images were deleted successfully#>
    }
}];
```

#### Swift
```swift
sdlManager.fileManager.delete(fileNames: ["<#Save As Name#>", "<#Save as Name 2#>"]) { (error) in
    if (error == nil) {
        <#Images were deleted successfully#>
    }
}
```

## Image Specifics
### Image File Type
Images may be formatted as PNG, JPEG, or BMP. Check the `RegisterAppInterfaceResponse.displayCapability` properties to find out what image formats the head unit supports.

### Image Sizes
If an image is uploaded that is larger than the supported size, that image will be scaled down to accomodate. All image sizes are available from the `SDLManager`'s `registerResponse` property once in the completion handler for `startWithReadyHandler`.

#### Image Specifications
| ImageName | Used in RPC | Details | Height | Width | Type |
|:--------------|:----------------|:--------|:---------|:-------|:-------|
softButtonImage		 | Show 					  | Will be shown on softbuttons on the base screen														  | 70px         | 70px   | png, jpg, bmp
choiceImage 		 | CreateInteractionChoiceSet | Will be shown in the manual part of an performInteraction either big (ICON_ONLY) or small (LIST_ONLY) | 70px         | 70px   | png, jpg, bmp
choiceSecondaryImage | CreateInteractionChoiceSet | Will be shown on the right side of an entry in (LIST_ONLY) performInteraction						  | 35px 		 | 35px   | png, jpg, bmp
vrHelpItem			 | SetGlobalProperties		  | Will be shown during voice interaction 																  | 35px 		 | 35px   | png, jpg, bmp
menuIcon			 | SetGlobalProperties		  | Will be shown on the “More…” button 																  | 35px 		 | 35px   | png, jpg, bmp
cmdIcon				 | AddCommand				  | Will be shown for commands in the "More…" menu 														  | 35px 		 | 35px   | png, jpg, bmp
appIcon 			 | SetAppIcon				  | Will be shown as Icon in the "Mobile Apps" menu 													  | 70px 		 | 70px   | png, jpg, bmp
graphic 			 | Show 					  | Will be shown on the basescreen as cover art 														  | 185px 		 | 185px  | png, jpg, bmp
