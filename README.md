# UChicago DSI Container Registry

A private Docker/OCI container registry running on Cloudflare Workers with R2 storage, forked from [cloudflare/serverless-registry](https://github.com/cloudflare/serverless-registry).

**Registry URL:** `registry.uchicago-dsi.org`
**Domain:** Managed under `uchicago-dsi.org` in Cloudflare (DNS and SSL handled automatically)
**Credentials:** Username and password are stored in Bitwarden.

## Using the Registry

### Login

```bash
docker login registry.uchicago-dsi.org -u core-facility-registry
```

You'll be prompted for the password. Look up the credentials in Bitwarden.

### Pushing Images

```bash
# Tag your image for the registry
docker tag myapp:latest registry.uchicago-dsi.org/myapp:latest

# Push
docker push registry.uchicago-dsi.org/myapp:latest
```

### Pulling Images

```bash
docker pull registry.uchicago-dsi.org/myapp:latest
```

### Listing Repositories

```bash
curl -u core-facility-registry:$PASSWORD https://registry.uchicago-dsi.org/v2/_catalog
```

## Infrastructure

- **Worker:** `uchicago-dsi-registry-production`
- **R2 Bucket:** `r2-registry`
- **Domain:** `registry.uchicago-dsi.org` (custom domain on Cloudflare)
- **Auth:** Username/password (stored as Wrangler secrets)

## Administration

### Rotating Credentials

```bash
npx wrangler secret put USERNAME --env production
npx wrangler secret put PASSWORD --env production
```

### Deploying Updates

```bash
pnpm install
npx wrangler deploy --env production
```

### Adding Read-Only Credentials

For users/services that should only be able to pull (not push):

```bash
npx wrangler secret put READONLY_USERNAME --env production
npx wrangler secret put READONLY_PASSWORD --env production
```

### Garbage Collection

Remove unreferenced layers for a specific image:

```bash
curl -X POST -u core-facility-registry:$PASSWORD \
  https://registry.uchicago-dsi.org/myapp/gc
```

### Configuring Pull Fallback

To mirror images from another registry (e.g., Docker Hub), add to `wrangler.toml`:

```toml
[env.production.vars]
REGISTRIES_JSON = "[{ \"registry\": \"https://index.docker.io/\" }]"
```

For authenticated fallback registries, set the token as a secret:

```bash
echo $TOKEN | npx wrangler secret put REGISTRY_TOKEN --env production
```

Then reference it in the config:

```toml
REGISTRIES_JSON = "[{ \"registry\": \"https://index.docker.io/\", \"password_env\": \"REGISTRY_TOKEN\", \"username\": \"your-docker-username\" }]"
```

## Known Limitations

- Pushing with Docker is limited to layers of maximum 500MB (Cloudflare Workers request body size limit).
- To work around this, see the `./push` folder for a tool that handles large layers.
- Local dev with `npx wrangler dev` buffers requests in the Worker, which may be slower.

## Development

```bash
pnpm install
npx wrangler dev --env dev
# Dev credentials: username=hello, password=world
```

## License

Licensed under the [Apache License](https://opensource.org/licenses/apache-2.0/). See `CONTRIBUTING.md` for contribution guidelines.
