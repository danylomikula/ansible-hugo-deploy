# Firewall Role

Ansible role for configuring firewalld on Rocky Linux systems.

## Features

- Configures firewalld zones
- Manages allowed services (SSH, HTTP, HTTPS)
- Supports custom port rules
- ICMP (ping) configuration
- Automatic firewall reload on changes

## Requirements

- Rocky Linux 10 or RHEL 9+ family distributions
- firewalld package (installed by role)

## Role Variables

See [main README](../../README.md#configuration-options) for all available variables.

### Key Variables

- `firewall_zone`: Firewall zone to configure (default: `public`)
- `firewall_allowed_services`: List of services to allow (default: `[ssh, http, https]`)
- `firewall_allowed_ports`: Custom ports to allow (default: `[]`)
- `firewall_allowed_icmp`: Enable ICMP traffic (default: `true`)

## Dependencies

None.

## Example Playbook

```yaml
- hosts: webservers
  become: true
  roles:
    - role: danylomikula.ansible_hugo_deploy.firewall
      vars:
        firewall_zone: "public"
        firewall_allowed_services:
          - ssh
          - http
          - https
        firewall_allowed_ports:
          - { port: 2222, proto: tcp, comment: "Custom SSH" }
```

## Authors

Module managed by [Danylo Mikula](https://github.com/danylomikula).

## License

Apache 2.0
