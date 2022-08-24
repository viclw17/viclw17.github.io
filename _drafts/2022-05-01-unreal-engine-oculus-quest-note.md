---
title: "Unreal Engine Oculus Quest Dev Note"
layout: post
# image: 2022-03-01-raymarching-visualization-shaders/1.png
---


https://medium.com/@kavanbahrami/oculus-quest-2-development-using-unreal-engine-getting-started-833c50ebe9ea

https://docs.unrealengine.com/4.27/en-US/SharingAndReleasing/Mobile/Android/AndroidManifestControl/

https://aj-21683.medium.com/tips-and-tricks-for-oculus-quest-development-with-unreal-engine-4-25-3-6d5cac7f3e19


---

Locate and Run:[EngineInstallLocation]\[UE_Version]\Engine\Extras\Android\SetupAndroid.bat

---

Under the Platforms tab, select Android

Edit > Project Settings > Platform > Android
Click Configure Now

Change Android Package Name
```com.VictorLi.VRTemplate```

store version???

Change the Minimum SDK Version to 25 ???
Change the Target SDK Version to 25


pack inside apk?
OBB ???
Enable Use ExternalFilesDir to UE4Game files?


Change Install Location to Auto (required by Oculus upload validator)
Change Orientation to Landscape (required by Oculus upload validator)

---

Enable Support arm64
Disable Support arm7

---

quotation!!! ''

Add 2 Array elements to the Extra Tags for <application> node
- android:allowBackup="False" (*this may no longer be correct due to the release of OculusCloudSave, more info in comments)
- android:usesCleartextTraffic="False"


Add 2 Array elements to the Extra Tags for UE4.GameActivity node
- android:excludeFromRecents="True"
- android:taskAffinity=""


1. Add 2 Array elements to the Extra Permissions
- android.permission.WRITE_EXTERNAL_STORAGE
- android.permission.READ_EXTERNAL_STORAGE


---


Update your Distribution Signing
https://docs.unrealengine.com/4.27/en-US/SharingAndReleasing/Mobile/Android/DistributionSigning/
Create and Place your [key].keystore file in [ProjectDirectory]\Build\Android\

---

Prior to final Packaging:
Edit > Project Settings > Packaging
Enable Full Rebuild ???
Enable For Distribution
Change Build Configuration to Shipping


```
Your manifest includes the following permissions restricted by Oculus:
- android.permission.WRITE_EXTERNAL_STORAGE
- android.permission.READ_EXTERNAL_STORAGE
- android.permission.ACCESS_MEDIA_LOCATION
Please remove these permissions if they are not needed by your application. If they are needed, you must include justification in the “Notes for the Reviewer” field when submitting your application for review. Failure to provide justification will result in the rejection of your application.
```


```
ERROR: We found issues with the APK during validation. Please check that the APK meets the `Application Manifest Requirements` and then resubmit your app.
* APK screen orientation is not landscape (android:screenOrientation in AndroidManifest.xml).
Build Upload Error - an error has occurred
```



UATHelper: Packaging (Android (ASTC)): ERROR: AndroidManifest.xml is invalid System.Xml.XmlException: '”' is an unexpected token. The expected token is '"' or '''. Line 11, position 35.



UATHelper: Packaging (Android (ASTC)): ERROR: Keystore file is missing. Check the DistributionSettings section in the Android tab of Project Settings


ERROR: One of your asset files is named [1mmain.1.com.VictorLi.EBuildingVR.obb[0m which contains the version code. This format is used for the single [1m--obb[0m file. To leverage the faster update system, asset file names should be persistent across build updates. Please remove the version [1m1[0m from the asset name.

