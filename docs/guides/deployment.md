# Deployment Guide

This guide explains how to deploy the Keycloak-Wazuh integration using Docker Compose.

## Prerequisites

- Docker and Docker Compose installed
- Git installed
- Basic knowledge of Keycloak and Wazuh
- 4GB+ of RAM available for the containers

## Quick Start

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/keycloak-wazuh-monitoring.git
   cd keycloak-wazuh-monitoring
   ```

2. Generate SSL certificates for Wazuh components:
   ```bash
   cd deployment/docker
   docker-compose -f ../../templates/generate-indexer-certs.yml up
   ```

3. Start the environment:
   ```bash
   docker-compose up -d
   ```

4. Access the components:
   - Keycloak: http://localhost:9000 (admin/password)
   - Wazuh Dashboard: https://localhost:5601 (admin/SecretPassword)

## Detailed Setup

### Environment Configuration

You can customize the deployment by creating a `.env` file in the `deployment/docker` directory. Copy the example file and modify as needed:

```bash
cp .env.example .env
```

Edit the `.env` file to change:
- Passwords
- Ports
- Memory allocation
- Other configuration options

### SSL Certificates

The integration uses SSL certificates for secure communication between components. The certificates are generated using the Wazuh certificate generator.

If you need to regenerate the certificates:

1. Remove the existing certificates:
   ```bash
   rm -rf config/wazuh_indexer_ssl_certs/*
   ```

2. Run the certificate generator:
   ```bash
   docker-compose -f templates/generate-indexer-certs.yml up
   ```

### Keycloak Configuration

The default setup includes a preconfigured realm with the webhook-syslog provider enabled. If you want to use your own realm:

1. Export your realm configuration from your existing Keycloak instance
2. Place the exported JSON file in `config/keycloak/realm/`
3. Update the `docker-compose.yml` file to use your realm file

Make sure your realm has the webhook-syslog provider enabled in the events listeners.

### Wazuh Configuration

The default setup includes custom decoders and rules for Keycloak events. If you want to customize these:

1. Edit the files in `config/wazuh/decoders/` and `config/wazuh/rules/`
2. Restart the Wazuh Manager:
   ```bash
   docker-compose restart wazuh.manager
   ```

## Troubleshooting

### Logs

You can check the logs of each component:

```bash
# Keycloak logs
docker-compose logs keycloak

# Wazuh Manager logs
docker-compose logs wazuh.manager

# Syslog-ng logs
docker-compose logs syslog-ng
```

### Common Issues

#### Keycloak not sending events

- Check that the webhook-syslog provider is enabled in your realm
- Check the Keycloak logs for errors
- Verify the Syslog-ng configuration

#### Wazuh not receiving events

- Check the Syslog-ng logs
- Verify the Wazuh Manager configuration
- Check the network connectivity between containers

#### Wazuh Dashboard not showing events

- Check that the Wazuh Indexer is running
- Verify the Wazuh Dashboard configuration
- Check the Wazuh Manager logs for indexing errors

## Production Deployment

For production deployments, consider:

- Using external volumes for persistent data
- Setting up a reverse proxy with proper TLS termination
- Implementing proper password management
- Setting up regular backups
- Monitoring the health of the containers

## Next Steps

After deployment, you can:

- [Configure Keycloak](keycloak-configuration.md) for your specific use case
- [Customize Wazuh rules](wazuh-configuration.md) for your security requirements
- Explore the [example use cases](../examples/README.md)