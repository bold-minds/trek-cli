# trek-cli

Command-line interface for Trek — create and manage debug sessions.

## Installation

```bash
go install github.com/bold-minds/trek-cli@latest
```

This installs the `trek` binary.

## Usage

### Authentication (Clerk)

```bash
# Login via Clerk device flow
trek auth login --clerk-domain your-domain.clerk.accounts.dev --client-id <client-id>

# Check authentication status
trek auth whoami

# Logout
trek auth logout
```

### Start a debug session

```bash
# Debug a specific user for 15 minutes
trek start --user u123 --ttl 15m --level debug --reason "investigating order issue"

# Debug a specific route
trek start --route "/api/orders*" --ttl 10m --level trace

# Debug a tenant
trek start --tenant t456 --ttl 30m --level debug
```

### List active sessions

```bash
trek list
trek list --status active
trek list --status expired
```

### Stop a session

```bash
trek stop --session s_abc123
```

### Inspect a request context (test matching locally)

```bash
trek inspect --request-context '{"user_id":"u123","route":"/api/orders/789"}'
```

### Token management

```bash
# Create a new service token
trek tokens create --name "api-server"

# List tokens
trek tokens list

# Revoke a token
trek tokens revoke --id tok_abc123
```

## Configuration

Set via environment variables or `~/.trek/config.yaml`:

| Env Var | Description |
|---------|-------------|
| `TREK_API_ENDPOINT` | Control plane URL |
| `TREK_API_TOKEN` | Service token for API access |
| `TREK_ORG_ID` | Default organization |
| `TREK_ENV` | Default environment (dev/stage/prod) |
| `TREK_CLERK_DOMAIN` | Clerk domain for auth |
| `TREK_CLERK_CLIENT_ID` | Clerk OAuth client ID |

### Config file example

```yaml
# ~/.trek/config.yaml
endpoint: https://trek.example.com
org: org_abc123
env: prod
```

## Commands

| Command | Description |
|---------|-------------|
| `trek auth login` | Authenticate via Clerk |
| `trek auth logout` | Remove stored credentials |
| `trek auth whoami` | Show auth status |
| `trek start` | Create a debug session |
| `trek stop` | Revoke a session |
| `trek list` | List sessions |
| `trek inspect` | Test request matching |
| `trek tokens create` | Create service token |
| `trek tokens list` | List tokens |
| `trek tokens revoke` | Revoke a token |

## Related Repos

| Repo | Purpose |
|------|---------|
| [trek-go](https://github.com/bold-minds/trek-go) | Go SDK |
| [trek](https://github.com/bold-minds/trek) | Control plane server |
| [trek-spec](https://github.com/bold-minds/trek-spec) | Conformance fixtures |