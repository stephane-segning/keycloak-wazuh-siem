# Keycloak Configuration Guide

This guide explains how to configure Keycloak to send events to Wazuh via Syslog.

## Overview

Keycloak can be configured to send events to external systems using event listeners. In this integration, we use the webhook-syslog provider to send events to Syslog-ng, which then forwards them to Wazuh.

## Prerequisites

- A running Keycloak instance
- The webhook-syslog provider installed
- Syslog-ng configured to receive events

## Installing the Webhook-Syslog Provider

The webhook-syslog provider is a Keycloak extension that sends events to a Syslog server. In our Docker setup, this is automatically installed, but if you're using your own Keycloak instance, you'll need to install it manually.

1. Download the provider JARs from the GitHub repository:
   ```bash
   wget -O keycloak-webhook-provider-core.jar "https://github.com/vymalo/keycloak-webhook/releases/download/v0.9.1/keycloak-webhook-provider-core-0.9.1-all.jar"
   wget -O keycloak-webhook-provider-syslog.jar "https://github.com/vymalo/keycloak-webhook/releases/download/v0.9.1/keycloak-webhook-provider-syslog-0.9.1-all.jar"
   ```

2. Copy the JARs to the Keycloak providers directory:
   ```bash
   cp keycloak-webhook-provider-*.jar /path/to/keycloak/providers/
   ```

3. Restart Keycloak to load the new providers.

## Configuring the Event Listener

Once the provider is installed, you need to configure your realm to use it:

1. Log in to the Keycloak Admin Console.

2. Select your realm from the dropdown in the top-left corner.

3. Go to **Realm Settings** > **Events**.

4. In the **Event Listeners** field, add `webhook-syslog` to the list.

5. Make sure **Save Events** is enabled.

6. Click **Save**.

## Environment Variables

The webhook-syslog provider is configured using environment variables. In our Docker setup, these are defined in the `docker-compose.yml` file:

```yaml
environment:
  WEBHOOK_SYSLOG_PROTOCOL: udp
  WEBHOOK_SYSLOG_HOSTNAME: keycloak
  WEBHOOK_SYSLOG_APP_NAME: keycloak_events
  WEBHOOK_SYSLOG_FACILITY: SYSLOG
  WEBHOOK_SYSLOG_SEVERITY: INFORMATIONAL
  WEBHOOK_SYSLOG_SERVER_HOSTNAME: syslog-ng
  WEBHOOK_SYSLOG_SERVER_PORT: 5514
  WEBHOOK_SYSLOG_MESSAGE_FORMAT: RFC_5424
```

If you're using your own Keycloak instance, you'll need to set these environment variables before starting Keycloak.

### Environment Variable Reference

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| `WEBHOOK_SYSLOG_PROTOCOL` | Protocol to use (udp/tcp) | udp | udp |
| `WEBHOOK_SYSLOG_HOSTNAME` | Hostname to include in Syslog messages | localhost | keycloak |
| `WEBHOOK_SYSLOG_APP_NAME` | Application name to include in Syslog messages | keycloak | keycloak_events |
| `WEBHOOK_SYSLOG_FACILITY` | Syslog facility to use | USER | SYSLOG |
| `WEBHOOK_SYSLOG_SEVERITY` | Syslog severity to use | INFORMATIONAL | INFORMATIONAL |
| `WEBHOOK_SYSLOG_SERVER_HOSTNAME` | Syslog server hostname | localhost | syslog-ng |
| `WEBHOOK_SYSLOG_SERVER_PORT` | Syslog server port | 514 | 5514 |
| `WEBHOOK_SYSLOG_MESSAGE_FORMAT` | Syslog message format (RFC_3164/RFC_5424) | RFC_5424 | RFC_5424 |

## Event Types

Keycloak generates various types of events that can be monitored:

### User Events

- `LOGIN` - User login
- `LOGIN_ERROR` - Failed login attempt
- `LOGOUT` - User logout
- `REGISTER` - User registration
- `UPDATE_PASSWORD` - Password update
- `UPDATE_PROFILE` - Profile update
- `VERIFY_EMAIL` - Email verification
- `REMOVE_TOTP` - TOTP removal
- `UPDATE_TOTP` - TOTP update
- `SEND_VERIFY_EMAIL` - Verification email sent
- `SEND_RESET_PASSWORD` - Password reset email sent

### Admin Events

- `CREATE` - Resource creation
- `UPDATE` - Resource update
- `DELETE` - Resource deletion
- `ACTION` - Action on a resource

## Testing the Configuration

To test that events are being sent correctly:

1. Log in to Keycloak with a test user.
2. Check the Syslog-ng logs:
   ```bash
   docker-compose logs syslog-ng
   ```
3. Check the Wazuh Manager logs:
   ```bash
   docker-compose logs wazuh.manager
   ```
4. Look for events in the Wazuh Dashboard.

## Troubleshooting

### No Events in Syslog

- Check that the webhook-syslog provider is installed correctly.
- Verify that the event listener is configured in your realm.
- Check the Keycloak logs for errors.
- Verify the environment variables are set correctly.

### Events in Syslog but Not in Wazuh

- Check that Syslog-ng is forwarding events to Wazuh.
- Verify the Wazuh decoder is configured correctly.
- Check the Wazuh Manager logs for decoding errors.

## Advanced Configuration

### Custom Event Filtering

You can configure which events are sent to Syslog by creating a custom event listener that extends the webhook-syslog provider. This requires Java development and is beyond the scope of this guide.

### High Availability

For high availability setups, you can configure multiple Syslog servers and use a load balancer to distribute the events.

## References

- [Keycloak Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/)
- [Keycloak Event Listener SPI](https://www.keycloak.org/docs/latest/server_development/#_events)
- [webhook-syslog Provider Documentation](https://github.com/vymalo/keycloak-webhook)