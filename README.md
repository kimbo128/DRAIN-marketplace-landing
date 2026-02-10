# Handshake58 Marketplace

<p align="center">
  <strong>AI Provider Directory powered by DRAIN Protocol & Bittensor Subnet 58</strong>
</p>

Discover, register, and manage AI providers with trustless micropayments on Polygon.

**Live Marketplace**: [handshake58.com](https://handshake58.com)

---

## What is Handshake58?

Handshake58 is a decentralized AI provider marketplace where agents discover providers, pay per request via the DRAIN Protocol, and providers are scored trustlessly through Bittensor Subnet 58.

- **Provider Discovery** — Find AI providers by model, tier, or score
- **Trustless Scoring** — Bittensor validators score providers based on real on-chain usage
- **Micropayments** — Pay-per-request with USDC on Polygon via payment channels
- **Two Provider Tiers** — Bittensor Miners (auto-verified) and Community Providers (admin-approved)
- **MCP Integration** — AI agents discover providers automatically

---

## Provider Tiers

### TAO Verified (Bittensor Miners)

Providers that run as Bittensor Subnet 58 miners. They are:
- Cryptographically verified via hotkey signatures (sr25519)
- Auto-approved on registration
- Scored by validators: 60% DRAIN claims + 40% availability
- Listed with their Bittensor metagraph score

### Community Providers

Independent providers that register manually:
- Admin-reviewed and tested before approval
- Full access to the marketplace API
- Can upgrade to Bittensor tier by running miner software

---

## Provider SDK & Templates

Ready-to-deploy provider templates for popular AI backends:

| Template | Backend | Models |
|----------|---------|--------|
| [`hs58-openai`](providers/hs58-openai) | OpenAI | GPT-4o, o1, o3-mini, GPT-3.5 |
| [`hs58-claude`](providers/hs58-claude) | Anthropic | Claude 3.5 Sonnet, Haiku, Opus |
| [`hs58-grok`](providers/hs58-grok) | xAI | Grok-2, Grok-2 Mini |
| [`hs58-openrouter`](providers/hs58-openrouter) | OpenRouter | 200+ models |
| [`hs58-chutes`](providers/hs58-chutes) | Chutes | Bittensor inference models |

Each template includes:
- Full DRAIN voucher validation (EIP-712 signatures)
- Automatic payment claiming with expiry protection
- OpenAI-compatible API (`/v1/chat/completions`)
- Configurable pricing with upstream markup
- Health monitoring endpoints
- One-click Railway deployment

### Quick Start

```bash
# Clone provider templates
git clone https://github.com/kimbo128/DRAIN-marketplace-landing.git
cd DRAIN-marketplace-landing/providers/hs58-openai

# Install and configure
npm install
cp .env.example .env
# Edit .env with your API key and Polygon wallet

# Run
npm start
```

---

## For AI Agents

### MCP Server (Recommended)

```bash
npm install -g drain-mcp
```

Add to your Claude Desktop or Cursor config:

```json
{
  "mcpServers": {
    "drain": {
      "command": "drain-mcp",
      "env": {
        "DRAIN_PRIVATE_KEY": "your-polygon-wallet-private-key"
      }
    }
  }
}
```

### API Discovery

```bash
# All providers
GET https://handshake58.com/api/mcp/providers

# Smart filters
GET https://handshake58.com/api/mcp/providers?model=gpt-4o&tier=bittensor&limit=3&format=compact
```

**Filters:** `model`, `tier` (bittensor/community), `minScore`, `limit`, `format` (compact/full)

### Agent Documentation

- [Agent Quick Start](https://handshake58.com/agent.md)
- [MCP Skill File](https://handshake58.com/skill.md)

---

## How It Works

### For Agents

1. **Discover** — Query the marketplace API for providers
2. **Pay Session Fee** — Transfer $0.01 USDC to marketplace fee wallet (from `GET /api/directory/config`)
3. **Open Channel** — Deposit USDC into a DRAIN payment channel (~$0.02 gas)
4. **Use AI** — Send requests with signed vouchers (free, off-chain)
5. **Close Channel** — Withdraw unused USDC when done

### For Providers

1. **Deploy** — Use a provider template or build your own
2. **Register** — Submit via marketplace or auto-register as Bittensor miner
3. **Serve** — Accept voucher-based payments, serve inference
4. **Claim** — Provider claims earned USDC from the contract (auto-claim protects against expiry)

### Scoring (Bittensor Subnet 58)

Validators on Subnet 58 score providers trustlessly:

- **60% DRAIN Claims** — Real USDC claimed from payment channels (7-day window)
- **40% Availability** — Provider responds to validator health checks with valid wallet proof

Higher scores = more visibility in the marketplace + higher Bittensor incentive.

---

## Architecture

```
Agent ──── discovers ────→ Marketplace (handshake58.com)
  │                              │
  │ opens DRAIN channel          │ syncs scores from
  │ pays per request             │ Bittensor metagraph
  ↓                              ↓
Provider ←── scores ──── Validator (Subnet 58)
  │                        │
  │ claims USDC            │ scans DRAIN events
  ↓                        ↓
Polygon ────────────────── Polygon
(DRAIN Contract)           (ChannelClaimed events)
```

---

## Pricing

- **Session fee:** $0.01 per channel (flat)
- **Protocol fee:** 0% on payments
- **Gas cost:** ~$0.02 per channel open/claim on Polygon
- **Provider markup:** Set by each provider (typically 20-50% on upstream costs)

---

## Contract Addresses

| Contract | Address | Network |
|----------|---------|---------|
| DRAIN Channel | `0x1C1918C99b6DcE977392E4131C91654d8aB71e64` | Polygon Mainnet |
| USDC | `0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359` | Polygon Mainnet |

[View on Polygonscan](https://polygonscan.com/address/0x1C1918C99b6DcE977392E4131C91654d8aB71e64)

---

## Repositories

| Repo | Description |
|------|-------------|
| [DRAIN-marketplace-landing](https://github.com/kimbo128/DRAIN-marketplace-landing) | Public landing page, provider templates, documentation |
| [HS58-validator](https://github.com/Handshake58/HS58-validator) | Bittensor Subnet 58 validator |
| [DRAIN Protocol](https://github.com/kimbo128/DRAIN) | Core protocol, SDK, smart contracts |

---

## Documentation

- [Become a Provider](docs/BECOME_A_PROVIDER.md) — Full integration guide
- [Agent Quick Start](https://handshake58.com/agent.md) — For AI agents and developers
- [For Agents](https://handshake58.com/for-agents) — Session fee flow, wallet setup, contract info

---

## License

MIT License

---

Handshake58 &copy; 2026 — Trustless AI payments powered by DRAIN Protocol & Bittensor
