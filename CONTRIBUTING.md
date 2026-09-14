# Contributing

Thank you for helping improve this Zabbix template.

## Useful reports

Issues are especially helpful when they include:

- Zabbix version
- Avigilon Unity Video and Web Endpoint versions
- Whether the Zabbix server or a proxy executes the template
- The affected item or trigger name
- Sanitized API field names and status values
- Import or preprocessing errors with credentials, hostnames, IP addresses, camera names, and IDs removed

Never post client IDs, client secrets, bearer tokens, internal addresses, camera names, or other sensitive deployment information.

## Proposed changes

Keep the template importable by Zabbix 7.0 unless a major version change is clearly documented. Preserve the root trigger dependencies on all per-camera availability triggers.

Before submitting a pull request:

1. Parse the YAML successfully.
2. Import it into a test Zabbix instance when possible.
3. Confirm that no live credentials or deployment-specific information are present.
4. Explain which Unity and Zabbix versions were tested.

Please keep changes focused and update the README when configuration or monitored behavior changes.
