# Zabbix template for Avigilon Unity Video

An importable Zabbix template for monitoring Avigilon Unity Video 8 systems through the Unity API Gateway and Web Endpoint.

The template uses Unity 8 Integration Management client credentials. It does not require or store an interactive Unity username and password.

## Features

- Web Endpoint reachability and service-health monitoring
- Unity API Gateway OAuth authentication monitoring
- Site and server availability
- ACC and Web Endpoint version inventory
- Total, online, disconnected, disabled, and unknown video-source counts
- Low-level discovery of video sources
- Individual camera-disconnection triggers with a configurable grace period
- Parent-device-aware camera alert suppression
- Camera status and diagnostic-error monitoring
- Physical camera and encoder counts, inventory, and low-level discovery
- Optional per-device connection, firmware, IP address, and health monitoring
- Device health flags for error status, lost network, and prolonged errors
- Expected camera/device count checks to detect removed resources
- Server software display and full build versions
- Paginated API collection for larger deployments
- Trigger dependencies that suppress per-camera alerts during server, API, or site outages
- Optional Windows service monitoring through Zabbix Agent or Agent 2

Unity's `Disabled` state is counted separately and does not generate a disconnection alert.

## Compatibility

- Zabbix 7.0 export format
- Avigilon Unity Video 8
- Validated against Unity Video 8.8.0.22 and Web Endpoint build 36.0.2

Newer Zabbix 7.x releases should accept the 7.0 export format. Zabbix 6.x may require conversion or manual recreation.

## Upgrading from version 1

Import the new YAML over the existing template and select the normal update-existing option. The template, original items, discovery rule, and trigger UUIDs are preserved, so existing hosts remain linked and retain their history. Version 2 adds new items and a second discovery rule.

## Installation

1. Download [`zabbix_avigilon_unity_video_7.0.yaml`](zabbix_avigilon_unity_video_7.0.yaml).
2. In Zabbix, open **Data collection → Templates → Import** and import the file.
3. Create or select a host representing the Unity server.
4. Configure the host interface DNS name or IP address so `{HOST.CONN}` resolves to the Unity server.
5. Link the **Avigilon Unity Video by HTTP** template.
6. Set the required host macros described below.

## Unity Integration Management setup

Create an integration in Unity Video and associate it with a user group that can see the monitored site and cameras. Use a dedicated least-privilege group in production.

The template uses the OAuth 2.0 client-credentials flow at:

```text
https://HOST:38880/connect/token
```

It requests scope `tq:api` and reads the Unity API Gateway resources under `/tq/v2`.

## Required host macros

| Macro | Default | Description |
|---|---:|---|
| `{$AVIGILON.CLIENT.ID}` | `CHANGE_ME` | Client ID from Unity Integration Management |
| `{$AVIGILON.CLIENT.SECRET}` | `CHANGE_ME` | Client secret; stored as Zabbix secret text |
| `{$AVIGILON.API.PORT}` | `38880` | Unity API Gateway HTTPS port |
| `{$AVIGILON.WEB.PORT}` | `8443` | Unity Web Endpoint HTTPS port |
| `{$AVIGILON.EXPECTED.CAMERA.COUNT}` | `0` | Minimum expected video-source count; zero disables the trigger |
| `{$AVIGILON.EXPECTED.DEVICE.COUNT}` | `0` | Minimum expected physical-device count; zero disables the trigger |
| `{$AVIGILON.CAMERA.OFFLINE.TIME}` | `2m` | Camera disconnection grace period |
| `{$AVIGILON.CAMERA.PARENT.SUPPRESSION}` | `1` | Suppress camera alerts while the mapped parent device is disconnected |
| `{$AVIGILON.DEVICE.MONITORING.ENABLED}` | `1` | Discover and alert on physical devices; set to `0` for camera-only monitoring |
| `{$AVIGILON.DEVICE.OFFLINE.TIME}` | `2m` | Physical-device disconnection grace period |
| `{$AVIGILON.HEALTH.ENABLED}` | `0` | Enable intensive per-device health requests; disabled by default |
| `{$AVIGILON.HEALTH.INTERVAL}` | `10m` | Detailed physical-device health interval |
| `{$AVIGILON.HEALTH.MAX.DEVICES}` | `50` | Devices checked per detailed-health poll; zero means all |
| `{$AVIGILON.HEALTH.TIMEOUT}` | `60s` | Detailed-health collector timeout |

Additional macros control polling, trigger delays, and the optional Windows service check.

## Alert-storm prevention

Each discovered camera and physical-device trigger depends on the applicable root conditions:

1. Unity server/Web Endpoint availability
2. API authentication and collection health
3. Site reachability

When one of those root conditions fails, Zabbix suppresses the dependent camera problems. This prevents a server outage from generating a separate notification for every camera.

Camera disconnection triggers also inspect the lightweight connection state of their mapped parent physical device. If the device is disconnected, its child video-source problems remain closed and the physical-device problem is the single alert. Multi-channel cameras and encoders therefore do not create one notification per channel during a device outage.

Set `{$AVIGILON.DEVICE.MONITORING.ENABLED}` to `0` for camera-only monitoring. This excludes physical devices from low-level discovery and automatically bypasses parent-device suppression, ensuring camera outages are still reported. The lightweight `/Devices` list remains part of the main collection because it supplies camera-to-device mapping and summary counts; it is one paginated collection request, not one request per device.

## Video sources versus physical devices

Unity reports video data sources separately from physical cameras and encoders. A multi-sensor camera or encoder can provide several video sources, so the two totals are not expected to match.

Version 2 discovers both layers:

- Video-source items monitor the status and diagnostic result of every view/channel.
- Physical-device items monitor the camera or encoder connection, firmware, IP address, and detailed device health.

When physical-device monitoring and parent suppression are enabled, a device outage creates the physical-device problem while its child video-source problems remain suppressed. If a video source disconnects while its parent device remains connected, the camera problem opens normally. An unknown or unmapped parent never suppresses a camera problem.

## Detailed health polling

Unity exposes device health through one resource per physical device. For that reason, detailed health is disabled by default. Set `{$AVIGILON.HEALTH.ENABLED}` to `1` on a host to enable it. The lower-frequency collector then performs one health request for each checked device. It is separate from the one-minute availability poll and checks at most 50 devices by default.

With detailed health disabled, the collector reports state `2` (disabled/not checked) and makes no per-device `/Health` requests. It still performs a low-frequency authentication and device-list request so existing dependent items remain supported. Detailed health also remains inactive when `{$AVIGILON.DEVICE.MONITORING.ENABLED}` is `0`.

If the site contains more than the configured limit, the template records the skipped count and opens an informational capacity event. Increase the limit carefully, reduce the health polling frequency, or leave the remaining devices unchecked. A value of zero removes the limit but can exceed Zabbix's script timeout on large sites.

## Network and TLS requirements

Allow the Zabbix server or proxy to reach TCP ports 38880 and 8443 on each monitored Unity server.

Unity deployments commonly use locally issued certificates. Restrict the monitoring path appropriately and install trusted certificates where possible.

## First-run checks

After linking the template, confirm in **Latest data** that:

- Server available is `1`.
- Web Endpoint health is good is `1`.
- API collection successful is `1`.
- Site reachable is `1`.
- Camera totals match Unity Video.
- Physical-device totals match Unity Video.
- Cameras with unknown state is `0`.
- Device health collection successful is `2` when detailed health is disabled, or `1` after it is enabled.
- Devices skipped by detailed-health limit is `0`, unless detailed health is enabled and intentionally limited.

Perform a controlled camera disconnect and confirm that only its camera trigger opens after the configured grace period. Then disconnect a physical device and confirm that its device problem opens without child camera problems. Also test a server/API outage to verify that the root dependency chain suppresses individual notifications.

## Security

Never place a real client ID or secret in the template, an issue, or a diagnostic log. Configure credentials as host-level macros and store the client secret as secret text.

See [SECURITY.md](SECURITY.md) for vulnerability reporting guidance.

## Contributing

Bug reports, compatibility results, and improvements are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License and trademarks

Released under the [MIT License](LICENSE).

Avigilon and Unity are trademarks of their respective owners. This community project is not affiliated with or endorsed by Avigilon or Motorola Solutions.
