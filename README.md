# ns8-opnform

Create beautiful forms and share them anywhere. It's super fast, you don't need to know how to code

Module id: `opnform`. Images are pinned in `build-images.sh` and updated through reviewed changes.

## Install

```bash
add-module ghcr.io/geniusdynamics/opnform:latest 1
```

The command returns the instance id, for example `opnform1`. Use the id NS8 returns.

## Configure

```bash
api-cli run configure-module --agent module/opnform1 --data - <<'EOF'
{
  "host": "opnform.example.org",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

| Parameter | Required | Type | Description |
|---|---|---|---|
| `host` | yes | string | Host name for the application, like 'opnform.domain.org' |
| `lets_encrypt` | yes | boolean | Request a valid Let's Encrypt certificate. |
| `http2https` | yes | boolean | Redirect all the HTTP requests to HTTPS |

## Configuration output

`get-configuration` returns: `host`, `lets_encrypt`, `http2https`.

## Images

| Image | Version |
|---|---|
| `docker.io/postgres` | `18.7-alpine3.24` |
| `docker.io/jhumanj/opnform-client` | `2.8.4` |
| `docker.io/jhumanj/opnform-api` | `2.8.4` |
| `docker.io/redis` | `8.10.3` |
| `docker.io/nginx` | `1.31.8` |

## Network and access

- Reserves 2 TCP port(s) on the node. The web application is published through Traefik on the configured host name.
- Runs rootless.
- Declares these NS8 authorizations: `traefik@node:routeadm`.

## Backup

The following paths are included in the NS8 backup (`imageroot/etc/state-include.conf`):

- `state/database.env`
- `state/opnform-db.env`
- `state/opnform-api.env`
- `state/opnform.pg_dump`
- `volumes/opnform-logs`
- `volumes/opnform-storage`

## Services

- `nginx.service`
- `opnform-app.service`
- `opnform-ui.service`
- `opnform.service`
- `postgresql-app.service`
- `redis.service`

## Get the configuration

```bash
api-cli run get-configuration --agent module/opnform1
```

## Update and uninstall

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/opnform:latest","instances":["opnform1"],"force":true}'
remove-module --no-preserve opnform1
```

## Testing

Continuous integration:

- `Publish images` builds the module image on every push.
- `Test module` installs the image on a QEMU NS8 node and runs the Robot Framework suite in `tests/`. It runs after `Publish images` succeeds on `dev` and on `main`.
- `Build API JSON Schema docs` and the security scan run alongside.

The suite installs the module, configures a host name, checks that the web application answers through Traefik, takes UI screenshots and removes the module.

Run it locally with the NS8 standard infrastructure described in the [ns8-github-actions README](https://github.com/NethServer/ns8-github-actions/blob/v1/README.md#running-tests-locally).

## Open items

Found by the NS8 module standards check. These must be resolved before a stable release:

- metadata: documentation_url is a placeholder
- nginx.service: uses %S/state, kickstart uses %E/state
- opnform-app.service: uses %S/state, kickstart uses %E/state
- nginx.service: %S/state changed to %E/state (review the bind mounts that remain)
- opnform-app.service: %S/state changed to %E/state (review the bind mounts that remain)
- NGINX_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- OPNFORM_API_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- OPNFORM_CLIENT_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- POSTGRES_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- REDIS_IMAGE is used by the module but no image in the label provides it: leftover from the template?

## Upstream

Application documentation: https://docs.opnform.com/

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
