# Uploading Files
In almost all cases, graphics are uploaded using the `ScreenManager`. You can find out about setting images in templates, soft buttons, and menus in the [Text Images and Buttons](Displaying a User Interface/Text Images and Buttons) guide. Other situations, such as `PerformInteraction`s, VR help lists, and turn by turn directions, are not currently covered by the `ScreenManager`. To upload an image, see the [Uploading Images](Other SDL Features/Uploading Images) guide.

## Uploading an mp3 Using SDLFileManager
The `SDLFileManager` uploads files and keeps track of all the uploaded files names during a session. To send data with the `SDLFileManager`, you need to create either a `SDLFile` or `SDLArtwork` object. `SDLFile` objects are created with a local `NSURL` or `NSData`; `SDLArtwork` uses a `UIImage`.

##### Objective-C
```objc
NSData *mp3Data = <#Get the File Data#>;
SDLFile *file = [SDLFile fileWithData:mp3Data name:<#File name to be referenced later#> fileExtension:<#File Extension#>];

[self.sdlManager.fileManager uploadFile:file completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
    if (error != nil) { return; }
    <#File Upload Successful#>
}];]
```

##### Swift
```swift
let mp3Data = <#Get MP3 Data#>
let file = SDLFile(data: mp3Data, name: <#File name#>, fileExtension: <#File Extension#>)

sdlManager.fileManager.upload(file: file) { (success, bytesAvailable, error) in
    guard error == nil else { return }
    <#File Upload Successful#>
}
```

## Batch File Uploads
If you want to upload a group of files, you can use the `SDLFileManager`'s batch upload methods. Once all of the uploads have completed you will be notified if any of the uploads failed. If desired, you can also track the progress of each file in the group.

##### Objective-C
```objc
SDLFile *file1 = [SDLFile fileWithData:<#Data#> name:<#File name to be referenced later#> fileExtension:<#File Extension#>];
SDLFile *file2 = [SDLFile fileWithData:<#Data#> name:<#File name to be referenced later#> fileExtension:<#File Extension#>];

[self.sdlManager.fileManager uploadFiles:@[file1, file2] progressHandler:^BOOL(SDLFileName * _Nonnull fileName, float uploadPercentage, NSError * _Nullable error) {
    <#Called as each upload completes#>
    // Return true to continue sending files. Return false to cancel any files that have not yet been sent.
    return YES;
} completionHandler:^(NSError * _Nullable error) {
    <#Called when all uploads complete#>
}];
```

##### Swift
```swift
let file1 = SDLFile(data: <#Data#>, name: <#File name to be referenced later#>, fileExtension: <#File Extension#>)
let file2 = SDLFile(data: <#Data#>, name: <#File name to be referenced later#>, fileExtension: <#File Extension#>)

sdlManager.fileManager.upload(files: [file1, file2], progressHandler: { (fileName, uploadPercentage, error) -> Bool in
    <#Called as each upload completes#>
    // Return true to continue sending files. Return false to cancel any files that have not yet been sent.
    return true
}) { (error) in
    <#Called when all uploads complete#>
}
```

## File Persistance
`SDLFile` and its subclass `SDLArtwork` support uploading persistant files, i.e. files that are not deleted when the car turns off. Persistance should be used for files that will be used every time the user opens the app. If the file is only displayed for short time the file should not be persistant because it will take up unnecessary space on the head unit. You can check the persistence via:

##### Objective-C
```objc
if (file.isPersistent) {
    <#File was initialized as persistent#>
}
```

##### Swift
```swift
if file.isPersistent {
    <#File was initialized as persistent#>
}
```

!!! NOTE
Be aware that persistance will not work if space on the head unit is limited. `SDLFileManager` will always handle uploading images if they are non-existent.
!!!

## Overwriting Stored Files
If a file being uploaded has the same name as an already uploaded file, the new file will be ignored. To override this setting, set the `SDLFile`’s `overwrite` property to true.

##### Objective-C
```objc
file.overwrite = YES;
```

##### Swift
```swift
file.overwrite = true
```

## Checking the Amount of File Storage
To find the amount of file storage left for your app on the head unit, use the `SDLFileManager`’s `bytesAvailable` property.

##### Objective-C
```objc
NSUInteger bytesAvailable = self.sdlManager.fileManager.bytesAvailable;
```

##### Swift
```swift
let bytesAvailable = sdlManager.fileManager.bytesAvailable
```

## Checking if a File Has Already Been Uploaded
You can check out if an image has already been uploaded to the head unit via the `SDLFileManager`'s `remoteFileNames` property.

##### Objective-C
```objc
BOOL isFileOnHeadUnit = [self.sdlManager.fileManager.remoteFileNames containsObject:<#Name Uploaded As#>];
```

##### Swift
```swift
let isFileOnHeadUnit = sdlManager.fileManager.remoteFileNames.contains(<#Name Uploaded As#>)
```

## Deleting Stored Files
Use the file manager’s delete request to delete a file associated with a file name.

##### Objective-C
```objc
[self.sdlManager.fileManager deleteRemoteFileWithName:@"<#Save As Name#>" completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError *error) {
    if (success) {
        <#Image was deleted successfully#>
    }
}];
```

##### Swift
```swift
sdlManager.fileManager.delete(fileName: "<#Save As Name#>") { (success, bytesAvailable, error) in
    if success {
        <#Image was deleted successfully#>
    }
}
```

## Batch Deleting Files
##### Objective-C
```objc
[self.sdlManager.fileManager deleteRemoteFileWithNames:@[@"<#Save As Name#>", @"<#Save As Name 2#>"] completionHandler:^(NSError *error) {
    if (error == nil) {
        <#Images were deleted successfully#>
    }
}];
```

##### Swift
```swift
sdlManager.fileManager.delete(fileNames: ["<#Save As Name#>", "<#Save as Name 2#>"]) { (error) in
    if (error == nil) {
        <#Images were deleted successfully#>
    }
}
```

