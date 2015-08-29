---
layout: post
title: "Creating Home Screen Shortcut's in Titanium"
---

The following guide will explain how to create home screen shortcuts on Android using the [Titanium][titanium] platform.

As of version `4.1.0.GA` of the Titanium SDK creating home screen shortcuts is not possible out of the box. Luckily, I have created a [module][intent-module] to fill in the missing features and created a [pull request][pull-request] that will add these features to Titanium.

First, we have to add the necessary permissions to the Android Manifest in our `tiapp.xml` file. These two permissions allow your app to add/remove home screen shortcuts programmatically. Users can still manually remove a shortcut from the home screen without the uninstall permission.

```xml
<ti:app xmlns:ti="http://ti.appcelerator.org">
  ...
  <android xmlns:android="http://schemas.android.com/apk/res/android">
    <manifest>

      <!-- Permissions to add -->
      <uses-permission android:name="com.android.launcher.permission.INSTALL_SHORTCUT" />
      <uses-permission android:name="com.android.launcher.permission.UNINSTALL_SHORTCUT" />

    </manifest>
  </android>
  ...
</ti:app>
```

Next, you have to download the [latest release][intent-module-release] of the Intent Extension module. If you have never manually installed a titanium module there is a [great guide][install-module] in Appcelerator's documentation. The Intent Extension module is currently required because the method `putExtra` does not accept `Intent` or `Blob` types as values. More information can be found in the modules README.

Lastly, here is a code sample for creating a home screen shortcut.

```javascript
var currentIntent = Ti.Android.currentActivity.getIntent();
currentIntent.setAction(Ti.Android.ACTION_MAIN);

// Add custom metadata to read when your App is launched from the shortcut.
currentIntent.putExtra("shortcut", "gallery");

// Create an Intent and pass it to the module.
var IntentExtension = require('ca.intentextension');
var shortcutIntent = IntentExtension.getIntentExtension(Ti.Android.createIntent({
  action: "com.android.launcher.action.INSTALL_SHORTCUT",
  // Use com.android.launcher.action.UNINSTALL_SHORTCUT to remove the shortcut
}));

// Title that will appear with the shortcut
shortcutIntent.putExtra(Ti.Android.EXTRA_SHORTCUT_NAME, "Gallery");

shortcutIntent.putExtra(Ti.Android.EXTRA_SHORTCUT_INTENT, currentIntent);
shortcutIntent.putExtra("duplicate", false);

// Set the icon for the shortcut
var iconImage = Ti.Filesystem.getFile(Ti.Filesystem.resourcesDirectory, "appicon.png").read();
shortcutIntent.putExtra(Ti.Android.EXTRA_SHORTCUT_ICON, iconImage);

// Adds the shortcut to the home screen.
Ti.Android.currentActivity.sendBroadcast(shortcutIntent);
```

If Titanium's Intent's allowed `Intent` or `Blob` types to be passed to the `putExtra` method, the `shortcutIntent` instantiation line would look like this:

```javascript
var shortcutIntent = Ti.Android.createIntent({
  action: "com.android.launcher.action.INSTALL_SHORTCUT",
});
```

In Android, [Intent's][intent] are descriptions of operations to be performed. In the code sample we created a `shortcutIntent` that will perform the INSTALL_SHORTCUT action. 


[titanium]: https://github.com/appcelerator/titanium_mobile
[intent-module]: https://github.com/collinprice/ca.intentextension
[pull-request]: https://github.com/appcelerator/titanium_mobile/pull/7073
[intent-module-release]: https://github.com/collinprice/ca.intentextension/releases
[install-module]: http://docs.appcelerator.com/platform/latest/#!/guide/Using_a_Module
[intent]: http://developer.android.com/reference/android/content/Intent.html