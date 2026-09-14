# Security policy

## Reporting a vulnerability

Please do not disclose vulnerabilities or exposed credentials in a public issue.

Use GitHub's private vulnerability reporting feature for this repository. Include enough sanitized detail to reproduce the issue without including real Unity credentials, bearer tokens, internal addresses, camera names, or video-system topology.

## Credential handling

The template expects a Unity Integration Management client ID and secret as host-level Zabbix macros. The secret macro uses Zabbix's secret-text type.

If credentials are accidentally committed, posted in an issue, or written to a log, revoke and rotate the client secret immediately. Removing a secret from the latest commit does not remove it from Git history.
