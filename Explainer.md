# Managed Device Web API
## What is this?
This is a proposal to add a series of device related Web APIs that are expected to be used by managed applications (e.g. applications installed by the administrator). These APIs are explicitly enabled by device owners or administrators through an enterprise policy or an equivalent mechanism. Supporting the similar functionalities in unmanaged devices is a non-goal.
## What is the motivation?
It’s a common requirement for web developers to provide diversified user experience or build a verified operating environment based on device-specific characteristics and configurations, especially in the field of commercial applications.

Chrome browser started providing such kinds of capabilities in the form of Chrome API many years ago. Although they have been widely used in a variety of scenarios, the drawback is still apparent - Chrome API is not platform independent. In order to better fulfill the ever-increasing demand on web applications, it will be helpful to add a set of similar Web APIs to fill this gap.
## What are managed applications?
The APIs that are able to get device information are usually treated as powerful capabilities because this data belongs to [Personally Identifiable Information (PII)](https://en.wikipedia.org/wiki/Personal_data), thus exposing it to the general web would be a privacy violation. Therefore, these Web APIs are only avaliable in the managed devices with consent from the device owners or administrators. They usually only want to allow them in appropriate cases, and limit to a set of pre-approved web applications they understand and trust.

To reduce the potential privacy and security risks, these API should cowork with a matching authentication mechanism defined by various browsers. There should be a central management console to control applications setup and permissions. Ideally the application permissions should be decided by a IT administrator role (rather than a regular user role) because individual users may expose their sensitive information unconsciously.

Origin of a managed web application is considered trusted. This is a condition to expose the API, as origin is de facto the boundary mechanism on the Web (permissions, local storage, requests are all scoped/restricted per origin). 

E.g. in Chrome browser the status of managed applications is given to those origins, which correspond to web applications selected by organization administrators, that are configured in the [Google Admin Console](https://support.google.com/a/topic/2413312) and forced-installed on the enterprise managed devices. The permissions can be revoked if administrators don't want to provide specific functionalities on the managed devices.
## How is Managed Device Web API defined?
We propose to add a new read-only property ‘managed’ into the [navigator](https://developer.mozilla.org/en-US/docs/Web/API/Navigator) interface. It contains all powerful methods and related properties enabled for managed applications.

Technically speaking, the API signatures are always exposed to any caller in Javascript, but only the applications that meet the criteria can get a meaningful result. For specific browsers that provide the ability to switch users, the availability of Managed Device Web API is verified and reconfigured whenever the active user is changed to another. The same web applications cannot use these APIs any longer if they are not considered a managed application in a different session.

## Detailed description
### Managed Configuration
Managed Configuration Web API is a subset of Managed Device Web API, that provides web applications the capabilities to access external configuration set by the device administrator.

On devices that are managed by an organization, there is a need to thoroughly set up the environment for the web applications before use. The configuration for each device may be not exactly the same, or even changes over time. What’s more important, the device administrator is not necessarily the owner of web applications they are using, which means that it’s impossible to set up everything on the web application side. Some real use cases are as follows.
* **Signage configuration**: an enterprise application pulls the commercial content from the management console, then demonstrates it to the customers on in-store devices.
* **Personalization**: an enterprise application pulls the application settings (i.e., single-touch or multi-touch on a tablet) from the management console preferred by administrators, then applies them to the local devices.

Managed Configuration API only returns meaningful values when the web application is trusted by the user agent, otherwise it will fail with an exception.
#### Interface definition
**Promise\<object\> navigator.managed.getManagedConfiguration(sequence\<DOMString\> keys)**  
Returns a promise, which will be resolved into an object containing key-value pairs for each of the key from |keys| that is configured for this application.
* This promise is rejected with a 'NotAllowedError' DOMException in case the web application is not configurable by the administrator. This is done so that applications are able to distinguish between such two cases and notify the user accordingly.

**void navigator.managed.addEventListener(‘managedconfigurationchange’, onChange)**  
Registers an observer for any changes in managed configuration. |onChange| is called upon any configuration update.
#### Usage example
Suppose a device administrator owns a department store chain, which includes a fleet of devices of different purposes. Their roles and means of interaction vary. For some devices, the following configuration is set: 
```json
{
   "interactable" : "false",
   "deviceType" : "map"
}
```

A client, which would like to get a configuration for a particular _key_ would call:

```javascript
navigator.managed.getManagedConfiguration(["interactable"])
 .then(function(result) {
     // result = { “interactable” : “false” } 
     // Process the value of the key.
});
```
If the value is unset for a _key_, the result will not include that _key_.

Example requesting multiple keys:
```javascript
navigator.managed.getManagedConfiguration(["interactable","deviceType","theme"])
 .then(function(result) {
      // result = { “interactable” : "false", 
      //            “deviceType” : "map" }   
      // Process the value of the keys.
});
```

For web apps that are not _trusted_, the promise gets rejected.

```javascript
navigator.managed.getManagedConfiguration(["interactable","deviceType","theme"])
 .then(onSuccess, function(error) { 
      console.log(error.name); // Will print "NotAllowedArror");
});
```

Users can track updates in managed configuration by calling addEventListener method:

```javascript
navigator.managed.addEventListener("managedconfigurationchange", 
 function() { 
     // Whenever something changes in the configuration, this method is                
     // called.
});
```
###  Device Attributes
Device Attributes Web API is a subset of Managed Device Web API, that provides to web applications the capability to query device information (device ID, serial number, location, etc). Some real use cases are as follows.
* **Virtual Desktop Infrastructure (VDI)**: an enterprise application launched on the client side needs to pull the device ID / serial number from the local device it is running on. Then the VDI provider can rely on this information to determine which user is using which device at any point in time.
* **Context-based configuration**: an enterprise application needs to apply a specific configuration to a local device based on device attributes like location, asset ID. Then different users can have appropriate experience respectively.

In addition to the requirement of being called by a managed application, Device Attributes Web API usually asks for more consents from the device users or administrators before they are able to return meaningful results. Please check the rules in the definition of each.
#### Interface definition
**Promise\<DOMString\> navigator.managed.getAnnotatedAssetId()**
Returns a promise for a string. It contains an administrator-defined value which identifies a device within an organization.
* If this API is not called by a managed application, the promise is rejected with a ‘NotAllowedError’ DOMException.
* If no Annotated Asset Id has been set by the administrator, the promise is resolved with ‘undefined’ value.

**Promise\<DOMString\> navigator.managed.getAnnotatedLocation()**
Returns a promise for a string. It contains an administrator-defined value which identifies a location within an organization.
* If this API is not called by a managed application, this promise is rejected with a ‘NotAllowedError’ DOMException.
* If no Annotated Location has been set by the administrator, the promise is resolved with ‘undefined’ value.

**Promise\<DOMString\> navigator.managed.getDirectoryId()**
Returns a promise for a string. It contains an inventory management system-defined value which identifies a device within an organization.
* If this API is not called by a managed application, the promise is rejected with a ‘NotAllowedError’ DOMException.
* If no Directory Id has been set, the promise is resolved with ‘undefined’ value.

**Promise\<DOMString\> navigator.managed.getHostname()**
Returns a promise for a string. It contains an administrator-defined value which is used as the device hostname during DHCP request.
* If this API is not called by a managed application, the promise is rejected with a ‘NotAllowedError’ DOMException.
* If no Hostname has been set by the administrator, the promise is resolved with ‘undefined’ value.

**Promise\<DOMString\> navigator.managed.getSerialNumber()**
Returns a promise for a string. It contains a manufacturer-defined value which uniquely identifies a device.
* If this API is not called by a managed application, the promise is rejected with a ‘NotAllowedError’ DOMException.
#### Usage example
Assuming there is a retail enterprise that relies on an online sales system. The backend service pushes different tariffs to the in-store devices based on their annotated location (country, city or sales region) in the morning, and collects sales reports in the afternoon.

With the help of Device Attributes Web API, this sales application can get the device location by using **getAnnotatedLocation()** method. The operation will fail if the API call is triggered by a phishing application - looks similar but not the expected one.

```javascript
// request sensitive data if the current environment is valid.
function successCallback(location) {
  const tariff = backend.requestTariff(location);
  console.log(tariff);
}

// stop the workflow if the current environment is unexpected.
function failureCallback(error) {
  backend.reportFailure(error);
  console.error(error.message);
}

function PrepareTariff() {
  navigator.managed.getAnnotatedLocation()
                   .then(successCallback, failureCallback);
}
```

It is easy to write another similar code snippet to report the sales data including a device serial number by using **getAnnotatedAssetId()** method. The service side can rely on this additional information to double check whether the data comes from an expected device.

```javascript
function ReportSalesData() {
  navigator.managed.getAnnotatedAssetId.then(reportCallback);
}
```
#### Design considerations
##### Single getter method with keys
This option was considered in the beginning, but wasn't chosen because of the following reasons:
* It doesn't simplify the specification of various device attributes. Each reserved key still needs to be defined separately.
* It also needs to be checked if implemented in a specific API version.
##### Reusing managed configuration
Adding new special keys for device attributes to managed configuration has difficulties:
* Ensuring that the new key isn't already used by some managed configurations. Such conflict may suddenly break the existing web applications.
* Managed configuration is set as an application setting, meaning that it is the same config received by all devices in some organisational unit that host this application. On the other hand device attributes (e.g. annotated location) can be set uniquely per device (as device settings).
* Managed configuration API doesn't require a managed (force-installed) application, but the Device Attributes API does.
