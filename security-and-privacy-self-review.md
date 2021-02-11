## Security and Privacy Self-Review for Managed Device Web API

Responses to the [Self-Review Questionnaire: Security and Privacy](https://www.w3.org/TR/security-privacy-questionnaire/) for the [Managed Device Web API](https://github.com/WICG/WebApiDevice).

### 1. What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?
There are two subsets in the Managed Device Web API.
- **Managed Configuration:** the web sites are able to get the managed configuration prepared by the device administrators by using this API, then provide customized user experience for a group of managed devices according to the configuration values.
- **Device Attributes:** the web sites are able to get the specific device attributes from the managed devices by using this API. Some of them can be overwrited by device users during the enrollment.

More specific [use cases](https://github.com/WICG/WebApiDevice/blob/master/Explainer.md) can be found in the Explainer.md.

### 2. Is this specification exposing the minimum amount of information necessary to power the feature?
Yes.

### 3. How does this specification deal with personal information or personally-identifiable information or information derived thereof?
This is discussed in [What are trusted applications?](https://github.com/WICG/WebApiDevice/blob/master/Explainer.md#what-are-trusted-applications) paragraph of the Explainer.md. These APIs are required to be used in the managed devices and trusted applications. The device administrators are responsible to understand the potential risk and control the accessibility when the APIs are allowed to be used.

### 4. How does this specification deal with sensitive information?
These APIs are required to be used in the managed devices and trusted applications. The device administrators are responsible to understand the potential risk and control the accessibility when the APIs are allowed to be used.

### 5. Does this specification introduce new state for an origin that persists across browsing sessions?
No.

### 6. What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?
The API exposes data, which is configured by the device administrator per app and per device along with a serial number of the device which is running the website.

### 7. Does this specification allow an origin access to sensors on a user’s device?
No.

### 8. What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.
The API exposes data, which is configured by the device administrator per app and per device along with a serial number of the device which is running the website. This data is exposed only to the trusted applications, while for all other origins the API calls will always fail.

### 9. Does this specification enable new script execution/loading mechanisms?
No.

### 10. Does this specification allow an origin to access other devices?
No.

### 11. Does this specification allow an origin some measure of control over a user agent’s native UI?
No.

### 12. What temporary identifiers might this  specification create or expose to the web?
A website can use any of device attributes provided by the API to create a temporary identifier of the device (annoted asset id, annotated location, directory id) and, if explicitly allowed by the admin, a persistent identifier (serial number).

### 13. How does this specification distinguish between behavior in first-party and third-party contexts?
It does not distinguish between them.

### 14. How does this specification work in the context of a user agent’s Private Browsing or "incognito" mode?
These APIs are not avaiable in Private Browsing or "incognito" modes, since there a different user profile is used.

### 15. Does this specification have a "Security Considerations" and "Privacy Considerations" section?
Not yet, as we're not at the specification stage.

### 16. Does this specification allow downgrading default security characteristics?
No.
