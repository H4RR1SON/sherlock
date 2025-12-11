# Configuration

Sherlock is designed to work out of the box. But when you need to customize behavior—CI/CD pipelines, corporate proxies, debugging—here's how.

## Environment Variables

### `SHERLOCK_API_URL`

The base URL for all API requests.

```bash
export SHERLOCK_API_URL="https://api.covertlabs.io"
```

**Default:** Stored value from `sherlock auth login`, or `http://localhost:8000`

**When to use:**
- CI/CD pipelines where you don't want to run `auth login`
- Switching between environments (staging, production)
- Docker containers

### `SHERLOCK_API_TIMEOUT`

Request timeout in milliseconds.

```bash
export SHERLOCK_API_TIMEOUT="60000"  # 60 seconds
```

**Default:** `30000` (30 seconds)

**When to use:**
- Slow network connections
- Large result sets that take longer to process
- Debugging timeout issues

### `SHERLOCK_LOGIN_URL`

The browser URL for generating CLI tokens.

```bash
export SHERLOCK_LOGIN_URL="https://app.covertlabs.io/cli/token"
```

**Default:** Auto-detected based on API URL

**When to use:**
- Custom deployment where the login page is on a different domain
- Testing against local development servers

### `SHERLOCK_DEBUG`

Enable debug output for troubleshooting.

```bash
export SHERLOCK_DEBUG="true"
```

**Default:** `false`

**When to use:**
- Debugging API issues
- Reporting bugs (include debug output in your report)

## Configuration File

After running `sherlock auth login`, Sherlock stores your configuration locally:

| Platform | Location |
|----------|----------|
| macOS | `~/Library/Preferences/sherlock-nodejs/config.json` |
| Linux | `~/.config/sherlock/config.json` |
| Windows | `%APPDATA%/sherlock/config.json` |

**Stored values:**
- `api_url` — Your configured API URL

**Not stored in config file:**
- Tokens (stored in OS keychain when possible)

You generally don't need to edit this file directly. Use `sherlock auth login` to update settings.

## Output Formats

All search and victim commands support three output formats:

### Table (default)

```bash
sherlock search domain acme.com
sherlock search domain acme.com --format table
```

Pretty-printed tables with:
- Color-coded columns
- Truncated long values
- Summary statistics

Best for: Interactive terminal use, quick lookups

### JSON

```bash
sherlock search domain acme.com --format json
```

Raw JSON output. Includes all fields, no truncation.

Best for:
- Piping to `jq` for processing
- Scripting and automation
- Integration with other tools

**Example with jq:**

```bash
# Get just victim IDs
sherlock search domain acme.com --format json | jq -r '.results[].victim_id'

# Count results
sherlock search domain acme.com --format json | jq '.total'

# Filter by country
sherlock search domain acme.com --format json | jq '.results[] | select(.country_code == "US")'
```

### CSV

```bash
sherlock search domain acme.com --format csv
```

Comma-separated values with header row.

Best for:
- Importing into Excel/Google Sheets
- Database imports
- Reporting

**Example:**

```bash
sherlock search domain acme.com --format csv > results.csv
```

## Precedence

Configuration is resolved in this order (highest priority first):

1. **Command-line flags** (`--api-url`, `--format`, etc.)
2. **Environment variables** (`SHERLOCK_API_URL`, etc.)
3. **Stored configuration** (from `sherlock auth login`)
4. **Defaults**

This means you can override any stored setting with an environment variable, and override that with a flag.

## CI/CD Configuration

For automated pipelines, you typically:

### GitHub Actions

```yaml
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - run: npm install -g @covertlabs/sherlock
      
      - run: sherlock auth login --api-url https://api.covertlabs.io --token "${{ secrets.SHERLOCK_TOKEN }}"
      
      - run: sherlock search domain mycompany.com --format json > results.json
```

### GitLab CI

```yaml
scan:
  image: node:20
  script:
    - npm install -g @covertlabs/sherlock
    - sherlock auth login --api-url https://api.covertlabs.io --token "$SHERLOCK_TOKEN"
    - sherlock search domain mycompany.com --format json > results.json
  artifacts:
    paths:
      - results.json
```

### Environment-based (no login)

If you prefer environment variables over `auth login`:

```bash
export SHERLOCK_API_URL="https://api.covertlabs.io"
sherlock auth login --token "$SHERLOCK_TOKEN"
sherlock search domain mycompany.com
```

## Docker

```dockerfile
FROM node:20-slim

RUN npm install -g @covertlabs/sherlock

ENV SHERLOCK_API_URL="https://api.covertlabs.io"

# Token should be passed at runtime, not baked into image
# docker run -e SHERLOCK_TOKEN=xxx myimage sherlock auth login --token "$SHERLOCK_TOKEN"
```

## Shell Aliases

Add to your `~/.bashrc` or `~/.zshrc`:

```bash
# Quick domain search
alias sdom='sherlock search domain'

# Quick email search with JSON output
alias semail='sherlock search email --format json'

# Quick victim lookup
alias svic='sherlock victim profile'

# Search and open in jq
sjq() {
  sherlock search "$@" --format json | jq
}
```

## Proxy Configuration

Sherlock uses axios for HTTP requests, which respects standard proxy environment variables:

```bash
export HTTP_PROXY="http://proxy.corp.com:8080"
export HTTPS_PROXY="http://proxy.corp.com:8080"
export NO_PROXY="localhost,127.0.0.1"
```

For authenticated proxies:

```bash
export HTTPS_PROXY="http://user:password@proxy.corp.com:8080"
```

---

Questions about configuration? Check [Troubleshooting](troubleshooting.md) or reach out to support.
