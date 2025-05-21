# Wazuh Configuration Guide

This guide explains how to configure Wazuh to process and analyze Keycloak events.

## Overview

Wazuh uses decoders to parse incoming log messages and rules to analyze the parsed data and generate alerts. In this integration, we use custom decoders and rules specifically designed for Keycloak events.

## Prerequisites

- A running Wazuh Manager
- Keycloak configured to send events to Syslog
- Syslog-ng configured to forward events to Wazuh

## Decoder Configuration

The decoder is responsible for parsing the JSON events sent by Keycloak. In our setup, we use a simple JSON decoder:

```xml
<decoder name="keycloak_json">
    <program_name>keycloak_events</program_name>
    <plugin_decoder>JSON_Decoder</plugin_decoder>
</decoder>
```

This decoder:
- Matches log messages with the program name `keycloak_events`
- Uses the built-in JSON decoder to parse the message content

### Installing the Decoder

1. Create a file named `local_decoder.xml` in the Wazuh decoders directory:
   ```bash
   vi /var/ossec/etc/decoders/local_decoder.xml
   ```

2. Add the decoder configuration:
   ```xml
   <decoder name="keycloak_json">
       <program_name>keycloak_events</program_name>
       <plugin_decoder>JSON_Decoder</plugin_decoder>
   </decoder>
   ```

3. Restart the Wazuh Manager:
   ```bash
   systemctl restart wazuh-manager
   ```

## Rule Configuration

Rules define conditions that trigger alerts when matched. Our setup includes rules for various Keycloak events, such as:

- Failed login attempts
- Brute force attacks
- Suspicious login locations
- Administrative changes

### Example Rules

#### Base Rule

```xml
<rule id="110001" level="3">
    <decoded_as>keycloak_json</decoded_as>
    <description>Keycloak event detected</description>
    <group>keycloak,authentication,</group>
</rule>
```

This is the base rule that matches all Keycloak events.

#### Failed Login Rule

```xml
<rule id="110100" level="5">
    <if_matched_sid>110001</if_matched_sid>
    <field name="type">LOGIN_ERROR</field>
    <description>Keycloak: Failed login attempt from IP $(ipAddress)</description>
    <group>authentication_failures,</group>
</rule>
```

This rule matches failed login attempts.

#### Brute Force Detection Rule

```xml
<rule id="110101" level="10" frequency="5" timeframe="300">
    <if_matched_sid>110001</if_matched_sid>
    <same_source_ip/>
    <description>Keycloak: Possible brute force attack from IP $(ipAddress) - multiple failed logins</description>
    <group>authentication_failures,brute_force,attack,</group>
    <mitre>
        <id>T1110</id> <!-- Brute Force -->
    </mitre>
</rule>
```

This rule detects brute force attacks by identifying multiple failed login attempts from the same IP address within a short timeframe.

### Installing the Rules

1. Create a file named `local_rules.xml` in the Wazuh rules directory:
   ```bash
   vi /var/ossec/etc/rules/local_rules.xml
   ```

2. Add the rule configurations (see the [complete rule set](../../config/wazuh/rules/local_rules.xml) for all available rules).

3. Restart the Wazuh Manager:
   ```bash
   systemctl restart wazuh-manager
   ```

## MITRE ATT&CK Framework Integration

Our rules include MITRE ATT&CK framework references to help you understand the tactics and techniques used by attackers. For example:

```xml
<mitre>
    <id>T1110</id> <!-- Brute Force -->
</mitre>
```

This indicates that the rule is detecting the MITRE ATT&CK technique T1110 (Brute Force).

## Custom Rule Development

You can create your own rules to detect specific events or patterns. Here's how:

1. Identify the event type or pattern you want to detect.
2. Create a rule that matches the appropriate fields in the JSON event.
3. Assign a unique ID to your rule (use a range that doesn't conflict with existing rules).
4. Set an appropriate alert level (0-15, with higher numbers indicating more severe alerts).
5. Add your rule to the `local_rules.xml` file.
6. Restart the Wazuh Manager.

### Rule Syntax

```xml
<rule id="RULE_ID" level="LEVEL">
    <if_sid>PARENT_RULE_ID</if_sid>
    <field name="FIELD_NAME">FIELD_VALUE</field>
    <description>DESCRIPTION</description>
    <group>GROUP1,GROUP2,</group>
    <mitre>
        <id>TECHNIQUE_ID</id>
    </mitre>
</rule>
```

### Alert Levels

- 0-3: Low severity (informational)
- 4-7: Medium severity (warning)
- 8-11: High severity (critical)
- 12-15: Very high severity (alert)

## Testing and Validation

To test your configuration:

1. Generate events in Keycloak (e.g., by attempting to log in).
2. Check the Wazuh Manager logs:
   ```bash
   tail -f /var/ossec/logs/alerts/alerts.log
   ```
3. Verify that the events are being decoded and that the rules are triggering as expected.
4. Check the Wazuh Dashboard for alerts.

## Troubleshooting

### Events Not Being Decoded

- Check that the program name in the logs matches the one in the decoder.
- Verify that the JSON format is correct.
- Check the Wazuh Manager logs for decoding errors.

### Rules Not Triggering

- Check that the field names and values in your rules match those in the events.
- Verify that the parent rule is triggering.
- Check the rule syntax for errors.

## Advanced Configuration

### Custom Fields

You can extract custom fields from the JSON events by modifying the decoder or using the built-in JSON decoder capabilities.

### Correlation Rules

You can create correlation rules that trigger based on multiple events or conditions. For example, you might want to alert on a successful login after multiple failed attempts.

### Response Actions

Wazuh can be configured to take automatic actions in response to alerts, such as blocking an IP address or sending a notification.

## References

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Wazuh Rule Syntax](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)