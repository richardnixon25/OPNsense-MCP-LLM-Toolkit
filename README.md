# OPNsense LLM Assistant

Configure your OPNsense firewall using natural language with any MCP-compatible LLM.

## What This Is

An integrated package that lets you use Claude, Cursor, OpenCode, or other LLMs to configure OPNsense firewalls through conversation. Instead of clicking through the web UI, just tell your LLM what you want:

```
"Create a guest VLAN that can access the internet but not my LAN"
"Forward port 443 to my web server at 192.168.1.50"  
"Show me all firewall rules blocking traffic"
```

## Quick Start

1. **Get API credentials** from OPNsense (System > Access > Users > API keys)
2. **Add MCP server** to your LLM client config:
   ```json
   {
     "mcpServers": {
       "opnsense": {
         "command": "npx",
         "args": ["opnsense-mcp-server"],
         "env": {
           "OPNSENSE_HOST": "https://192.168.1.1",
           "OPNSENSE_API_KEY": "your_key",
           "OPNSENSE_API_SECRET": "your_secret",
           "OPNSENSE_VERIFY_SSL": "false"
         }
       }
     }
   }
   ```
3. **Add the system prompt** from `llm/SYSTEM_PROMPT.md` to your LLM's instructions
4. **Start chatting** - "Test my OPNsense connection"

**Full setup instructions:** [SETUP.md](SETUP.md)

## What's Included

| File | Purpose |
|------|---------|
| `llm/SYSTEM_PROMPT.md` | Copy-paste prompt for your LLM |
| `llm/OPNSENSE_KNOWLEDGE.md` | Detailed reference (MCP tools, workflows, patterns) |
| `docs/OPNsense_User_Guide.pdf` | 78-page human-readable guide |
| `SETUP.md` | Step-by-step setup instructions |

## How It Works

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   You       │────▶│   LLM       │────▶│  OPNsense   │
│  (natural   │     │ (with MCP   │     │  (firewall) │
│  language)  │◀────│  tools)     │◀────│             │
└─────────────┘     └─────────────┘     └─────────────┘
                          │
                    Knowledge from
                    this repo
```

The LLM uses:
- **MCP tools** to execute commands on OPNsense
- **Knowledge files** to understand OPNsense concepts and best practices
- **System prompt** to follow safety guidelines (backup first, explain changes, etc.)

## Requirements

- OPNsense firewall with API access enabled
- LLM client with MCP support (Claude Desktop, Cursor, OpenCode, Continue.dev)
- Node.js (for npx)

## Project Structure

```
opnsense-user-guide/
├── README.md
├── SETUP.md                 # Setup instructions
├── llm/
│   ├── SYSTEM_PROMPT.md     # LLM instructions
│   └── OPNSENSE_KNOWLEDGE.md # Reference material
├── docs/
│   └── OPNsense_User_Guide.pdf
└── src/
    └── opnsense_user_guide.py  # PDF generator
```

## Related

- [OPNsense MCP Server](https://github.com/vespo92/OPNsenseMCP) - The MCP server that provides the tools
- [OPNsense](https://opnsense.org/) - The firewall itself

## License

MIT

---

*Not affiliated with the OPNsense project. OPNsense is a trademark of Deciso B.V.*
