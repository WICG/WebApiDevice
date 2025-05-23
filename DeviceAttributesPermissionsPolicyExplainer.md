# Device Attributes Permissions Policy Explainer

## Introduction

This document suggests two changes to the existing
[Device Attributes API](https://wicg.github.io/WebApiDevice/device_attributes/):

- Establish the Device Attributes API as a
  [policy controlled feature](https://www.w3.org/TR/permissions-policy-1/#policy-controlled-feature)
  identified by the `device-attributes` token.
- Remove the need for configuring an allowlist to grant the device attributes
  permission in
  [isolated contexts](https://wicg.github.io/isolated-web-apps/isolated-contexts.html).

The Device Attributes API allows retrieving basic device properties, namely:

- Directory ID
- Hostname
- Serial Number
- Annotated Asset ID
- Annotated Location

The API is available only on managed devices and only for applications installed
by the organizations' administrators. The administrator needs to set an
enterprise policy to allow access to the API.

## Motivation

The goal of establishing the Device Attributes as a policy-controlled feature is
to provide high-watermark permissions.

In the
[isolated context](https://wicg.github.io/isolated-web-apps/isolated-contexts.html),
the web application needs to list all required permissions upfront in the app
manifest. Because isolated context applications are considered more secure, they
can grant the permission instead of requiring the enterprise policy like in the
regular web applications.

## Implementation

The
[Device Attributes API](https://wicg.github.io/WebApiDevice/device_attributes/),
which is available only for policy-installed applications on managed devices,
would become a
[policy controlled feature](https://www.w3.org/TR/permissions-policy-1/#policy-controlled-feature).
In the permissions policy it would be referred to as `device-attributes`.

Additionally, in an isolated context the permission to use the API would be
automatically granted. Regular web applications would still need the permission
granted through enterprise policy, as it is now.

## Usage

For the regular web applications, if the policy for a feature is not declared or
inharited, the application can request the permission. The developer can opt out
of using the API with HTTP headers:

```html
Permissions-Policy: device-attributes=()
```

or specify which origins can request it

```html
Permissions-Policy: device-attributes=(self, "https://a.example.com" "https://b.example.com")
```

In an isolated context, the application needs to list all the policies upfront.
In the [app manifest](https://www.w3.org/TR/appmanifest/), this can be done by
listing the permissions policy like:

```html
"permissions_policy": {
    "device-attributes": ["self"]
}
```
