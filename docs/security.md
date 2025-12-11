# Security

You're working with sensitive data. Sherlock is built with that in mind.

## Token Security

### How Tokens Work

Sherlock uses Personal Access Tokens (PATs) for authentication. When you authenticate:

1. You sign in via SSO (WorkOS AuthKit)
2. A one-time PAT is generated
3. You paste it into the CLI
4. The CLI stores it securely

Every API request includes your token in the `Authorization` header. The server validates the token and logs the request against your account.

### Token Storage

Sherlock stores your token in the most secure location available on your system:

| Platform | Primary Storage | Security |
|----------|-----------------|----------|
| macOS | Keychain | Encrypted, requires user authentication |
| Windows | Credential Manager | Encrypted, tied to user account |
| Linux | Secret Service | GNOME Keyring or KWallet, encrypted |

**Fallback:** If keychain access fails (headless servers, containers), tokens are stored in `~/.config/sherlock/config.json`. This file is readable only by your user account, but it's not encrypted.

### Token Properties

- **Long-lived:** Tokens don't expire automatically (but you should rotate them)
- **Revocable:** Kill a token instantly from the dashboard
- **Auditable:** Every request is logged with the token ID
- **Scoped:** Tokens inherit your account's permissions

## Best Practices

### Do

- **Generate separate tokens per machine.** Laptop token, CI token, script token. If one is compromised, revoke just that one.

- **Rotate tokens periodically.** Quarterly is reasonable. Monthly if you're paranoid.

- **Use environment variables in CI/CD.** Store tokens in your CI's secrets manager, not in code.

- **Revoke tokens you don't need.** Left a job? Decommissioned a server? Revoke those tokens.

- **Check `sherlock auth status` regularly.** Know what's configured on your machine.

### Don't

- **Commit tokens to version control.** Ever. Use `.gitignore` and pre-commit hooks.

- **Share tokens between team members.** Each person should have their own account and tokens.

- **Paste tokens in Slack, email, or tickets.** If you need to share output, redact the token first.

- **Store tokens in plaintext files.** If you must use the config file fallback, ensure proper file permissions.

- **Screenshot tokens.** Screenshots get synced to cloud services, shared accidentally, etc.

## Handling Sensitive Output

Sherlock retrieves sensitive data: passwords, cookies, session tokens. Handle output carefully.

### Terminal Output

By default, passwords are partially masked in table view:
```
Pa******
```

JSON and CSV output includes full values. Be careful where you pipe that data.

### File Output

```bash
# This file contains plaintext passwords
sherlock victim credentials abc123 --format csv > creds.csv

# Secure it
chmod 600 creds.csv

# Delete when done
rm -P creds.csv  # macOS secure delete
shred -u creds.csv  # Linux secure delete
```

### Logging

If you're logging Sherlock output:
- Don't log JSON/CSV output to shared log systems
- Redact sensitive fields before logging
- Consider using `--format table` for logs (passwords are masked)

## Incident Response: Compromised Token

If you suspect a token is compromised:

1. **Revoke immediately.** Log into the Covertlabs dashboard → API Tokens → Revoke.

2. **Check audit logs.** Review what queries were made with that token.

3. **Generate a new token.** Run `sherlock auth login` again.

4. **Update any scripts/CI.** Replace the old token everywhere it was used.

5. **Investigate the source.** How was it exposed? Fix the root cause.

## Network Security

### TLS

All API communication uses HTTPS (TLS 1.2+). The CLI validates certificates by default.

### Proxy Support

If you're behind a corporate proxy:

```bash
export HTTPS_PROXY="http://proxy.corp.com:8080"
```

For authenticated proxies:

```bash
export HTTPS_PROXY="http://user:password@proxy.corp.com:8080"
```

**Warning:** Proxy credentials in environment variables may be logged. Use a proxy that supports certificate-based auth if possible.

### Firewall Rules

Sherlock needs outbound HTTPS (port 443) to:
- `api.covertlabs.io` — API requests
- `app.covertlabs.io` — Token generation (browser flow)

## Data Handling

### What Sherlock Stores Locally

| Data | Location | Encryption |
|------|----------|------------|
| API Token | OS Keychain | Yes |
| API URL | Config file | No |
| Query results | Not stored | — |

Sherlock does **not** cache query results locally. Every search hits the API.

### What the API Logs

The Covertlabs API logs:
- Timestamp
- Token ID (not the token itself)
- Query type and parameters
- Response size
- Client IP

This enables:
- Usage auditing
- Abuse detection
- Debugging support requests

## Compliance Considerations

### Data Access

Access to stealer log data should be:
- **Need-to-know:** Only users who require it for their job
- **Logged:** All access is auditable
- **Time-limited:** Revoke access when no longer needed

### Data Handling

When you export data from Sherlock:
- Store it securely (encrypted at rest)
- Limit retention (delete when no longer needed)
- Control access (don't share broadly)
- Document usage (for compliance audits)

### Regulatory Notes

Depending on your jurisdiction and use case, stealer log data may be subject to:
- GDPR (if processing EU resident data)
- CCPA (if processing California resident data)
- Industry-specific regulations (PCI-DSS, HIPAA, etc.)

Consult your legal/compliance team for guidance specific to your situation.

## Reporting Security Issues

Found a vulnerability in Sherlock or the Covertlabs platform?

**Email:** security@covertlabs.io

Please include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Your contact information

We take security reports seriously and will respond within 48 hours.

---

Questions about security? Contact support@covertlabs.io or your account manager.
