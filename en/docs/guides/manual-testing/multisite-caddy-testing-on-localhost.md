# Multisite Caddy Testing on Localhost

## Requirements

* Incus

## Administration API

Port 2019 is Caddy's administration API, not its web hosting interface. It is deliberately restricted to the container's loopback interface and should not be publicly exposed.

## Dummy Test Sites

The Molecule scenario establishes two really boring dummy websites.

| Address inside container   | Purpose                  |
| -------------------------- | ------------------------ |
| `localhost:2019`           | Caddy administration API |
| `first.example.test:8080`  | First dummy website      |
| `second.example.test:8080` | Second dummy website     |

## Testing  dummy sites with molecule

Run the Molecule container with destroy=never:

```
molecule test --destroy=never
```

Then enter the container:

```
incus exec caddy-molecule-debian13 -- bash
```

Install curl if necessary:

```
apt-get install curl
```

Test the first website:

```
curl -H 'Host: first.example.test:8080' \
    http://127.0.0.1:8080/
```

Expected response:

You will see this output just before your root prompt

```
first siteroot@caddy-molecule-de
```

Test the second website:

```
curl -H 'Host: second.example.test:8080' \
    http://127.0.0.1:8080/
```

Expected response:

again, you will see this output just before your root prompt

```
second siteroot@caddy-molecule-d
```

## Access the websites from your desktop web browser

Incus supports proxy devices, which allow us to forward a port on your workstation to a port inside the container.

On your workstation, run:

```
incus config device add caddy-molecule-debian13 \
    web-test proxy \
    listen=tcp:127.0.0.1:8080 \
    connect=tcp:127.0.0.1:8080
```

Add the following entry to your workstation's `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Entry:

```bash
127.0.0.1 first.example.test second.example.test
```

You may need to adjust browser setting to allow for **Insecure content** to be displayed. Then you should then be able to visit and see your test:

- [http://first.example.test:8080/](http://first.example.test:8080/)
- [http://second.example.test:8080/](http://second.example.test:8080/)

No firewall changes or public DNS records are required.

## **Recommendation for v0.1.1**

The Molecule scenario could be set up to provide a more useful visual testing environment:

- Two distinct HTML dummy websites.
- An automatically configured local Incus proxy device.
- A test of actual static file serving rather than just Caddy's `respond` directive.
- Independent document roots under `/srv/www/`.
- Browser-based manual testing instructions.
- Automatic proxy removal during Molecule destruction.

This would give us both automated functional verification and convenient visual verification without exposing the container publicly.

The current v0.1.0 intentionally uses plain-text responses rather than HTML files. The proxy configuration above is a proposed manual extension and has not yet been integration-tested.