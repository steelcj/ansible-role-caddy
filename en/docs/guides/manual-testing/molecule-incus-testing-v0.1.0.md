# Testing ansible-role-caddy with Molecule and Incus

Version: 0.1.0

Date: 2026-11-25

Adapted from the supplied Foomuuri manual-testing guide. Run the commands on the controller where Incus is installed and initialized, not on a production Caddy server.

## Prepare the controller

Activate your existing Ansible development environment, or create a dedicated one. The existing Foomuuri workflow uses the current Ansible venv.

```sh
. ~/.venvs/ansible/current/bin/activate
python -m pip install molecule ansible-lint yamllint
ansible --version
molecule --version
incus list local:
incus image list images: debian/13 --format table
```

The Incus client must be permitted to operate the local server without sudo. No additional Molecule driver is needed. The scenario installs `community.general` 13.5.1 from Galaxy; use an ansible-core release compatible with it (the intended development environment is ansible-core 2.20). Internet access is needed to fetch the image, collection and Debian packages on the first run. Incus must have functioning storage and container networking.

## Run the scenario

From the directory in which you want the repository extracted, ensuring no older checkout of the same name exists:

```sh
tar -xzf ansible-role-caddy-v0.1.0.tar.gz
cd ansible-role-caddy
molecule test
```

The sequence is dependency, destroy, syntax, create, prepare, converge, idempotence, verify, destroy. It uses `caddy-molecule-debian13` in the local default project; any existing container with this role's ownership marker is disposable. A container without the marker causes a refusal. Run only one instance of the scenario at a time.

## Troubleshoot

```sh
molecule test --destroy=never
incus exec local:caddy-molecule-debian13 -- bash
systemctl status caddy --no-pager
journalctl -u caddy --no-pager -n 100
caddy version
runuser -u caddy -- env HOME=/var/lib/caddy caddy validate --config /etc/caddy/Caddyfile --adapter caddyfile
ss -ltn
exit
molecule destroy
```

You can also run individual actions: `molecule dependency`, `molecule create`, `molecule prepare`, `molecule converge`, `molecule idempotence`, `molecule verify`, and `molecule destroy`. The test fragments only listen on container loopback, and use reserved `.test` names with explicit HTTP, so no public certificates or DNS records are needed.

## Record results

Record the controller's Ansible, Molecule, community.general and Incus versions, the actual Debian image fingerprint, installed Caddy package version, and the scenario recap. Idempotence should show no changes on the second converge, and all verification steps should pass (the intentionally rejected invalid configuration is rescued). The archive author has not run an Incus integration test; this is the first development test on the controller.
