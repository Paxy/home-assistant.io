---
title: HTML5 Push Notifications
description: Instructions on how to use the HTML5 push notifications platform from Home Assistant.
ha_category:
  - Notifications
ha_release: 0.27
ha_iot_class: Cloud Push
ha_domain: html5
ha_platforms:
  - notify
ha_integration_type: integration
---

The `html5` notification platform enables you to receive push notifications to Chrome or Firefox, no matter where you are in the world. `html5` also supports Chrome and Firefox on Android, which enables native-app-like integrations without actually needing a native app.

<div class='note'>

HTML5 push notifications **do not** work on iOS.

</div>

## Configuration

To enable this platform, add the following lines to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
notify:
  - platform: html5
    vapid_pub_key: YOUR_PUBLIC_KEY
    vapid_prv_key: YOUR_PRIVATE_KEY
    vapid_email: YOUR_EMAIL
```

{% configuration %}
vapid_pub_key:
  description: The VAPID public key generated by Google (this is the key that is immediately visible under "webpush certificates"), [see configuring the platform](#configuring-the-platform).
  required: true
  type: string
vapid_prv_key:
  description: The VAPID private key generated by Google, [see configuring the platform](#configuring-the-platform).
  required: true
  type: string
vapid_email:
  description: The e-mail account of your Google account associated with your Firebase project, [see configuring the platform](#configuring-the-platform).
  required: true
  type: string
{% endconfiguration %}

### Requirements

The `html5` platform can only function if all of the following requirements are met:

* You are using Chrome and/or Firefox on any desktop platform, ChromeOS or Android.
* Your Home Assistant instance is accessible from outside your network over HTTPS or can perform an alternative [Domain Name Verification Method](https://support.google.com/webmasters/answer/9008080#domain_name_verification) on the domain used by Home Assistant.
* If using a proxy, HTTP basic authentication must be off for registering or unregistering for push notifications. It can be re-enabled afterwards.
* If you don't run Hass.io: `pywebpush` must be installed. `libffi-dev`, `libpython-dev` and `libssl-dev` must be installed prior to `pywebpush` (i.e., `pywebpush` probably won't automatically install).
* You have configured SSL/TLS for your Home Assistant. It doesn't need to be configured in Home Assistant though, e.g., you can be running NGINX in front of Home Assistant and this will still work. The certificate must be trustworthy (i.e., not self-signed).
* You are willing to accept the notification permission in your browser.

### Configuring the platform

1. Create a new project at [https://console.cloud.google.com/home/dashboard](https://console.cloud.google.com/home/dashboard), this project will be imported into Firebase later (alternatively, the project can also be created in the next step).
2. Go to [https://console.firebase.google.com](https://console.firebase.google.com), and click the "Add project" button
3. Choose your Google Cloud project for the name field (or create a new one). Decline analytics.
4. Then, click the cogwheel on top left and select "Project settings".
5. Select the "Cloud Messaging" tab.
6. Generate a new key pair in "Web configuration" at the bottom of the page. Add the public key as `vapid_pub_key` in the config, then choose the 3 dots, and copy the private key for `vapid_prv_key` in the config.
7. Input your Google email as `vapid_email` in the config.

### Setting up your browser

Assuming you have already configured the platform:

{% my profile badge %}

1. Open Home Assistant in Chrome or Firefox and load profile page by clicking the My button above or by clicking on the badge next to the Home Assistant title in the sidebar. Assuming you have met all the [requirements](#requirements) above then you should see a new slider for Push Notifications. If the slider is greyed out, ensure you are viewing Home Assistant via its external HTTPS address (and that you have configured the `notify` HTML5 integration in Home Assistant). If the slider is not visible, ensure you are not in the user configuration (Sidebar, Configuration, Users, View User).
2. Turn on the slider, and name the device you're using in the alert that appears.
3. Within a few seconds you should be prompted to allow notifications from Home Assistant.
4. Assuming you accept, that's all there is to it!

**Note:** If you aren't prompted for a device name when enabling notifications, open the `html5_push_registrations.conf` file in your configuration directory. You will see a new entry for the browser you just added. Rename it from `unnamed device` to a name of your choice, which will make it easier to identify later. _Do not change anything else in this file!_ You need to restart Home Assistant after making any changes to the file.

### Testing

Assuming the previous test completed successfully and your browser was registered, you can test the notification as follows:

{% my developer_services badge %}

1. Click on the My button above, or open Home Assistant in Chrome or Firefox, open the sidebar and click the Services button at the bottom (shaped like a remote control), located below the Developer Tools.
2. From the Services dropdown, search for your HTML5 notify service (notify.html5) and select it.
3. In the Service Data text box enter: `{"message":"hello world"}`, then press the CALL SERVICE button.
4. If everything worked you should see a popup notification.

### Usage

The `html5` platform accepts a standard notify payload. However, there are also some special features built in which you can control in the payload.


#### Actions

Chrome supports notification actions, which are configurable buttons that arrive with the notification and can cause actions on Home Assistant to happen when pressed. You can send up to 2 actions.

```yaml
message: Anne has arrived home
data:
  actions:
    - action: open
      icon: "/static/icons/favicon-192x192.png"
      title: Open Home Assistant
    - action: open_door
      title: Open door
```

#### Data

Any parameters that you pass in the notify payload that aren't valid for use in the HTML5 notification (`actions`, `badge`, `body`, `dir`, `icon`, `image`, `lang`, `renotify`, `requireInteraction`, `tag`, `timestamp`, `vibrate`, `priority`, `ttl`) will be sent back to you in the [callback events](#automating-notification-events).

```yaml
title: Front door
message: The front door is open
data:
  my-custom-parameter: front-door-open
```

#### Tag

By default, every notification sent has a randomly generated UUID (v4) set as its _tag_ or unique identifier. The tag is unique to the notification, _not_ to a specific target. If you pass your own tag in the notify payload you can replace the notification by sending another notification with the same tag. You can provide a `tag` like so:

```yaml
title: Front door
message: The front door is open
data:
  tag: front-door-notification
```

Example of adding a tag to your notification. This won't create new notification if there already exists one with the same tag.

{% raw %}

```yaml
  - alias: "Push/update notification of sensor state with tag"
    trigger:
      - platform: state
        entity_id: sensor.sensor
    action:
      service: notify.html5
      data:
        message: "Last known sensor state is {{ states('sensor.sensor') }}."
      data:
        data:
          tag: "notification-about-sensor"
```

{% endraw %}

#### Targets

If you do not provide a `target` parameter in the notify payload a notification will be sent to all registered targets as listed in `html5_push_registrations.conf`. You can provide a `target` parameter like so:

```yaml
title: Front door
message: The front door is open
target: unnamed device
```

`target` can also be a string array of targets like so:

```yaml
title: Front door
message: The front door is open
target:
  - unnamed device
  - unnamed device 2
```

#### Overrides

You can pass any of the parameters listed [here](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration/showNotification#Parameters) in the `data` dictionary. Please note, Chrome specifies that the maximum size for an icon is 320px by 320px, the maximum `badge` size is 96px by 96px and the maximum icon size for an action button is 128px by 128px.

#### URL

You can provide a URL to open when the notification is clicked by putting `url` in the data dictionary like so:

```yaml
title: Front door
message: The front door is open
data:
  url: https://google.com
```

If no URL or actions are provided, interacting with a notification will open your Home Assistant in the browser. You can use relative URLs to refer to Home Assistant, i.e., `/map` would turn into `https://192.168.1.2:8123/map`.

#### TTL and Priority

Newer Android versions introduced stronger battery optimization, so notifications by default are delivered only when phone is awake.
Options TTL and priority tries to help users solve those problems. Default value of TTL is `86400s` and priority is `normal`.
You can set priority to either `normal` or `high`. TTL is any integer value.

```yaml
title: Front door
message: The front door is open
data:
  ttl: 86400
  priority: high
```

### Dismiss

You can dismiss notifications by using service html5.dismiss like so:

```yaml
target: ['my phone']
data:
  tag: notification_tag
```

If no target is provided, it dismisses for all.
If no tag is provided, it dismisses all notifications.

### Automating notification events

During the lifespan of a single push notification, Home Assistant will emit a few different events to the event bus which you can use to write automations against.

Common event payload parameters are:

| Parameter | Description                                                                                                                                                                                                                                                    |
|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `action`  | The `action` key that you set when sending the notification of the action clicked. Only appears in the `clicked` event.                                                                                                                                        |
| `data`    | The data dictionary you originally passed in the notify payload, minus any parameters that were added to the HTML5 notification (`actions`, `badge`, `body`, `dir`, `icon`, `image`, `lang`, `renotify`, `requireInteraction`, `tag`, `timestamp`, `vibrate`). |
| `tag`     | The unique identifier of the notification. Can be overridden when sending a notification to allow for replacing existing notifications.                                                                                                                        |
| `target`  | The target that this notification callback describes.                                                                                                                                                                                                          |
| `type`    | The type of event callback received. Can be `received`, `clicked` or `closed`.                                                                                                                                                                                 |

You can use the `target` parameter to write automations against a single `target`. For more granularity, use `action` and `target` together to write automations which will do specific things based on what target clicked an action.

#### received event

You will receive an event named `html5_notification.received` when the
notification is received on the device.

```yaml
- alias: "HTML5 push notification received and displayed on device"
  trigger:
    platform: event
    event_type: html5_notification.received
```

#### clicked event

You will receive an event named `html5_notification.clicked` when the notification or a notification action button is clicked. The action button clicked is available as `action` in the `event_data`.

```yaml
- alias: "HTML5 push notification clicked"
  trigger:
    platform: event
    event_type: html5_notification.clicked
```

or

```yaml
- alias: "HTML5 push notification action button clicked"
  trigger:
    platform: event
    event_type: html5_notification.clicked
    event_data:
      action: open_door
```

#### closed event

You will receive an event named `html5_notification.closed` when the notification is closed.

```yaml
- alias: "HTML5 push notification clicked"
  trigger:
    platform: event
    event_type: html5_notification.closed
```

### Making notifications work with NGINX proxy

If you use NGINX as a proxy with authentication in front of your Home Assistant instance, you may have trouble with receiving events back to Home Assistant. It's because of an authentication token that cannot be passed through the proxy.

To solve the issue put additional location into your NGINX site's configuration:

```bash
location /api/notify.html5/callback {
    if ($http_authorization = "") { return 403; }
    allow all;
    proxy_pass http://localhost:8123;
    proxy_set_header Host $host;
    proxy_redirect http:// https://;
}
```

This rule check if request have `Authorization` HTTP header and bypass the htpasswd (if you use one).

If you still have the problem, even with mentioned rule, try to add this code:

```bash
    proxy_set_header Authorization $http_authorization;
    proxy_pass_header Authorization;
```
