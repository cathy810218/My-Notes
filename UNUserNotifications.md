# User Notifications (iOS 10 swift)

### **How it works?**

Import the new api in `AppDelegate.swift`
```swift
import UserNotifications
```

First you will have to ask the user for notification permission.
```swift
let center = UNUserNotificationCenter.current()
center.delegate = self
center.requestAuthorization(options: [.alert, .sound, .badge]) { (granted, error) in
    if granted == true {
        // If you're sending a REMOTE notification, you will have to ask for device token.
        application.registerForRemoteNotifications()
    }
}
```
**Note: This will prompt a permission alert, if user taps on "Don't Allow",
the user will have to go to Settings to change the permission.**



Once 
`application.registerForRemoteNotifications()` is called,
the below function will be called to get the device token.

```swift
 func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    // TODO: authenticate your token 
}
```

**You will have to have the device token to send a remote notification**



--------------------------------------------------------------------------------------------

## **Local Notification**

1. Create a `UNMutableNotificationContent`:
```swift
let content = UNMutableNotificationContent()
content.title = "Notification title"
content.body = "Notification body"
```

2. Create a `UNTimeIntervalNotificationTrigger` , so you can send the notification in the time interval
that you set (in secs).

```swift
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 60, 
													 repeats: false)
```

3. Create a `UNNotificationRequest`
```swift
let request = UNNotificationRequest(identifier: "identifier", 
									   content: content, 
                                       trigger: trigger)
```

4. Then finally, add the request to the notification center
````swift
UNUserNotificationCenter.current().add(request) { error in
	
}
````

When localizing notification string, you may use
`String.localizedUserNotificationString(forKey: "your_key", arguments: [])`

There are 3 types of NotificationTrigger:

* UNTimeIntervalNotificationTrigger
* UNCalendarNotificationTrigger
* UNLocationNotificationTrigger



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
    "sound":"default", // or myMusic.caf
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

-----------------------------------------------------------------------------------

## **Notification Action**

Register notification category with actions:

````swift
private func registerNotificationCategory() {
    let myCategory: UNNotificationCategory = {
        // 1
        let inputAction = UNTextInputNotificationAction(
            identifier: "action.input",
            title: "Input",
            options: [.foreground],
            textInputButtonTitle: "Send",
            textInputPlaceholder: "What do you want to say...")

        // 2
        let goodbyeAction = UNNotificationAction(
            identifier: "action.goodbye",
            title: "Goodbye",
            options: [.foreground])

        let cancelAction = UNNotificationAction(
            identifier: "action.cancel",
            title: "Cancel",
            options: [.destructive])

        // 3
        return UNNotificationCategory(identifier:"myCategory", actions: [inputAction, goodbyeAction, cancelAction], intentIdentifiers: [], options: [.customDismissAction])
    }()

    UNUserNotificationCenter.current().setNotificationCategories([myCategory])
}
````


When creating notification content, set

`content.categoryIdentifier = "myCategory"`


Then in the remote notification payload, just add 
````
{
  "aps":{
    "alert":"Please say something",
    "category":"myCategory"
  }
}
````


Handling actions

````swift
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {

    if let category = UserNotificationCategoryType(rawValue: response.notification.request.content.categoryIdentifier) {
        switch category {
        case .myCategory:
            handleMyCategory(response: response)
        }
    }
    completionHandler()
}

private func handleMyCategory(response: UNNotificationResponse) {
    let text: String

    if let actionType = myCategoryAction(rawValue: response.actionIdentifier) {
        switch actionType {
        case .input: text = (response as! UNTextInputNotificationResponse).userText
        case .goodbye: text = "Goodbye"
        case .none: text = ""
        }
    } else {
        // Only tap or clear. (You will not receive this callback when user clear your notification unless you set .customDismissAction as the option of category)
        text = ""
    }

    if !text.isEmpty {
        UIAlertController.showConfirmAlertFromTopViewController(message: "You just said \(text)")
    }
}
````




------------------------------------------------------------------------------------

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

Go to your `NotificationService.swift`

````swift
class NotificationService: UNNotificationServiceExtension {

        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        
        // Need to create a local container URL in order to share downloaded media between targets
        let containerURL: URL? = FileManager.default.containerURL(forSecurityApplicationGroupIdentifier: "group.cardinalblue.com.app.series")
        var attachement: UNNotificationAttachment? = nil
        
        //  use semaphore convert async download to sync
        let semaphore = DispatchSemaphore(value: 0)
        if let imageURLString = bestAttemptContent?.userInfo["attachment"] as? String {
            let url = URL(string: imageURLString)!
            let localURL = containerURL?.appendingPathComponent(url.lastPathComponent)
            URLSession.downloadImage(atURL: url, withCompletionHandler: { (data, error) in
                if (error == nil) {
                    // wrtie to app group
                    try? data?.write(to: localURL!)
                    
                    // create attachment
                    attachement = try! UNNotificationAttachment(identifier: "attachment", url: localURL!, options: nil)
                    semaphore.signal()
                }
            })
        }
        semaphore.wait()
    
        //  success set attachments
        if let bestAttemptContent = bestAttemptContent {
            bestAttemptContent.attachments = [attachement!]
            contentHandler(bestAttemptContent) // content extension now will have the attachment
        }

}
````

In order to show more when you expend the remote notification,
you will have to add `"mutable-content"` in the payload.
````
{
  "aps":{
    "alert":{
      "title":"Image Notification",
      "body":"Show me an image from web!"
    },
    "mutable-content":1 // This tells the api I have an image attachment to display
  },
  "attachment-url": "https://myimageurl.com/myimage.jpg // Then it'll read this url
}
````



* ### **UNNotificationContentExtension**

After the service extension reads and downloads the media from `"attachment-url"` and passes in to content extension, content extension will be able to modify the notifications layouts.

Because in the service extension, you've already downloaded the media, and saves it in the local container, you now can read the downloaded image directly.

**Note: In order to read the image downloaded from service extension, you have to create an app group.**

```swift
func didReceive(_ notification: UNNotification) {
    let content = notification.request.content
    self.label?.text = content.body

    if let attachment = content.attachments.first {
        if attachment.url.startAccessingSecurityScopedResource() {
            DispatchQueue.main.async {
                self.imageView.image = UIImage(contentsOfFile: attachment.url.path)
                attachment.url.stopAccessingSecurityScopedResource()
            }
        }  
    }
}
```



---------------------

Source:
1. https://onevcat.com/2016/08/notification/
2. https://github.com/maquannene/UserNotifications


