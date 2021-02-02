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
#### Setting up Chrome Policy
In order to test out managed configuration API, some additional preparation has to be done to the device. More specifically, we have to set up a values for ManagedConfigurationPerOrigin policy.  

In this instruction, we will currently describe a way of doing so for Chrome OS and Linux platforms.

#### Hosting configuration

The configuration that is provided to the application is loaded dynamically and not sent as a part of a policy blob. What this means for testing is that the policy providing server(for Google Chrome, it is Google Admin Console) is sending a *link* to the configuration file to the browser instead of the raw configuration. This is done to cover cases where the configuration value is larger than expected. 

In the future, this should not be an obstacle, as the corresponding UI will be implemented in the Admin Console. However, now, in order to test it, you may want to host your JSON configuration somewhere, where you can get a direct link to the configuration. For example, at [JsonKeeper](https://jsonkeeper.com/).

##### Configuration file
One can create a file */etc/opt/chrome/policies/managed/test_policy.json* on a device, which will contain the information about the managed configuration to be set.
If you are using a Chrome OS device, you may want to switch the device into a [developer mode](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/developer_mode.md#dev-mode). After that, this filesystem should be accessible.

In the file, one can override any user policy. We will override *ManagedConfigurationPerOrigin*. This policy is defined as a list of JSON dictionaries, with the following keys:
- __origin__ -- defines the Web App origin this configuration applies to
- __managed_configuration_url__ -- the url, where the configuration is hosted
- __managed_configuration_hash__ -- the unique identifier, which is usually calculated by the policy server which distinguishes one configuration from another

Here is an example of the configuration:

```
{
  "ManagedConfigurationPerOrigin": [
  {
    "origin": "https://trustedorigin.com",
    "managed_configuration_url": "__Insert your configuration URL here__",
    "managed_configuration_hash": "__Insert a random string here, configuration URL, for example__"
  }
  ]
}
```

To verify whether the value was set correctly for the policy, one can open *chrome://policy* page and search for *ManagedConfigurationPerOrigin* entry.

#### Verification in the Chrome DevTools Console

After an app is force installed by the administrator and the managed configuration is set, user can open Chrome DevTools Console on that page.

There, the user may call the managed configuration API like this:
```
navigator.device.getManagedConfiguration(["key"]).then(console.log)
```

If the configuration is set properly, the value from the JSON configuration shall be printed in the console. 

### Device Attributes
#### Verification in the Chrome DevTools Console
The easiest way to verify the attribute results is calling these APIs in the [Chrome DevTools Console](https://developers.google.com/web/tools/chrome-devtools/console). Take an example, the following code snippet can be used to print the current device's serial number in the console.
```javascript
navigator.device.getSerialNumber(console.log);
```
#### Annotated Asset ID and Location
Please follow the guide [Asset identifier during enrollment](https://support.google.com/chrome/a/answer/2657289?hl=en#allow_to_update_device_attribute) to customize the annotated Asset ID and Location during enrollment if you want to test the corresponding Web APIs.
