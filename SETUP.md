# OPNsense LLM Assistant Setup Guide

Get your LLM configuring OPNsense in 10 minutes.

## Prerequisites

1. **OPNsense firewall** with API access enabled
2. **API credentials** (key + secret)
3. **LLM client** that supports MCP (Claude Desktop, Cursor, OpenCode, etc.)

---

## Step 1: Enable OPNsense API Access

### Create API User

1. Log into OPNsense Web UI
2. Go to **System > Access > Users**
3. Click **+ Add**
4. Create a user:
   - Username: `api`
   - Password: (generate a strong password)
   - Group: `admins` (or create a custom group with specific privileges)
5. Save

### Generate API Key

1. Edit the user you just created
2. Scroll to **API keys**
3. Click **+** to generate a new key
4. **Download and save** the key file - it contains:
   ```
   key=your_api_key_here
   secret=your_api_secret_here
   ```
   
> **Important:** The secret is only shown once. Save it securely.

### Test API Access

```bash
curl -k -u "your_api_key:your_api_secret" \
  https://your-opnsense-ip/api/core/firmware/status
```

You should get a JSON response with firmware info.

---

## Step 2: Install OPNsense MCP Server

### Option A: NPX (Recommended)

No installation needed - runs directly:

```bash
npx opnsense-mcp-server
```

### Option B: Global Install

```bash
npm install -g opnsense-mcp-server
```

### Option C: From Source

```bash
git clone https://github.com/vespo92/OPNsenseMCP.git
cd OPNsenseMCP
npm install
npm run build
```

---

## Step 3: Configure Your LLM Client

Choose your platform below.

### Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "opnsense": {
      "command": "npx",
      "args": ["opnsense-mcp-server"],
      "env": {
        "OPNSENSE_HOST": "https://192.168.1.1",
        "OPNSENSE_API_KEY": "your_api_key",
        "OPNSENSE_API_SECRET": "your_api_secret",
        "OPNSENSE_VERIFY_SSL": "false"
      }
    }
  }
}
```

Restart Claude Desktop.

### OpenCode

Edit `~/.config/opencode/opencode.json`:

```json
{
  "mcp": {
    "servers": {
      "opnsense": {
        "type": "stdio",
        "command": "npx",
        "args": ["opnsense-mcp-server"],
        "env": {
          "OPNSENSE_HOST": "https://192.168.1.1",
          "OPNSENSE_API_KEY": "your_api_key",
          "OPNSENSE_API_SECRET": "your_api_secret",
          "OPNSENSE_VERIFY_SSL": "false"
        }
      }
    }
  }
}
```

### Cursor

Add to your project's `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "opnsense": {
      "command": "npx",
      "args": ["opnsense-mcp-server"],
      "env": {
        "OPNSENSE_HOST": "https://192.168.1.1",
        "OPNSENSE_API_KEY": "your_api_key",
        "OPNSENSE_API_SECRET": "your_api_secret",
        "OPNSENSE_VERIFY_SSL": "false"
      }
    }
  }
}
```

### Continue.dev

Add to `~/.continue/config.json`:

```json
{
  "experimental": {
    "modelContextProtocolServers": [
      {
        "transport": {
          "type": "stdio",
          "command": "npx",
          "args": ["opnsense-mcp-server"]
        },
        "env": {
          "OPNSENSE_HOST": "https://192.168.1.1",
          "OPNSENSE_API_KEY": "your_api_key",
          "OPNSENSE_API_SECRET": "your_api_secret"
        }
      }
    ]
  }
}
```

---

## Step 4: Add Knowledge Context

For best results, give your LLM the OPNsense knowledge base.

### Option A: Project Instructions (Recommended)

Copy the contents of `llm/SYSTEM_PROMPT.md` into your:
- Claude Desktop: Project instructions
- Cursor: `.cursorrules`
- OpenCode: `CLAUDE.md` in project root

### Option B: Include Knowledge File

For complex setups, also include `llm/OPNSENSE_KNOWLEDGE.md` in your context:

**Claude Desktop:** Add to project knowledge or paste at conversation start

**OpenCode:** Reference in CLAUDE.md:
```markdown
## OPNsense Reference
See: /path/to/opnsense-user-guide/llm/OPNSENSE_KNOWLEDGE.md
```

**Cursor:** Add to context with `@file` reference

---

## Step 5: Verify Setup

Start a conversation and test:

```
Test my OPNsense connection and list the current firewall rules.
```

The LLM should:
1. Call `opnsense_test_connection`
2. Call `opnsense_list_firewall_rules`
3. Show you the results

If it works, you're ready to go!

---

## Configuration Reference

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPNSENSE_HOST` | Yes | Firewall URL (e.g., `https://192.168.1.1`) |
| `OPNSENSE_API_KEY` | Yes | API key from OPNsense |
| `OPNSENSE_API_SECRET` | Yes | API secret from OPNsense |
| `OPNSENSE_VERIFY_SSL` | No | Set to `false` for self-signed certs |

### SSH Access (Optional)

For advanced CLI commands (`opnsense_ssh_*` tools), add SSH credentials:

```json
{
  "env": {
    "OPNSENSE_SSH_HOST": "192.168.1.1",
    "OPNSENSE_SSH_USER": "root",
    "OPNSENSE_SSH_KEY": "/path/to/private_key"
  }
}
```

---

## Example Conversations

**Check Status:**
```
You: What's the status of my OPNsense firewall?
LLM: [calls opnsense_test_connection, opnsense_get_interfaces]
     Your firewall is online. WAN has IP 203.0.113.5, LAN is 192.168.1.1/24...
```

**Create Port Forward:**
```
You: Forward port 8080 from the internet to my web server at 192.168.1.50
LLM: I'll create a port forward. [calls opnsense_nat_create_port_forward]
     Done. External port 8080 now forwards to 192.168.1.50:8080.
```

**Set Up Guest VLAN:**
```
You: Create an isolated guest network on VLAN 20
LLM: I'll create VLAN 20. [calls opnsense_create_vlan]
     VLAN created. You'll need to assign it to an interface in the UI at
     Interfaces > Assignments. Then I can create the firewall rules.
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Connection refused" | Check OPNSENSE_HOST URL, ensure API is enabled |
| "401 Unauthorized" | Verify API key/secret, check user has admin group |
| "SSL certificate error" | Set `OPNSENSE_VERIFY_SSL=false` for self-signed certs |
| MCP tools not showing | Restart your LLM client after config changes |
| SSH commands fail | Add SSH env vars, ensure root SSH is enabled |

### Debug: Test MCP Server Directly

```bash
# Set env vars
export OPNSENSE_HOST="https://192.168.1.1"
export OPNSENSE_API_KEY="your_key"
export OPNSENSE_API_SECRET="your_secret"
export OPNSENSE_VERIFY_SSL="false"

# Run server manually to see errors
npx opnsense-mcp-server
```

---

## Next Steps

- Read `llm/OPNSENSE_KNOWLEDGE.md` for detailed configuration patterns
- Check `docs/OPNsense_User_Guide.pdf` for comprehensive human-readable docs
- Visit [OPNsense MCP Server repo](https://github.com/vespo92/OPNsenseMCP) for tool updates