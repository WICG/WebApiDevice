# Device Attributes Permissions Policy Explainer

## Introduction

This document suggests two changes to the existing
[Device Attributes API](https://wicg.github.io/WebApiDevice/device_attributes/):

- Establish the Device Attributes API as a
  [policy controlled feature](https://www.w3.org/TR/permissions-policy-1/#policy-controlled-feature)
  identified by the `device-attributes` token.
- Establish the Device Attributes API as a
  [powerful feature](https://w3c.github.io/permissions/#dfn-powerful-feature)
  identified by the `device-attributes` token.

The Device Attributes API allows retrieving basic device properties, namely:

- Directory ID
- Hostname
- Serial Number
- Annotated Asset ID
- Annotated Location

The API is intended only for use on managed devices, where permission for an
origin to access this feature is controlled by the organizations'
administrators.

## Motivation

The Device Attributes API allows web developers to query information about the
device. This information can be used for context-based configuration or other
device-aware use cases such as licensing.

The goal of this change is to make the Device Attributes API usable with less
configuration, while still maintaining administrators' control over what
applications are allowed to access the API.

## Implementation

The
[Device Attributes API](https://wicg.github.io/WebApiDevice/device_attributes/)
would become a
[policy controlled feature](https://www.w3.org/TR/permissions-policy-1/#policy-controlled-feature)
and a
[powerful feature](https://w3c.github.io/permissions/#dfn-powerful-feature). In
the permissions policy it would be referred to as `device-attributes`. The
permission to use the API would be granted based on the policies set by the
device administrators.

## Usage

If the permission to use the Device Attributes API for an origin is granted, the
application can either use it without specifying the permissions policy, or
explicitly declare it, for example with the Permissions-Policy header:

```html
Permissions-Policy: device-attributes=(self)
```

or

```html
Permissions-Policy: device-attributes=("https://a.example.com")
```

The developer can also opt out of using the API:

```html
Permissions-Policy: device-attributes=()
```

In an isolated context, the application needs to list all the policies upfront.
In the [app manifest](https://www.w3.org/TR/appmanifest/), this can be done by
listing the permissions policy like:

```html
"permissions_policy": {
    "device-attributes": ["self"]
}
```
