# Trek CLI Guidelines

> These guidelines are specific to the Trek CLI (`trek-cli`).

## Overview

The Trek CLI provides command-line access to Trek functionality for developers and operators.

## Design Principles

### 1. Consistent Command Structure

Follow standard CLI conventions:

```bash
trek <resource> <action> [flags]

# Examples
trek session create --selector user:123 --level debug --ttl 1h
trek session list --environment prod
trek session revoke sess_abc123
trek org list
trek env create staging
```

### 2. Human and Machine Output

Support both human-readable and machine-parseable output:

```bash
# Human-readable (default)
trek session list

# JSON output for scripting
trek session list --output json

# Quiet mode (IDs only)
trek session list --quiet
```

### 3. Helpful Error Messages

```bash
# Bad
Error: invalid selector

# Good
Error: Invalid selector format "user123"
  Selectors must be in format "type:value"
  Valid types: user, tenant, request, route
  Example: trek session create --selector user:123
```

---

## Command Structure

### Global Flags

```go
// Available on all commands
--config string      Config file (default: ~/.trek/config.yaml)
--org string         Organization ID (overrides config)
--env string         Environment (overrides config)
--output string      Output format: table, json, yaml (default: table)
--quiet              Only output IDs
--verbose            Verbose output
--no-color           Disable colored output
```

### Session Commands

```bash
trek session create   # Create a new debug session
trek session list     # List active sessions
trek session get      # Get session details
trek session revoke   # Revoke a session
trek session extend   # Extend session TTL
```

### Organization Commands

```bash
trek org list         # List organizations
trek org switch       # Switch active organization
trek org members      # List organization members
```

### Environment Commands

```bash
trek env list         # List environments
trek env create       # Create environment
trek env switch       # Switch active environment
```

### Config Commands

```bash
trek config init      # Initialize configuration
trek config show      # Show current configuration
trek config set       # Set configuration value
```

---

## Configuration

### Config File Structure

```yaml
# ~/.trek/config.yaml
current_context: default

contexts:
  default:
    org_id: acme-corp
    environment: production
    api_url: https://api.trek.dev
    
  staging:
    org_id: acme-corp
    environment: staging
    api_url: https://api.trek.dev

auth:
  token: trek_xxx...  # Or use TREK_API_KEY env var
```

### Authentication Priority

1. `--token` flag
2. `TREK_API_KEY` environment variable
3. Config file token
4. Interactive login

---

## Output Formatting

### Table Output (Default)

```
ID              SELECTOR        LEVEL   STATUS   EXPIRES
sess_abc123     user:123        debug   active   in 45m
sess_def456     tenant:acme     info    active   in 2h
```

### JSON Output

```json
{
  "sessions": [
    {
      "id": "sess_abc123",
      "selector": "user:123",
      "level": "debug",
      "status": "active",
      "expires_at": "2024-01-15T12:00:00Z"
    }
  ]
}
```

---

## Error Handling

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Authentication error |
| 4 | Not found |
| 5 | Permission denied |

### Error Format

```go
type CLIError struct {
    Message    string
    Hint       string
    ExitCode   int
}

func (e *CLIError) Error() string {
    if e.Hint != "" {
        return fmt.Sprintf("Error: %s\n  %s", e.Message, e.Hint)
    }
    return fmt.Sprintf("Error: %s", e.Message)
}
```

---

## Interactive Features

### Confirmations

```bash
# Dangerous operations require confirmation
trek session revoke sess_abc123
> This will revoke the debug session for user:123
> Are you sure? [y/N]: y
Session revoked.

# Skip with --yes flag
trek session revoke sess_abc123 --yes
```

### Selection Prompts

```bash
trek env switch
> Select environment:
  1. development
  2. staging
> 3. production (current)
> Enter number: 2
Switched to staging
```

---

## Testing

### Command Tests

```go
func TestSessionCreate(t *testing.T) {
    // Test with all required flags
    cmd := NewRootCmd()
    cmd.SetArgs([]string{
        "session", "create",
        "--selector", "user:123",
        "--level", "debug",
        "--ttl", "1h",
    })
    
    err := cmd.Execute()
    assert.NoError(t, err)
}
```

### Integration Tests

```bash
# Test against real API (integration tests)
TREK_API_URL=http://localhost:8080 go test -tags=integration ./...
```

---

## Directory Structure

```
trek-cli/
├── cmd/
│   ├── root.go           # Root command
│   ├── session.go        # Session commands
│   ├── org.go            # Organization commands
│   ├── env.go            # Environment commands
│   └── config.go         # Config commands
├── internal/
│   ├── api/              # API client
│   ├── config/           # Configuration handling
│   └── output/           # Output formatting
├── main.go               # Entrypoint
└── go.mod
```

## Release

- Build binaries for linux/amd64, linux/arm64, darwin/amd64, darwin/arm64, windows/amd64
- Publish to GitHub releases
- Update Homebrew formula
