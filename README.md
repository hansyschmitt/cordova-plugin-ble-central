# BLE plugin for Apache Cordova with Nordic DFU

This plugin enables communication between a phone and Bluetooth Low Energy (BLE) peripherals. It is
a fork of excellent
[don/cordova-plugin-ble-central](https://github.com/don/cordova-plugin-ble-central) plugin enriched
with Nordic Semiconductors [Android](https://github.com/NordicSemiconductor/Android-DFU-Library) and
[iOS](https://github.com/NordicSemiconductor/IOS-Pods-DFU-Library) DFU libraries.

**For the main documentation, please visit the [base plugin GitHub
page](https://github.com/don/cordova-plugin-ble-central).**
This page covers only additional installation requirements and extended API.

---

# Requirements

For using this plugin on iOS, there are some additional requirements:

- `cordova` version >= 6.4
- `cordova-ios` version >=4.3.0
- [CocoaPods](https://guides.cocoapods.org/using/getting-started.html#installation)

# Installing

    $ cordova plugin add https://github.com/flemmingdjensen/cordova-plugin-ble-central.git

# Extended API

### PhoneGap Build

Edit config.xml to install the plugin for [PhoneGap Build](http://build.phonegap.com).

    <gap:plugin name="cordova-plugin-ble-central" source="npm" />
    <preference name="phonegap-version" value="cli-6.1.0" />

### PhoneGap Developer App

This plugin is included in iOS and Android versions of the [PhoneGap Developer App](http://app.phonegap.com/).

Note that this plugin's id changed from `com.megster.cordova.ble` to `cordova-plugin-ble-central` as part of the migration from the [Cordova plugin repo](http://plugins.cordova.io/) to [npm](https://www.npmjs.com/).

### iOS

For iOS, apps will crash unless they include usage description keys for the types of data they access. Applications targeting iOS 13 and later, define [NSBluetoothAlwaysUsageDescription](https://developer.apple.com/documentation/bundleresources/information_property_list/nsbluetoothalwaysusagedescription?language=objc) to tell the user why the application needs Bluetooth. For apps with a deployment target earlier than iOS 13, add [NSBluetoothPeripheralUsageDescription](https://developer.apple.com/documentation/bundleresources/information_property_list/nsbluetoothperipheralusagedescription?language=objc). Both of these keys can be set when installing the plugin by passing the BLUETOOTH_USAGE_DESCRIPTION variable.

    $ cordova plugin add cordova-plugin-ble-central --variable BLUETOOTH_USAGE_DESCRIPTION="Your description here"

See Apple's documentation about [Protected Resources](https://developer.apple.com/documentation/bundleresources/information_property_list/protected_resources) for more details. If your app needs other permissions like location, try the [cordova-custom-config plugin](https://github.com/don/cordova-plugin-ble-central/issues/700#issuecomment-538312656).

It is possible to delay the initialization of the plugin on iOS. Normally the Bluetooth permission dialog is shown when the app loads for the first time. Delaying the initialization of the plugin shows the permission dialog the first time the Bluetooth API is called. Set `IOS_INIT_ON_LOAD` to false when installing.

    --variable IOS_INIT_ON_LOAD=false

### Android

If your app targets Android 10 (API level 29) or higher, you have also the option of requesting the ACCESS_BACKGROUND_LOCATION permission. If your app has a feature that requires it, set `ACCESS_BACKGROUND_LOCATION ` to true when installing.

    --variable ACCESS_BACKGROUND_LOCATION=true

# API

## Methods

- [all methods from the original plugin](https://github.com/don/cordova-plugin-ble-central#methods)
- [`ble.upgradeFirmware`](#upgradeFirmware)

## upgradeFirmware

Upgrade a peripheral firmware.

```javascript
ble.upgradeFirmware(device_id, uri, progress, failure);
```

### Description

Function `upgradeFirmware` upgrades peripheral firmware using the Nordic Semiconductors'
[proprietary DFU
protocol](https://devzone.nordicsemi.com/documentation/nrf51/6.0.0/s110/html/a00062.html) (hence
only Nordic nRF5x series devices can be upgraded). It uses the official DFU libraries for each
platform and wraps them for use with Apache Cordova. Currently only supported firmware format is a
[ZIP file](https://devzone.nordicsemi.com/blogs/715/creating-zip-package-for-dfu/) prepared using
Nordic CLI utilities.

The function presumes a connected BLE peripheral. A progress callback is called multiple times with
upgrade status info, which is a JSON object of the following format:

```javascript
{
    "status": "--machineFriendlyString--"
}
```

A complete list of possible status strings is:

- `deviceConnecting`
- `deviceConnected`
- `enablingDfuMode`
- `dfuProcessStarting`
- `dfuProcessStarted`
- `firmwareUploading`
- `progressChanged` - extended status info
- `firmwareValidating`
- `dfuCompleted`
- `deviceDisconnecting`
- `deviceDisconnected` - the last callback on successful upgrade
- `dfuAborted` - the last callback on user abort

The list is only approximately ordered. *Not all statuses all presented on both platforms.*
If `status` is `progressChanged`, the object is extended by a `progress` key like so:

```javascript
{
    "status": "progressChanged",
    "progress": {
        "percent": 12,
        "speed": 2505.912325285,
        "avgSpeed": 1801.8598291,
        "currentPart": 1,
        partsTotal: 1
    }
}
```

In a case of error, the JSON object passed to failure callback has following structure:

```javascript
{
    "errorMessage": "Hopefully human readable error message"
}
```

Please note, that the device will disconnect (possibly multiple times) during the upgrade, so the
[ble.connect](https://github.com/don/cordova-plugin-ble-central#connect) error callback will
trigger. This is intentional.

### Parameters

- __device_id__: UUID or MAC address of the peripheral
- __uri__: URI of a firmware ZIP file on the **local filesystem**
  (see [cordova-plugin-file](https://github.com/apache/cordova-plugin-file))
- __progress__: Progress callback function that is invoked multiple times with upgrade status info
- __failure__: Error callback function, invoked when an error occurs

### Quick Example

```javascript
// presume connected device

var device_id = "BD922605-1B07-4D55-8D09-B66653E51BBA"
var uri = "file:///var/mobile/Applications/12312-1231-1231-123312-123123/Documents/firmware.zip";

ble.upgradeFirmware(device_id, uri, console.log, console.error);
```
