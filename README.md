# WebApiDevice
## How to test these features?

### Precondition
#### Supported Platform
Currently, the Managed Device Web API is only available on Chrome OS devices (Chrome version >= 90.0.4400.8).
#### Feature flag
To test these new APIs, the following two feature flags need to be turned on in the Experiments page (**chrome://flags**).
* **enable-experimental-web-platform-features**: enables experimental Web Platform features that are in development.
* **enable-restricted-web-apis**: enables the restricted web APIs for the 'Dev Trials' stage.
#### Trusted application
Only trusted applications are available to use the Managed Web API. Please follow the guide [Automatically install apps and extensions](https://support.google.com/chrome/a/answer/6306504) to configure a web application for test purpose.

### Managed Configuration
TBD.

### Device Attributes
#### Verification in the Chrome DevTools Console
The easiest way to verify the attribute results is calling these APIs in the [Chrome DevTools Console](https://developers.google.com/web/tools/chrome-devtools/console). Take an example, the following code snippet can be used to print the current device's serial number in the console.
```javascript
navigator.device.getSerialNumber(console.log);
```
#### Annotated Asset ID and Location
Please follow the guide [Asset identifier during enrollment](https://support.google.com/chrome/a/answer/2657289?hl=en#allow_to_update_device_attribute) to customize the annotated Asset ID and Location during enrollment if you want to test the corresponding Web APIs.
