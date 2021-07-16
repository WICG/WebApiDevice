# Draft Specifications

* [Managed Configuration API](https://wicg.github.io/WebApiDevice/managed_config)
* [Device Attributes API](https://wicg.github.io/WebApiDevice/device_attributes/)

# How to test these features?

## Preconditions
### Supported platforms
Managed Device Web API is only available on Chrome OS devices (Chrome version >= 90.0.4400.8).

### Managed applications
Only managed applications are allowed to use the Managed Web API. Please follow the guide [Automatically install apps and extensions](https://support.google.com/chrome/a/answer/6306504) to configure a web application for test purpose.

## Managed Configuration API
### Setting up Chrome Policy
In order to test out the managed configuration API, some additional preparation has to be done to the device. More specifically, you have to set up a proper value for the ManagedConfigurationPerOrigin policy.

In this instruction, we introduce the approach for Chrome OS and Linux platforms.

#### Configuration file
You can create a file `/etc/opt/chrome/policies/managed/test_policy.json` on a device, which will contain the information about the managed configuration to be set.
If you are using a Chrome OS device, you may want to switch the device into a [developer mode](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/developer_mode.md#dev-mode). After that, this filesystem should be accessible.

Since this API is still in the trial stage, the corresponding server-side UI is not yet ready. Because of that, in order to test it, you will need to host the JSON  configuration in a place, which can provide a direct link to it. For example, at [JsonKeeper](https://jsonkeeper.com/).

In the `test_policy.json` file, you need to override the `ManagedConfigurationPerOrigin` policy to indicate the managed configuration to be used. It is defined as a list of JSON dictionaries, with the following keys for each:
* __origin__ -- defines the Web App origin this configuration applies to
* __managed_configuration_url__ -- the url, where the configuration is hosted
* __managed_configuration_hash__ -- the unique identifier, which is usually calculated by the policy server which distinguishes you configuration from another.

Here is an example of the configuration:

```yaml
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

To verify whether the value was set correctly for the policy, you can open `chrome://policy` page and search for `ManagedConfigurationPerOrigin` entry.

### Verification in the Chrome DevTools Console

After an app is force installed by the administrator and the managed configuration is set, please open the [Chrome DevTools Console](https://developers.google.com/web/tools/chrome-devtools/console) on that page and use the following code snippet to print the configuration in the console.
```javascript
navigator.managed.getManagedConfiguration(["key"]).then(console.log)
```

If the configuration is set properly, the value from the JSON configuration shall be printed in the console. 

## Device Attributes API
### [Optional] Setting up the annotated Asset ID and Location
Please follow the guide [Asset identifier during enrollment](https://support.google.com/chrome/a/answer/2657289?hl=en#allow_to_update_device_attribute) to customize the annotated Asset ID and Location during enrollment if you want to test the corresponding Web APIs.

### Enable developer mode on the test device
Please prepare a Chrome OS device, and switch it into [developer mode](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/developer_mode.md#dev-mode).

### Setting up feature flags
In order to enable this API in Chrome, the experimental web platform features should be enabled. It can be done either by either of the following approaches.
* Turn on the `enable-experimental-web-platform-features` flag in the Experiments page (`chrome://flags`).
* Add `--enable-experimental-web-platform-features` into `/etc/chrome_dev.conf` file on your test device.

### Setting up test policies
Please add a `DeviceAttributesAllowedForOrigins` policy into `test_policy.json` file to allow specific origins to access device attributes. Here is an example of enabling the permissions for Google search website.

```yaml
{
  "DeviceAttributesAllowedForOrigins": [
    "https://www.google.com"
  ]
}
```

### Verification in the Chrome DevTools Console
To verify the correctness of this API, you can test it by using [Chrome DevTools Console](https://developers.google.com/web/tools/chrome-devtools/console). Take an example, the following code snippet can be used to print the current device's serial number in the console.
```javascript
navigator.managed.getSerialNumber(console.log);
```

### Attention!
* Device attributes API is unavailable if the current user is unaffiliated (i.e. the account enrolled into the device and session don't have the same domain).
* Device attributes API is unavailable if the current application is not managed.
* Device attributes API is unavailable in the Incognito mode.
