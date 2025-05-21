# Use Cases and Examples

This document provides examples of how the Keycloak-Wazuh integration can be used in real-world scenarios.

## Security Monitoring Use Cases

### 1. Detecting Brute Force Attacks

Brute force attacks involve repeated login attempts to guess user credentials. The integration can detect these attacks using correlation rules.

**Scenario:**
An attacker attempts to log in to a user account multiple times with different passwords.

**Detection:**
The rule `110101` detects multiple failed login attempts from the same IP address within a short timeframe:

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

**Response:**
- Alert security team
- Block the IP address temporarily
- Increase authentication requirements for the targeted user

### 2. Detecting Password Spraying

Password spraying is a type of brute force attack where an attacker tries a few common passwords against many user accounts.

**Scenario:**
An attacker attempts to log in to multiple user accounts using the same password.

**Detection:**
The rule `110102` detects multiple failed login attempts from the same IP address targeting different users:

```xml
<rule id="110102" level="12" frequency="5" timeframe="300">
    <if_matched_sid>110001</if_matched_sid>
    <same_srcip/>
    <different_user/>
    <description>Keycloak: Brute force attack on multiple users from IP $(ipAddress)</description>
    <group>authentication_failures,brute_force,attack,</group>
    <mitre>
        <id>T1110.003</id> <!-- Brute Force: Password Spraying -->
    </mitre>
</rule>
```

**Response:**
- Alert security team
- Block the IP address
- Review password policies
- Consider implementing additional authentication factors

### 3. Detecting Suspicious Login Locations

Detecting logins from unusual geographic locations can help identify compromised accounts.

**Scenario:**
A user who normally logs in from the United States suddenly logs in from a different country.

**Detection:**
The rule `110150` detects logins from new geographic locations:

```xml
<rule id="110150" level="7">
    <if_matched_sid>110001</if_matched_sid>
    <field name="type">LOGIN</field>
    <different_srcgeoip/>
    <description>Keycloak: Login from new geographic location - IP: $(ipAddress)</description>
    <group>authentication_success,unusual_location,</group>
</rule>
```

**Response:**
- Alert security team
- Notify the user
- Consider requiring additional authentication
- Monitor the account for suspicious activity

### 4. Detecting Account Sharing

Detecting multiple logins from different locations for the same user can help identify account sharing.

**Scenario:**
A user account is being used by multiple people from different locations.

**Detection:**
The rule `110210` detects logins from multiple IP addresses for the same user:

```xml
<rule id="110210" level="8" frequency="4" timeframe="300">
    <if_matched_sid>110001</if_matched_sid>
    <field name="type">LOGIN</field>
    <same_user/>
    <different_srcip/>
    <description>Keycloak: User $(userId) logged in from multiple IP addresses</description>
    <group>authentication_success,account_sharing,suspicious_activity,</group>
    <mitre>
        <id>T1078</id> <!-- Valid Accounts -->
    </mitre>
</rule>
```

**Response:**
- Alert security team
- Contact the user
- Review account usage policies
- Consider implementing IP restrictions

## Compliance Monitoring Use Cases

### 1. Tracking Administrative Changes

Tracking administrative changes is important for compliance with regulations like SOX, HIPAA, and GDPR.

**Scenario:**
An administrator makes changes to the realm configuration.

**Detection:**
The rule `110005` detects realm configuration changes:

```xml
<rule id="110005" level="4">
    <if_sid>110001</if_sid>
    <field name="type">REALM-UPDATE</field>
    <description>Keycloak: User $(userId) updated Realm configuration</description>
    <options>no_full_log</options>
    <group>configuration_changes,system_changes,</group>
</rule>
```

**Response:**
- Log the change for audit purposes
- Review the change for compliance
- Document the change in the compliance register

### 2. Monitoring Client Registration

Monitoring client registration is important for maintaining control over which applications can access your identity provider.

**Scenario:**
A new client is registered in Keycloak.

**Detection:**
The rule `110180` detects client registration:

```xml
<rule id="110180" level="8">
    <if_matched_sid>110001</if_matched_sid>
    <field name="type">CLIENT-CREATE</field>
    <description>Keycloak: New client $(clientId) registered by user $(userId)</description>
    <group>configuration_changes,</group>
</rule>
```

**Response:**
- Alert security team
- Review the client registration
- Verify that the client is authorized
- Document the client for compliance purposes

## Integration Examples

### 1. Integration with SIEM

The Keycloak-Wazuh integration can be further integrated with other SIEM solutions for centralized security monitoring.

**Example:**
Forward Wazuh alerts to Splunk or Elasticsearch for correlation with other security events.

**Implementation:**
Configure Wazuh to forward alerts to your SIEM solution using the appropriate integration module.

### 2. Integration with Incident Response

The integration can be used to trigger automated incident response actions.

**Example:**
Automatically block an IP address after detecting a brute force attack.

**Implementation:**
Configure Wazuh to execute a custom script when a brute force attack is detected. The script can update firewall rules to block the offending IP address.

### 3. Integration with Compliance Reporting

The integration can be used to generate compliance reports.

**Example:**
Generate a weekly report of administrative changes for compliance review.

**Implementation:**
Configure Wazuh to generate a custom report of administrative changes and send it to the compliance team.

## Custom Rule Examples

### 1. Detecting After-Hours Logins

Detecting logins outside of business hours can help identify suspicious activity.

**Rule:**
```xml
<rule id="110140" level="7">
    <if_matched_sid>110001</if_matched_sid>
    <field name="type">LOGIN</field>
    <time>22:00-06:00</time>
    <description>Keycloak: Login outside business hours from IP $(ipAddress)</description>
    <group>policy_violation,suspicious_time,</group>
</rule>
```

### 2. Detecting Unusual Scope Requests

Detecting requests for unusual scopes can help identify potential data exfiltration attempts.

**Rule:**
```xml
<rule id="110270" level="7">
    <if_sid>110001</if_sid>
    <field name="type">CODE_TO_TOKEN</field>
    <field name="details.scope">\.offline_access\.</field>
    <description>Keycloak: Offline access token requested for user $(userId) from IP $(ipAddress)</description>
    <group>authentication_success,suspicious_scope,</group>
</rule>
```

## Visualization Examples

The Wazuh Dashboard provides various visualization options for Keycloak events:

1. **Authentication Dashboard**: Visualize login success and failure rates over time.
2. **Security Events Dashboard**: Display security-related events such as brute force attacks and suspicious logins.
3. **Compliance Dashboard**: Show administrative changes and other compliance-related events.
4. **User Activity Dashboard**: Track user activity across the system.

These dashboards can be customized to meet your specific monitoring needs.