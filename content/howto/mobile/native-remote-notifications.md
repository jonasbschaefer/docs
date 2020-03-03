---
title: "Use Remote Notifications"
#category: "Native Mobile"
#parent: "native-mobile"
#menu_order: 11
#description: "Learn to set up native push notifications with the Native builder."
#tags: ["mobile", "push notification", "remote", "push", "notification"]
---

## 1 Introduction

This tutorial will teach you to configure and test remote notifications. Remote notifications only work with apps created through the Native Builder. You cannot send notifications to an app inside the Make it Native app.

## 2 Prerequisites

If you want to use push notifications with custom apps which created with native builder, make sure you have completed the following prerequisite:

* Complete [How to Deploy Your First Mendix Native App](/howto/mobile/deploying-native-app) through the end of the *Making Your First Build* section

## 3 Setting Up Your Mendix Project

Create a Mendix project using the Native Mobile Quickstart starter app:

1. Open Mendix Studio Pro. **Select File** > **New Project** , and then select the **Native Mobile Quickstart app**.
2. Click **Use this starting point**.
3. Click **Create app** to close the dialog box.

### 3.1 Installing Your Module 

1. Add the [Community Commons](https://appstore.home.mendix.com/link/app/170/) module to your app project.
2. Add the [Encryption](https://appstore.home.mendix.com/link/app/1011/) module to your app project.
3. Set the encryption private key by doing the following:<br />
	a. Double-click **Settings**.<br />
	b. Click **Edit**.<br />
	c. Click **Constants**.<br />
	d. Click **New**.<br />
	e. Type *encryptionkey*, select it, and add a 16 character **Value**:

	![Capabilities](attachments/native-remote-push/modeler/setEncryption.png)

3. Add the [Push Notifications Connector](https://appstore.home.mendix.com/link/app/3003/) module to your app project.

### 3.2 Set Up a Notification Widget

1. Drag and drop an app events widget onto your app's home page, double-click it, and do the following:<br />
    a. Set **Page load** > **On load** to **Call a nanoflow**, then specify **PushNotifications.OnPageLoad_RegisterPushNotifications**.<br />
    b. Set **App resume** > **On resume** to **Call a nanoflow**, then specify **PushNotifications.OnPageLoad_RegisterPushNotifications**:<br />
    
    ![AppEvents](attachments/native-remote-push/modeler/AppEvents.png)

    This configured app events widget will allow for user devices to register with your notification interface, so that you can choose who to send push notifications to. Specifically it will register devices when they open the app or resume the app.

2. Create an entity called *NativeNotification* in your domain model with one `objectGUID` field:

    ![NotificationEntity](attachments/native-remote-push/modeler/NotificationEntity.png)

3. Create a new *DS_Notification* nanoflow which creates a **NativeNotification** entity object and then returns it:

    ![DS_Notification](attachments/native-remote-push/modeler/DS_Notification.png)

4. Drag and drop a data view widget onto your home page, set its **Data source** > **Nanoflow** to **DS_Notification**:

    ![Dataview](attachments/native-remote-push/modeler/Dataview.png)

5. Inside of this data view, drag and drop a notifications widget.
6. Set the notifications widget's GUID to **NativeNotification.objectGUID**:

    ![NotificationsGUID](attachments/native-remote-push/modeler/NotificationsGUID.png)

    This will allow you to pass objects with notifications.

7.  Open **Navigation**, in the **Responsive** pane click **New Item**, then add a new **Show page** item **PushNotifications/_USE ME/Administration**: 

    ![ProfileHomePage](attachments/native-remote-push/modeler/ProfileHomePage.png)

### 3.3 Adding Actions to Your Notifications Widget

To set up actions which will occur after tapping or receiving a notification, do the following:

1. Create two nanoflows (*ACT_OnRecieve* and *ACT_OnOpen*) which will create two different logs (**onRecieve triggered** and **onOpen triggered**):

    ![ACT_OnRecieve](attachments/native-remote-push/modeler/ACT_OnRecieve.png)

2. Double-click your notifications widget and do the following:<br />
    a. Add an action called *logIt*.<br />
    b. For **On recieve** select **ACT_OnRecieve**.<br />
    c. For **On open** select **ACT_OnOpen**:

    ![LogitAction](attachments/native-remote-push/modeler/logitAction.png)

### 3.4 Adding Firebase Configurations

Deploy your project and open your administration page in a web browser. Do the following:

1. Add a new FCM configuration.
2. Select **Enabled**.
3. Name your configuration.
4. Set it as **Development** (this will not affect any functionality, it is only a label).
5. Set the **Project id** to the project id you set in the [Native Apps](setting-up-google-firebase-cloud-messaging-server#native-apps) section of *How to Set Up the Google Firebase Cloud Messaging Server* [todo: fix link].
6.  Upload your private key (which you got in the [SOMETHING] section of *Setting up Remote Push Notifications* [todo: fix link]:

    ![FCMConfig](attachments/native-remote-push/modeler/FCMConfig.png)

7. For both your Android and iOS **Messaging service settings**, select your FCM configuration.
8.  For the Android and iOS **Messaging service types**, select FCM:

    ![FCMConfig2](attachments/native-remote-push/modeler/FCMConfig2.png)

Next you will test the implementation of your configurations.

### 3.5 Sending a Push Notification to a Single Device {#sending-single}

1. Reload the app on your phone.
2. Put the app in the background by returning to your phone's home screen.
3. On your desktop browser, open the **Administration** page and click the **Devices** tab.

Now you should be able to see registered devices (which is probably only one: your testing device):

1. Click your device listed under **Registered Devices** and click **New Message**.
2.  Type some text into **Title** and **Body** fields, and in **Action name** type *logIt*:

    ![SimpleMessage](attachments/native-remote-push/modeler/SimpleMessage.png)

3. Click **Send**.

You will see your notification with the text you configured:

    ![PushRecieved](attachments/native-remote-push/modeler/PushRecieved.png)

1. Tap the notification. You will see a log message in your modeler console: **onOpen triggered**.
2. Now send and tap a notification while keeping the app open. You will see a different log in your modeler console: **onRecieve triggered**. 

## 4 Sending Data Using Push Notifications

Imagine your business has several products, and you want to send one product to a user using an administration module interface. How would you achieve this goal?

In this section you will learn the following:

* How to show a push notification to a user if their app is in the backgroud — when a user taps it, they will be brought to a product page
* How to show a small view to a user if their app is in the foreground — when a user taps the button in the animation, they will be brought to a product page

### 4.1 Push Notifications for an App in the Background

#### 4.1.1 Setting Up an Example Entity

1. In the **Domain Model**, add a *Product* entity with a *ProductName* attribute.
2.  Right-click **Product** > **Generate overview pages**
3. Make sure the **Navigation layouts** are Atlas layouts. Click **OK** which will make the pages **Product_NewEdit** and **Product_Overview**:

    ![GeneratePages](attachments/native-remote-push/modeler/GeneratePages.png)

    ![GeneratePages](attachments/native-remote-push/modeler/GeneratePages2.png)

3. Drag and drop **Product_Overview** onto your app's home page to make a button which navigates to your new page.
4. Create a native page *NativeProductOverview* that has a data view with the **Data source** set to **Context** and Entity set to **Product**. Click **Ok** and then **Accept**. Now when you tap a notification, a page will be opened using the proper product object:

	![NativeProductOverview](attachments/native-remote-push/modeler/NativeProductOverview.png)

#### 4.1.2 Synchronizing Unused Entities

Studio Pro does smart data syncing, meaning if an entity has not been retrieved in native side, it will not be there. This situation will not occur since most Mendix native apps do retrieve entities which you want to show. For more information, see the [Sychronization](/refguide/offline-first#sychronization) section of the *Offline-First Reference Guide*.

Your app does not retrieve any products in any of its pages. Fix this by doing the following:

1. In **Navigation** > **Native mobile**, click **Sync configuration**.
2. Change **Product** to download **All Objects**:

    ![SyncConfig](attachments/native-remote-push/modeler/SyncConfig.png)

#### 4.1.3 Determining GUIDs {#guid-objects}

In order to send a particular object to a page, first an object's GUID must be determined and sent using a push notification. To begin this process, you will log the object GUID to your Mendix Studio Pro's console (to be used later in the [Testing the Implementation](#testing-guid) section):

1.  Create a nanoflow *ACT_GetGUIDAndLog* which does the following:<br />
    a. Accepts **Product** object as a parameter.<br />
    b. Uses a JavaScript action **Get guid**, which sets the object as **$Product**.<br />
    c. Logs the returned value:
    
    ![ACT_GetGUIDAndLog](attachments/native-remote-push/modeler/ACT_GetGUIDAndLog.png)

2.  Drag and drop this nanoflow inside **Product_NewEdit**'s data view:

    ![getGUIdAndLogButton](attachments/native-remote-push/modeler/getGUIDAndLogButton.png)

#### 4.1.4 Creating a Data Passing Nanoflow

1.  Create a nanoflow *ACT_GetProductAndShowPage* which does the following:<br />
    a. Accepts a **Notification** object as a parameter:<br />
    
    ![getGUIdAndLogButton](attachments/native-remote-push/modeler/ACT_GetProductAndShowPage.png)
    
    b. Uses a JavaScript action **Get object by guid**, which sets **Entity** as **Product**, the **Object guid** as **parameter/objectGUID**, and the **Object Name** to *ProductObject*:<br />
    
    ![GetProductAndShowPage2](attachments/native-remote-push/modeler/ACT_GetProductAndShowPage2.png)
    
    c. Shows the **NativeProductOverview** page using the passed object **ProductObject**:
    
    ![GetProductAndShowPage3](attachments/native-remote-push/modeler/ACT_GetProductAndShowPage3.png)

2.  In the notification widget on **Home_Native**, create a new action named *sendProduct* which **On open** triggers **ACT_GetProductAndShowPage**:

    ![pushSendProduct](attachments/native-remote-push/modeler/pushSendProduct.png)

#### 4.1.5 Testing the Implementation {#testing-guid}

1. Click the nanoflow button you created in [Determining GUIDs](#guid-objects) to log the object's GUID (which you can see in your Studio Pro Console).
2. Follow the steps in the [Sending a Push Notification to a Single Device](#sending-single) section again, but with the following changes:<br />
    a. Set the action name to *sendProduct*.<br />
    b. Set **Context object guid** to the GUID you just logged:

    ![openProductPage](attachments/native-remote-push/modeler/openProductPage.png)

3. With your app running in the background, send a notification and tap it. This will navigate to the **NativeProductOverview** page with the proper object.

## 4.2 Push Notifications for an App in the Foreground

1.  Add one more **boolean** field named *showNotification* to the **NativeNotification** entity:

    ![showNotification](attachments/native-remote-push/modeler/showNotification.png)

2. In your **Home_Native** page inside of the **NativeNotification** data view, do the following:<br />
    a. Add a container.<br />
    b. Sets its visibility to **NativeNotification/showNotification**.<br />
    c. Add a text field saying *You have recieved a product*.<br />
    d. Drag and drop your **ACT_GetProductAndShowPage** nanoflow next to it.

    ![ContainerVisibility](attachments/native-remote-push/modeler/ContainerVisibility.png)

3. Create a nanoflow called *ACT_ShowNotificationOnRecieve* which will be responsible for toggling the **NativeNotification/showNotification** attribute:<br />
    a. Set **NativeNotification** as a parameter.<br />
    b. Change **NativeNotification/showNotification** to **true** without committing.<br />
    c. Drag and drop a **Wait** JavaScript action, and set it for *5000* ms.<br />
    d. Change the **NativeNotification/showNotification** to **false** without committing.
    
	![ACT_ShowNotificationOnRecieve](attachments/native-remote-push/modeler/ACT_ShowNotificationOnRecieve.png)

4. Navigate to your **Home_Native** page and do the following:<br />
	a. Double-click your notification widget<br />
	b. Change **sendProduct** so that on recieve it triggers **ACT_ShowNotificationOnRecieve**.

	![sendProductOnRecieve](attachments/native-remote-push/modeler/sendProductOnRecieve.png)

Now repeat the steps in [Testing the Implementation](#testing-guid), but this time put your app in the foreground. You will see a notification with your text and a button for five seconds.

![onRecieveShowDV](attachments/native-remote-push/modeler/onRecieveShowDV.png)

## 5 Sending Notifications to Multiple Devices

What if you want to send messages to all your users' devices with a single button push, but you do not want to handle the GUID retrieval? The section below will illustrate this. Specifically, you will send a push notification containing a data object to your users' devices via the Push Notifications API.

### 5.1 Creating a Microflow to Send a Data Object Push Notification 

Create a microflow *ACT_SendProductToAllDevices* with the following elements:

![SendProductToAll](attachments/native-remote-push/modeler/SendProductToAll.png)

1. Add a *Product* data parameter to your microflow.
2. Retrieve the *PushNotifications.Device* entity list from a database:

    ![retrieveDevices](attachments/native-remote-push/modeler/retrieveDevices.png)
    
3. Drag and drop the **PrepareMessageData** microflow from *PushNotifications/_USE ME/API* onto **ACT_SendProductToAllDevices** and configure the following:<br />
	a. Title: *myTitle*.<br />
	b. Body: *myBody*.<br />
	c. TimeToLive: *0*.<br />
	d. Badge: *0*.<br />
	e. ActionName: *sendProduct*.<br />
	f. ContextObjectGuid: *empty*:
	
	![prepareMessageData](attachments/native-remote-push/modeler/prepareMessageData.png)

	**ContextObjectGuid** is set to empty since you will pass the object itself to the **SendMessageToDevices** Java action where it will be retrieved automatically. 

4. Drag and drop the **SendMessageToDevices** Java action from `PushNotifications/_USE ME/API` onto **ACT_SendProductToAllDevices** and configure the following:<br />
	a. Message data param: **$MessageToBeSent**.<br />
	b. Device param: **$Devices**.<br />
	c. Context object: **$Product**:
	
	![sendMessagesJava](attachments/native-remote-push/modeler/sendMessagesJava.png)
  
5. Go to **Product_NewEdit**, drag and drop **ACT_SendProductToAllDevices** inside of that page's data view:

	![sendProductToAllButton](attachments/native-remote-push/modeler/sendProductToAllButton.png)

### 5.2 Testing the Implementation

Test your new push notificaiton capabilities by doing the following:

1. Run your native app in the background.
2. In your web browser, go to **Product_NewEdit** and click your **ACT_SendProductToAllDevices** microflow button. 

This will send a notification to all available devices and when you tap the notification you will be redirected to the particular product page you modeled.

### 5.3 More Java Action Explanations

For more detail on Java actions available in the Push notifications module, see the subsections below.

#### 5.3.1 PrepareMessageData Microflow

This allows users to create their own user interfaces in order to alter and create a push notification message. 

#### 5.3.2 SendMessageToDevice and SendMessageToDevices Java Actions

These Java actions have the following parameters:

* **MessageDataParam** (PushNotifications.MessageData): This parameter can be generated by the **PrepareMessageData** microflow
* **DeviceParam** (List of PushNotifications.Device or PushNotification.Device): This parameter can be used to send the same message to a list of devices
* **ContextObject**: This parameter will allow any Mendix object to be passed to the notification

#### 5.3.3 SendMessageToUsers and SendMessageToUser Java Actions

Every user is allowed to have more than one device. When sending push notifications to every device of a particular user, use  the **SendMessageToUser** Java action.

To send a push notification to all users, use the **SendMessageToUsers** Java action.

## 6 Read More

* [Use Local Notifications](local-notif-parent)

todo add links (firebase, local notifs)