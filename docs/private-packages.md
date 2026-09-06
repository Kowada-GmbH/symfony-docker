# Using Private Composer Packages

A project built from this template sometimes needs a private VCS package
(e.g. `kowada-gmbh/starter-bundle`). Composer needs GitHub credentials to
resolve it, in three different places:

## Local development

The `php` and `messenger` services read an optional, git-ignored `.env.local`
file (via `env_file`, `required: false`) for a `COMPOSER_AUTH` variable -
composer's own `auth.json` format, as a compact, unquoted single-line JSON
value:

```env
COMPOSER_AUTH={"github-oauth":{"github.com":"<token>"}}
```

Without it, `composer.json` requiring only public packages works exactly as
before - this is opt-in and adds nothing for projects that don't need it.

## CI and production builds

`frankenphp_test` and `frankenphp_prod_builder` (the Dockerfile stages that
`composer install` at _build_ time, not container start) read the same
`COMPOSER_AUTH` value through a BuildKit build secret (`composer_auth`,
declared in `compose.yaml`, sourced from the `COMPOSER_AUTH` environment
variable at build time) rather than a build `ARG`, so it never lands in an
image layer.

Set a `COMPOSER_AUTH` (or equivalent) GitHub Actions secret and pass it into
the build step's environment, e.g. in `.github/workflows/ci.yaml`:

```yaml
- name: Build Docker images
  env:
    COMPOSER_AUTH: '{"github-oauth":{"github.com":"${{ secrets.COMPOSER_GITHUB_TOKEN }}"}}'
  uses: docker/bake-action@v7
  ...
```

`cd.yaml`'s plain `docker compose ... build` needs the same `env:` on its
build step.

Use a token scoped as narrowly as possible - a fine-grained GitHub PAT with
read-only Contents access to just the private package repositories a project
actually needs, not a broad personal token. Org-level secrets are the more
maintainable default: set once, every project derived from this template
picks it up without repeating the setup.
