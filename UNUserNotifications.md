# User Notifications (iOS 10)

### **How it works?**

Import the new api in `AppDelegate.swift`
```Objective-c
#import <UserNotificationsUI/UserNotificationsUI.h>
```

First you will have to ask the user for notification permission.
Since UserNotifications only works for iOS10, we check user's operating system first.
Put the code below in `- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`

```Objective-C
if ([NSProcessInfo.processInfo isOperatingSystemAtLeastVersion:(NSOperatingSystemVersion){10,0,0}]) {
        UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
        center.delegate = self;
        [center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert + UNAuthorizationOptionSound + UNAuthorizationOptionBadge)
                              completionHandler:^(BOOL granted, NSError * _Nullable error) {
                                  if (granted) {
                                      NSLog(@"permission!!");
                                      [[UIApplication sharedApplication] registerForRemoteNotifications];
                                  } else {
                                      NSLog(@"no permission");
                                  }
                              }];
        [center setNotificationCategories:[self createNotificationActionCategories]];
    }
```
**Note: This will prompt a permission alert, if user taps on "Don't Allow",
he/she will have to go to Settings to change the permission.**

Once 
`if (granted)`, it will call `- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken`

```objective-c
 func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    // TODO: authenticate your token 
}
```

**You will have to have the device token to send a remote notification**

## UNUserNotificationCenterDelegate method

This only works for iOS10

```Objective-C
- (void)userNotificationCenter:(UNUserNotificationCenter *)center
didReceiveNotificationResponse:(UNNotificationResponse *)response
         withCompletionHandler:(void(^)())completionHandler
{
    NSDictionary *userInfo = response.notification.request.content.userInfo;
    
    if ([UIApplication sharedApplication].applicationState == UIApplicationStateActive) {
        // Ignore push notification, but update badge count
        [self updateBadge];
    } else { 
        // in background mode do something
        if (completionHandler) {
            NSString *clickURL = userInfo[@"clickURL"];
            if (clickURL) {
                // deeplink to clickURL
            }
            completionHandler();
        }
    }
}
```

## UNUserNotificationActions
In order to add the buttons under the notification, we will need to create
action categories.

```objective-c
- (NSSet<UNNotificationCategory *> *)createNotificationActionCategories
{
    UNNotificationAction *acceptAction = [UNNotificationAction actionWithIdentifier:@"accept"
                                                                              title: I18n(@"Buy",nil)
                                                                            options:UNNotificationActionOptionForeground];
    // create notification category
    UNNotificationCategory *stickerCategory = [UNNotificationCategory categoryWithIdentifier:@"sticker"
                                                                                     actions:@[acceptAction]
                                                                           intentIdentifiers:@[]
                                                                                     options:UNNotificationCategoryOptionNone];
    UNNotificationCategory *noActionCategory = [UNNotificationCategory categoryWithIdentifier:@"regular"
                                                                                      actions:@[]
                                                                            intentIdentifiers:@[]
                                                                                      options:UNNotificationCategoryOptionNone];
    
    return [NSSet setWithArray:@[stickerCategory, noActionCategory]];
}
```
This method is being called when asking user for permission.
`[center setNotificationCategories:[self createNotificationActionCategories]];`
in `- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`

-------------------------------------------------------------------------------------

## **Remote Notification**
The new payload allows you to add title, subtitle and body now.
It will look like this
````
{
  "aps":{
    "alert":{
      "title":"Title",
      "subtitle":"Subtitle",
      "body":"Body"
    },
    "sound":"default", // ex. myMusic.caf
    "badge":1
  }
}
````

For localizing remote notification, 
````
"aps": {
    "badge": 1,
    "alert": {
        "loc-key": "localized-body-key",
        "title-loc-key": "localized-title-key"
    }
}
````
-------------------------------------------------------------------------------------

Now, on to the extensions implementation!

-----------------------------------------------------------------------------------

# **Notification Extension**

The new features in User Notifications includes:
1. UNNotificationAttachment
3. UNNotificationServiceExtension
4. UNNotificationContentExtension

Add new notifications targets to continue the below steps.

* ### **UNNotification Attachment**
Apple now supports audio, video, and image/gif to be attached in a notification.
With 3D touch on the notification, user will be able to see the full-size of the attachment.


* ### **UNNotificationServiceExtension**
Now when a user receives a remote notification payload,
BEFORE the notification shows up, the service extension will be called.
It reads the notification payload and downloads the media through the given URL to display
in the notification.

Note: You only have 30 secs to do everything before the notification shows up.

Go to your `NotificationService.m`

First add a category for `UNNotificationAttachment` so we can create the attachment for different media and save it to the disk.

````Objective-C
@interface UNNotificationAttachment (PIC)
// save file to disk
+ (UNNotificationAttachment *)createAttachmentWithIdentifier:(NSString *)identifier imageData:(NSData *)attachmentData fileString:(NSString *)fileString;
@end

@implementation UNNotificationAttachment (PIC)
+ (UNNotificationAttachment *)createAttachmentWithIdentifier:(NSString *)identifier imageData:(NSData *)attachmentData fileString:(NSString *)fileString
{
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSString *subFolderName = @"notification";
    NSURL *tmpFolderURL = [[NSURL fileURLWithPath:NSTemporaryDirectory()]
                           URLByAppendingPathComponent:subFolderName
                           isDirectory:YES];
    NSError *error;
    [fileManager createDirectoryAtURL:tmpFolderURL
          withIntermediateDirectories:YES
                           attributes:nil
                                error:&error];
    
    NSURL *fileURL = [tmpFolderURL URLByAppendingPathComponent:fileString];
    if ([fileManager fileExistsAtPath:fileURL.path]) {
        [fileManager removeItemAtPath:fileURL.path error:nil];
    }
    if ([attachmentData writeToURL:fileURL atomically:YES]) {
        UNNotificationAttachment *attachment =
        [UNNotificationAttachment attachmentWithIdentifier:identifier
                                                       URL:fileURL
                                                   options:nil
                                                     error:nil];
        return attachment;
    }
    return nil;
}
@end
````

And now inside `NotificationService` implementation 

````Objective-C
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request
                   withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler
{
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];

    // creates an array of attachments
    NSMutableArray<UNNotificationAttachment*> *attachments = [[NSMutableArray alloc] init];
    
    UNMutableNotificationContent *content = [request.content mutableCopy];
    
    // This is the icon image shows up in the push notification
    // content.userInfo is the payload we get from the server
    NSURL *iconURL = [NSURL URLWithString:content.userInfo[@"icon"]];
    if (iconURL) {
        NSData *iconData = [NSData dataWithContentsOfURL: iconURL];
        UNNotificationAttachment *iconAttachment =
        [UNNotificationAttachment createAttachmentWithIdentifier:@"icon"
                                                      imageData:iconData
                                                     fileString:iconURL.lastPathComponent];
        // if it exists, add it to the attachments array
        if (iconAttachment) {
            [attachments addObject:iconAttachment];
        }
    }
    
    // This is the image that shows up when user pulls down/3d touch the notification
    NSURL *imageURL = [NSURL URLWithString:content.userInfo[@"image"]];
    if (imageURL) {
        NSData *imageData = [NSData dataWithContentsOfURL:imageURL];
        UNNotificationAttachment *imageAttachment =
        [UNNotificationAttachment createAttachmentWithIdentifier:@"image"
                                                  imageData:imageData
                                                 fileString:imageURL.lastPathComponent];
        if (imageAttachment) {
            [attachments addObject:imageAttachment];
        }
    }
    
    // This is the audio that plays when user pulls down/3d touch the notification
    NSURL *audioURL = [NSURL URLWithString:content.userInfo[@"audio"]];
    if (audioURL) {
        NSData *audioData = [NSData dataWithContentsOfURL:audioURL];
        UNNotificationAttachment *audioAttachment =
        [UNNotificationAttachment createAttachmentWithIdentifier:@"audio"
                                                       imageData:audioData
                                                      fileString:audioURL.lastPathComponent];
        if (audioAttachment) {
            [attachments addObject:audioAttachment];
        }
    }
    
    // This sends the attachments to content extension
    content.attachments = attachments;
    contentHandler([content copy]);
}

````

In order to display rich notification, you will have to add `"mutable-content"` in the payload.

````
{
  "aps":{
    "alert":{
      "title":"Image Notification",
      "body":"Show me an image from web!"
    },
    "mutable-content":1 // This tells the api I have an image attachment to display
  },
  "icon": "https://68.media.tumblr.com/76c96e72a43dd60fca4e8321b37b5c84/tumblr_nzkmnoz0gf1u7gnm9o1_500.gif",
  "image": "https://68.media.tumblr.com/76c96e72a43dd60fca4e8321b37b5c84/tumblr_nzkmnoz0gf1u7gnm9o1_500.gif",
  "audio": "https://cardinalblue-personal.s3.amazonaws.com/yy-jim/xmas.mp3"}
}
````

------------------------------------------------------------------------------------------
* ### **UNNotificationContentExtension**

In order to make your action buttons show up on the notification,
do the following steps in the `info.plist`:

Since we add actions using categories, we will need to configure the `info.plist` in `NotificationContentExtension`.
Under `NSExtension` (Dictionary) -> `NSExtensionAttributes` (Dictionary) add 
`UNNotificationExtensionCategory` (Array)
Then add your category (String).

And in the remote notification payload, just add 

````
{
  "aps":{
    "alert":{
      "title":"Image Notification",
      "body":"Show me an image from web!"
    },
    "mutable-content":1, // This tells the api I have an image attachment to display
    "category":"sticker"
  },
  "icon": "https://68.media.tumblr.com/76c96e72a43dd60fca4e8321b37b5c84/tumblr_nzkmnoz0gf1u7gnm9o1_500.gif",
  "image": "https://68.media.tumblr.com/76c96e72a43dd60fca4e8321b37b5c84/tumblr_nzkmnoz0gf1u7gnm9o1_500.gif",
  "audio": "https://cardinalblue-personal.s3.amazonaws.com/yy-jim/xmas.mp3"}
}
````

If you want to customize your notification content in content extension,
again, in the `info.plist`, go to `NSExtension` -> `NSExtensionAttributes` add 
`UNNotificationExtensionDefaultContentHidden` (Boolean) set it to `YES`.
Then it will hide the default content.


In the NotificationViewController.m file 

```Objective-C
- (void)didReceiveNotification:(UNNotification *)notification {
    UNNotificationContent *content = notification.request.content;

    // Check each attachment in the attachments array
    for (UNNotificationAttachment *attachment in content.attachments) {
        if ([attachment.identifier isEqualToString:@"audio"]) {
            if ([attachment.URL startAccessingSecurityScopedResource]) { // This is very important!!!
                self.audioPlayer = [[AVAudioPlayer alloc] initWithContentsOfURL:attachment.URL error:nil];
                [[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayback error:nil];
                [[AVAudioSession sharedInstance] setActive: YES error: nil];
            }
            [attachment.URL stopAccessingSecurityScopedResource]; // This is very important!!!
            [self.audioPlayer play];
        } else {
            if ([attachment.URL startAccessingSecurityScopedResource]) {
                NSData *imageData = [NSData dataWithContentsOfURL:attachment.URL];
                dispatch_async(dispatch_get_main_queue(), ^{
                    if ([attachment.type isEqualToString:@"com.compuserve.gif"]) {
                        self.imageView.image = [UIImage animatedImageWithAnimatedGIFData:imageData];
                    } else {
                        self.imageView.image = [UIImage imageWithData:imageData];
                    }
                    self.titleLabel.text = content.title;
                    self.bodyLabel.text = content.body;
                });
                [attachment.URL stopAccessingSecurityScopedResource];
            }
        }
    }
}
```

For actions:

```Objective-C
- (void)didReceiveNotificationResponse:(UNNotificationResponse *)response completionHandler:(void (^)(UNNotificationContentExtensionResponseOption))completion
{
    NSString *actionIdentifier = response.actionIdentifier;
    if ([actionIdentifier isEqualToString:@"accept"]) {
        completion(UNNotificationContentExtensionResponseOptionDismissAndForwardAction); // Forward the actions to the app!
    }
}
```

We also created a `UIImage` category for displaying GIF on `UIImageView`.


---------------------

Source:

1. https://onevcat.com/2016/08/notification/
2. https://github.com/maquannene/UserNotifications
3. https://blog.pusher.com/how-to-send-ios-10-notifications-using-the-push-notifications-api/
4. https://github.com/nomad/houston
