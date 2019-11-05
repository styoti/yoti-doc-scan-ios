# Yoti Doc Scan iOS SDK

![Illustration](./Illustration.png)

The Yoti Doc Scan iOS SDK allows a user of your app to take a photo of their ID, as well as to scan their face, we then verify this instantly and prepare a response, which your system can then retrieve on your hosted site.

## Prerequisites
In order to integrate with the iOS SDK of Yoti Doc Scan, a working infrastructure is needed.
Please see [developers.yoti.com](https://developers.yoti.com/yoti-doc-scan/yoti-doc-scan-integration-introduction) for more details.

## Requirements
- iOS 11+
- Swift 5+

## Installation

### Carthage
Make sure you are running the latest version of [Carthage](https://github.com/carthage/carthage) and [Git LFS](https://git-lfs.github.com) by running:
```bash
brew update
brew upgrade carthage
brew upgrade git-lfs
```

Create a [Cartfile](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md#cartfile) in the same directory where your `.xcodeproj` or `.xcworkspace` is and add the following lines to it:
```bash
github "getyoti/yoti-doc-scan-ios" == 1.1.0
github "getyoti/yoti-doc-capture-ios" == 1.5.5
github "BlinkID/blinkid-ios" == 4.7.0
```

Run `carthage update`.

**Note:** This will fetch dependencies into a `Carthage/Checkouts` folder, then copy `YotiDocScan.framework` and `ScanDocument.framework` into a `Carthage/Build/iOS` folder. Please find `Microblink.framework` and `Microblink.bundle` in the `Carthage/Checkouts/blinkid-ios` folder.


On your application targets' `Build Phases` tab:
- Click `+` icon and choose `New Run Script Phase`.
- Create a script with a shell of your choice (e.g. `/bin/sh`).
- Add the following to the script area below the shell:
```bash
/usr/local/bin/carthage copy-frameworks
```

- Add the paths to the frameworks you want to use under `Input Files`:
```bash
$(SRCROOT)/Carthage/Build/iOS/YotiDocScan.framework
$(SRCROOT)/Carthage/Build/iOS/ScanDocument.framework
$(SRCROOT)/Carthage/Build/iOS/ZoomAuthenticationHybrid.framework
```

- Then add the paths to the copied frameworks under `Output Files`:
```bash
$(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/YotiDocScan.framework
$(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/ScanDocument.framework
$(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/ZoomAuthenticationHybrid.framework
```

### Add Libraries and Resources
Add the following libraries at `Build Phases` → `Link Binary With Libraries`
- `Microblink.framework`
- `AVFoundation.framework`
- `AudioToolbox.framework`
- `CoreMedia.framework`
- `libc++.tbd`
- `libiconv.tbd`
- `libz.tbd`

And the following resources at `Build Phases` → `Copy Bundle Resources`
- `Microblink.bundle`

In addition, you should use [Git LFS](https://git-lfs.github.com) to track the `Microblink.bundle`:
```bash
git lfs track "<path>/Microblink.bundle/**"
```

### Configuration
Add a `Strip Unused Architectures` script

On your application targets' `Build Phases` tab:
- Click `+` icon and choose `New Run Script Phase`.
- Create a script with a shell of your choice (e.g. `/bin/sh`).
- Add the following to the script area below the shell:
```bash
APP_PATH="${TARGET_BUILD_DIR}/${WRAPPER_NAME}"

# This script loops through the frameworks embedded in the application and
# removes unused architectures.
find "$APP_PATH" -name '*.framework' -type d | while read -r FRAMEWORK
do
FRAMEWORK_EXECUTABLE_NAME=$(defaults read "$FRAMEWORK/Info.plist" CFBundleExecutable)
FRAMEWORK_EXECUTABLE_PATH="$FRAMEWORK/$FRAMEWORK_EXECUTABLE_NAME"
echo "Executable is $FRAMEWORK_EXECUTABLE_PATH"

EXTRACTED_ARCHS=()

for ARCH in $ARCHS
do
echo "Extracting $ARCH from $FRAMEWORK_EXECUTABLE_NAME"
lipo -extract "$ARCH" "$FRAMEWORK_EXECUTABLE_PATH" -o "$FRAMEWORK_EXECUTABLE_PATH-$ARCH"
EXTRACTED_ARCHS+=("$FRAMEWORK_EXECUTABLE_PATH-$ARCH")
done

echo "Merging extracted architectures: ${ARCHS}"
lipo -o "$FRAMEWORK_EXECUTABLE_PATH-merged" -create "${EXTRACTED_ARCHS[@]}"
rm "${EXTRACTED_ARCHS[@]}"

echo "Replacing original executable with thinned version"
rm "$FRAMEWORK_EXECUTABLE_PATH"
mv "$FRAMEWORK_EXECUTABLE_PATH-merged" "$FRAMEWORK_EXECUTABLE_PATH"
done
```

## Integration

### 1. Launching the SDK
Perform the following actions to initialize and present the SDK.
```swift
// Create an instance of our navigation controller.
let navigationController = YotiDocScanNavigationController()

// To specify the `sessionID` and `clientSessionToken`.
navigationController.yotiDocScanDataSource = self

// To perform UI customizations and to handle the verification result.
navigationController.yotiDocScanDelegate = self

// Present the navigation controller.
present(navigationController, animated: true, completion: nil)
```

### 2. Specifying the Session ID and Client Session Token
Conform to `YotiDocScanDataSource`.
```swift
// Configuring the Session ID is required.
func sessionID(for navigationController: YotiDocScanNavigationController) -> String {
    return "Please insert the [Session ID] here"
}

// Configuring the Client Session Token is required.
func clientSessionToken(for navigationController: YotiDocScanNavigationController) -> String {
    return "Please insert the [Client Session Token] here"
}
```

### 3. UI customizations and handling the verification result
Conform to `YotiDocScanDelegate`.
```swift
// Configuring the primary color is optional.
func primaryColor(for navigationController: YotiDocScanNavigationController) -> UIColor {
    return .blue
}

// Handle the result of the verification process.
func yotiDocScan(
    _ navigationController: YotiDocScanNavigationController,
    didFinishWithResult result: YotiDocScanResult) {

    // Dismiss the SDK.
    dismiss(animated: true)

    // Handle the result from the SDK.
    switch result {
    case .success:
        return
    case .failure(let error):
        print(error)
    }
}
```

## Error Handling
Please refer to the following table for a description of error codes that may be returned to you as part of a failed verification.

Code | Description | Retry possible (same session)
:-- | :-- | :--
1000 | No error occurred. The user cancelled the session for an unknown reason | Yes
2000 | Unauthorised request (wrong or expired session token) | Yes
2001 | Session not found | Yes
2002 | Session expired | Yes
2003 | SDK launched without session Token | Yes
2004 | SDK launched without session ID | Yes
3000 | Yoti's services are down or unable to process the request | Yes
3001 | An error occurred during a network request | Yes
3002 | User has no network | Yes
4000 | The user did not grant permission to the camera | Yes
5000 | No camera. The user's camera was not found and file upload is not allowed | No
6000 | SDK is out-of-date. Please update the SDK to the latest version | No
6001 | Unexpected internal error | No
6002 | Unexpected document scanning error | No

## Support
If you have any other questions please do not hesitate to contact sdksupport@yoti.com.
Once we have answered your question we may contact you again to discuss Yoti products and services. If you'd prefer us not to do this, please let us know when you e-mail.

## Licence
Please find the licence for the Yoti Doc Scan iOS SDK [here](https://www.yoti.com/wp-content/uploads/2019/08/Yoti-Doc-Scan-SDK-Terms.pdf).
