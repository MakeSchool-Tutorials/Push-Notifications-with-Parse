---
title: "Push Notifications with Parse"
slug: push-notifications-with-parse
---

In order to keep users engaged with your apps, it is helpful to send users messages while outside of your running apps.  Apple provides a mechanism to do this, and its called a push notification.  However, this requires some infrastructure (i.e., servers) and is a time consuming task when done with a small team.  Fortunately, Parse provides the infrastructure and has simplified the process of sending push notifications.

This guide will help you get started with push notifications and help you use it along with your Parse backend.  So this assumes that the code will be added to an existing project with core Parse functionality implemented.  If you don't have a project with Parse implemented and are looking for a place to start, check out our [Makestagram tutorial](https://www.makeschool.com/tutorials/build-a-photo-sharing-app-part-1/getting-started).  
​
## What Is A Push Notification?

You have most likely seen them before. They are the banners or alerts you see appear on your phone screen whenever someone messages you. Games even use them to notify you that you have won a game or it is your turn to play.
​
## How Do Push Notifications Work?

When someone is sending a message to someone else, the backend receives this message and saves it in the database. In addition, the backend needs to notify the receiving user that there is a message for them to look at. In order to do that, your backend (or Parse) has to bundle a message for the receiving user and send it to Apple Push Notification Service (APNS). Once the APNS receives this bundle, it will send a push notification to the user's phone, and that is when they see a banner show up on their phone.
​
## Let's Get Started

First, we need to set up some certificates to allow our backend to talk to APNS securely and to make sure our backend is the only one able to send messages to APNS on our behalf. Please go to [Parse Push Notifications](https://parse.com/tutorials/ios-push-notifications) in order to get started with the certificates. Once you have reached step 5, please come back and continue this tutorial.
​
## Registering User's Device For Push Notifications

Now that we have the certificates set up, it is time to write some code! First off, we need to ask the user for permission to send them notifications.
​
> [action]
> Go to your AppDelegate.swift file and within the didFinishLaunchingWithOptions function, write this code:
>
>		let settings = UIUserNotificationSettings(forTypes: [.Alert, .Sound, .Badge], categories: nil)
>       UIApplication.sharedApplication().registerUserNotificationSettings(settings)
>       UIApplication.sharedApplication().registerForRemoteNotifications()

As soon as the registerForRemoteNotifications function is called, the user gets an alert to enable push notifications. If you have a login page, it is best to run the above code after the user has logged in for better user experience.
​
## Providing Backend With Device Information
​
> [action]
> In your AppDelegate.swift, write this function:
>
>       func application(application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: NSData) {
>           let installation = PFInstallation.currentInstallation()
>           installation["user"] = PFUser.currentUser()
>           installation.setDeviceTokenFromData(deviceToken)
>           installation.saveInBackground()
>       }

PFInstallation.currentInstallation() represents the current device. We save within it user's unique device token that is provided by the iOS to the app, which APNS will use to send push to the right device. We also save the current user within it to be able to distinguish the installation against other users' installations.
​
## Getting App Ready To Receive Notifications
​
> [action]
> In your AppDelegate.swift, write this function:
>
>       func application(application: UIApplication, didReceiveRemoteNotification userInfo: [NSObject : AnyObject]) {
>           PFPush.handlePush(userInfo)
>       }

If the app is active, the above function will be called to notify the app that a push notification is received, and handlePush will simply show an alert to the user. You may change the code within this function to deal with incoming push notifications anyway you'd like. The only problem with this function is that, it does not get called when the user is outside of the app and taps on the banner of push notification. In order to catch that action, we need to call this function manually when the app is launched due to a push notification.
​
> [action]
> In your didFinishLaunchingWithOptions function, write this code:
>
>		if let launchOptions = launchOptions as? [String : AnyObject] {
>          	if let notificationDictionary = launchOptions[UIApplicationLaunchOptionsRemoteNotificationKey] as? [NSObject : AnyObject] {
> 				self.application(application, didReceiveRemoteNotification: notificationDictionary)
>           }
>       }

The above nested if-statement simply checks if the app was launched because of a notification, and if so, we call our didReceiveRemoteNotification function.
​
## How To Trigger A Push Notification

In order to send a push notification, first we need to grab the PFInstallation for the receiving user. In order to do that, we will first query for that PFInstallation. Note that PFInstallations are just like other objects on the Parse backend. You will see a table of them appear on your project website when a PFInstallation object is created.
​
> [action]
> Querying a PFInstallation object is like querying any other object:
>
>       let pushQuery = PFInstallation.query()!
>       pushQuery.whereKey("user", equalTo: friend) //friend is a PFUser object
>
> To actually send a push notification to that PFInstallation:
>
>       let push = PFPush()
>       push.setQuery(pushQuery)
>       push.setMessage("New message from \(PFUser.currentUser()!.username!)")
>       push.sendPushInBackground()

Once the above code runs, your friend should get a push notification on her device.
​
## Badges

It is common to also increment the badge number on the app icon to indicate to the user that a message is waiting for them.
​
> [action]
> Change your code to this:
>
>       let data = ["alert" : "New message from \(PFUser.currentUser()!.username!)", "badge" : "Increment"]
>       let push = PFPush()
>       push.setQuery(pushQuery)
>       push.setData(data)
>       push.sendPushInBackground()

It is also necessary to reset the badge number to zero and make the badge icon disappear, when the user opens the app or sees the new message.
​
> [action]
> Add this function to your AppDelegate.swift:
>
> 		func clearBadges() {
>        	let installation = PFInstallation.currentInstallation()
>	        installation.badge = 0
>	        installation.saveInBackgroundWithBlock { (success, error) -> Void in
>	            if success {
>	                println("cleared badges")
>	                UIApplication.sharedApplication().applicationIconBadgeNumber = 0
>	            }
>	            else {
>	                println("failed to clear badges")
>	            }
>         }
>  		}


The above function will reset the badge number of the PFInstallation, and once that succeeds, we reset the badge number locally so the badge icon disappears off the app icon.
​
> [action]
> Call clearBadges() whenever your app becomes active:
>
>		func applicationDidBecomeActive(application: UIApplication) {
>        	clearBadges()
>		}

## Summary

Push notifications are a necessary part of keeping users engaged in your apps.  Although difficult to implement from scratch, Parse offers a service that simplifies sending notifications.  After providing the APNS certificate to Parse, only a small amount of code needs to be added to an existing Parse enabled project.  This code registers the device for notifications, saves device information to Parse, sends notifications and manages recieved notifications.  Using push notifications effectively is a great way to increase user retention.  Happy coding!
