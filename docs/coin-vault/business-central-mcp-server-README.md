# Coin Vault Business Central MCP server

Exposes Business Central customer \& sales order data as MCP tools, so a
Retell voice agent can look things up mid-call.

```
Retell --\[HTTPS + RETELL\_MCP\_API\_KEY]--> this server --\[Azure AD OAuth]--> Business Central
```

Retell never sees the Business Central credentials — it only holds
`RETELL\_MCP\_API\_KEY`, a separate secret you generate yourself.

## Prerequisites on the Business Central side

The Azure AD app registration alone is not enough. The **same Application
(client) ID** must also be registered inside Business Central:

Business Central → **Settings → Microsoft Entra Applications** → add an
entry with this Client ID → assign it a **Permission Set** covering the
entities you want exposed (customers, sales orders). Without this step,
API calls will fail with 403 even with a valid Azure AD token.

## Setup

```bash
npm install
cp .env.example .env
# fill in .env:
#   BC\_TENANT\_ID, BC\_CLIENT\_ID, BC\_CLIENT\_SECRET  -- from IT (rotate the
#     secret first if it was ever shared over chat/email)
#   BC\_ENVIRONMENT                                -- e.g. "Production", as
#     named in the Business Central Administration Center
#   RETELL\_MCP\_API\_KEY                            -- generate your own,
#     e.g. `openssl rand -hex 32`
npm run build
npm start
```

Confirm it's up: `curl http://localhost:3000/health` should return
`{"status":"ok"}`.

Find your `companyId` (needed by every tool call) by calling the
`list\_companies` tool once connected, or:

```bash
curl -s http://localhost:3000/mcp \\
  -H "Authorization: Bearer $RETELL\_MCP\_API\_KEY" \\
  -H "Content-Type: application/json" \\
  -H "Accept: application/json, text/event-stream" \\
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"list\_companies","arguments":{}}}'
```

## Making it reachable by Retell

Retell requires a **public HTTPS URL** — it explicitly blocks localhost and
private network addresses. Since this runs on an internal server, put a
named **Cloudflare Tunnel** in front of just this port (not the whole local
LLM server):

```bash
cloudflared tunnel login
cloudflared tunnel create coinvault-bc-mcp
cloudflared tunnel route dns coinvault-bc-mcp mcp.thecoinvault.com
cloudflared tunnel run --url http://localhost:3000 coinvault-bc-mcp
```

(Requires DNS access to `thecoinvault.com` in a Cloudflare account — if
that's managed by IT rather than you, they'll need to do the `route dns`
step, or hand you access to do it.)

A **quick tunnel** (`cloudflared tunnel --url http://localhost:3000`, no
domain needed) works for a one-off test, but its URL is random and changes
every restart — not usable for a saved Retell configuration.

## Configuring Retell

In the agent's MCP tool/connection settings:

* **Server URL**: `https://mcp.thecoinvault.com/mcp`
* **Header**: `Authorization: Bearer <RETELL\_MCP\_API\_KEY value>`
* Select which tools (`list\_companies`, `list\_customers`, `get\_customer`,
`list\_sales\_orders`, `get\_sales\_order`) the agent may call.

## Security notes

* `.env` is gitignored — never commit it.
* The Business Central secret and the Retell-facing API key are
intentionally separate: leaking one never exposes the other.



