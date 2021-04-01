# Managed Configuration API for Enterprise Web Applications

## Objective
Provide an API which will be able to asynchronously obtain/monitor configuration provided by the enterprise admin. This configuration is set up in the Admin Panel by uploading a JSON file, which contains key-value configuration.

## Overview
We will add a new policy, which receive the key-value configuration from the Admin Panel, store it in a client read-only leveldb database associated with the web app id, and provide the access to it via new javascript bindings.

## Detailed design
### Scoping managed configuration
There are several ways we could scope the managed configuration provided to the website – by origin, by web app “scope” or some other heuristic.

It is possible to host(and force-install) several different web applications(with their own manifest, service worker, etc) under the same origin.  One candidate for their identification is the “scope” manifest field, which defines a subset of navigations, which from the UI perspective will behave like one application. However, there are many issues of separating web applications not by their origin, including unreliability of the web app installation pipeline(more research here). 

Hence, the decision was made to restrict managed configuration to be set only by origin, not by each force-installed web application entry. This requires additional server-side work to synchronize managed configuration of several web applications under the same origin.

### Policy stack
Since the app configuration can potentially be big, we should not send the data to the browser directly, but rather send a link to the data, which will be downloaded asynchronously. 

We created a new policy called “ManagedConfigurationPerOrigin”, which holds a list of links to the configuration file and the hashed value of that file for each of the configured origins. We will redownload the configuration in case of the hash change. 

```js
[ 
    {
        'origin': 'https://www.google.com',
        'managed_configuration_url' :
           'https://gstatic.google.com/configuration.json',
        'managed_configuration_hash' : 'asd891jedasd12ue9h'
    }, 
    {
        'origin': 'https://www.example.com',
        'managed_configuration_url' :
            'https://gstatic.google.com/configuration2.json',
        'managed_configuration_hash' : 'djio12easd89u12aws'
    }
]
```

### Renderer API exposure
This API will be exposed in the way that is described in the infrastructure doc. It will be exposed to all webpages, however the promises returned by the API will fail for origins without a force-installed app.

We will add new fields under *navigator.managed.** .

```java
[
  ContextEnabled=TrustedContext,
  SecureContext,
  Exposed=Window,
] interface NavigatorManagedData : EventTarget {
  [CallWith=ScriptState]
   Promise<record<DOMString, object>> getManagedConfiguration
                                        (sequence<DOMString> keys);

  attribute EventHandler managedconfigurationchange;
};
```

In order to obtain and track managed configuration, we will implement ManagedConfigurationAPI keyed service, which will be connected to the frame service called ManagedConfigurationService.

Their interaction will be defined like this:
```java
interface ManagedConfigurationObserver {
  OnConfigurationChanged();
};


interface ManagedConfigurationSource {
  // Get the managed configuration for the web app associated with the     
  // keys.
  GetManagedConfiguration(array<string> keys) =>
                          (map<string, string>? configurations);

  // Allows to subscribe to the managed configuration updates.
  SubscribeToManagedConfiguration(
    pending_remote<ManagedConfigurationObserver> observer) =>            
                   (DeviceAPIResult result);
};
```

This service is attached to the RenderFrameHost upon creation. 

Whenever the user tries to observe the changes in the managed configuration, the renderer will subscribe to the configuration updates by pushing its remote via the *ManagedConfigurationSource* interface.

 Since the *NavigatorManagedData* is bound to the *DOMWindow*(a.k.a. Javascript Frame) and *ManagedConfigurationSource* is bound to the *Frame*, there is 1:1 correspondence between them.
 
### Managing the configuration

A new browser keyed service ManagedConfigurationApi will be created, which will be responsible for policy configuration, downloading and interaction with stored configurations. For each app we would compare the last downloaded configuration hash with the current policy value. If they differ, we will redownload it.

When downloaded, we would parse it as a JSON using a sandboxed [parser](https://source.chromium.org/chromium/chromium/src/+/master:services/data_decoder/public/cpp/data_decoder.h;drc=5b8933e94139a0ab5be46141666fdfcce0f624f6;l=100) to unpack it, and afterwards store the data into a corresponding Web App leveldb.

```c++
class ManagedConfigurationAPI : public KeyedService {
 public:
 class Observer : public base::CheckedObserver {
   public:
    virtual void OnManagedConfigurationChanged() = 0;
  };
   // Tries to retrieve the managed configuration for the |origin|, mapped
   //  by |keys|. Returns a dictionary, mapping each found key to a value.
  void GetOriginPolicyConfiguration(
      const url::Origin& origin,
      const std::vector<std::string>& keys,
      base::OnceCallback<void(std::unique_ptr<base::DictionaryValue>)>
          callback);
          
  void AddObserver(Observer* observer);
  void RemoveObserver(Observer* observer);

 private:
  class ManagedConfigurationDownloader;

  ..
 
   std::map<url::Origin, std::unique_ptr<ManagedConfigurationStore>> store_map_;
```

It will contain in-memory loaded connections to the [levelDb](https://source.chromium.org/chromium/chromium/src/+/master:extensions/browser/value_store/leveldb_value_store.h) databases per domain, or create such on an on-demand basis. 

```c++
class ManagedConfigurationStore {
 public:
  ManagedConfigurationStore(
      const url::Origin& origin,
      const base::FilePath& path);
      
  void AddObserver(ManagedConfigurationAPI::Observer* observer);
  void RemoveObserver(ManagedConfigurationAPI::Observer* observer);

  // Read/Write operations must be called on |backend_sequence_|.
  void SetCurrentPolicy(const base::DictionaryValue& current_configuration);
  ValueStore::ReadResult Get(const std::vector<std::string>& keys);
 private:
  std::unique_ptr<ValueStore> store_;
  ..
};
```

## Alternatives Considered
### New policy namespace
Alternatively, we could have an approach similar to Extensions -- create a custom policy namespace, which will in turn store and synchronize configuration for each Web App separately. This also gives a way to visually observe the policies for Web Apps in a manner that we currently do for extensions.

To avoid encoding issues, the component id for such policies will be the Web App domain.

```c++
enum PolicyDomain {
  // Domain for Chrome policies. |component_id| must be empty.
  POLICY_DOMAIN_CHROME = 0;

  // Domain for policies for regular Chrome extensions. |component_id| must be
  // equal to the extension ID.
  POLICY_DOMAIN_EXTENSIONS = 1;

  // Domain for policies for Chrome extensions running under the Chrome OS
  // signin profile. |component_id| must be equal to the extension ID.
  POLICY_DOMAIN_SIGNIN_EXTENSIONS = 2;

  // Domain for policies for Web Apps. |component_id| must be equal to 
  // Web App domain.
  POLICY_DOMAIN_WEB_APPS = 3;

  // Next ID to use: 4
};
```
The issue with this approach is that its benefits are only present with a strictly defined origin schema, which is unfeasible to enforce in the wild web.

### Extending WebAppInstallForceList
We could include the “External policy” logic into the existing policy which is used for force-installing Web Apps. Currently, the policy value is a JSON string defined like this:
```js
[
  {
    "url": "https://www.example.com/app1",
    "create_desktop_shortcut": true,
    "default_launch_container": "window"
  },
  {
    "url": "https://sub.myexample.com",
    "default_launch_container": "tab"
  }
]
```
By adding an additional “managed_configuration” field to the configuration for each individual app, we could provide a configuration url and its hash there. 
```js
[
  {
    "url": "https://www.example.com/app1",
    "create_desktop_shortcut": true,
    "default_launch_container": "window",
    "managed_configuration" : {
        "url" : "https://static.example.com/conf.txt",
        "hash" : "1j89ejasdo19f1j89ejasdo1061j89ejasdo10a"
    } 
  },
  {
    "url": "https://sub.myexample.com",
    "default_launch_container": "tab"
  }
]
```

We would track the updates to the hash value in order to invalidate current configuration. 

The problem with this approach, as opposed to the chosen one, is that:
1. The managed configuration is tightly bound with Web App installation logic, which is not 100% reliable and not instantaneous.
2. We cannot enforce the rule "one configuration per origin" using this policy schema. This may be a problem for other non-Google Chromium users.

## Security considerations
From a security perspective, the only unsafe thing we do is parsing unknown JSON files provided by the Admin panel. This happens inside of a sandboxed process on a user profile, which should be safe in terms of decoding.

## Privacy considerations
We do not upload any user data into the cloud. However, we do upload admin-preset data to some of the origins, which could lead to a website identifying a managed user among others. To ensure user privacy, we are exposing the list of such origins at the User Transparency Page.
