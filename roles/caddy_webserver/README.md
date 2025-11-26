# Caddy Webserver Role

Ansible role for deploying Hugo static sites with a custom-built Caddy web server.

## Features

- Builds Caddy from source with custom modules
- Installs Hugo static site generator
- Configures automated SSL/TLS with Let's Encrypt
- Supports Cloudflare DNS-01 ACME challenge
- Built-in rate limiting protection
- Automated Git repository deployment
- SystemD timer for automatic rebuilds

## Requirements

- Rocky Linux 10 or RHEL 9+ family distributions
- SSH access with sudo privileges
- Git repository with Hugo site

## Role Variables

See [main README](../../README.md#configuration-options) for all available variables.

### Key Variables

- `domain`: Your website domain
- `website_repo_url`: Git repository SSH URL
- `hugo_version`: Hugo version to install
- `caddy_version`: Caddy version to build
- `caddy_modules`: List of Caddy modules to compile
- `cloudflare_api_token`: Cloudflare API token for DNS-01 challenge

## Dependencies

None.

## Example Playbook

```yaml
- hosts: webservers
  become: true
  roles:
    - role: danylomikula.ansible_hugo_deploy.caddy_webserver
      vars:
        domain: "example.com"
        website_repo_url: "git@github.com:user/site.git"
        hugo_version: "0.152.2"
        caddy_version: "2.10.2"
```

## Authors

Module managed by [Danylo Mikula](https://github.com/danylomikula).

## License

Apache 2.0
