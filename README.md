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
- Trigger dependencies that suppress per-camera alerts during server, API, or site outages
- Optional Windows service monitoring through Zabbix Agent or Agent 2

Unity's `Disabled` state is counted separately and does not generate a disconnection alert.

## Compatibility

- Zabbix 7.0 export format
- Avigilon Unity Video 8
- Validated against Unity Video 8.8.0.22 and Web Endpoint build 36.0.2

Newer Zabbix 7.x releases should accept the 7.0 export format. Zabbix 6.x may require conversion or manual recreation.

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

Additional macros control polling, trigger delays, and the optional Windows service check.

## Alert-storm prevention

Each discovered camera trigger depends on three root conditions:

1. Unity server/Web Endpoint availability
2. API authentication and collection health
3. Site reachability

When one of those root conditions fails, Zabbix suppresses the dependent camera problems. This prevents a server outage from generating a separate notification for every camera.

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
- Cameras with unknown state is `0`.

Perform a controlled camera disconnect and confirm that only its camera trigger opens after the configured grace period. Also test a server/API outage to verify that the dependency chain suppresses the individual camera notifications.

## Security

Never place a real client ID or secret in the template, an issue, or a diagnostic log. Configure credentials as host-level macros and store the client secret as secret text.

See [SECURITY.md](SECURITY.md) for vulnerability reporting guidance.

## Contributing

Bug reports, compatibility results, and improvements are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License and trademarks

Released under the [MIT License](LICENSE).

Avigilon and Unity are trademarks of their respective owners. This community project is not affiliated with or endorsed by Avigilon or Motorola Solutions.
