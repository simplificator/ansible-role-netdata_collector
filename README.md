# Ansible Role: Netdata Collector

[![CI](https://github.com/simplificator/ansible-role-netdata_collector/workflows/CI/badge.svg?event=push)](https://github.com/simplificator/ansible-role-netdata_collector/actions?query=workflow%3ACI)

This role configures an existing Netdata base installation as a server which can collect data from multiple streaming nodes.

Its companion role is the [Netdata node](https://github.com/simplificator/ansible-role-netdata_node).

## Requirements

An empty Netdata installation, see dependencies.

## Role Variables

The setup in general is highly opiniated.

* Caddy as a reverse proxy will be installed.
* All routes to the dashboard are protected with basic authentication.
* We differ between two types of streaming node: `silence` will deactivate all alarms but still collect data. `default` will activate all alarms.
* Historic data will be stored within Netdatas own database mechanism.

This setup is reflected within the variables:

* `netdata_server_page_cache_size`: The page cache size option determines the amount of RAM in MiB dedicated to caching Netdata metric values. The actual page cache size will be slightly larger than this figure — see the [memory requirements section](https://learn.netdata.cloud/docs/agent/database/engine/#memory-requirements) for details.
* `netdata_db_multihost_disk_space`: The dbengine multihost disk space option determines the amount of disk space in MiB that is dedicated to storing Netdata metric values and all related metadata describing them. You can use the [database engine calculator](https://learn.netdata.cloud/docs/store/change-metrics-storage#calculate-the-system-resources-ram-disk-space-needed-to-store-metrics) to correctly set dbengine multihost disk space based on your metrics retention policy. The calculator gives an accurate estimate based on how many child nodes you have, how many metrics your agent collects, and more.
* `netdata_registry_to_announce`: The central registry where this client should be registered. More information is available [here](https://learn.netdata.cloud/docs/agent/registry).
* `netdata_registry_domain`: Will be used for the registry cookie, so you can request different registries but all nodes will be available to browse from the dashboard.
* `netdata_server_basic_auth_user`: Name for the basic authentication user name.
* `netdata_server_basic_auth_password`: Password for the basic authentication of the reverse proxy. The password needs to be in Caddys hashed base64 format (more about that [here](https://caddyserver.com/docs/caddyfile/directives/basicauth#basicauth)).
* `netdata_stream_default_api_key`: API key to identify incoming stream traffic. Can be generated via `uuidgen`.
* `netdata_stream_silence_api_key`: Same thing as `netdata_stream_default_api_key`, but alarms for traffic with this UUID won't be sent to any notification system.
  
Optional variables:

* `netdata_hostname`: Allows you to overwrite the hostname of this Netdata host. Default value is "auto-detected". Note that a change of this variable could result in a drop of historic data.
* `netdata_server_domain`:  Domain where the reverse proxy should listen to. If no value provided, it'll be set to the value of `netdata_registry_to_announce`.

Additionally, we expect that a certificate is placed at `/etc/netdata/ssl/cert.pem`. It'll be used to encrypt the traffic between node and collector. More information can be found in the [netdata installation role](https://github.com/simplificator/ansible-role-netdata_installation).

## Dependencies

* [simplificator.netdata_installation](https://github.com/simplificator/ansible-role-netdata_installation)

## License

MIT / BSD
