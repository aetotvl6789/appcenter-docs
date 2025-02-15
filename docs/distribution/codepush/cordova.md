---
title: Using the Cordova SDK with CodePush
description: "How to use to the Cordova SDK with CodePush"
keywords: distribution
author: lucen-ms
ms.author: lucen
ms.date: 07/06/2021
ms.topic: article
ms.assetid: 591025F9-C7DA-400E-8CCA-DF39514B3DD0
ms.service: vs-appcenter
ms.custom: distribute
---

# Cordova Client SDK

This plugin provides client-side integration for the CodePush service, allowing you to easily add a dynamic update experience to your Cordova app(s).

> [!NOTE]
> Support for Cordova Apps has ended in April 2022. Find more information in the [App Center blog](https://devblogs.microsoft.com/appcenter/announcing-apache-cordova-retirement/).

## How does it work?

A Cordova app is composed of HTML, CSS and JavaScript files and any accompanying images, which are bundled together by the Cordova CLI and distributed as part of a platform-specific binary (i.e. an .ipa or .apk file). Once the app is released, updating either the code or image assets requires you to recompile and redistribute the entire binary. This process includes review time for the store(s) you're publishing to.

The CodePush plugin helps get product improvements in front of your end users instantly, by keeping your code and images synchronized with updates you release to the CodePush server. This way, your app gets the benefits of an offline mobile experience, as well as the "web-like" agility of side-loading updates as soon as they're available. It's a win-win!

To ensure that your end users always have a functioning version of your app, the CodePush plugin maintains a copy of the previous update, so that in the event that you accidentally push an update that includes a crash, it can automatically roll back. This way, you can rest assured that your newfound release agility won't result in users becoming blocked before you have a chance to roll back on the server. It's a win-win-win!

> [!NOTE]
> Any product changes that touch native code (e.g. upgrading Cordova versions, adding a new plugin) can't be distributed via CodePush, and so, must be updated via the appropriate store(s).

## Supported Cordova Platforms

Cordova 5.0.0+ is fully supported, along with the following associated platforms:

* Android ([cordova-android](https://github.com/apache/cordova-android) 4.0.0+) - Including CrossWalk!
* iOS ([cordova-ios](https://github.com/apache/cordova-ios) 3.9.0+) - (To use CodePush along with the [`cordova-plugin-wkwebview-engine`](https://github.com/apache/cordova-plugin-wkwebview-engine) plugin, you need to install `v1.5.1-beta+`, which includes full support for apps using either WebView.)

To check the versions of each Cordova platform you're currently using, you can run the following command and inspect the `Installed platforms` list:

```shell
    cordova platform ls
```

If you're running an older Android or iOS platform than is mentioned above, and would be open to upgrading, you can easily do so by running the following commands (omitting a platform if it isn't necessary):

```shell
    cordova platform update android
    cordova platform update ios
```

## Getting started

> [!NOTE]
> This article contains references to the term *whitelist*, a term that Microsoft no longer uses. When the term is removed from the software, we'll remove it from this article.

Once you've followed the general-purpose ["getting started"](~/distribution/codepush/index.md) instructions for setting up your CodePush account, you can start CodePush-ifying your Cordova app by running the following command from within your app's root directory:

```shell
cordova plugin add cordova-plugin-code-push@latest
```

With the CodePush plugin installed, configure your app to use it via the following steps:

1. Add your deployment keys to the **config.xml** file, making sure to include the right key for each Cordova platform:

```xml
    <platform name="android">
        <preference name="CodePushDeploymentKey" value="YOUR-ANDROID-DEPLOYMENT-KEY" />
    </platform>
    <platform name="ios">
        <preference name="CodePushDeploymentKey" value="YOUR-IOS-DEPLOYMENT-KEY" />
    </platform>
```

As a reminder, these keys are generated for you when you created your CodePush app via the CLI. If you need to retrieve them, you can run `appcenter codepush deployment list -a <ownerName>/<appName> --displayKeys`, and grab the key for the specific deployment you want to use (e.g. `Staging`, `Production`).

   > [!IMPORTANT]
   > We [recommend](./cli.md#app-management) creating a separate CodePush app for iOS and Android, which is why the above sample declares separate keys for Android and iOS. If you're only developing for a single platform, then you only need to specify the deployment key for either Android or iOS, so you don't need to add the additional `<platform>` element as illustrated above.

Beginning from version **1.10.0** you can sign your update bundles (for more information about code signing refer to relevant documentation [section](cli.md#code-signing)). To enable code signing for a Cordova application you should set up a public key to verify the bundle's signature by providing following a `preference` setting in `config.xml`:

```xml
    <platform name="android">
        ...
        <preference name="CodePushPublicKey" value="YOUR-PUBLIC-KEY" />
    </platform>
    <platform name="ios">
        ...
        <preference name="CodePushPublicKey" value="YOUR-PUBLIC-KEY" />
    </platform>
```

You can use the same private/public key pair for each platform.

1. If you're using an `<access origin="*" />` element in your **config.xml** file, then your app is already allowed to communicate with the CodePush servers and you can safely skip this step. Otherwise, add the following additional `<access />` elements:

```xml
    <access origin="https://codepush.appcenter.ms" />
    <access origin="https://codepush.blob.core.windows.net" />
    <access origin="https://codepushupdates.azureedge.net" />
```

1. To ensure that your app can access the CodePush server on [CSP](https://developer.mozilla.org/docs/Web/Security/CSP/Introducing_Content_Security_Policy)-compliant platforms, add `https://codepush.appcenter.ms` to the `Content-Security-Policy` `meta` tag in your `index.html` file:

```xml
    <meta http-equiv="Content-Security-Policy" content="default-src https://codepush.appcenter.ms 'self' data: gap: https://ssl.gstatic.com 'unsafe-eval'; style-src 'self' 'unsafe-inline'; media-src *" />
```

1. Finally, double-check that you already have the [`cordova-plugin-whitelist`](https://github.com/apache/cordova-plugin-whitelist) plugin installed (most apps will). To check this, run the following command:

```shell
    cordova plugin ls    
```

If `cordova-plugin-whitelist` is in the list, then you're good to go. Otherwise, run the following command to add it:

```shell
    cordova plugin add cordova-plugin-whitelist
```

You're now ready to use the plugin in the application code. See the sample applications for examples and the API documentation for more details.

## Plugin Usage

With the CodePush plugin installed and configured, the only thing left is to add the necessary code to your app to control the following policies:

1. When (and how often) to check for an update? (e.g. app start, in response to clicking a button in a settings page, periodically at some fixed interval)
1. When an update is available, how to present it to the end user?
The simplest way to do this is to run the following in your app's `deviceready` event handler:

```javascript
codePush.sync();
```

If an update is available, it will be silently downloaded, and installed the next time the app is restarted (either explicitly by the end user or by the OS), which ensures the least invasive experience for your end users. If an available update is mandatory, then it will be installed immediately, ensuring that the end user gets it as soon as possible.

If you want your app to discover updates more quickly, you can also choose to call `sync` every time the app resumes from the background, by adding the following code (or something equivalent) as part of your app's startup behavior. You can call `sync` as frequently as you want, so when and where you call it depends on your personal preference.

```javascript
document.addEventListener("resume", function () {
    codePush.sync();
});
```

Additionally, if you want to display an update confirmation dialog (an "active install"), configure when an available update is installed (e.g. force an immediate restart) or customize the update experience in any way, refer to the `sync` method's API reference for information on how to tweak this default behavior.

> [!IMPORTANT]
> While [Apple's developer agreement](https://developer.apple.com/app-store/review/guidelines/#software-requirements) fully allows over-the-air updates of JavaScript and assets (which is what enables CodePush!), it's against their policy for an app to display an update prompt. Because of this, we recommend that App Store-distributed apps don't enable the `updateDialog` option when calling `sync`, whereas Google Play and internally distributed apps (e.g. Enterprise, Fabric, HockeyApp) can choose to enable/customize it.

## Releasing Updates
Once your app has been configured and distributed to your users, and you've made some code or asset changes, it's time to instantly release them! The simplest (and recommended) way to do this is to use the `release-cordova` command in the CodePush CLI, which handles preparing and releasing your update to the CodePush server.

> [!NOTE]
> Before you can start releasing updates, log into App Center by running the `appcenter login` command

In its most basic form, this command only requires one parameter: your owner name + "/" + app name.

```shell
appcenter codepush release-cordova -a <ownerName>/<appName>
appcenter codepush release-cordova -a <ownerName>/MyApp-ios
appcenter codepush release-cordova -a <ownerName>/MyApp-android
```

> [!TIP]
> By using the App Center CLI `set-current` function you don't have to use the -a flag to specify the app a command is directed at.
>
> [!TIP]
> When releasing updates to CodePush, you don't need to bump your app's version in the **config.xml** file, since you aren't modifying the binary version at all. You only need to bump this version when you upgrade Cordova or one of your plugins, at which point, you need to release an update to the native store(s). CodePush will automatically generate a "label" for each release you make (e.g. `v3`) to help identify it within your release history.

The `release-cordova` command enables such a simple workflow because it understands the standard layout of a Cordova app, and so, can generate your update and know exactly which files to upload. Additionally, to support flexible release strategies, the `release-cordova` command exposes numerous optional parameters that let you customize how the update should be distributed to your end users (e.g. Which binary versions are compatible with it? Should the release be viewed as mandatory?).

```shell
# Release a mandatory update with a changelog
appcenter codepush release-cordova -a <ownerName>/MyApp-ios -m --description "Modified the header color"

# Release a dev Android build to 1/4 of your end users
appcenter codepush release-cordova -a <ownerName>/MyApp-android --rollout 25

# Release an update that targets users running any 1.1.* binary, as opposed to
# limiting the update to exact version name in the config.xml file
appcenter codepush release-cordova -a <ownerName>/MyApp-android --target-binary-version "~1.1.0"

# Release the update now but mark it as disabled
# so that no users can download it yet
appcenter codepush release-cordova -a <ownerName>/MyApp-ios -x
```

The CodePush client supports differential updates, so even though you're releasing your app code on every update, your end users will only actually download the files they need. The service handles this automatically so that you can focus on creating awesome apps and we can worry about optimizing end-user downloads.

For more details about how the `release-cordova` command works, as well as the various parameters it exposes, refer to the [CLI docs](./cli.md#releasing-updates-cordova). Additionally, if you'd prefer to handle running the `cordova prepare` command yourself, and so, want an even more flexible solution than `release-cordova`, refer to the [`release` command](./cli.md#releasing-updates-general) for more details.

If you run into any issues, or have any questions/comments/feedback, you can [e-mail us](mailto:codepushfeed@microsoft.com) or open a new issue on this repo and we'll respond ASAP! See also [help and feedback](../../help.md).

## API Reference
The CodePush API is exposed to your app via the global `codePush` object, which is available after the `deviceready` event fires. This API exposes the following top-level methods:
* **[checkForUpdate](#codepushcheckforupdate)**: Asks the CodePush service whether the configured app deployment has an update available.
* **[getCurrentPackage](#codepushgetcurrentpackage)**: Retrieves the metadata about the currently installed update (e.g. description, installation time, size).
* **[getPendingPackage](#codepushgetpendingpackage)**: Retrieves the metadata for an update (if one exists) that was downloaded and installed, but hasn't been applied yet via a restart.
* **[notifyApplicationReady](#codepushnotifyapplicationready)**: Notifies the CodePush runtime that an installed update is considered successful. If you're manually checking for and installing updates (i.e. not using the sync method to handle it all for you), then this method **MUST** be called; otherwise CodePush will treat the update as failed and roll back to the previous version when the app next restarts.
* **[restartApplication](#codepushrestartapplication)**: Immediately restarts the app. If there's an update pending, it will be immediately displayed to the end user.
* **[sync](#codepushsync)**: Allows checking for an update, downloading it and installing it, all with a single call. Unless you need custom UI or behavior, we recommend most developers to use this method when integrating CodePush into their apps.

Additionally, the following objects and enums are also exposed globally as part of the CodePush API:
* **[InstallMode](#installmode)**: Defines the available install modes for updates.
* **[LocalPackage](#localpackage)**: Contains information about a locally installed package.
* **[RemotePackage](#remotepackage)**: Contains information about an update package available for download.
* **[SyncStatus](#syncstatus)**: Defines the possible intermediate and result statuses of the [sync](#codepushsync) operation.

### codePush.checkForUpdate
```javascript
codePush.checkForUpdate(onSuccess, onError?, deploymentKey?: String);
```

Queries the CodePush service to see whether the configured app deployment has an update available. By default, it will use the deployment key that's configured in your **config.xml** file, but you can override that by specifying a value via the optional `deploymentKey` parameter. This can be useful when you want to dynamically "redirect" a user to a specific deployment, such as allowing "Early access" via an easter egg or a user setting switch.

When the update check completes, it will trigger the `onUpdateCheck` callback with one of two possible values:
1. `null` if there's no update available. This occurs in the following scenarios:
   - The configured deployment doesn't contain any releases, so there's nothing to update.
   - The latest release within the configured deployment is targeting a different binary version than what you're currently running (either older or newer).
   - The currently running app already has the latest release from the configured deployment, so it doesn't need the release again.
2. A `RemotePackage` instance that represents an available update that can be inspected and later downloaded.
    
Parameters:
* **onSuccess**: Callback that's invoked upon receiving a successful response from the server. The callback receives a single parameter, which is described above.
* **onError**: Optional callback that's invoked in the event of an error. The callback takes one error parameter, containing the details of the error.
* **deploymentKey**: Optional deployment key that overrides the **config.xml** setting.

Example usage:
```javascript
codePush.checkForUpdate(function (update) {
    if (!update) {
        console.log("The app is up to date.");
    } else {
        console.log("An update is available! Should we download it?");
    }
});
```

### codePush.getCurrentPackage
```javascript
codePush.getCurrentPackage(onSuccess, onError?);
```

Retrieves the metadata about the currently installed "package" (e.g. description, installation time). This can be useful for scenarios such as displaying a "what's new?" dialog after an update has been applied or checking whether there's a pending update that's waiting to be applied via a resume or restart.

When the update retrieval completes, it will trigger the `onSuccess` callback with one of two possible values:
1. `null` if the app is currently running the HTML start page from the binary and not a CodePush update. This occurs in the following scenarios:
   - The end user installed the app binary and has yet to install a CodePush update
   - The end user installed an update of the binary (e.g. from the store), which cleared away the old CodePush updates, and gave precedence back to the binary.
2. A `LocalPackage` instance that represents the metadata for the currently running CodePush update.

Parameters:
* **onSuccess**: Callback that's invoked upon receiving the metadata about the currently running update. The callback receives a single parameter, which is described above.
* **onError**: Optional callback that's invoked in the event of an error. The callback takes one error parameter, containing the details of the error.

Example Usage:
```javascript
codePush.getCurrentPackage(function (update) {
    if (!update) {
        console.log("No updates have been installed");
        return;
    }

    // If the current app "session" represents the first time
    // this update has run, and it had a description provided
    // with it upon release, let's show it to the end user
    if (update.isFirstRun && update.description) {
        // Display a "what's new?" modal
    }
});
```

### codePush.getPendingPackage
```javascript
codePush.getPendingPackage(onSuccess, onError?);
```

Gets the metadata for the currently pending update (if one exists). An update is considered "pending" if it's been downloaded and installed, but hasn't been applied yet via an app restart. An update could only ever be in this state if   `InstallMode.ON_NEXT_RESTART` or `InstallMode.ON_NEXT_RESUME` were specified upon calling `sync` or `LocalPackage.install`, and the app hasn't yet been restarted or resumed (respectively). This method can be useful if you want to determine whether there's a pending update and then prompt the user if they want to restart immediately (via `codePush.restartApplication`) to apply it.

When the update retrieval completes, it will trigger the `onSuccess` callback with one of two possible values:
1. `null` if the app doesn't currently have a pending update (e.g. the app is already running the latest available version).
2. A `LocalPackage` instance that represents the metadata for the currently pending CodePush update.

Parameters:
* **onSuccess**: Callback that's invoked upon receiving the metadata about the currently pending update. The callback receives a single parameter, which is described above.
* **onError**: Optional callback that's invoked in the event of an error. The callback takes one error parameter, containing the details of the error.

Example Usage:
```javascript
codePush.getPendingPackage(function (update) {
    if (update) {
        // An update is currently pending, ask the
        // user if they want to restart
    }
});
```

### codePush.notifyApplicationReady
```javascript
codePush.notifyApplicationReady(notifySucceeded?, notifyFailed?);
```

Notifies the CodePush runtime that a freshly installed update should be considered successful, so an automatic client-side roll back isn't necessary. It's mandatory to call this function somewhere in the code of the updated bundle. Otherwise, when the app next restarts, the CodePush runtime will assume that the installed update has failed and roll back to the previous version. This behavior exists to help ensure that your end users aren't blocked by a broken update.

If you're using the `sync` function, and doing your update check on app start, then you don't need to manually call `notifyApplicationReady` since `sync` will call it for you. This behavior exists because of the assumption that when `sync` is called in your app represents a good approximation of a successful startup.

Parameters:
* **notifySucceeded**: Optional callback invoked if the plugin was successfully notified.
* **notifyFailed**: Optional callback invoked if there's an error notifying the plugin.

### codePush.restartApplication
```javascript
codePush.restartApplication();
```

Immediately restarts the app. This method is for advanced scenarios, and is primarily useful when the following conditions are true:
1. Your app is specifying an install mode value of `ON_NEXT_RESTART` or `ON_NEXT_RESUME` when calling the `sync` or `LocalPackage.install` methods. This doesn't apply your update until the app restarts (by either the end user or OS) or resumes, so the update won't be immediately displayed to the end user.
2. You have an app-specific user event (e.g. the end user navigated back to the app's home route) that allows you to apply the update in an unobtrusive way, and potentially gets the update in front of the end user sooner than waiting until the next restart or resume.

### codePush.sync
```javascript
codePush.sync(syncCallback?, syncOptions?, downloadProgress?, syncErrback?);
```

Synchronizes your app's code and images with the latest release to the configured deployment. Unlike the `checkForUpdate` method, which checks for the presence of an update, and lets you control what to do next, `sync` handles the update check, download and installation experience for you.

This method provides support for two different (but customizable) "modes" to easily enable apps with different requirements:
1. **Silent mode** *(the default behavior)*, which automatically downloads available updates, and applies them the next time the app restarts (e.g. the OS or end user killed it, or the device was restarted). This way, the entire update experience is "silent" to the end user, since they don't see any update prompt or "synthetic" app restarts.
2. **Active mode**, which when an update is available, prompts the end user for permission before downloading it, and then immediately applies the update. If an update was released using the mandatory flag, the end user would still be notified about the update, but they wouldn't have the choice to ignore it.

Example Usage:
```javascript
// Fully silent update that keeps the app in
// sync with the server, without ever
// interrupting the end user
codePush.sync();

// Active update that lets the end user know
// about each update, and displays it to them
// immediately after downloading it
codePush.sync(null, { updateDialog: true, installMode: InstallMode.IMMEDIATE });
```

> [!TIP]
> If you want to decide whether you check or download an available update based on the end user's device battery level, network conditions, etc. then wrap the call to sync in a condition that ensures you only call it when wanted.

While the sync method tries to make it easy to do silent and active updates with little configuration, it accepts the following optional parameters that allow you to customize numerous aspects of the default behavior mentioned above:
* **syncCallback**: Called when the sync process moves from one stage to another in the overall update process. The method is called with a status code that represents the current state, and can be any of the [`SyncStatus`](#syncstatus) values.
* **syncOptions**: Optional [`SyncOptions`](#syncoptions) parameter configuring the behavior of the sync operation.
* **downloadProgress**: Called periodically when an available update is being downloaded from the CodePush server. The method is called with a `DownloadProgress` object, which contains the following two properties:
  * **totalBytes** *(Number)* - The total number of bytes expected to be received for this update (i.e. the size of the set of files that changed from the previous release).
  * **receivedBytes** *(Number)* - The number of bytes downloaded thus far, which can be used to track download progress.
* **syncErrback**: Called when there's an error in any of sync internal steps. The method is called with a standard javascript `Error` object as first argument.

#### SyncOptions
While the `sync` method tries to make it easy to do silent and active updates with little configuration, it accepts an "options" object that allows you to customize numerous aspects of the default behavior mentioned above:
* **deploymentKey** *(String)* - Specifies the deployment key you want to query for an update against. By default, this value is derived from the **config.xml** file, but this option allows you to override it from the script-side if you need to dynamically use a different deployment for a specific call to `sync`.
* **installMode** *(InstallMode)* - Specifies when you want to install optional updates (i.e. those that aren't marked as mandatory). Defaults to `InstallMode.ON_NEXT_RESTART`. Refer to the [`InstallMode`](#installmode) enum reference for a description of the available options and what they do.
* **mandatoryInstallMode** *(InstallMode)* - Specifies when you want to install updates that are marked as mandatory. Defaults to `InstallMode.IMMEDIATE`. Refer to the [`InstallMode`](#installmode) enum reference for a description of the available options and what they do.
* **minimumBackgroundDuration** *(Number)* - Specifies the minimum number of seconds for the app to be in the background before restarting the app. This property only applies to updates which are installed using `InstallMode.ON_NEXT_RESUME`, and can be useful for getting your update in front of end users sooner, without being too obtrusive. Defaults to `0`, which applies the update immediately after a resume, however long it was in the background.
* **ignoreFailedUpdates** *(Boolean)* - Specifies whether an available update should be ignored if it had been previously installed and rolled back on the client (because `notifyApplicationReady` wasn't successfully called). Defaults to `true`.
* **updateDialog** *(UpdateDialogOptions)* - An "options" object used to determine whether a confirmation dialog should be displayed to the end user when an update is available, and if so, what strings to use. Defaults to `null`, which disables the dialog. Setting this to any `true` value will enable the dialog with the default strings, and passing an object to this parameter allows enabling the dialog as well as overriding one or more of the default strings.

The following list represents the available options and their defaults:
* **appendReleaseDescription** *(Boolean)* - Indicates whether you want to append the description of an available release to the notification message that's displayed to the end user. Defaults to `false`.
* **descriptionPrefix** *(String)* - Indicates the string you want to prefix the release description with, if any, when displaying the update notification to the end user. Defaults to `" Description: "`.
* **mandatoryContinueButtonLabel** *(String)*: The text to use for the button the end user must press to install a mandatory update. Defaults to `"Continue"`.
* **mandatoryUpdateMessage** *(String)* - The text used as the body of an update notification, when the update is specified as mandatory. Defaults to `"An update is available that must be installed."`.
* **optionalIgnoreButtonLabel** *(String)* - The text to use for the button the end user can press to ignore an optional update that's available. Defaults to `"Ignore"`.
* **optionalInstallButtonLabel** *(String)* - The text to use for the button the end user can press to install an optional update. Defaults to `"Install"`.
* **optionalUpdateMessage** *(String)* - The text used as the body of an update notification, when the update is optional. Defaults to `"An update is available. Would you like to install it?"`.
*- **updateTitle** *(String)* - The text used as the header of an update notification that's displayed to the end user. Defaults to `"Update available"`.

Example Usage:
```javascript
// Download the update silently, but install it on
// the next resume, as long as at least 5 minutes
// has passed since the app was put into the background.
codePush.sync(null, { installMode: InstallMode.ON_NEXT_RESUME, minimumBackgroundDuration: 60 * 5 });

// Download the update silently, and install optional updates
// on the next restart, but install mandatory updates on the next resume.
codePush.sync(null, { mandatoryInstallMode: InstallMode.ON_NEXT_RESUME });

// Changing the title displayed in the
// confirmation dialog of an "active" update
codePush.sync(null, { updateDialog: { title: "An update is available!" } });

// Displaying an update prompt that includes the
// description for the CodePush release
codePush.sync(null, {
   updateDialog: {
    appendReleaseDescription: true,
    descriptionPrefix: "\n\nChange log:\n"
   },
   installMode: InstallMode.IMMEDIATE
});

// Silently check for the update, but
// display a custom downloading UI
// via the SyncStatus and DownloadProgress callbacks
codePush.sync(syncStatus, null, downloadProgress);

function syncStatus(status) {
    switch (status) {
        case SyncStatus.DOWNLOADING_PACKAGE:
            // Show "downloading" modal
            break;
        case SyncStatus.INSTALLING_UPDATE:
            // Hide "downloading" modal
            break;
    }
}

function downloadProgress(downloadProgress) {
    if (downloadProgress) {
    	// Update "downloading" modal with current download %
        //console.log("Downloading " + downloadProgress.receivedBytes + " of " + downloadProgress.totalBytes);
    }
}
```

The `sync` method can be called anywhere you want to check for an update. That could be in the `deviceready` event handler, the `click` event of a button, in the callback of a periodic timer, or whatever else makes sense for your needs. Like the `checkForUpdate` method, it will run the network request to check for an update in the background, so it won't impact your UI thread or JavaScript thread's responsiveness.

### Package objects
The `checkForUpdate` and `getCurrentPackage` methods invoke success callbacks, that when triggered, provide access to "package" objects. The package represents your code update as well as any extra metadata (e.g. description, mandatory?). The CodePush API has the distinction between the following types of packages:
1. `LocalPackage`: Represents a downloaded update that's either already running, or has been installed and is pending an app restart.
2. `RemotePackage`: Represents an available update on the CodePush server that hasn't been downloaded yet.

#### LocalPackage
Contains details about an update that has been downloaded locally or already installed. You can get a reference to an instance of this object either by calling the `codePush.getCurrentPackage` method, or as the value provided to the success callback of the `RemotePackage.download` method.

##### Properties
* **appVersion**: The native version of the application this package update is intended for. *(String)*
* **deploymentKey**: Deployment key of the package. *(String)*
* **description**: The description of the update. This is the same value that you specified in the CLI when you released the update. *(String)*
* **failedInstall**: Indicates whether this update has been previously installed but was rolled back. The `sync` method will automatically ignore updates that have previously failed, so you only need to worry about this property if using `checkForUpdate`. *(Boolean)*
* **isFirstRun**: Flag indicating if the current application run is the first one after the package was applied. *(Boolean)*
* **isMandatory**: Indicates whether the update is considered mandatory. This is the value that was specified in the CLI when the update was released. *(Boolean)*
* **label**: The internal label automatically given to the update by the CodePush server, such as `v5`. This value uniquely identifies the update within its deployment. *(String)*
* **packageHash**: The SHA hash value of the update. *(String)*
* **packageSize**: The size of the code contained within the update, in bytes. *(Number)*

##### Methods
* **install(installSuccess, installError, installOptions)**: Installs this package to the application.
  The install behavior is dependent on the provided `installOptions`. By default, the update package is silently installed and the application is reloaded with the new content on the next application start.
  On the first run after the update, the application will wait for a `codePush.notifyApplicationReady()` call. Once this call is made, the install operation is considered a success.
  Otherwise, the install operation will be marked as failed, and the application is reverted to its previous version on the next run.

###### InstallOptions
Interface defining several options for customizing install operation behavior.
* **installMode**: Used to specify the [InstallMode](#installmode) used for the install operation. Defaults to `InstallMode.ON_NEXT_RESTART`.
* **mandatoryInstallMode**: Used to specify the [InstallMode](#installmode) used for the install operation if the package is mandatory. Defaults to `InstallMode.IMMEDIATE`.
* **minimumBackgroundDuration**: If **installMode** is `InstallMode.ON_NEXT_RESUME`, used to specify the amount of time the app must be in the background before the update is installed when it's resumed. Defaults to `0`.

Example Usage:
```javascript
// App version 1 (current version)

var onError = function (error) {
    console.log("An error occurred. " + error);
};

var onInstallSuccess = function () {
    console.log("Installation succeeded.");
};

var onPackageDownloaded = function (localPackage) {
    // Install regular updates after someone navigates away from the app for more than 2 minutes
    // Install mandatory updates after someone restarts the app
    localPackage.install(onInstallSuccess, onError, { installMode: InstallMode.ON_NEXT_RESUME, minimumBackgroundDuration: 120, mandatoryInstallMode: InstallMode.ON_NEXT_RESTART });
};

var onUpdateCheck = function (remotePackage) {
    if (!remotePackage) {
        console.log("The application is up to date.");
    } else {
        // The hash of each previously reverted package is stored for later use.
        // This way, we avoid going into an infinite bad update/revert loop.
        if (!remotePackage.failedInstall) {
            console.log("A CodePush update is available. Package hash: " + remotePackage.packageHash);
            remotePackage.download(onPackageDownloaded, onError);
        } else {
            console.log("The available update was attempted before and failed.");
        }
    }
};

window.codePush.checkForUpdate(onUpdateCheck, onError);

//------------------------------------------------
// App version 2 (updated version)
var app = {
    onDeviceReady: function () {
        // Calling this function is required during the first application run after an update.
        // If not called, the application will be reverted to the previous version.
        window.codePush.notifyApplicationReady();
        // ...
    }
}
```

For an example on how you're protected against a bad update, see the [notifyApplicationReady() documentation](#codepushnotifyapplicationready).

#### RemotePackage
Contains details about an update that's available for download from the CodePush server. You get a reference to an instance of this object by calling the `codePush.checkForUpdate` method when an update is available. If you're using the sync API, you don't need to worry about the `RemotePackage`, since it will handle the download and installation process automatically for you.

##### Properties
The `RemotePackage` inherits all of the same properties as the `LocalPackage`, but includes one additional one:
* **downloadUrl**: The URL where the package is available for download. This property is only needed for advanced usage, since the `download` method will automatically handle the acquisition of updates for you. *(String)*

##### Methods
* **abortDownload(abortSuccess, abortError)**: Aborts the current download session, if any.
* **download(downloadSuccess, downloadError, downloadProgress)**: Downloads the package update from the CodePush service. The ```downloadSuccess``` callback is invoked with a [LocalPackage](#localpackage) argument, representing the downloaded package.
  The optional `downloadProgress` callback is invoked several times during the download progress with one `DownloadProgress` parameter.

###### DownloadProgress
Defines the format of the DownloadProgress object, used to send periodical update notifications on the progress of the update download.

###### Properties
* **totalBytes**: The size of the downloading update package, in bytes. (Number)
* **receivedBytes**: The number of bytes already downloaded. (Number)

Example Usage:
```javascript
var onError = function (error) {
    console.log("An error occurred. " + error);
};

var onPackageDownloaded = function (localPackage) {
    console.log("Package downloaded at: " + localPackage.localPath);
    // you can now update your application to the downloaded version by calling localPackage.install()
};

var onProgress = function (downloadProgress) {
    console.log("Downloading " + downloadProgress.receivedBytes + " of " + downloadProgress.totalBytes + " bytes.");
};

var onUpdateCheck = function (remotePackage) {
    if (!remotePackage) {
        console.log("The application is up to date.");
    } else {
        console.log("A CodePush update is available. Package hash: " + remotePackage.packageHash);
        remotePackage.download(onPackageDownloaded, onError, onProgress);
    }
};

window.codePush.checkForUpdate(onUpdateCheck, onError);
```

### Enums
The CodePush API includes the following "enum" objects that can be used to customize the update experience, and are available globally off of the `window` object:

#### InstallMode
This enum specified when you want an installed update to actually be applied, and can be passed to either the `sync` or `LocalPackage.install` methods. It includes the following values:
* **IMMEDIATE**: The update will be applied to the running application immediately. The application will be reloaded with the new content immediately.
* **ON_NEXT_RESTART**: Indicates that you want to install the update, but not forcibly restart the app. When the app is "naturally" restarted (because of the OS or end user killing it), the update will be seamlessly picked up. This value is appropriate when doing silent updates, since it would likely be disruptive to the end user if the app suddenly restarted out of nowhere, since they wouldn't have realized an update was even downloaded. This is the default mode used for both the `sync` and `LocalPackage.install` methods.

For an example on how you're protected against a bad update, see the [notifyApplicationReady() documentation](#codepushnotifyapplicationready).

#### RemotePackage
Contains details about an update that's available for download from the CodePush server. You get a reference to an instance of this object by calling the `codePush.checkForUpdate` method when an update is available. If you're using the sync API, you don't need to worry about the `RemotePackage`, since it will handle the download and installation process automatically for you.

##### Properties
The `RemotePackage` inherits all of the same properties as the `LocalPackage`, but includes one additional one:
* **downloadUrl**: The URL where the package is available for download. This property is only needed for advanced usage, since the `download` method will automatically handle the acquisition of updates for you. *(String)*

##### Methods
* **abortDownload(abortSuccess, abortError)**: Aborts the current download session, if any.
* **download(downloadSuccess, downloadError, downloadProgress)**: Downloads the package update from the CodePush service. The ```downloadSuccess``` callback is invoked with a [LocalPackage](#localpackage) argument, representing the downloaded package.
  The optional `downloadProgress` callback is invoked several times during the download progress with one `DownloadProgress` parameter.

###### DownloadProgress
Defines the format of the DownloadProgress object, used to send periodical update notifications on the progress of the update download.

###### # Properties
* **totalBytes**: The size of the downloading update package, in bytes. (Number)
* **receivedBytes**: The number of bytes already downloaded. (Number)

Example Usage:
```javascript
var onError = function (error) {
    console.log("An error occurred. " + error);
};

var onPackageDownloaded = function (localPackage) {
    console.log("Package downloaded at: " + localPackage.localPath);
    // you can now update your application to the downloaded version by calling localPackage.install()
};

var onProgress = function (downloadProgress) {
    console.log("Downloading " + downloadProgress.receivedBytes + " of " + downloadProgress.totalBytes + " bytes.");
};

var onUpdateCheck = function (remotePackage) {
    if (!remotePackage) {
        console.log("The application is up to date.");
    } else {
        console.log("A CodePush update is available. Package hash: " + remotePackage.packageHash);
        remotePackage.download(onPackageDownloaded, onError, onProgress);
    }
};

window.codePush.checkForUpdate(onUpdateCheck, onError);
```

### Enums
The CodePush API includes the following "enum" objects that can be used to customize the update experience, and are available globally off of the `window` object:

#### InstallMode
This enum specified when you want an installed update to actually be applied, and can be passed to either the `sync` or `LocalPackage.install` methods. It includes the following values:
* **IMMEDIATE**: The update will be applied to the running application immediately. The application will be reloaded with the new content immediately.
* **ON_NEXT_RESTART**: Indicates that you want to install the update, but not forcibly restart the app. When the app is "naturally" restarted (because of the OS or end user killing it), the update will be seamlessly picked up. This value is appropriate when doing silent updates, since it would likely be disruptive to the end user if the app suddenly restarted out of nowhere, since they wouldn't have realized an update was even downloaded. This is the default mode used for both the `sync` and `LocalPackage.install` methods.
* **ON_NEXT_RESUME**: Indicates that you want to install the update, but don't want to restart the app until the next time the end user resumes it from the background. This way, you don't disrupt their current session, but you can get the update in front of them sooner than having to wait for the next natural restart. This value is appropriate for silent installs that can be applied on resume in a non-invasive way.

#### SyncStatus
Defines the possible statuses of the [sync](#codepushsync) operation. There are two categories of statuses: intermediate and result (final). The intermediate statuses represent progress statuses of the sync operation, and aren't final. The result statuses represent final statuses of the sync operation. Every sync operation ends with only one result status, but can have zero or more intermediate statuses.

* **UP_TO_DATE**: The app is fully up-to-date with the configured deployment.
* **UPDATE_INSTALLED**: An available update has been installed and will be run either immediately after the callback function returns or the next time the app resumes/restarts, depending on the `InstallMode` specified in `SyncOptions`.
* **UPDATE_IGNORED**: The app has an optional update, which the end user chose to ignore. *(This is only applicable when the `updateDialog` is used)*
* **ERROR**: An error occurred during the `sync` operation. This might be an error while communicating with the server, downloading or unzipping the update. The console logs should contain more information about what happened. No update has been applied in this case.
* **IN_PROGRESS**: Another sync is already running, so this attempt to sync has been aborted.
* **CHECKING_FOR_UPDATE**: The CodePush server is being queried for an update.
* **AWAITING_USER_ACTION**: An update is available, and a confirmation dialog was shown to the end user. *(This is only applicable when the `updateDialog` is used)*
* **DOWNLOADING_PACKAGE**: An available update is being downloaded from the CodePush server.
* **INSTALLING_UPDATE**: An available update was downloaded and is about to be installed.

## PhoneGap Build
This plugin is compatible with [PhoneGap Build](https://build.phonegap.com), and supports creating Android and iOS builds out-of-the-box. However, in order for CodePush to calculate the hash of your binary contents on Android, PhoneGap Build needs to use Gradle to build your app, which isn't its default behavior (it uses Ant). To resolve this, add the following element to the project's **config.xml** file, as a child of the `<platform name="android">` element:
```xml
<preference name="android-build-tool" value="gradle" />
```

## Example Apps
The Cordova community has graciously created some awesome open source apps that can serve as examples for developers that are getting started. The following list is of OSS Cordova apps that are also using CodePush, and can be used to see how others are using the service:
* [PGDay CodePush Demo](https://github.com/rangle/pgdays-codepush-demo) - Demo app created by [Rangle.io](https://rangle.io) used for [PhoneGap Day Europe 2016](https://pgday.phonegap.com/eu2016/).

> [!NOTE]
> If you've developed a Cordova app using CodePush, that's open-source, let us know. We'd love to add it to this list!