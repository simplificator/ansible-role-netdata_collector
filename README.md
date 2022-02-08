# Ansible Role: Netdata Collector

[![CI](https://github.com/simplificator/ansible-role-netdata_collector/workflows/CI/badge.svg?event=push)](https://github.com/simplificator/ansible-role-netdata_collector/actions?query=workflow%3ACI)

This role configures an existing Netdata base installation as a server which can collect data from multiple streaming nodes.

Its companion role is the [Netdata node](https://github.com/simplificator/ansible-role-netdata_node).

## Requirements

An empty Netdata installation, see dependencies.

## Role Variables

The setup is highly opinionated.

* Caddy as a reverse proxy will be installed.
* All routes to the dashboard are protected with basic authentication.
* The stream port is exposed to port 19999 and requires certificate-based authentication.
* We differ between two types of streaming nodes: `silence` will deactivate all alarms but still collect data. `default` will activate all alarms.
* Historic data will be stored within Netdatas own database mechanism.
* We deactivated several TCP/UDP-related metrics

This setup is reflected within the variables:

* `netdata_collector_basic_auth_user`: Name for the basic authentication user name.
* `netdata_collector_basic_auth_password`: Password for the basic authentication of the reverse proxy. The password needs to be in Caddys hashed base64 format (more about that [here](https://caddyserver.com/docs/caddyfile/directives/basicauth#basicauth)).
* `netdata_collector_registry_domain`: Will be used for the registry cookie, so you can request different registries but all nodes will be available to browse from the dashboard.
* `netdata_registry_to_announce`: The central registry where this client should be registered. More information is available [here](https://learn.netdata.cloud/docs/agent/registry).
* `netdata_collector_default_stream_key`: API key to identify incoming stream traffic. Can be generated via `uuidgen`.
* `netdata_collector_silent_stream_key`: Same thing as `netdata_collector_default_stream_key`, but alarms for traffic with this UUID won't be sent to any notification system.
  
Optional variables:

* `netdata_collector_page_cache_size`: The page cache size option determines the amount of RAM in MiB dedicated to caching Netdata metric values. The actual page cache size will be slightly larger than this figure — see the [memory requirements section](https://learn.netdata.cloud/docs/agent/database/engine/#memory-requirements) for details.
* `netdata_collector_domain`:  Domain where the reverse proxy should listen to. If no value provided, it'll be set to the value of `netdata_registry_to_announce`.
* `netdata_db_multihost_disk_space`: The dbengine multihost disk space option determines the amount of disk space in MiB that is dedicated to storing Netdata metric values and all related metadata describing them. You can use the [database engine calculator](https://learn.netdata.cloud/docs/store/change-metrics-storage#calculate-the-system-resources-ram-disk-space-needed-to-store-metrics) to correctly set dbengine multihost disk space based on your metrics retention policy. The calculator gives an accurate estimate based on how many child nodes you have, how many metrics your agent collects, and more.
* `netdata_hostname`: Allows you to overwrite the hostname of this Netdata host. Default value is "auto-detected". Note that a change of this variable could result in a drop of historic data.

Additionally, we expect that a certificate is placed at `/etc/netdata/ssl/cert.pem`. It'll be used to encrypt the traffic between node and collector. More information can be found in the [netdata installation role](https://github.com/simplificator/ansible-role-netdata_installation).

### Notifications

The role currently supports Microsoft Teams and Slack as "recipients" for notifications.

For Slack, you need the following values:

* `netdata_collector_custom_slack_sender`: If set to "yes", this will activate a custom slack notifier which makes smaller messages.
* `netdata_alarms_slack_channel`: The channel where netdata should sent its messages.
* `netdata_collector_slack_webhook`: Webhook to send slack messages.

For Teams, set the following ones:

* `netdata_collector_teams_recipients`: The channel where netdata should sent its messages.
* `netdata_collector_teams_webhook`: Webhook URL where Netdata can send its notifications.

## Dependencies

* [simplificator.caddy](https://github.com/simplificator/ansible-role-caddy)
* [simplificator.netdata_installation](https://github.com/simplificator/ansible-role-netdata_installation)

## Example playbook

```yaml
---
- name: Converge
  hosts: all
  become: true

  roles:
    - role: simplificator.netdata_collector

  vars:
    netdata_collector_basic_auth_user: admin
    netdata_collector_basic_auth_password: JDJhJDE0JE9FYmltMG9LbVJGNVRld3hWRHMvek9Mb3FhNno5T05hYjFDYllPcjVPOFJrTEtScFBmN1Fl # admin123$
    netdata_collector_default_stream_key: 267E1130-EDDA-4574-82FA-EF17286B0816
    netdata_collector_registry_domain: example.com
    netdata_collector_silent_stream_key: 6840CAB7-AE62-49C3-B5D9-C4323BBAAF94
    netdata_server_certificate: certificate
    netdata_server_certificate_key: key
    netdata_registry_to_announce: "https://registry.example.com"
```

## License

MIT / BSD
