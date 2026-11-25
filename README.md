# Ansible role caddy

Version: 0.1.0 (development draft, 2026-11-25)

Role: `steelcj.caddy`

Establish the Caddy service foundation of MCSPF on Debian 13 (trixie). The role is deliberately independent of website publishing, site definitions and firewall policy. It uses Debian's authenticated APT sources and preserves the package's systemd unit and dedicated service account.

## Example playbook

```yaml
---
- name: Establish the Caddy service foundation
  hosts: caddy_servers
  become: true
  gather_facts: true
  roles:
    - role: steelcj.caddy
      caddy_email: admin@example.org
```

Use a real email address for production ACME notifications. The default is empty and automatic HTTPS remains enabled for any subsequently configured sites. No public site, DNS change, certificate request for a public hostname or firewall opening is introduced by the foundation alone.

To use the role in an ordinary playbook, install or link the extracted directory under the name `steelcj.caddy` in your Ansible roles path. For example, from the extracted repository:

```sh
mkdir -p ~/.ansible/roles
ln -s "$PWD" ~/.ansible/roles/steelcj.caddy
```

The Molecule scenario uses the checkout directly and does not need that link. The production prerequisites are `foundation_ssh` and `baseline_secure`; they are not role dependencies automatically executed by this role.

## Role variables

| Variable | Default | Purpose |
| --- | --- | --- |
| `caddy_email` | `""` | Optional global ACME account email. |
| `caddy_package_version` | `""` | Exact APT package version if supplied; otherwise install the trusted candidate if absent. |
| `caddy_apt_cache_valid_time` | `3600` | Maximum age of APT metadata in seconds. |

No external package repository is added. APT is expected to have trustworthy sources and keyrings already configured; the role does not audit source trust overrides. The package is not automatically upgraded on every run. Exact version selection is not an APT hold or a snapshot pin; an existing newer version is not deliberately downgraded. Review `apt-cache policy caddy` and the distribution's security maintenance before promotion. The package in Debian trixie is not necessarily the newest upstream Caddy.

## Configuration architecture

```text
/etc/caddy/                 root:caddy 0750
├── Caddyfile               root:caddy 0640
├── snippets/               root:caddy 0750
└── sites.d/                root:caddy 0750
/var/lib/caddy/             caddy:caddy 0700
```

The default global configuration is:

```caddyfile
{
    admin localhost:2019
}

import /etc/caddy/sites.d/*.caddy
```

An empty site directory is intentional. Snippets are available for explicit import by site fragments, not imported indiscriminately. Caddy manages its certificate and configuration storage under its package-provided home. Do not grant publishing accounts membership in the service group or write access to configuration or state. The local TCP admin endpoint is not an authentication boundary against local users; trusted local accounts are a prerequisite. Unix-socket administration is a future hardening decision.

The role stages a changed global file in `/etc/caddy`, validates the complete configuration with the Caddy service identity and normal Debian HOME, atomically copies it into place, then reloads an active service or starts an inactive one. A failed validation leaves the previous file untouched. If activation fails, the role attempts to restore the previous file and runtime configuration and fails the play; a failed rollback needs manual intervention. Temporary staging files are removed. There is no persistent backup of the prior global file; retain the authoritative configuration in version control and back up existing installations before adoption.

An existing global Caddyfile is replaced by the role. Migrate inline sites into `sites.d` first. Site files are not created, purged, chmodded or updated by the role. Ensure fragments are readable by Caddy and writable only by trusted administrators. Reapplying the role with unchanged global content does not reload Caddy; external site changes need their own validation and reload. The role does not serialize competing external configuration writers: run provisioning serially. Custom service units, custom environment files, custom admin endpoints and other Caddy installation layouts have not been supported or validated. Check mode covers assertions and APT only, not configuration or runtime validation.

## Testing

The scenario follows `ansible-role-foomuuri`: Molecule's built-in default driver runs local Incus commands, and `community.general.incus` supplies Ansible connectivity without SSH. It creates an unprivileged Debian 13 system container using `images:debian/13`, bootstraps Python and sudo, converges twice, verifies, and destroys. No privileged mode or Docker is required. The fixed test container name `caddy-molecule-debian13` is reserved for this scenario; a marker guards against accidental deletion of unrelated instances. Do not run two scenarios concurrently under the same name. The local Incus project is `default`.

```sh
incus list local:
incus image list images: debian/13 --format table
molecule test
```

See [the Incus testing guide](en/docs/guides/manual-testing/molecule-incus-testing-v0.1.0.md) for setup and debugging. The scenario tests two loopback-only HTTP virtual hosts, service identity, permissions, local admin binding, idempotence, candidate rejection and graceful reload. It does not issue a public ACME certificate or test the complete MCSPF baseline/firewall.

| Platform | Role | Result |
| --- | --- | --- |
| Debian 13 Incus | 0.1.0 | Pending execution on an Incus controller |

## Role selection and license

See [role selection findings](en/docs/design/role-selection-v0.1.0.md). The code and adapted Molecule harness use Apache-2.0, matching the supplied Foomuuri role. See [LICENSE](LICENSE). This is a development draft, not a production-validated release.
