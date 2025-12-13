# Getting Started

You're five minutes away from your first query. Let's go.

## Prerequisites

- **macOS or Linux** (Windows supported via WSL)
- **curl** and **tar**
- **A Covertlabs account** — if you don't have one, talk to your account manager

## Step 1: Install Sherlock

```bash
curl -fsSL https://covertlabs.io/install.sh | bash
```

Confirm it's installed:

```bash
command -v sherlock
```

If you get "command not found," close and reopen your terminal. If it still fails, ensure `/usr/local/bin` (or `~/.local/bin`) is on your PATH.

## Step 2: Authenticate

Point Sherlock at the API and grab a token:

```bash
sherlock auth login --api-url https://api.covertlabs.io
```

This prints a URL. Open it in your browser, sign in with your Covertlabs credentials (SSO via WorkOS), and you'll see a one-time token. Copy it, paste it back into the terminal prompt.

That's it. Your token is now stored securely in your OS keychain.

## Step 3: Verify Your Setup

```bash
sherlock auth status
```

You should see:
- ✓ Token stored
- API URL: `https://api.covertlabs.io`

If something's wrong, run `sherlock auth login` again.

## Step 4: Run Your First Search

Let's find compromised accounts for a domain:

```bash
sherlock search domain gmail.com --limit 5
```

You'll see a table of results with victim IDs, IP addresses, country codes, and stealer types.

**Important**: Domain searches require a full domain (e.g., `gmail.com`), not just `gmail`. The API validates input strictly.

## Step 5: Dive Deeper

Pick a victim ID from your results and pull their profile:

```bash
sherlock victim profile <victim_id> --include emails,domains --format json
```

Now grab their credentials:

```bash
sherlock victim credentials <victim_id>
```

## What's Next

You've got the basics. Here's where to go from here:

- **[Command Reference](commands.md)** — every command, every flag, every option
- **[Authentication Deep Dive](auth.md)** — how tokens work, rotation, revocation
- **[Configuration](configuration.md)** — environment variables, output formats, timeouts
- **[Troubleshooting](troubleshooting.md)** — when things break

## Quick Reference

```bash
# Search commands
sherlock search email user@example.com
sherlock search domain example.com
sherlock search ip 8.8.8.8
sherlock search username admin
sherlock search password "Password123"
sherlock search country US
sherlock search stealer redline
sherlock search text "banking credentials"

# Victim commands
sherlock victim profile <id>
sherlock victim credentials <id>
sherlock victim cookies <id>
sherlock victim history <id>

# Auth commands
sherlock auth login
sherlock auth status
sherlock auth logout

# Common flags
--limit 50          # Results per page (max 100)
--format json       # Output as JSON
--format csv        # Output as CSV
--cursor <cursor>   # Pagination cursor
```

---

Questions? Something not working? Check [Troubleshooting](troubleshooting.md) or hit up support.
