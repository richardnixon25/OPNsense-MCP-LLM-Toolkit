# OPNsense MCP Server Enhancements Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Fork and extend the OPNsense MCP server (vespo92/OPNSenseMCP) with P0 tools for WireGuard management, PF state inspection, and gateway monitoring.

**Architecture:** Fork upstream repo, add new tool handlers following existing patterns (axios for API, ssh2 for CLI fallback). Each feature area gets its own section in index.ts with tool definitions and handlers.

**Tech Stack:** TypeScript, Node.js, @modelcontextprotocol/sdk, axios (HTTP), ssh2 (SSH), OPNsense REST API

---

## Phase 0: Repository Setup

### Task 0.1: Clone and Configure Fork

**Files:**
- Create: `~/Projects/OPNSenseMCP-fork/` (working directory)

**Step 1: Clone upstream repository**

```bash
cd ~/Projects/OPNSenseMCP-fork
git clone https://github.com/vespo92/OPNSenseMCP.git .
```

**Step 2: Verify clone**

Run: `ls -la && git remote -v`
Expected: See src/, package.json, origin pointing to vespo92/OPNSenseMCP

**Step 3: Create feature branch**

```bash
git checkout -b feature/p0-enhancements
```

**Step 4: Install dependencies**

```bash
npm install
```

**Step 5: Verify build works**

Run: `npm run build`
Expected: Compiles without errors, creates dist/

**Step 6: Commit baseline**

```bash
git add -A
git commit -m "chore: initial fork baseline from upstream v0.9.4"
```

---

## Phase 1: WireGuard Management Tools

### Task 1.1: Add WireGuard Service Status Tool

**Files:**
- Modify: `src/index.ts` (add tool definition + handler)

**Step 1: Add tool definition after existing tool definitions**

Find the `tools: [` array and add:

```typescript
{
  name: "opnsense_wireguard_status",
  description: "Get WireGuard service status (running/stopped)",
  inputSchema: {
    type: "object",
    properties: {},
    required: []
  }
},
```

**Step 2: Add handler in the switch statement**

Find the tool handler switch and add case:

```typescript
case "opnsense_wireguard_status": {
  const response = await apiClient.get('/api/wireguard/service/status');
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(wireguard): add service status tool"
```

---

### Task 1.2: Add WireGuard Show Tunnel Details Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_wireguard_show",
  description: "Show WireGuard tunnel details including handshakes, transfer stats, and endpoints",
  inputSchema: {
    type: "object",
    properties: {},
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_wireguard_show": {
  const response = await apiClient.get('/api/wireguard/service/show');
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(wireguard): add show tunnel details tool"
```

---

### Task 1.3: Add WireGuard List Servers Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_wireguard_list_servers",
  description: "List all WireGuard server (local) configurations",
  inputSchema: {
    type: "object",
    properties: {},
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_wireguard_list_servers": {
  const response = await apiClient.get('/api/wireguard/server/searchServer');
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(wireguard): add list servers tool"
```

---

### Task 1.4: Add WireGuard List Clients/Peers Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_wireguard_list_clients",
  description: "List all WireGuard client (peer) configurations",
  inputSchema: {
    type: "object",
    properties: {},
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_wireguard_list_clients": {
  const response = await apiClient.get('/api/wireguard/client/searchClient');
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(wireguard): add list clients/peers tool"
```

---

### Task 1.5: Add WireGuard Restart Service Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_wireguard_restart",
  description: "Restart the WireGuard service",
  inputSchema: {
    type: "object",
    properties: {},
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_wireguard_restart": {
  const response = await apiClient.post('/api/wireguard/service/restart');
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(wireguard): add restart service tool"
```

---

### Task 1.6: Add WireGuard Get Server Details Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_wireguard_get_server",
  description: "Get detailed configuration for a specific WireGuard server",
  inputSchema: {
    type: "object",
    properties: {
      uuid: {
        type: "string",
        description: "Server UUID (from list_servers)"
      }
    },
    required: ["uuid"]
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_wireguard_get_server": {
  const { uuid } = args as { uuid: string };
  const response = await apiClient.get(`/api/wireguard/server/getServer/${uuid}`);
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(wireguard): add get server details tool"
```

---

## Phase 2: PF States / Firewall Diagnostics Tools

### Task 2.1: Add PF States Query Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_pf_states",
  description: "Get current PF (packet filter) state table - shows active connections",
  inputSchema: {
    type: "object",
    properties: {},
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_pf_states": {
  const response = await apiClient.get('/api/diagnostics/firewall/pf_states');
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(diagnostics): add pf_states tool for connection tracking"
```

---

### Task 2.2: Add PF States Search/Query Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_pf_query_states",
  description: "Search/filter PF states by criteria (source, destination, interface)",
  inputSchema: {
    type: "object",
    properties: {
      filter: {
        type: "string",
        description: "Filter expression (e.g., IP address, interface name)"
      }
    },
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_pf_query_states": {
  const { filter } = args as { filter?: string };
  const response = await apiClient.post('/api/diagnostics/firewall/query_states', {
    searchPhrase: filter || ''
  });
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(diagnostics): add pf_query_states tool with filtering"
```

---

### Task 2.3: Add PF Kill States Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_pf_kill_states",
  description: "Kill PF states matching criteria (use with caution)",
  inputSchema: {
    type: "object",
    properties: {
      filter: {
        type: "string",
        description: "Filter for states to kill (e.g., source IP, interface)"
      }
    },
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_pf_kill_states": {
  const { filter } = args as { filter?: string };
  const response = await apiClient.post('/api/diagnostics/firewall/kill_states', {
    filter: filter || ''
  });
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(diagnostics): add pf_kill_states tool"
```

---

### Task 2.4: Add PF Flush States Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_pf_flush_states",
  description: "Flush all PF states (WARNING: will drop all active connections)",
  inputSchema: {
    type: "object",
    properties: {},
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_pf_flush_states": {
  const response = await apiClient.post('/api/diagnostics/firewall/flush_states');
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(diagnostics): add pf_flush_states tool"
```

---

### Task 2.5: Add PF Statistics Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_pf_statistics",
  description: "Get PF statistics (packets passed/blocked, state table info)",
  inputSchema: {
    type: "object",
    properties: {
      section: {
        type: "string",
        description: "Statistics section (optional - all if omitted)"
      }
    },
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_pf_statistics": {
  const { section } = args as { section?: string };
  const url = section 
    ? `/api/diagnostics/firewall/pf_statistics/${section}`
    : '/api/diagnostics/firewall/pf_statistics';
  const response = await apiClient.get(url);
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(diagnostics): add pf_statistics tool"
```

---

### Task 2.6: Add Delete Specific State Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_pf_delete_state",
  description: "Delete a specific PF state by ID",
  inputSchema: {
    type: "object",
    properties: {
      stateId: {
        type: "string",
        description: "State ID to delete"
      },
      creatorId: {
        type: "string",
        description: "Creator ID of the state"
      }
    },
    required: ["stateId", "creatorId"]
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_pf_delete_state": {
  const { stateId, creatorId } = args as { stateId: string; creatorId: string };
  const response = await apiClient.post(`/api/diagnostics/firewall/del_state/${stateId}/${creatorId}`);
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(diagnostics): add delete specific state tool"
```

---

## Phase 3: Gateway Monitoring Tools

### Task 3.1: Add Gateway Status Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_gateway_status",
  description: "Get status of all gateways including latency, packet loss, and online status",
  inputSchema: {
    type: "object",
    properties: {},
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_gateway_status": {
  const response = await apiClient.get('/api/routes/gateway/status');
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(gateway): add gateway status monitoring tool"
```

---

## Phase 4: Firewall Aliases Tools

### Task 4.1: Add List Aliases Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_alias_list",
  description: "List all firewall aliases",
  inputSchema: {
    type: "object",
    properties: {},
    required: []
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_alias_list": {
  const response = await apiClient.get('/api/firewall/alias/searchItem');
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(alias): add list aliases tool"
```

---

### Task 4.2: Add Get Alias Details Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_alias_get",
  description: "Get detailed configuration for a specific alias",
  inputSchema: {
    type: "object",
    properties: {
      uuid: {
        type: "string",
        description: "Alias UUID"
      }
    },
    required: ["uuid"]
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_alias_get": {
  const { uuid } = args as { uuid: string };
  const response = await apiClient.get(`/api/firewall/alias/getItem/${uuid}`);
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(alias): add get alias details tool"
```

---

### Task 4.3: Add Get Alias by Name Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_alias_get_by_name",
  description: "Get alias UUID by name",
  inputSchema: {
    type: "object",
    properties: {
      name: {
        type: "string",
        description: "Alias name (e.g., TAILSCALE_REDIRECTORS)"
      }
    },
    required: ["name"]
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_alias_get_by_name": {
  const { name } = args as { name: string };
  const response = await apiClient.get(`/api/firewall/alias/getAliasUUID/${name}`);
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(alias): add get alias by name tool"
```

---

### Task 4.4: Add Alias Content Listing Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_alias_list_content",
  description: "List the actual IPs/hosts in an alias (runtime content)",
  inputSchema: {
    type: "object",
    properties: {
      alias: {
        type: "string",
        description: "Alias name"
      }
    },
    required: ["alias"]
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_alias_list_content": {
  const { alias } = args as { alias: string };
  const response = await apiClient.get(`/api/firewall/alias_util/list/${alias}`);
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(alias): add list alias content tool"
```

---

### Task 4.5: Add Entry to Alias Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_alias_add_entry",
  description: "Add an IP/host entry to an alias",
  inputSchema: {
    type: "object",
    properties: {
      alias: {
        type: "string",
        description: "Alias name"
      },
      entry: {
        type: "string",
        description: "IP address or hostname to add"
      }
    },
    required: ["alias", "entry"]
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_alias_add_entry": {
  const { alias, entry } = args as { alias: string; entry: string };
  const response = await apiClient.post(`/api/firewall/alias_util/add/${alias}`, {
    address: entry
  });
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(alias): add entry to alias tool"
```

---

### Task 4.6: Add Remove Entry from Alias Tool

**Files:**
- Modify: `src/index.ts`

**Step 1: Add tool definition**

```typescript
{
  name: "opnsense_alias_delete_entry",
  description: "Remove an IP/host entry from an alias",
  inputSchema: {
    type: "object",
    properties: {
      alias: {
        type: "string",
        description: "Alias name"
      },
      entry: {
        type: "string",
        description: "IP address or hostname to remove"
      }
    },
    required: ["alias", "entry"]
  }
},
```

**Step 2: Add handler**

```typescript
case "opnsense_alias_delete_entry": {
  const { alias, entry } = args as { alias: string; entry: string };
  const response = await apiClient.post(`/api/firewall/alias_util/delete/${alias}`, {
    address: entry
  });
  return {
    content: [{
      type: "text",
      text: JSON.stringify(response.data, null, 2)
    }]
  };
}
```

**Step 3: Build and verify**

Run: `npm run build`
Expected: Compiles without errors

**Step 4: Commit**

```bash
git add src/index.ts
git commit -m "feat(alias): add remove entry from alias tool"
```

---

## Phase 5: Integration and Testing

### Task 5.1: Update OpenCode Configuration

**Files:**
- Modify: `~/.config/opencode/opencode.json`

**Step 1: Update MCP server path to use local fork**

Change the opnsense entry from:
```json
"command": "npx",
"args": ["-y", "opnsense-mcp-server"]
```

To:
```json
"command": "node",
"args": ["/path/to/OPNSenseMCP-fork/dist/index.js"]
```

**Step 2: Restart OpenCode**

Close and reopen OpenCode to load the new MCP server.

**Step 3: Verify tools are available**

In OpenCode, try calling one of the new tools:
- `opnsense_wireguard_status`
- `opnsense_gateway_status`
- `opnsense_pf_states`

Expected: Tools should execute and return data from OPNsense

---

### Task 5.2: Test WireGuard Tools Against Live System

**Step 1: Test wireguard_status**

Run tool and verify:
- Returns service status (running/stopped)
- No API errors

**Step 2: Test wireguard_show**

Run tool and verify:
- Returns tunnel details
- Shows wg0/wg1 interfaces
- Shows peer handshakes

**Step 3: Test wireguard_list_servers**

Run tool and verify:
- Returns server configurations
- Matches OPNsense UI

---

### Task 5.3: Test PF State Tools Against Live System

**Step 1: Test pf_states**

Run tool and verify:
- Returns active connection states
- Shows source/destination IPs

**Step 2: Test pf_query_states with filter**

Run: `opnsense_pf_query_states` with filter="10.0.50"
Verify: Returns only VLAN 50 states

**Step 3: Test pf_statistics**

Run tool and verify:
- Returns packet counts
- Shows state table size

---

### Task 5.4: Test Gateway Tools Against Live System

**Step 1: Test gateway_status**

Run tool and verify:
- Returns all gateways
- Shows Mullvad gateway (wg0) status
- Shows latency and packet loss

---

### Task 5.5: Final Commit and Version Bump

**Files:**
- Modify: `package.json`

**Step 1: Update version**

Change version from "0.9.4" to "0.9.5-fork.1"

**Step 2: Commit all changes**

```bash
git add -A
git commit -m "feat: add P0 tools - WireGuard, PF states, Gateway, Aliases

New tools added:
- WireGuard: status, show, list_servers, list_clients, restart, get_server
- PF States: pf_states, query_states, kill_states, flush_states, statistics, delete_state
- Gateway: gateway_status
- Aliases: list, get, get_by_name, list_content, add_entry, delete_entry

Closes P0 requirements for VLAN 50 WireGuard monitoring and firewall debugging."
```

---

## Summary: New Tools Added

| Category | Tool | OPNsense API Endpoint |
|----------|------|----------------------|
| WireGuard | `wireguard_status` | GET /api/wireguard/service/status |
| WireGuard | `wireguard_show` | GET /api/wireguard/service/show |
| WireGuard | `wireguard_list_servers` | GET /api/wireguard/server/searchServer |
| WireGuard | `wireguard_list_clients` | GET /api/wireguard/client/searchClient |
| WireGuard | `wireguard_restart` | POST /api/wireguard/service/restart |
| WireGuard | `wireguard_get_server` | GET /api/wireguard/server/getServer/{uuid} |
| PF States | `pf_states` | GET /api/diagnostics/firewall/pf_states |
| PF States | `pf_query_states` | POST /api/diagnostics/firewall/query_states |
| PF States | `pf_kill_states` | POST /api/diagnostics/firewall/kill_states |
| PF States | `pf_flush_states` | POST /api/diagnostics/firewall/flush_states |
| PF States | `pf_statistics` | GET /api/diagnostics/firewall/pf_statistics |
| PF States | `pf_delete_state` | POST /api/diagnostics/firewall/del_state |
| Gateway | `gateway_status` | GET /api/routes/gateway/status |
| Aliases | `alias_list` | GET /api/firewall/alias/searchItem |
| Aliases | `alias_get` | GET /api/firewall/alias/getItem/{uuid} |
| Aliases | `alias_get_by_name` | GET /api/firewall/alias/getAliasUUID/{name} |
| Aliases | `alias_list_content` | GET /api/firewall/alias_util/list/{alias} |
| Aliases | `alias_add_entry` | POST /api/firewall/alias_util/add/{alias} |
| Aliases | `alias_delete_entry` | POST /api/firewall/alias_util/delete/{alias} |

**Total: 19 new tools**
