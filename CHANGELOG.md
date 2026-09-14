# Changelog

All notable changes to this project will be documented here.

## 2.1.0 - 2026-09-14

- Added mapping from each Unity video source to its parent physical camera or encoder.
- Suppressed child camera-disconnection problems while the mapped parent device is disconnected.
- Added `{$AVIGILON.DEVICE.MONITORING.ENABLED}` for camera-only hosts; disabling it also bypasses parent suppression so camera outages remain visible.
- Added `{$AVIGILON.CAMERA.PARENT.SUPPRESSION}` to independently control parent-device suppression.
- Added `{$AVIGILON.HEALTH.ENABLED}` and disabled intensive per-device health requests by default.
- Kept lightweight physical-device collection enabled for parent mapping and summary counts.

## 2.0.0 - 2026-09-14

- Added paginated collection for Unity API list resources.
- Added physical camera and encoder counts and low-level discovery.
- Added per-device connection, state, firmware, and IP-address items.
- Added a lower-frequency detailed device-health collector.
- Added device error-status, lost-network, and prolonged-error health metrics.
- Added camera diagnostic status and diagnostic-error triggers.
- Added expected video-source and physical-device count triggers.
- Added server count/availability and full Unity software build inventory.
- Preserved root trigger dependencies for camera and physical-device alert-storm suppression.

## 1.0.0 - 2026-09-14

- Initial public release.
- Added OAuth client-credentials authentication through Unity Integration Management.
- Added Web Endpoint, API, site, server, version, and camera-count monitoring.
- Added low-level camera discovery and disconnection alerts.
- Added dependency-based camera alert-storm suppression.
- Tracked disabled and unknown states separately from disconnected cameras.
