# **Ansible role Caddy v0.1.0**

**Download ansible-role-caddy-v0.1.0.tar.gz**

Initial development release · steelcj.caddy · Debian 13

Download SHA-256 checksum

The role follows the organizational conventions and local Incus testing architecture of your existing Foomuuri role, while implementing the service foundation described in the MCSPF Caddy multisite customization.&#x20;

customization--caddy-multisite-v0.1.0.md

## **Implemented capabilities**

| Capability               | Implementation                           |
| ------------------------ | ---------------------------------------- |
| Role name                | `steelcj.caddy`                          |
| Version                  | `0.1.0`                                  |
| Reference distribution   | Debian 13 (trixie)                       |
| Package management       | Authenticated Debian APT repositories    |
| Service management       | Package-provided systemd unit            |
| Service account          | Dedicated `caddy` user                   |
| Certificate management   | Native Caddy automatic HTTPS             |
| Administration interface | Loopback only, port 2019                 |
| Configuration validation | Staged, before activation                |
| Configuration recovery   | Attempted rollback on activation failure |
| Graceful reload          | Implemented                              |
| Molecule testing         | Local Incus system container             |
| License                  | Apache-2.0                               |

## **Repository structure**

```
ansible-role-caddy/
├── .ansible-lint
├── .gitignore
├── .yamllint
├── CHANGELOG.md
├── LICENSE
├── README.md
├── ROADMAP.md
├── requirements.yml
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   ├── main.yml
│   ├── assert.yml
│   ├── configure.yml
│   └── activate.yml
├── templates/
│   └── Caddyfile.j2
├── vars/
│   └── main.yml
├── molecule/
│   └── default/
│       ├── molecule.yml
│       ├── requirements.yml
│       ├── create.yml
│       ├── prepare.yml
│       ├── converge.yml
│       ├── verify.yml
│       └── destroy.yml
└── en/
    └── docs/
        ├── design/
        │   └── role-selection-v0.1.0.md
        └── guides/
            └── manual-testing/
                └── molecule-incus-testing-v0.1.0.md
```

## **Molecule and Incus testing**

The scenario uses the same basic approach as `ansible-role-foomuuri`, without requiring Docker, SSH connectivity or an additional Molecule driver.

Local Molecule testing lifecycle

The test container is named `caddy-molecule-debian13`. I also added an Incus ownership marker to prevent the scenario from inadvertently deleting an unrelated container with the same name.

### **Verification tests**

The Molecule scenario is designed to verify:

- Successful installation and service activation.
- Service enabled at boot and running as a non-root user.
- Restrictive configuration and persistent storage permissions.
- Administration interface restricted to loopback.
- Two independent websites with correct hostname routing.
- Idempotence.
- Rejection of an invalid configuration without replacing the working global configuration.
- Graceful reload without restarting the Caddy process.
- Preservation of independently managed website configurations.

The test websites use reserved `.test` domains and loopback HTTP, so no public DNS records or TLS certificates are required.

## **Running the first test**

On your controller, using your existing Ansible development environment:

### **Extract the archive**

```
tar -xzf ansible-role-caddy-v0.1.0.tar.gz

cd ansible-role-caddy
```

### **Verify Incus connectivity**

```
incus list
incus image list images: debian/13 \
    --format table
```

### **Run Molecule**

For the initial development run, I recommend retaining the container for inspection.

```
molecule test --destroy=never
```

### **Inspect the container**

```
incus exec caddy-molecule-debian13 -- bash
```

Once inside:

```
caddy version

systemctl status caddy

cat /etc/caddy/Caddyfile

ls -la /etc/caddy/sites.d/

ss -ltn

journalctl -u caddy --no-pager
```

When finished:

```
exit
molecule destroy
```

## **Default Caddy configuration**

The role establishes the following configuration architecture:

```
/etc/caddy/
├── Caddyfile
├── snippets/
└── sites.d/

/var/lib/caddy/
```

The default Caddyfile is deliberately minimal.

```
{
    admin localhost:2019
}

import /etc/caddy/sites.d/*.caddy
```

An administrative email address can be supplied through inventory.

```
caddy_email: admin@example.org
```

Automatic HTTPS remains enabled for subsequently configured websites.

## **Example playbook**

```
---
- name: Establish the Caddy service foundation
  hosts: caddy_servers
  become: true
  gather_facts: true

  roles:
    - role: steelcj.caddy
      caddy_email: admin@example.org
```

## **Consult before creating**

In accordance with your Robert de Bock CFAP, I documented the existing-role evaluation within the repository.&#x20;

robertdebock-ansible-roles-cfap-v0.1.1.md



| Candidate                     | Finding                                                      |
| ----------------------------- | ------------------------------------------------------------ |
| `robertdebock.caddy`          | No dedicated role located                                    |
| `robertdebock.software`       | Generic package installation, but not Caddy-specific configuration management |
| `caddy-ansible/caddy-ansible` | Existing implementation, but uses a different binary installation, service management and configuration model |
| `steelcj.caddy`               | Initial dedicated service foundation implementation          |

The existing third-party Caddy role defaults to downloaded binaries, a templated systemd unit and the `www-data` service account. Our implementation instead preserves the distribution-provided package and service.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## **Decisions and limitations**

I made several initial implementation decisions that remain subject to your approval:

- Debian's native package repository rather than introducing another repository.
- Debian 13 as the only currently supported distribution.
- Apache-2.0 licensing, matching the supplied Foomuuri role.
- No firewall management, website publishing or website lifecycle management within this role.
- Synchronous configuration activation to permit rollback attempts if activation fails.

The archive has passed YAML parsing, duplicate-key detection, Jinja template rendering and SHA-256 verification.

**Ansible syntax checking and actual Molecule integration testing remain pending**, because Ansible, Caddy, Molecule and Incus are not installed in my execution environment. This is therefore a development release, not a production-validated role.

The next milestone is a successful local `molecule test` on your controller, followed by development of the independent `steelcj.caddy_sites` role.