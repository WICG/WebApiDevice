# Device Attributes Permissions Policy Explainer

## Introduction

The `device-attributes` is a new Permissions-Policy regulating the access to the
[Device Attributes API](https://wicg.github.io/WebApiDevice/device_attributes/)
permission.

The Device Attributes API allows retrieving basic device properties, namely:

- Directory ID
- Hostname
- Serial Number
- Annotated Asset ID
- Annotated Location

The API is available only on managed devices, and only for applications
installed by the organizations' administrators. The administrator needs to set
an enterprise policy to allow access to the API.

This proposal suggest two changes to the existing Device Attributes API:

- Introduce the device-attributes permissions policy.
- Remove the need for configuring an allowlist to grant the device attributes
  permission in
  [isolated contexts](https://wicg.github.io/isolated-web-apps/isolated-contexts.html).

## Motivation

The goal of the new Permissions Policy for Device Attributes is to provide
high-watermark permissions.

In the
[isolated context](https://wicg.github.io/isolated-web-apps/isolated-contexts.html),
the web application needs to list all required permissions upfront in the app
manifest. Because isolated context applications are considered more secure, they
can grant the permission, instead of requiring the enterprise policy like in the
regular web applications.

## Implementation

The Permissions-Policy is called `device-attributes`. It allows requesting
permission to use the
[Device Attributes API](https://wicg.github.io/WebApiDevice/device_attributes/),
which is available only for policy-installed applications on managed devices.
Regular web applications need the permission granted through enterprise policy.
In an isolated context, the permission is automatically granted.

## Usage

For the regular web applications, if the permissions policy is not explicitly
declared, the application can request the permission. The developer can opt out
of using the API with HTTP headers:

```html
Permissions-Policy: device-attributes=()
```

or specify which origins can request it

```html
Permissions-Policy: device-attributes=(self, "https://a.example.com" "https://b.example.com")
```

In an isolated context, the application needs to list the required Permissions
Policies upfront. In the [app manifest](https://www.w3.org/TR/appmanifest/),
this can be done by listing the permissions policy like:

```html
"permissions_policy": {
    "device-attributes": ["self"]
}
```
