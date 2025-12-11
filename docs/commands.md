# Command Reference

Every command. Every flag. No fluff.

## Global Options

These work on any command:

| Flag | Description |
|------|-------------|
| `--help` | Show help for any command |
| `--version` | Show CLI version |
| `--json` | Force JSON output (overrides `--format`) |

## Authentication Commands

### `sherlock auth login`

Authenticate and store your Personal Access Token.

```bash
sherlock auth login --api-url https://api.covertlabs.io
```

| Flag | Description | Default |
|------|-------------|---------|
| `--api-url, -a` | API base URL | Stored value or `http://localhost:8000` |
| `--token, -t` | PAT (skips browser flow) | — |
| `--login-url` | Override browser login URL | Auto-detected |

**Examples:**

```bash
# Interactive (opens browser)
sherlock auth login --api-url https://api.covertlabs.io

# Non-interactive (CI/CD)
sherlock auth login --api-url https://api.covertlabs.io --token "$MY_TOKEN"

# Custom login URL
sherlock auth login --login-url https://app.covertlabs.io/cli/token
```

### `sherlock auth status`

Check your authentication status.

```bash
sherlock auth status
```

Shows:
- Whether a token is stored
- Current API URL
- Config file location

### `sherlock auth logout`

Remove stored credentials.

```bash
sherlock auth logout
```

Clears your token from the keychain and config file.

---

## Search Commands

All search commands share these common flags:

| Flag | Description | Default |
|------|-------------|---------|
| `--limit, -l` | Results per page (1-100) | `20` |
| `--format, -f` | Output format: `table`, `json`, `csv` | `table` |
| `--cursor, -c` | Pagination cursor for next page | — |

### `sherlock search email <email>`

Find victims by email address.

```bash
sherlock search email ceo@acme.com --limit 50
```

**Arguments:**
- `email` — Full email address to search

**Examples:**

```bash
sherlock search email admin@company.com
sherlock search email user@gmail.com --format json
sherlock search email test@example.org --limit 100 --format csv > results.csv
```

### `sherlock search domain <domain>`

Find victims by domain.

```bash
sherlock search domain acme.com --limit 20
```

**Arguments:**
- `domain` — Full domain name (e.g., `acme.com`, not `acme`)

**Examples:**

```bash
sherlock search domain company.com
sherlock search domain gmail.com --format json | jq '.results | length'
sherlock search domain yahoo.com --limit 50
```

**Note:** The API validates domain format. `gmail` will fail; `gmail.com` works.

### `sherlock search ip <ip>`

Find victims by IP address.

```bash
sherlock search ip 203.0.113.50
```

**Arguments:**
- `ip` — IPv4 address

**Examples:**

```bash
sherlock search ip 8.8.8.8
sherlock search ip 192.168.1.1 --format json
sherlock search ip 10.0.0.1 --limit 100
```

### `sherlock search username <username>`

Find victims by username.

```bash
sherlock search username admin
```

**Arguments:**
- `username` — Username to search (not email)

**Examples:**

```bash
sherlock search username administrator
sherlock search username jsmith --format json
sherlock search username root --limit 50
```

### `sherlock search password <password>`

Find victims by password.

```bash
sherlock search password "Summer2024!"
```

**Arguments:**
- `password` — Password string to search

**Examples:**

```bash
sherlock search password "Password123"
sherlock search password "qwerty" --format json
sherlock search password "company2024" --limit 100
```

**Note:** Password search may be restricted based on your account permissions.

### `sherlock search country <code>`

Find victims by country code.

```bash
sherlock search country US
```

**Arguments:**
- `code` — ISO 3166-1 alpha-2 country code (e.g., `US`, `GB`, `DE`, `FR`)

**Examples:**

```bash
sherlock search country US --limit 50
sherlock search country GB --format json
sherlock search country RU --format csv > russian_victims.csv
```

### `sherlock search stealer <family>`

Find victims by stealer malware family.

```bash
sherlock search stealer redline
```

**Arguments:**
- `family` — Malware family name (case-insensitive)

**Common stealer families:**
- `redline` — RedLine Stealer
- `vidar` — Vidar Stealer
- `raccoon` — Raccoon Stealer
- `azorult` — AZORult
- `meta` — META Stealer
- `lumma` — Lumma Stealer
- `stealc` — StealC

**Examples:**

```bash
sherlock search stealer redline --limit 100
sherlock search stealer vidar --format json
sherlock search stealer raccoon --format csv
```

### `sherlock search text <query>`

Full-text search across all fields.

```bash
sherlock search text "vpn credentials"
```

**Arguments:**
- `query` — Search term or phrase

**Examples:**

```bash
sherlock search text banking
sherlock search text "corporate vpn" --format json
sherlock search text "slack token" --limit 50
```

Use this when you're not sure which field contains what you're looking for.

---

## Victim Commands

Victim commands retrieve detailed data for a specific compromised machine.

Common flags for victim commands:

| Flag | Description | Default |
|------|-------------|---------|
| `--format, -f` | Output format: `table`, `json`, `csv` | `table` |
| `--limit, -l` | Results per page (1-100) | `20` |
| `--offset` | Pagination offset | `0` |

### `sherlock victim profile <victim_id>`

Get detailed profile information for a victim.

```bash
sherlock victim profile abc123def456
```

**Arguments:**
- `victim_id` — Unique victim identifier (from search results)

**Flags:**

| Flag | Description |
|------|-------------|
| `--include, -i` | Extra fields to include (comma-separated) |
| `--format, -f` | Output format |

**Include options:**
- `emails` — Associated email addresses
- `domains` — Domains found in credentials
- `usernames` — Usernames found

**Examples:**

```bash
sherlock victim profile abc123
sherlock victim profile abc123 --include emails,domains
sherlock victim profile abc123 --include emails,domains,usernames --format json
```

**Output fields:**
- `victim_id` — Unique identifier
- `folder_name` — Original log folder name
- `ip_address` — Victim's IP at time of infection
- `country_code` — Geolocation
- `stealer_type` — Malware family
- `created_at` — When the log was processed
- `emails` — (if included) List of email addresses
- `domains` — (if included) List of domains
- `usernames` — (if included) List of usernames

### `sherlock victim credentials <victim_id>`

Get stored credentials for a victim.

```bash
sherlock victim credentials abc123def456
```

**Arguments:**
- `victim_id` — Unique victim identifier

**Flags:**

| Flag | Description | Default |
|------|-------------|---------|
| `--domain, -d` | Filter by domain | — |
| `--limit, -l` | Results per page | `20` |
| `--offset` | Pagination offset | `0` |
| `--format, -f` | Output format | `table` |

**Examples:**

```bash
sherlock victim credentials abc123
sherlock victim credentials abc123 --domain gmail.com
sherlock victim credentials abc123 --domain slack.com --format json
sherlock victim credentials abc123 --limit 100 --format csv > creds.csv
```

**Output fields:**
- `domain` — Website domain
- `url` — Full URL
- `username` — Stored username
- `password` — Stored password

### `sherlock victim cookies <victim_id>`

Get browser cookies for a victim.

```bash
sherlock victim cookies abc123def456
```

**Arguments:**
- `victim_id` — Unique victim identifier

**Flags:**

| Flag | Description | Default |
|------|-------------|---------|
| `--domain, -d` | Filter by domain | — |
| `--limit, -l` | Results per page | `20` |
| `--offset` | Pagination offset | `0` |
| `--format, -f` | Output format | `table` |

**Examples:**

```bash
sherlock victim cookies abc123
sherlock victim cookies abc123 --domain slack.com
sherlock victim cookies abc123 --domain google.com --format json
sherlock victim cookies abc123 --limit 50 --format csv
```

**Output fields:**
- `domain` — Cookie domain
- `name` — Cookie name
- `value` — Cookie value (may be truncated in table view)

### `sherlock victim history <victim_id>`

Get browser history for a victim.

```bash
sherlock victim history abc123def456
```

**Arguments:**
- `victim_id` — Unique victim identifier

**Flags:**

| Flag | Description | Default |
|------|-------------|---------|
| `--search, -s` | Filter history entries | — |
| `--limit, -l` | Results per page | `20` |
| `--offset` | Pagination offset | `0` |
| `--format, -f` | Output format | `table` |

**Examples:**

```bash
sherlock victim history abc123
sherlock victim history abc123 --search amazon
sherlock victim history abc123 --search banking --format json
sherlock victim history abc123 --limit 100 --format csv
```

**Output fields:**
- `url` — Visited URL
- `title` — Page title
- `visited_at` — Timestamp (if available)

---

## Pagination

### Search Pagination (Cursor-based)

Search results use cursor-based pagination. When more results are available, the response includes a `next_cursor` value.

```bash
# First page
sherlock search domain acme.com --limit 20

# Next page (use the cursor from previous response)
sherlock search domain acme.com --limit 20 --cursor "eyJzb3J0IjpbMTcz..."
```

### Victim Pagination (Offset-based)

Victim artifact commands use offset-based pagination.

```bash
# First page
sherlock victim credentials abc123 --limit 20

# Next page
sherlock victim credentials abc123 --limit 20 --offset 20

# Third page
sherlock victim credentials abc123 --limit 20 --offset 40
```

---

## Output Formats

### Table (default)

Human-readable tables with colors and formatting. Best for interactive use.

```bash
sherlock search domain acme.com
```

### JSON

Machine-readable JSON. Pipe to `jq` for processing.

```bash
sherlock search domain acme.com --format json
sherlock search domain acme.com --format json | jq '.results[0]'
```

### CSV

Comma-separated values. Import into spreadsheets or databases.

```bash
sherlock search domain acme.com --format csv > results.csv
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | General error |
| `2` | Authentication error |

---

## Examples: Real Workflows

**Incident Response: Find all compromised accounts for your domain**

```bash
sherlock search domain mycompany.com --limit 100 --format json > compromised.json
```

**Red Team: Extract credentials for a target**

```bash
sherlock search domain target.com --format json | \
  jq -r '.results[].victim_id' | \
  xargs -I {} sherlock victim credentials {} --format json
```

**Threat Intel: Track a stealer family**

```bash
sherlock search stealer lumma --limit 100 --format csv > lumma_victims.csv
```

**Monitoring: Check for new compromises (scripted)**

```bash
#!/bin/bash
sherlock search domain mycompany.com --format json | \
  jq -r '.results[] | "\(.victim_id) \(.created_at)"'
```
