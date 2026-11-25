# Caddy role selection findings

Version: 0.1.0

Date: 2026-11-25

Following the supplied Robert de Bock CFAP and the MCSPF Caddy multisite customization, consult existing roles before writing new ones.

| Field | Finding |
| --- | --- |
| Need | Install Caddy from a trusted distribution package; preserve the package unit/account; manage global configuration with staged validation and independent site imports. |
| Candidate | Robert de Bock Caddy role: none found in the supplied CFAP or public repository search. The expected repository returned HTTP 404. |
| Repository | https://github.com/robertdebock/ansible-role-caddy |
| Can it do it? | No dedicated role located; not proof that no alternative exists. |
| Evidence | Supplied CFAP v0.1.1, needs with no role; repository lookup. |
| Action | Evaluate alternatives and record the decision. |

| Field | Finding |
| --- | --- |
| Candidate | robertdebock.software |
| Repository | https://github.com/robertdebock/ansible-role-software |
| Can it do it? | Partial: generic package installation. |
| Evidence | `defaults/main.yml` exposes `software_packages`; does not express the Caddy configuration transaction. |
| Action | Keep package installation within the small Caddy foundation role so first installation can suppress package autostart and precede validation. |

| Field | Finding |
| --- | --- |
| Candidate | caddy-ansible/caddy-ansible |
| Repository | https://github.com/caddy-ansible/caddy-ansible |
| Can it do it? | Partial; would require adaptation to the chosen package-managed service and transaction model. |
| Evidence | Its README, defaults and tasks provide binary downloads from GitHub or Caddy's download API, a configurable full Caddyfile, default `www-data` identity, and templated systemd service. The tasks install the configuration and notify reload rather than staging complete validation first. |
| Action | Develop a minimal Debian-native role for this first iteration. Broader alternatives, including collection-based implementations, remain to be evaluated before claiming that a custom role is definitively necessary. |

## Implementation decisions

- Debian 13 only, as the reference distribution; additional distributions require explicit testing.
- Debian archive packages, not a new Cloudsmith repository or arbitrary binary download. APT authenticates metadata and verifies package checksums under the existing repository trust configuration.
- Retain the packaged systemd unit and service identity, keep the local admin endpoint available for graceful reloads, and keep site management outside the foundation.
- Synchronous reload inside the configuration transaction enables rollback attempts on activation failure. No implicit firewall, site or release deployment.
- Adapt the supplied Foomuuri Molecule scenario; pin its collection dependency for this draft. Image and package versions are not immutable by default.

## Resources

- https://github.com/robertdebock/ansible-role-software/blob/master/defaults/main.yml
- https://github.com/caddy-ansible/caddy-ansible/blob/master/defaults/main.yml
- https://github.com/caddy-ansible/caddy-ansible/blob/master/tasks/main.yml
- https://sources.debian.org/src/caddy/2.6.2-12%2Bdeb13u1/debian/README.Debian
- https://caddyserver.com/docs/caddyfile/options
- https://caddyserver.com/docs/automatic-https
- https://docs.ansible.com/projects/ansible/latest/collections/community/general/incus_connection.html
