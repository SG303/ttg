<img src="assets/icon.svg" width="64" height="64" alt="">

# ttg

A minimal Tailscale + Traefik gateway. It lets a machine that cannot join your tailnet directly
(e.g. a locked-down work laptop, using WSL/Docker) reach services in your tailnet by their
normal hostnames, with valid TLS certificates and almost nothing to maintain.

```
Browser ──443──> gateway (Docker: tailscale + traefik TCP pass-through)
                    └─ tailnet ─> your Traefik (TLS, ACME) ─> services
```

TLS is terminated by your existing Traefik, so the gateway holds no certificates and no DNS tokens.
It uses only official images, with no build step and no start script.

## Prerequisites

- A Traefik instance in your tailnet that already serves your hostnames over TLS (router with `tls` and a certificate resolver, e.g. DNS challenge).
- Port 443 of that host is reachable on its Tailscale IP.
- A Tailscale ACL that allows `tag:gateway` to reach `<home-tailscale-ip>:443`.
- A Tailscale auth key tagged `tag:gateway` (tagged devices have key expiry disabled by default).

## Setup

```bash
cd gateway
cp .env.example .env      # set TS_AUTHKEY
# set the Tailscale IP of your Traefik host in forward.yml
docker compose up -d
```

After the first login `TS_AUTHKEY` can be removed; the node state persists in the `ts-state` volume.

Point your hostnames at the gateway in the client's hosts file
(`/etc/hosts`, or `C:\Windows\System32\drivers\etc\hosts` on Windows):

```
127.0.0.1 app1.example.com app2.example.com
```

With WSL, `networkingMode=mirrored` in `%UserProfile%\.wslconfig` (`[wsl2]`, Windows 11) keeps
`127.0.0.1` stable. Otherwise use the WSL IP.

## Updates

```bash
docker compose pull && docker compose up -d
```

## Adding a service

Configure it on your home Traefik and add the hostname to the client's hosts file.
The gateway itself needs no change.

## Troubleshooting

- **Certificate error:** the request does not reach your Traefik. Check `docker compose logs proxy` and the ACL.
- **404 from Traefik:** the pass-through works, but there is no router for that host.
- **Timeout:** check reachability with `docker compose exec tailscale tailscale ping <home-tailscale-ip>`.
- **Port 80/443 already in use:** change the published ports in `docker-compose.yml`.
