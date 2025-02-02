# Attic Compose
Deploying Attic Nix Binary Cache With Docker Compose

## Server Install
Install docker and docker compose

`git clone`

See [/scr](./src), create a `prod.env` and `server.toml` files

then run

```bash
just up
just create_token <your username here>
```

### Exmaple Traefik Label
```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.attic.rule=Host(`nix.example.com`)"
  - "traefik.http.routers.attic.entrypoints=websecure"
  - "traefik.http.routers.attic.tls.certresolver=myhttpchallenge"
  - "traefik.http.services.attic.loadbalancer.server.port=8080"
  - "traefik.http.routers.attic-http.middlewares=redirect-to-https"
  - "traefik.docker.network=<network name>"
```

### Cloudflare
If you are using cloudflare make the subdomain DNS only

### Check if it works
If working nix.example.com should say `attic push`

## Client Install
Install `pkg.attic-client`

make sure your user is trusted
```nix
nix.settings = {
  trusted-users = [
    "root"
    "<your username here>"
  ];
};
```

```bash
# then login to attic
attic login <pick a name for server> https://nix.example.com <token from just create_token>

# create a cache to push to
attic cache create <cache name>

# use the cache
attic use <cache name>

# pushing to the cache
attic push <cache name> /nix/store/*/
```

## Github Actions Install
Add the token from `just create_token` to your repository secrets `https://github.com/<username>/<repo>/settings/secrets/actions`
```yaml
steps:
  - uses: actions/checkout@v3
  - uses: DeterminateSystems/nix-installer-action@main
  - run: nix run -I nixpkgs=channel:nixos-unstable nixpkgs#attic-client login <pick a name for server> https://nix.example.com ${{ secrets.ATTIC_TOKEN }}
  - run: nix run -I nixpkgs=channel:nixos-unstable nixpkgs#attic-client cache create <cache name> || true
  - run: nix run -I nixpkgs=channel:nixos-unstable nixpkgs#attic-client use <cache name>

  - run: nix flake check --all-systems

  - run: nix run -I nixpkgs=channel:nixos-unstable nixpkgs#attic-client push <cache name> /nix/store/*/ || true
```

# License
All code in this repository is dual-licensed under either [License-MIT](./LICENSE-MIT) or [LICENSE-APACHE](./LICENSE-Apache) at your option. This means you can select the license you prefer. [Why dual license](https://github.com/bevyengine/bevy/issues/2373).
