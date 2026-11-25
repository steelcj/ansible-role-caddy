# Roadmap

## Initial development gate

- Run Molecule on Debian 13 through Incus; fix any compatibility failures and record exact tool and image versions.
- Validate against `foundation_ssh` and `baseline_secure`, including umask, AppArmor and Foomuuri integration.
- Test package upgrades and configuration recovery, including a deliberately failed service activation and reboot.
- Evaluate local admin Unix socket permissions and stronger systemd restrictions without replacing the vendor unit.
- Decide package source, supported security maintenance and version pinning policy before production.

## Subsequent role

Implement `steelcj.caddy_sites` separately: static content roots, domain aliases, site lifecycle, per-site logging and transactional site configuration, while preserving active releases. Test public HTTPS only in a separately authorized environment.
