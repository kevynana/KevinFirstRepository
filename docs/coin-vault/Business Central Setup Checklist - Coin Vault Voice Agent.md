# Business Central Setup Checklist — Coin Vault Voice Agent

Sep 22, 2026 · @Someone

## What this sets up

This gives the voice agent read access to customer and sales order data in Business Central, so it can answer caller questions during a call instead of taking a message.

The connection uses an Azure AD app registration with client credentials — no user account, no interactive login, no shared password. Retell never receives Business Central credentials; it holds a separate key that only reaches our middleware.

Five steps follow. Steps 1 through 4 all happen in your tenant. Step 3 is the one that most often gets missed, so read it before you start.

## Step 1 — Azure AD app registration

In the Entra admin center for the tenant that hosts Business Central:

- [ ] New app registration. Name it something recognizable, e.g. `Coin Vault Voice Agent — BC API`
- [ ] Single tenant. No redirect URI — this is a client credentials flow, not interactive login
- [ ] API permissions → Add → Dynamics 365 Business Central → **Application permissions** (not Delegated) → `API.Read.All`
- [ ] Grant admin consent for the tenant
- [ ] Copy the **Application (client) ID** and **Directory (tenant) ID**

`API.Read.All` is the right scope if the agent only looks things up. If we later want it to create or modify records, that becomes a separate conversation and a separate permission grant — not something to pre-authorize now.

## Step 2 — Client secret

- [ ] In the same app registration: Certificates & secrets → New client secret
- [ ] Set expiry to 12 months (not 24 — a shorter cycle means it fails while someone still remembers what it's for)
- [ ] Copy the secret **Value** immediately; it's only shown once
- [ ] Note the expiry date somewhere you'll actually see it — a calendar reminder 30 days out

Send the secret through a password manager share or read it over a call. Not email, not Slack, not a text message. If it does end up in a chat thread at any point, rotate it rather than hoping.

An expired secret is the single most likely cause of the agent going dark six months from now, and it fails silently from the caller's side.

## Step 3 — Register the same app inside Business Central

This is a second, separate registration. The Azure AD side alone is not enough, and skipping this produces a 403 even with a perfectly valid token — which reads like a permissions bug and sends people back to re-check Step 1 for an hour.

In Business Central itself (not the Azure portal):

- [ ] Search for **Microsoft Entra Applications** → New
- [ ] Paste the same **Client ID** from Step 1
- [ ] Give it a description, e.g. `Voice agent — read-only customer and order lookup`
- [ ] Set **State** to Enabled
- [ ] Assign a permission set (see Step 4)

If you get a 403 after all of this, come back to this page first and confirm State is Enabled. It defaults to disabled on creation.

## Step 4 — Scope the permission set

Please don't assign `SUPER`. It's the fast path and it's what most setup guides default to, but it makes this credential able to post journal entries, modify pricing, and read payroll — none of which the agent needs.

- [ ] Create a custom permission set, e.g. `VOICE_AGENT_RO`
- [ ] Grant read-only access to the API pages backing customers and sales orders
- [ ] No insert, modify, or delete permissions
- [ ] Assign that set to the Entra Application entry from Step 3

The practical difference: if this credential is ever compromised, a scoped read-only set means someone can read customer records. `SUPER` means someone can alter the books. Worth the extra fifteen minutes.

## Step 5 — Sandbox first, if there is one

- [ ] Confirm whether a sandbox environment exists with representative customer and order data
- [ ] If yes, do Steps 1–4 against the sandbox and we'll test end-to-end there before touching Production

If there's no sandbox with usable data, say so and we'll plan the Production cutover deliberately — a read-only scope makes that much less risky, but I'd rather not have the first live call also be the first test.

## What to send back

All of this in one message, so nothing stalls halfway through wiring it up:

| Item | Where it comes from | Value |
| --- | --- | --- |
| Directory (tenant) ID | Step 1 |  |
| Application (client) ID | Step 1 |  |
| Environment name | BC Admin Center, exactly as shown |  |
| Entra Application State | Step 3 — confirm "Enabled" |  |
| Permission set name | Step 4 |  |
| Sandbox available? | Step 5 |  |

The environment name is case-sensitive. `Production` and `production` are not the same string to the API, so copy it rather than typing it from memory.

The client secret comes separately, through whichever channel we agree on — not in this message.

## What this credential ultimately reaches

Worth stating plainly rather than leaving in the architecture diagram: the end of this chain is an inbound phone caller. Someone dials the main line, the agent looks something up, and the answer comes back over the phone.

That's the intended behavior. It also means the lookups need to be bound to a verified caller, enforced on our side rather than only in the agent's instructions. Otherwise the line becomes a way to enumerate who buys what and where it ships — which for a precious metals business is exactly the information worth protecting.

We're handling that in the middleware, and I'll walk you through what verification looks like before anything goes live. Flagging it now so the read-only scoping in Step 4 reads as deliberate rather than as us being difficult about permissions.
