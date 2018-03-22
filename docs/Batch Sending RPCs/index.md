## Batch Sending RPCs
There are two ways to send multiple requests to the head unit: concurrently or sequentially. Which method you should use depends on the type of RPCs being sent. Concurrently sent requests will not finish in any set order and should only be used when none of the requests depend on the result of another, such as when uploading group of artworks. Sequentially sent requests only send the next request in the group when a response has been received for the previously sent RPC. Requests should be sent sequentially when Core expects RPCs to be sent in a particular order like when sending the several different requests needed to create a menu.

Both methods have optional progress and completion handlers. Use the optional `progressHandler` to check the status of each sent RPC. The `progressHandler` will tell you if there was an error sending the request and what percentage of the group has completed sending. The optional `completionHandler` is called when all RPCs in the group have been sent. Use it to check if all of the requests have been sent successfully or not.

### Send Concurrent Requests
When you send multiple RPCs concurrently there is no guarantee of the order in which the RPCs will be sent or in which order Core will return responses.

#### Objective-C
```objc
[self.sdlManager sendRequests:@[<#SDLRPCRequest#>] progressHandler:^(__kindof SDLRPCRequest * _Nonnull request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error, float percentComplete) {
    NSLog(@"Command %@ sent %@, percent complete %f%%", request.name, response.resultCode == SDLResultSuccess ? @"successfully" : @"unsuccessfully", percentComplete * 100);
} completionHandler:^(BOOL success) {
    NSLog(@"All requests sent %@", success ? @"successfully" : @"unsuccessfully");
}];
```

#### Swift
```swift
sdlManager.send([<#SDLRPCRequest#>], progressHandler: { (request, response, error, percentComplete) in
    print("Command \(request.name) sent \(response?.resultCode == .success ? "successfully" : "unsuccessfully"), percent complete \(percentComplete * 100)")
}) { success in
    print("All requests sent \(success ? "successfully" : "unsuccessfully")")
}
```

### Send Sequential Requests
Requests sent sequentially are sent in the same order as they were added to the array. Only when a response has been received for the previously sent request, is the next request sent.

#### Objective-C
```objc
[self.sdlManager sendSequentialRequests:@[<#SDLRPCRequest#>] progressHandler:^BOOL(__kindof SDLRPCRequest * _Nonnull request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error, float percentComplete) {
    NSLog(@"Command %@ sent %@, percent complete %f%%", request.name, response.resultCode == SDLResultSuccess ? @"successfully" : @"unsuccessfully", percentComplete * 100);
} completionHandler:^(BOOL success) {
    NSLog(@"All requests sent %@", success ? @"successfully" : @"unsuccessfully");
}];
```

#### Swift
```swift
sdlManager.sendSequential(requests: [<#SDLRPCRequest#>], progressHandler: { (request, response, error, percentageCompleted) -> Bool in
    print("Command \(request.name) sent \(response?.resultCode == .success ? "successfully" : "unsuccessfully"), percent complete \(percentComplete * 100)")
}) { success in
    print("All requests sent \(success ? "successfully" : "unsuccessfully")")
}
```
