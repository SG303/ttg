# ttg

Minimal Tailscale + Traefik gateway in Docker: lets a machine that cannot
join the tailnet (e.g. a locked-down work laptop with WSL) reach services in
the tailnet by their normal hostnames. Traefik only passes TLS through to the
home Traefik, so the gateway holds no certificates or DNS tokens. Public repo.

Follows the SG303 baseline v1 (`sgstack/global/AGENTS.md`,
`sgstack/baseline/tools.md`). If you cannot read those: work on a branch,
open a PR, merge only with green checks (squash); never commit secrets or
personal data; English for everything in the repo; talk to me in German.

## Commands

- Validate: `docker compose -f gateway/docker-compose.yml --env-file gateway/.env.example config --quiet`
- Run (on the client machine, not on this host): `cd gateway && cp .env.example .env`,
  set `TS_AUTHKEY` and the address in `forward.yml`, then `docker compose up -d`.

## Release

No releases or tags; users run `main`.

## Conventions

- Everything lives in `gateway/`: `docker-compose.yml`, `traefik.yml`
  (entry points) and `forward.yml` (TCP pass-through to the home Traefik).
- Keep it minimal: official images only, no build step, no start script, no
  certificates or DNS credentials on the gateway.
- `forward.yml` and the README use placeholders (`100.64.0.10`,
  `app1.example.com`); never commit real Tailscale IPs, hostnames or auth
  keys. `TS_AUTHKEY` lives only in the gitignored `.env`.

## Deviations from baseline

- `image-pinned`: `tailscale/tailscale:stable` follows Tailscale's stable
  channel on purpose, so `docker compose pull` picks up client fixes without
  repo changes. Traefik is pinned to major `v3`.
