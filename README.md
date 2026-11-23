# subspace

Managing a WireGuard VPN means creating client configurations and removing access when a device no longer needs it. subspace provides a web interface for those tasks, with downloadable configurations and QR codes for mobile clients.

## Features

- Add WireGuard devices for macOS, Windows, Linux, Android, and iOS clients.
- Remove a device's client key to revoke its VPN access.
- Generate a unique downloadable configuration for each client.
- Display QR codes for configuration import on iOS and Android.
- Authenticate users through SAML providers such as G Suite and Okta.
- Obtain TLS certificates through Let's Encrypt and redirect HTTP requests to HTTPS.

## Installation

The HTML interface is part of the server application; opening it in a browser alone does not provide a VPN server. Use a Linux host with Docker and WireGuard installed.

**The container example below uses the donor project's image, `subspacecloud/subspace:latest`.** It is an upstream deployment reference, not a separately published image for this standalone project.

On an Ubuntu host with WireGuard available in its package repositories:

```bash
apt-get update
apt-get install -y wireguard

# Load modules.
modprobe wireguard
modprobe iptable_nat
modprobe ip6table_nat

# Enable IP forwarding
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.ipv6.conf.all.forwarding=1
```

Run these commands as root. The forwarding settings above apply to the current session; persist them through your host's sysctl configuration before relying on them after a reboot.

Install Docker using the [Docker Engine installation instructions](https://docs.docker.com/engine/install/). The donor container runs dnsmasq internally, so check for host DNS services that could conflict before starting it. Do not remove a host DNS service without checking how the server resolves names.

Create a public DNS `A` record for your server, such as `subspace.example.com`. Allow inbound TCP ports 80 and 443 and UDP port 51820. Let's Encrypt needs port 80 for verification.

Create the data directory and container. Replace `subspace.example.com` with your domain, and confirm that `/usr/bin/wg` is the host's WireGuard executable.

```bash
mkdir /data

docker create \
    --name subspace \
    --restart always \
    --network host \
    --cap-add NET_ADMIN \
    --volume /usr/bin/wg:/usr/bin/wg \
    --volume /data:/data \
    --env SUBSPACE_HTTP_HOST=subspace.example.com \
    subspacecloud/subspace:latest
```

**Keep `/data` persistent and back it up.** The container uses this directory for its application data. Host networking and `NET_ADMIN` also give the container access to the host's network configuration; deploy it only on a host you administer.

## Usage

Start the container and inspect its logs:

```bash
sudo docker start subspace
sudo docker logs subspace
```

Open `https://subspace.example.com` in your browser. Add a device through the web interface, then download its configuration or scan its QR code with a WireGuard client. Remove the device in the interface when you need to revoke its access.

If you have a server binary installed instead of using Docker, the equivalent hostname argument is:

```bash
subspace --http-host subspace.example.com
```

The server defaults to `/data` for storage and enables Let's Encrypt by default. Run `subspace -help` to inspect the available options. SAML authentication requires configuration for your identity provider; a public hostname alone does not configure SSO.

## License

MIT. This project is based on [subspacecloud/subspace](https://github.com/subspacecloud/subspace), also licensed under MIT. Retain the original copyright notice and MIT permission notice in copies or substantial portions of the software.

