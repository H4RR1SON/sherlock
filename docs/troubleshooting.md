# Troubleshooting

Something broke. Let's fix it.

## Authentication Errors

### "Not authenticated. Run 'sherlock auth login' first"

**Cause:** No token stored, or token was cleared.

**Fix:**
```bash
sherlock auth login --api-url https://api.covertlabs.io
```

### 401 Unauthorized / "Invalid or expired token"

**Causes:**
- Token was revoked server-side
- Token was corrupted
- Wrong API URL (token valid for different environment)

**Fix:**
```bash
# Check current status
sherlock auth status

# Re-authenticate
sherlock auth login --api-url https://api.covertlabs.io
```

### 403 Forbidden / "Insufficient permissions"

**Cause:** Your account doesn't have permission for this operation.

**Fix:** Contact your account administrator or support@covertlabs.io to request access.

### Token not persisting between sessions

**Causes:**
- Keychain access denied
- Config directory not writable
- Running in a restricted environment (container, CI)

**Fix:**
```bash
# Check if config directory exists and is writable
ls -la ~/.config/sherlock/

# If missing, create it
mkdir -p ~/.config/sherlock

# For CI/CD, pass token directly
sherlock auth login --api-url https://api.covertlabs.io --token "$SHERLOCK_TOKEN"
```

## Request Errors

### 400 Bad Request

**Common causes:**

**Domain validation failed:**
```bash
# Wrong - just the name
sherlock search domain gmail

# Right - full domain
sherlock search domain gmail.com
```

**Email validation failed:**
```bash
# Wrong - username only
sherlock search email admin

# Right - use username search for usernames
sherlock search username admin

# Right - full email for email search
sherlock search email admin@company.com
```

**Invalid limit:**
```bash
# Wrong - limit must be 1-100
sherlock search domain acme.com --limit 0
sherlock search domain acme.com --limit 500

# Right
sherlock search domain acme.com --limit 50
```

### 404 Not Found

**Cause:** Victim ID doesn't exist.

**Fix:** Double-check the victim ID. They're case-sensitive.

```bash
# Get victim IDs from search results first
sherlock search domain acme.com --format json | jq '.results[].victim_id'

# Then use the exact ID
sherlock victim profile "exact-victim-id-here"
```

### 422 Unprocessable Entity

**Cause:** Query parameters are syntactically valid but semantically wrong.

**Examples:**
- `limit=0` (must be at least 1)
- Invalid cursor format
- Malformed date ranges

**Fix:** Check the [Command Reference](commands.md) for valid parameter values.

### 429 Too Many Requests

**Cause:** Rate limit exceeded.

**Fix:** Wait and retry. If you're hitting limits regularly:
- Add delays between requests in scripts
- Use larger `--limit` values to reduce request count
- Contact support about rate limit increases

### 502 Bad Gateway / "Query failed"

**Cause:** Backend service temporarily unavailable.

**Fix:**
1. Wait 30 seconds, retry
2. If persistent, check [status.covertlabs.io](https://status.covertlabs.io) (if available)
3. Contact support if the issue continues

### Timeout errors

**Cause:** Request took too long.

**Fix:**
```bash
# Increase timeout (default is 30000ms)
export SHERLOCK_API_TIMEOUT="60000"

# Or reduce result size
sherlock search domain acme.com --limit 20
```

## Installation Issues

### "Command not found: sherlock"

**Cause:** npm global bin directory not in PATH.

**Fix (macOS/Linux):**
```bash
# Find where npm installs global packages
npm config get prefix

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH="$(npm config get prefix)/bin:$PATH"

# Reload shell
source ~/.bashrc  # or ~/.zshrc
```

**Fix (Windows):**
```powershell
# Check npm prefix
npm config get prefix

# Add to PATH via System Properties > Environment Variables
```

**Alternative:** Use npx
```bash
npx @covertlabs/sherlock search domain acme.com
```

### npm install fails

**Cause:** Usually native module compilation (keytar).

**Fix (macOS):**
```bash
# Install Xcode command line tools
xcode-select --install
npm install -g @covertlabs/sherlock
```

**Fix (Linux):**
```bash
# Install build tools
sudo apt-get install build-essential libsecret-1-dev
npm install -g @covertlabs/sherlock
```

**Fix (Windows):**
```powershell
# Install build tools
npm install -g windows-build-tools
npm install -g @covertlabs/sherlock
```

### Node version errors

**Cause:** Node.js version too old.

**Fix:**
```bash
# Check version
node --version

# Need 18+
# Use nvm to upgrade
nvm install 20
nvm use 20
npm install -g @covertlabs/sherlock
```

## Output Issues

### JSON output is malformed

**Cause:** Usually stderr mixed with stdout.

**Fix:**
```bash
# Redirect stderr separately
sherlock search domain acme.com --format json 2>/dev/null

# Or check for error messages
sherlock search domain acme.com --format json 2>&1 | head -20
```

### Table output looks broken

**Cause:** Terminal width too narrow, or non-UTF8 terminal.

**Fix:**
- Widen your terminal window
- Use `--format json` or `--format csv` for narrow terminals
- Ensure terminal supports UTF-8

### Colors not showing

**Cause:** Terminal doesn't support colors, or output is piped.

**Note:** Colors are automatically disabled when output is piped. This is intentional—you don't want ANSI codes in your JSON files.

## Network Issues

### Connection refused

**Cause:** Wrong API URL or network issue.

**Fix:**
```bash
# Check current URL
sherlock auth status

# Test connectivity
curl -I https://api.covertlabs.io/health

# Re-authenticate with correct URL
sherlock auth login --api-url https://api.covertlabs.io
```

### SSL/TLS errors

**Cause:** Corporate proxy, expired cert, or network interception.

**Fix:**
```bash
# If behind corporate proxy
export HTTPS_PROXY="http://proxy.corp.com:8080"

# If cert issues (not recommended for production)
export NODE_TLS_REJECT_UNAUTHORIZED="0"
```

### Proxy issues

**Cause:** Proxy not configured or authentication failing.

**Fix:**
```bash
# Set proxy
export HTTPS_PROXY="http://user:password@proxy.corp.com:8080"

# Bypass proxy for local testing
export NO_PROXY="localhost,127.0.0.1"
```

## Debug Mode

When all else fails, enable debug output:

```bash
export SHERLOCK_DEBUG="true"
sherlock search domain acme.com
```

This shows:
- Request URLs
- Request headers (tokens redacted)
- Response status codes
- Timing information

Include debug output when reporting issues to support.

## Getting Help

### Self-service

1. Check this troubleshooting guide
2. Review [Command Reference](commands.md) for correct syntax
3. Check [Configuration](configuration.md) for environment setup

### Contact Support

**Email:** support@covertlabs.io

**Include:**
- Sherlock version (`sherlock --version`)
- Node.js version (`node --version`)
- Operating system
- Full error message
- Debug output (`SHERLOCK_DEBUG=true`)
- What you were trying to do

### Check Status

If the API seems down, check:
- Your network connectivity
- [status.covertlabs.io](https://status.covertlabs.io) (if available)
- Contact support for outage information

---

Still stuck? Email support@covertlabs.io with debug output and we'll sort it out.
