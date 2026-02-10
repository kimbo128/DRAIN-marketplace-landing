# Become a Handshake58 Provider

Earn revenue by offering AI inference through the Handshake58 marketplace. Accept trustless micropayments from AI agents — no credit cards, no chargebacks, no KYC.

---

## Why Become a Provider?

| Benefit | Description |
|---------|-------------|
| **Instant Payments** | Receive USDC directly to your wallet via smart contract |
| **Zero Chargebacks** | Blockchain-based payments are final and irreversible |
| **Global Access** | Accept payments from anywhere, no geographic restrictions |
| **Agent Compatible** | Tap into the growing autonomous AI agent economy |
| **Bittensor Scoring** | Earn TAO emissions as a Subnet 58 miner |
| **Auto-Claim Protection** | Built-in expiry monitoring ensures you never lose earned funds |

---

## Two Ways to Join

### Option A: Community Provider (Easy Start)

1. Deploy a provider template
2. Register on the [marketplace](https://handshake58.com/directory)
3. Get admin-approved and listed

### Option B: Bittensor Miner (TAO Verified)

1. Deploy a provider template
2. Register on Bittensor Subnet 58 as a miner
3. Run the miner software alongside your provider
4. Get auto-approved, scored, and earn TAO emissions

---

## Quick Start with Provider Templates

We provide ready-to-deploy templates for popular AI backends:

### 1. Choose a Template

```bash
git clone https://github.com/Handshake58/DRAIN-marketplace.git
cd DRAIN-marketplace/providers/
```

| Template | Backend | Best For |
|----------|---------|----------|
| `hs58-openai` | OpenAI API | GPT-4o, o1, o3-mini |
| `hs58-claude` | Anthropic API | Claude 3.5 Sonnet, Opus |
| `hs58-grok` | xAI API | Grok-2 |
| `hs58-openrouter` | OpenRouter | 200+ models from any provider |
| `hs58-chutes` | Chutes | Bittensor inference models |

### 2. Configure

```bash
cd hs58-openai  # or any template
npm install
cp .env.example .env
```

Edit `.env`:

```env
# Your upstream API key
OPENAI_API_KEY=sk-...

# Your Polygon wallet (receives USDC payments)
PROVIDER_PRIVATE_KEY=0x...

# Network
CHAIN_ID=137

# Pricing markup on upstream costs (e.g., 50 = 50% markup)
MARKUP_PERCENT=50

# Minimum USDC to trigger a claim (in wei, 6 decimals)
# 1000000 = $1.00 USDC
CLAIM_THRESHOLD=1000000

# Server
PORT=3000
```

### 3. Run

```bash
npm start
```

Your provider is now live at `http://localhost:3000` with:
- `GET /v1/pricing` — Your models and prices
- `GET /v1/models` — OpenAI-compatible model list
- `POST /v1/chat/completions` — Chat endpoint (requires `X-DRAIN-Voucher` header)
- `GET /health` — Health check

### 4. Deploy

**Railway (recommended):**
1. Connect your GitHub repo
2. Set root directory to `providers/hs58-openai`
3. Add environment variables
4. Deploy

**Any platform:** Docker, Render, Fly.io, VPS — anything that can run Node.js.

### 5. Register

Go to [handshake58.com/directory](https://handshake58.com/directory) and submit your provider.

---

## How DRAIN Payments Work

### Payment Flow

```
1. Agent opens a DRAIN channel
   → Deposits USDC into smart contract
   → Channel has an expiry (e.g., 24 hours)

2. Agent sends requests to your API
   → Each request includes a signed voucher (X-DRAIN-Voucher header)
   → Voucher = "I authorize payment of X USDC from channel Y"
   → Your provider validates the signature and serves the request

3. Provider claims earned USDC
   → Call contract.claim() with the highest voucher
   → USDC transfers to your wallet
   → Auto-claim runs every 10 minutes for expiring channels
```

### Voucher Format

Agents send this header with every request:

```http
X-DRAIN-Voucher: {
  "channelId": "0x...",
  "amount": "1500000",
  "nonce": "3",
  "signature": "0x..."
}
```

Your provider validates:
1. **Channel exists** on the DRAIN contract
2. **You are the provider** for this channel
3. **Signature is valid** (EIP-712 typed data, signed by channel consumer)
4. **Amount covers** the estimated request cost
5. **Nonce is incrementing** (prevents replay)

All of this is handled automatically by the provider templates.

### Response Headers

Your provider returns cost info to the agent:

```http
X-DRAIN-Cost: 8250          # Cost of this request (USDC wei)
X-DRAIN-Total: 1508250      # Total spent in this channel
X-DRAIN-Remaining: 8491750  # Remaining deposit
X-DRAIN-Channel: 0x...      # Channel ID
```

---

## Auto-Claim Protection

The provider templates include automatic expiry protection:

- **Every 10 minutes**, the provider checks all active channels
- If a channel is **expiring within 1 hour**, it claims immediately
- This ensures you **never lose earned funds** from expired channels
- Manual claim is also available: `POST /v1/admin/claim?force=true`

The DRAIN contract also protects you:
- Agents **cannot close** a channel before its expiry
- You can claim **at any time** before expiry
- After expiry, the agent can only reclaim the **unclaimed remainder**

---

## Pricing Your Services

Prices are set as a **markup** on upstream API costs. The provider template handles this automatically.

```env
# 50% markup means you charge 1.5x the upstream cost
MARKUP_PERCENT=50
```

Example with 50% markup on GPT-4o:
- OpenAI charges: $2.50 / $10.00 per million tokens (input/output)
- You charge: $3.75 / $15.00 per million tokens
- Your profit: $1.25 / $5.00 per million tokens

The marketplace fetches your pricing from `/v1/pricing` and displays it to agents.

---

## Building a Custom Provider

If the templates don't fit your use case, build your own using `DrainService`:

```typescript
import { DrainService } from './drain.js';
import { VoucherStorage } from './storage.js';

// Initialize
const storage = new VoucherStorage('./data/vouchers.json');
const drain = new DrainService(config, storage);

// Start auto-claim
drain.startAutoClaim(10, 3600);

// In your request handler:
app.post('/v1/chat/completions', async (req, res) => {
  // 1. Parse voucher
  const voucher = drain.parseVoucherHeader(req.headers['x-drain-voucher']);
  if (!voucher) return res.status(402).json({ error: 'voucher_required' });

  // 2. Validate (checks signature, channel, amount, nonce)
  const result = await drain.validateVoucher(voucher, estimatedCost);
  if (!result.valid) return res.status(402).json({ error: result.error });

  // 3. Serve your AI response
  const response = await yourAIBackend(req.body);

  // 4. Store voucher for claiming
  drain.storeVoucher(voucher, result.channel, actualCost);

  // 5. Return with cost headers
  res.set({
    'X-DRAIN-Cost': actualCost.toString(),
    'X-DRAIN-Total': result.channel.totalCharged.toString(),
  }).json(response);
});
```

### Required Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/pricing` | GET | Models and prices (called by marketplace) |
| `/v1/models` | GET | OpenAI-compatible model list |
| `/v1/chat/completions` | POST | Chat endpoint with `X-DRAIN-Voucher` |
| `/health` | GET | Health check (called by marketplace) |

### Optional Admin Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/admin/claim` | POST | Trigger manual claim (`?force=true` ignores threshold) |
| `/v1/admin/stats` | GET | Provider statistics |
| `/v1/admin/vouchers` | GET | View pending vouchers |

---

## Becoming a Bittensor Miner

To earn TAO emissions and get the "TAO Verified" tier:

1. **Create a Bittensor wallet** and register on Subnet 58
2. **Run the miner software** from [HS58-validator](https://github.com/Handshake58/HS58-validator)
3. The miner:
   - Responds to validator health checks with a wallet ownership proof
   - Auto-registers on the marketplace with sr25519 hotkey signature
   - Points to your existing provider API
4. **Validators score you** based on:
   - 60% DRAIN claims (real USDC payments you received)
   - 40% Availability (responding to validator checks)

```bash
# Miner environment variables
POLYGON_WALLET=0xYourProviderAddress
POLYGON_PRIVATE_KEY=0x...
API_URL=https://your-provider.com
MARKETPLACE_URL=https://handshake58.com
```

---

## Contract Details

| Contract | Address | Network |
|----------|---------|---------|
| DRAIN Channel | `0x1C1918C99b6DcE977392E4131C91654d8aB71e64` | Polygon Mainnet |
| USDC | `0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359` | Polygon Mainnet |

**Key contract functions:**
- `claim(channelId, amount, nonce, signature)` — Claim payment
- `getChannel(channelId)` — Get channel details (consumer, provider, deposit, claimed, expiry)
- `getBalance(channelId)` — Get remaining balance

[View on Polygonscan](https://polygonscan.com/address/0x1C1918C99b6DcE977392E4131C91654d8aB71e64)

---

## Production Checklist

- [ ] HTTPS enabled (required for marketplace registration)
- [ ] Private key secured in environment variables
- [ ] Persistent storage for vouchers (database recommended for high volume)
- [ ] Auto-claim running (enabled by default in templates)
- [ ] `/v1/pricing` returns correct models and prices
- [ ] `/health` returns 200 OK
- [ ] Registered on [handshake58.com/directory](https://handshake58.com/directory)

---

## FAQ

**How often are payments claimed?**
Auto-claim checks every 10 minutes and claims channels expiring within 1 hour. You can also trigger manual claims via `/v1/admin/claim`.

**What if a consumer doesn't use the full deposit?**
They get the unused portion back after channel expiry. You keep everything that was claimed.

**Can I change my prices?**
Yes, update your `MARKUP_PERCENT` env var and restart. The marketplace fetches fresh pricing from your `/v1/pricing` endpoint.

**What if my server goes down?**
Agents can't make new requests, but your unclaimed vouchers are still valid. The auto-claim will process them when you restart (if channels haven't expired).

**How much MATIC do I need for claiming?**
Each claim costs ~$0.02 in gas. Keep ~$1 worth of MATIC in your provider wallet.

---

## Support

- [Marketplace](https://handshake58.com)
- [GitHub Issues](https://github.com/Handshake58/DRAIN-marketplace/issues)
- [Architecture Docs](https://github.com/Handshake58/DRAIN-marketplace/blob/main/ARCHITECTURE.md)
- [Smart Contract on Polygonscan](https://polygonscan.com/address/0x1C1918C99b6DcE977392E4131C91654d8aB71e64)

---

Handshake58 &copy; 2026 — Trustless AI payments powered by DRAIN Protocol & Bittensor
