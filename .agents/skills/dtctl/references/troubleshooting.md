# dtctl Troubleshooting

## Installation

Install and upgrade instructions live in the dtctl repo: **[github.com/dynatrace-oss/dtctl](https://github.com/dynatrace-oss/dtctl)** (Homebrew, install script, or `go install`). After installing, verify with `dtctl version` and confirm it is on your `PATH`.

## Initial Setup

```bash
# Store credentials (use --token flag directly, NOT stdin piping)
dtctl config set-credentials "my-token" --token "$DYNATRACE_API_TOKEN"
dtctl config set-context "default" \
  --environment "$DYNATRACE_BASE_URL" \
  --token-ref "my-token"
dtctl config use-context "default"

# Verify
dtctl auth whoami --plain
```

**Note:** Always use `--token "$TOKEN"` directly. Stdin piping does not work reliably and stores corrupted values in the keychain.

## Common Issues

### 401/403 Authentication Errors
```bash
# Re-store credentials
dtctl config set-credentials "my-token" --token "$TOKEN"

# Verify identity
dtctl auth whoami --plain

# Check permissions
dtctl auth can-i <verb> <resource>
```

### Wrong Tenant
```bash
dtctl config get-contexts --plain
dtctl config use-context <name>
```

### Safety Level Blocks
Safety levels are client-side protections: `readonly`, `readwrite-mine`, `readwrite-all`, `dangerously-unrestricted`. API token scopes determine actual permissions.

### dtctl Not Found
Ensure binary is on PATH. Check `~/bin/dtctl` or `/usr/local/bin/dtctl`.

### Corrupted Keychain Entry (macOS)
```bash
security delete-generic-password -s "dtctl" -a "<token-ref>"
dtctl config set-credentials "<token-ref>" --token "$TOKEN"
```

## Debugging

```bash
# Verbose output
dtctl <command> -v     # Details
dtctl <command> -vv    # Full debug including auth headers
```

## Platform Notes
- **macOS**: Keychain must be unlocked
- **Linux**: Requires gnome-keyring or similar
- **Windows**: Uses Credential Manager
