# 🚀 Ansible Hugo Deploy

[![CI](https://img.shields.io/github/actions/workflow/status/danylomikula/ansible-hugo-deploy/pr-validation.yml?label=CI)](https://github.com/danylomikula/ansible-hugo-deploy/actions/workflows/pr-validation.yml)
[![Release](https://img.shields.io/github/v/release/danylomikula/ansible-hugo-deploy)](https://github.com/danylomikula/ansible-hugo-deploy/releases)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-danylomikula.ansible__hugo__deploy-blue.svg)](https://galaxy.ansible.com/ui/repo/published/danylomikula/ansible_hugo_deploy/)
[![License](https://img.shields.io/badge/license-Apache--2.0-green.svg)](LICENSE)
[![Ansible](https://img.shields.io/badge/ansible-2.14+-blue.svg)](https://www.ansible.com/)
[![Rocky Linux](https://img.shields.io/badge/rocky%20linux-10-green.svg)](https://rockylinux.org/)

Ansible collection for automated Hugo static site deployment with custom Caddy web server, Cloudflare DNS integration, rate limiting, and automated SSL on Rocky Linux 10.

## ✨ Features

- 🌐 **Hugo Static Site Deployment** - Automated cloning and building of Hugo websites
- ⚡ **Custom Caddy Build** - Compiles Caddy with custom plugins from source
- 🔒 **SSL/TLS Automation** - Automatic HTTPS certificates via Let's Encrypt with Cloudflare DNS challenge
- 🛡️ **Built-in Rate Limiting** - Protection against bots and abuse using [caddy-ratelimit](https://github.com/mholt/caddy-ratelimit)
- ☁️ **Cloudflare Integration** - DNS-01 ACME challenge support via [caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare)
- 🔐 **GitHub Deploy Key Generation** - Automatic SSH key generation for secure repository access
- ⏰ **Automated Updates** - SystemD timer for periodic Git pulls and site rebuilds
- 🔥 **Firewall Configuration** - Automated firewalld setup with sensible defaults
- 📦 **Version Pinning** - Full control over Hugo, Caddy, and Go versions

## 📋 Requirements

- **Control Node** (your local machine):
  - Ansible 2.14+
  - Python 3.8+

- **Managed Node** (target server):
  - Rocky Linux 10 (tested and verified)
  - RHEL 9+ family distributions should work
  - SSH access with sudo privileges

## 🔧 Quick Start

### Option A: Using from Ansible Galaxy (Recommended)

Install the collection directly from Ansible Galaxy:

```bash
# Install the collection
ansible-galaxy collection install danylomikula.ansible_hugo_deploy

# Create inventory file
cat > inventory.ini <<'EOF'
[webservers]
your-server ansible_host=your.server.ip.address ansible_user=root
EOF

# Create a playbook that uses the role
cat > deploy.yml <<'EOF'
---
- name: Deploy Hugo website
  hosts: webservers
  become: true
  vars:
    domain: "example.com"
    website_repo_url: "git@github.com:username/your-hugo-site.git"
    website_repo_branch: "main"
    hugo_version: "0.152.2"
    caddy_version: "2.10.2"
    cloudflare_api_token: "{{ vault_cloudflare_api_token }}"
  roles:
    - role: danylomikula.ansible_hugo_deploy.caddy_webserver
    - role: danylomikula.ansible_hugo_deploy.firewall
EOF

# Create vault file for secrets
ansible-vault create vault.yml
# Add: vault_cloudflare_api_token: "your-token-here"

# Run the playbook
ansible-playbook -i inventory.ini deploy.yml --ask-vault-pass -e @vault.yml
```

You can also use group_vars or host_vars for better organization. See [Configuration Options](#configuration-options) for all available variables.

### Option B: Using this Repository

Clone this repository and use the included playbook:

### 1. Install Ansible Dependencies

```bash
ansible-galaxy collection install -r requirements.yml
```

### 2. Configure Inventory

Edit `inventory/hosts.ini` with your server details:

```ini
[webservers]
your-server ansible_host=your.server.ip.address ansible_ssh_private_key_file=~/.ssh/your_ssh_key
```

**Note:** By default, `ansible.cfg` is configured to use `remote_user = ansible`. If you need to use a different user (e.g., `root`), either:
- Add `ansible_user=root` to your inventory line above, or
- Change `remote_user` in `ansible.cfg`

### 3. Configure Variables

Edit `group_vars/webservers/vars.yml`:

```yaml
# Set your domain
domain: "example.com"

# Configure your Hugo site repository
website_repo_url: "git@github.com:username/your-hugo-site.git"
website_repo_branch: "main"

# Adjust Hugo and Caddy versions as needed
hugo_version: "0.152.2"
caddy_version: "2.10.2"
```

### 4. Set Up Ansible Vault for Secrets

This playbook uses Ansible Vault to securely store sensitive information like your Cloudflare API token.

⚠️ **Note:** The `vault.yml` file is not included in this repository. You must create it yourself.

#### Create the vault file:

A template is provided at `group_vars/webservers/vault.yml.example`. Create your vault file:

```bash
# Create an encrypted vault file (you'll be prompted for a password)
ansible-vault create group_vars/webservers/vault.yml
```

You'll be prompted to enter a vault password. **Remember this password** - you'll need it every time you run the playbook!

#### Add your secrets to the vault:

When the editor opens, add the following content (also available in `vault.yml.example`):

```yaml
---
# Cloudflare API token for DNS-01 ACME challenge
# Get this from: https://dash.cloudflare.com/profile/api-tokens
# Required permissions: Zone:DNS:Edit, Zone:Zone:Read
vault_cloudflare_api_token: "your-actual-cloudflare-api-token-here"
```

Save and exit the editor (`:wq` in vim, `Ctrl+X` then `Y` in nano).

#### Managing the vault file:

```bash
# Edit the vault file later
ansible-vault edit group_vars/webservers/vault.yml

# View the vault file contents
ansible-vault view group_vars/webservers/vault.yml

# Change the vault password
ansible-vault rekey group_vars/webservers/vault.yml

# Decrypt the vault file (not recommended for production)
ansible-vault decrypt group_vars/webservers/vault.yml
```

#### Alternative: Use a password file

For automation, you can store the vault password in a file:

```bash
# Create a password file (make sure it's in .gitignore!)
echo "your-vault-password" > ~/.ansible_vault_pass
chmod 600 ~/.ansible_vault_pass

# Uncomment the vault_password_file line in ansible.cfg to use it automatically
# The option is already present in ansible.cfg, just uncomment it
```

⚠️ **Important:** Never commit the vault password file to git!

#### How to get a Cloudflare API token:

1. Log in to your [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Go to **My Profile** → **API Tokens**
3. Click **Create Token**
4. Use the **Edit zone DNS** template or create a custom token with these permissions:
   - **Zone** → **DNS** → **Edit**
   - **Zone** → **Zone** → **Read**
5. Set **Zone Resources** to include your specific domain
6. Copy the generated token and add it to your `vault.yml`

#### If you don't need Cloudflare DNS:

If you're using HTTP-01 ACME challenge instead of DNS-01, you can skip the vault setup and update `vars.yml`:

```yaml
# Remove the Cloudflare DNS module from the list
caddy_modules:
  - github.com/mholt/caddy-ratelimit
  # - github.com/caddy-dns/cloudflare  # Comment out or remove this line

# Set cloudflare_api_token to empty string
cloudflare_api_token: ""
```

### 5. Run the Playbook

```bash
# Run with vault password prompt
ansible-playbook site.yml --ask-vault-pass

# Or use a vault password file
ansible-playbook site.yml --vault-password-file ~/.ansible_vault_pass

# Run specific tags only
ansible-playbook site.yml --ask-vault-pass --tags caddy,firewall

# Check mode (dry run) - see what would change
ansible-playbook site.yml --ask-vault-pass --check

# Verbose mode for debugging
ansible-playbook site.yml --ask-vault-pass -vvv
```

### 6. Add Deploy Key to GitHub

During the first run, the playbook will:
1. Generate an SSH deploy key
2. Display the public key
3. Save it to `deploy_key.pub` in the project directory
4. Pause and wait for you to add it to GitHub

**To add the deploy key:**
1. Go to your GitHub repository → Settings → Deploy keys
2. Click "Add deploy key"
3. Paste the contents of `deploy_key.pub`
4. Click "Add key"
5. Return to your terminal and press Enter to continue

**Note:** The deploy key prompt will only appear on the first run. After the repository is successfully cloned, subsequent runs will skip this step automatically. To disable the prompt entirely, add this to `group_vars/webservers/vars.yml`:

```yaml
wait_for_deploy_key_confirmation: false
```

## 🎯 Playbook Structure

```
.
├── ansible.cfg                      # Ansible configuration
├── galaxy.yml                       # Ansible Galaxy metadata
├── site.yml                         # Main playbook
├── requirements.yml                 # Ansible collection dependencies
├── inventory/
│   └── hosts.ini                    # Server inventory
├── group_vars/
│   └── webservers/
│       ├── vars.yml                 # Main configuration variables
│       └── vault.yml.example        # Vault template (create vault.yml from this)
├── meta/
│   └── runtime.yml                  # Collection runtime requirements
└── roles/
    ├── caddy_webserver/             # Caddy and Hugo setup
    │   ├── tasks/
    │   │   └── main.yml             # Main tasks
    │   ├── templates/
    │   │   ├── Caddyfile.j2         # Caddy configuration
    │   │   ├── caddy-env.conf.j2    # Cloudflare credentials
    │   │   ├── webrebuild.service.j2 # SystemD service
    │   │   └── webrebuild.timer.j2  # SystemD timer
    │   ├── defaults/
    │   │   └── main.yml             # Default variables
    │   ├── handlers/
    │   │   └── main.yml             # Service handlers
    │   └── meta/
    │       └── main.yml             # Role metadata
    └── firewall/                    # Firewall configuration
        ├── tasks/
        │   └── main.yml             # Firewall tasks
        ├── handlers/
        │   └── main.yml             # Firewall handlers
        └── meta/
            └── main.yml             # Role metadata
```

<a id="configuration-options"></a>
## ⚙️ Configuration Options

### Core Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `domain` | Your website domain | `example.com` |
| `admin_email` | Email for Let's Encrypt | `admin@{{ domain }}` |
| `website_repo_url` | Git repository SSH URL | `git@github.com:username/repo.git` |
| `website_repo_branch` | Branch to deploy | `main` |

### Hugo Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `hugo_version` | Hugo version to install | `0.152.2` |
| `website_root` | Website clone location | `/var/www/{{ domain }}` |
| `website_public_dir` | Hugo build output directory | `{{ website_root }}/public` |

### Caddy Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `caddy_version` | Caddy version to build | `2.10.2` |
| `caddy_go_version` | Go version for building | `1.25.4` |
| `caddy_modules` | Custom Caddy modules | See below |
| `caddy_acme_ca` | ACME CA URL | Let's Encrypt production |
| `caddy_log_path` | Directory for Caddy logs | `/var/log/caddy` |
| `caddy_compression_formats` | Compression formats | `[gzip, zstd]` |
| `cloudflare_api_token` | Cloudflare API token for DNS-01 | `{{ vault_cloudflare_api_token }}` |

### Caddy Modules

| Variable | Description | Default |
|----------|-------------|---------|
| `caddy_modules` | List of Caddy modules to compile | `[github.com/mholt/caddy-ratelimit, github.com/caddy-dns/cloudflare]` |

### Rate Limiting

| Variable | Description | Default |
|----------|-------------|---------|
| `caddy_rate_limit.enabled` | Enable/disable rate limiting | `true` |
| `caddy_rate_limit.events` | Max requests per window | `60` |
| `caddy_rate_limit.window` | Time window for rate limit | `1m` |

### Deploy SSH Key Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `deploy_ssh_key_user` | User that owns the deploy SSH key | `caddy` |
| `deploy_ssh_key_group` | Group for the deploy SSH key | `{{ deploy_ssh_key_user }}` |
| `deploy_ssh_key_dir` | Directory for SSH keys | `/var/lib/{{ deploy_ssh_key_user }}/.ssh` |
| `deploy_ssh_key_path` | Path to deploy key | `{{ deploy_ssh_key_dir }}/deploy_key` |
| `deploy_ssh_key_type` | SSH key type (ed25519 recommended) | `ed25519` |
| `deploy_ssh_key_comment` | SSH key comment | `{{ domain }}-deploy-key` |

### Website Rebuild Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `webrebuild_schedule` | SystemD timer schedule (systemd.time format) | `*-*-* 04:00:00` (daily at 4:00 AM) |
| `webrebuild_boot_delay` | Delay after boot before first rebuild (seconds) | `180` |
| `webrebuild_service_user` | User to run the rebuild service | `caddy` |
| `webrebuild_service_group` | Group to run the rebuild service | `caddy` |
| `webrebuild_commands` | Commands to run during rebuild | `["git pull origin {{ website_repo_branch }}", "hugo --gc --minify"]` |

### Firewall Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `firewall_zone` | Default firewall zone | `public` |
| `firewall_allowed_services` | Services to allow through firewall | `[ssh, http, https]` |
| `firewall_allowed_ports` | Custom ports to allow (optional) | `[]` (empty) |
| `firewall_allowed_icmp` | Allow ICMP (ping) traffic | `true` |
| `firewall_allowed_icmp_types` | ICMP types to allow | `[echo-request]` |

## 🏷️ Ansible Tags

Run specific parts of the playbook using tags:

```bash
# Run entire web server role (all caddy_webserver tasks)
ansible-playbook site.yml --tags webserver

# Run entire firewall role (all firewall tasks)
ansible-playbook site.yml --tags security

# Install only Caddy
ansible-playbook site.yml --tags caddy

# Configure only firewall
ansible-playbook site.yml --tags firewall

# Update Hugo version
ansible-playbook site.yml --tags hugo

# Rebuild Caddy with new modules
ansible-playbook site.yml --tags caddy,build

# Update website configuration only
ansible-playbook site.yml --tags config

# Update systemd services
ansible-playbook site.yml --tags systemd

# Regenerate SSH deploy key
ansible-playbook site.yml --tags deploy-key
```

### Available Tags

| Tag | Description |
|-----|-------------|
| `webserver` | All web server tasks (entire caddy_webserver role) |
| `security` | All security tasks (entire firewall role) |
| `caddy` | Caddy installation and configuration |
| `hugo` | Hugo installation |
| `packages` | Package installation tasks |
| `repo` | COPR repository configuration |
| `build` | Caddy build from source |
| `config` | Configuration file updates |
| `directories` | Directory creation tasks |
| `systemd` | SystemD service management |
| `firewall` | Firewall configuration |
| `services` | Firewall services configuration |
| `ports` | Firewall ports configuration |
| `icmp` | Firewall ICMP configuration |
| `deploy-key` | SSH deploy key generation |
| `ssh` | SSH configuration |
| `website` | Website deployment tasks |
| `initial-build` | Initial Hugo build |
| `rebuild` | Website rebuild configuration |

## 🔄 Manual Operations

### Manual Site Rebuild

```bash
# SSH into your server
ssh user@your-server

# Trigger immediate rebuild
sudo systemctl start webrebuild.service

# Check rebuild status
sudo systemctl status webrebuild.service

# View rebuild logs
sudo journalctl -u webrebuild.service -f
```

### Check Timer Status

```bash
# View timer status and next run time
sudo systemctl status webrebuild.timer

# List all timers
sudo systemctl list-timers

# Check timer logs
sudo journalctl -u webrebuild.timer
```

### Caddy Management

```bash
# Reload Caddy configuration
sudo systemctl reload caddy

# Restart Caddy
sudo systemctl restart caddy

# Check Caddy status
sudo systemctl status caddy

# View Caddy logs
sudo journalctl -u caddy -f
```

### View Access Logs

```bash
sudo tail -f /var/log/caddy/access.log
```

## 🔐 Security Features

- ✅ Automatic HTTPS with Let's Encrypt
- ✅ HSTS headers with preload
- ✅ Content Security Policy (CSP)
- ✅ X-Content-Type-Options
- ✅ Strict referrer policy
- ✅ Rate limiting per client IP
- ✅ Automatic firewall configuration
- ✅ SSH key-based Git authentication
- ✅ Minimal attack surface (static site only)

## 🔄 Updating Components

### Update Hugo

```yaml
# Edit group_vars/webservers/vars.yml
hugo_version: "0.153.0"  # Update to new version
```

```bash
ansible-playbook site.yml --tags hugo
```

### Update Caddy

```yaml
# Edit group_vars/webservers/vars.yml
caddy_version: "2.11.0"  # Update to new version
```

```bash
ansible-playbook site.yml --tags caddy,build
```

### Add/Remove Caddy Modules

```yaml
# Edit group_vars/webservers/vars.yml
caddy_modules:
  - github.com/mholt/caddy-ratelimit
  - github.com/caddy-dns/cloudflare
  - github.com/caddyserver/cache-handler  # Add new module
```

```bash
# Rebuild Caddy with new modules
ansible-playbook site.yml --tags caddy,build
```

## 📚 Additional Resources

- [Hugo Documentation](https://gohugo.io/documentation/)
- [Caddy Documentation](https://caddyserver.com/docs/)
- [Ansible Documentation](https://docs.ansible.com/)
- [Rocky Linux Documentation](https://docs.rockylinux.org/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [Cloudflare DNS API](https://developers.cloudflare.com/api/)

## ⚠️ Disclaimer

This playbook has been tested on Rocky Linux 10. While it should work on other RHEL 9+ family distributions, some adjustments may be needed.

## Authors

Module managed by [Danylo Mikula](https://github.com/danylomikula).

## Contributing

Contributions are welcome! Please read the [Contributing Guide](.github/contributing.md) for details on the process and commit conventions.

## License

Apache 2.0 Licensed. See [LICENSE](LICENSE) for full details.
