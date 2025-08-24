# NS8 OPNForm Module

[![NethServer 8](https://img.shields.io/badge/NethServer-8-blue)](https://github.com/NethServer/ns8-core)

A NethServer 8 module that provides [OPNForm](https://opnform.com/), a powerful form builder that allows you to create beautiful forms with ease. OPNForm is designed to be user-friendly, customizable, and integrates seamlessly with various services.

## Features

- **Easy Form Creation**: Build forms with a drag-and-drop interface
- **Customizable Themes**: Customize the look and feel of your forms
- **Data Collection**: Collect and manage form responses efficiently
- **Integration Ready**: Connect with external services and APIs
- **Secure**: Built-in security features to protect your data

## Prerequisites

- NethServer 8 cluster
- Sufficient resources for containerized services (PostgreSQL, Redis, Nginx)

## Installation

Instantiate the module with:

```bash
add-module ghcr.io/geniusdynamics/opnform:latest 1
```

The output of the command will return the instance name. Output example:

```json
{
  "module_id": "opnform1",
  "image_name": "opnform",
  "image_url": "ghcr.io/geniusdynamics/opnform:latest"
}
```

## Configuration

Let's assume that the OPNForm instance is named `opnform1`.

Launch `configure-module`, by setting the following parameters:
- `host`: a fully qualified domain name for the application
- `http2https`: enable or disable HTTP to HTTPS redirection (true/false)
- `lets_encrypt`: enable or disable Let's Encrypt certificate (true/false)

Example:

```bash
api-cli run configure-module --agent module/opnform1 --data - <<EOF
{
  "host": "opnform.domain.com",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

The above command will:
- Start and configure the OPNForm instance
- Configure a virtual host for Traefik to access the instance

## Get Configuration

You can retrieve the current configuration with:

```bash
api-cli run get-configuration --agent module/opnform1
```

## Usage

Once configured, access your OPNForm instance at the specified host domain. The application will be available through the configured Traefik virtual host.

### Default Access

- **URL**: `https://your-domain.com` (as configured in the host parameter)
- **Admin Panel**: Access through the OPNForm web interface

## Maintenance

### Update

To update the instance to the latest version:

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/opnform:latest","instances":["opnform1"],"force":true}'
```

### Uninstall

To uninstall the instance:

```bash
remove-module --no-preserve opnform1
```

## Smarthost Configuration

Some configuration settings, like the smarthost setup, are not part of the `configure-module` action input: they are discovered by looking at some Redis keys.

To ensure the module is always up-to-date with the centralized [smarthost setup](https://geniusdynamics.github.io/ns8-core/core/smarthost/):

1. Every time OPNForm starts, the command `bin/discover-smarthost` runs and refreshes the `state/smarthost.env` file with fresh values from Redis
2. If smarthost setup is changed when OPNForm is already running, the event handler `events/smarthost-changed/10reload_services` restarts the main module service

See also the `systemd/user/opnform.service` file for more details.

> **Note**: This setting discovery mechanism is a standard NethServer 8 pattern and can be customized based on your specific needs.

## Debugging

Here are some useful commands for debugging the OPNForm module:

### Environment Variables

The module runs under an agent that initializes many environment variables (stored in `/home/opnform1/.config/state`). To verify them on the root terminal:

```bash
runagent -m opnform1 env
```

### Run Agent Shell

To become the runagent user for testing scripts and access all environment variables:

```bash
runagent -m opnform1
```

The PATH will be updated to include module-specific directories:
```bash
echo $PATH
# /home/opnform1/.config/bin:/usr/local/agent/pyenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/
```

### Container Debugging

To debug containers or inspect the environment inside them:

```bash
runagent -m opnform1
podman ps
```

Example output:
```
CONTAINER ID  IMAGE                                      COMMAND               CREATED        STATUS        PORTS                    NAMES
d292c6ff28e9  localhost/podman-pause:4.6.1-1702418000                          9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  80b8de25945f-infra
d8df02bf6f4a  docker.io/library/postgres:15.5-alpine3.19          --character-set-s...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  postgresql-app
9e58e5bd676f  docker.io/library/nginx:stable-alpine3.17  nginx -g daemon o...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  opnform-app
```

### Inspect Container Environment

To see environment variables inside a specific container:

```bash
podman exec opnform-app env
```

### Access Container Shell

To run a shell inside the container:

```bash
podman exec -ti opnform-app sh
```
## Testing

Test the module using the `test-module.sh` script:

```bash
./test-module.sh <NODE_ADDR> ghcr.io/geniusdynamics/opnform:latest
```

The tests are implemented using [Robot Framework](https://robotframework.org/), which provides automated testing capabilities for the module's functionality.

## UI Translation

The user interface supports multiple languages and is translated using [Weblate](https://hosted.weblate.org/projects/ns8/).

### Setting up Translation Process

To set up the translation process for your fork:

1. Add the [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
2. Add your repository to [hosted.weblate.org](https://hosted.weblate.org) or ask a NethServer developer to add it to the ns8 Weblate project

### Supported Languages

- English (en)
- German (de)
- Spanish (es)
- Basque (eu)
- Italian (it)
- Portuguese (pt)
- Brazilian Portuguese (pt_BR)

## Contributing

Contributions are welcome! Please feel free to submit issues, feature requests, or pull requests to improve this module.

## License

This project is licensed under the terms specified in the LICENSE file.

## Support

For support and questions:
- Check the [NethServer 8 documentation](https://nethserver.github.io/ns8-core/)
- Open an issue on the [GitHub repository](https://github.com/NethServer/ns8-opnform)
- Visit the [NethServer community forums](https://community.nethserver.org/)
