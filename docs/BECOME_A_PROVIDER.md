# Become a DRAIN Provider

Earn revenue by offering AI services through the DRAIN network. Accept trustless micropayments from autonomous AI agents without credit cards, chargebacks, or KYC requirements.

## Why Become a Provider?

| Benefit | Description |
|---------|-------------|
| 💰 **Instant Payments** | Receive USDC directly to your wallet via smart contract |
| 🔒 **Zero Chargebacks** | Blockchain-based payments are final and irreversible |
| 🌍 **Global Access** | Accept payments from anywhere, no geographic restrictions |
| 🤖 **Agent Compatible** | Tap into the growing autonomous AI agent economy |
| 📊 **Transparent Pricing** | You set your own prices per model |
| ⚡ **Low Fees** | Only ~$0.02 gas to claim accumulated payments |

---

## Overview

As a DRAIN provider, you offer AI inference services (LLMs, image generation, audio, etc.) and accept payment via DRAIN vouchers instead of traditional API keys. The integration is straightforward:

```
Consumer Opens Channel → Signs Vouchers → Sends to Your API → You Verify & Serve → Claim Payments
```

---

## Integration Steps

### Step 1: Set Up Your Wallet

Create an Ethereum wallet that will receive payments. This address is used to:
- Identify your provider in the DRAIN contract
- Receive claimed USDC payments
- Sign claim transactions

```bash
# Your provider address (public)
PROVIDER_ADDRESS=0xYourWalletAddress

# Your private key (keep secret!)
PROVIDER_PRIVATE_KEY=0x...
```

> ⚠️ **Security**: Never commit your private key. Use environment variables or a secrets manager.

### Step 2: Implement DRAIN Headers

Your API must accept the `X-DRAIN-Voucher` header and return DRAIN response headers.

#### Request Header

```http
X-DRAIN-Voucher: {
  "channelId": "0x...",
  "amount": "1000000",
  "nonce": "1", 
  "signature": "0x..."
}
```

#### Response Headers

```http
X-DRAIN-Cost: 8250
X-DRAIN-Total: 1008250
X-DRAIN-Remaining: 8991750
X-DRAIN-Channel: 0x...
```

### Step 3: Verify Vouchers

Before serving a request, verify:

1. **Signature is valid** (EIP-712 typed data)
2. **Channel exists** on the DRAIN contract
3. **Amount is sufficient** for the request cost
4. **Nonce is incrementing** (prevents replay attacks)
5. **Channel is yours** (provider address matches)

```javascript
// Pseudo-code for voucher verification
async function verifyVoucher(voucher) {
  // 1. Recover signer from EIP-712 signature
  const signer = recoverTypedDataSigner(voucher);
  
  // 2. Get channel from contract
  const channel = await drainContract.getChannel(voucher.channelId);
  
  // 3. Verify channel belongs to this provider
  if (channel.provider !== PROVIDER_ADDRESS) {
    throw new Error('wrong_provider');
  }
  
  // 4. Verify signer is channel consumer
  if (signer !== channel.consumer) {
    throw new Error('invalid_signature');
  }
  
  // 5. Verify amount is sufficient
  const estimatedCost = calculateCost(request);
  if (BigInt(voucher.amount) < channel.claimed + estimatedCost) {
    throw new Error('insufficient_funds');
  }
  
  return { channel, estimatedCost };
}
```

### Step 4: Store Vouchers & Claim Payments

Store the highest-value voucher for each channel. You can batch-claim payments periodically:

```javascript
// Store voucher if it's the highest for this channel
function storeVoucher(channelId, voucher) {
  const existing = vouchers.get(channelId);
  if (!existing || BigInt(voucher.amount) > BigInt(existing.amount)) {
    vouchers.set(channelId, voucher);
  }
}

// Claim payment from contract
async function claimPayment(channelId, voucher) {
  const tx = await drainContract.claim(
    channelId,
    voucher.amount,
    voucher.nonce,
    voucher.signature
  );
  await tx.wait();
  console.log('Payment claimed:', formatUSDC(voucher.amount));
}
```

---

## Quick Start with Reference Provider

Clone and run the DRAIN reference provider to get started quickly:

```bash
# Clone the repository
git clone https://github.com/kimbo128/DRAIN.git
cd DRAIN/provider

# Install dependencies
npm install

# Configure environment
cp env.example .env
# Edit .env with your settings

# Run the provider
npm run dev
```

The reference provider handles all voucher verification, token counting, and payment claiming out of the box.

📖 **Full technical documentation**: [provider/README.md](https://github.com/kimbo128/DRAIN/blob/main/provider/README.md)

---

## Pricing Your Services

Set competitive prices to attract users. Prices are specified in USDC (6 decimals):

```env
# Pricing in USDC wei per 1000 tokens
# $0.0001 per 1K tokens = 100 USDC wei
PRICE_GPT4O_MINI_INPUT=75       # $0.000075/1K
PRICE_GPT4O_MINI_OUTPUT=150     # $0.00015/1K
PRICE_GPT4O_INPUT=7500          # $0.0075/1K
PRICE_GPT4O_OUTPUT=22500        # $0.0225/1K
```

### Price Calculation Example

```
Request: 500 input tokens, 200 output tokens
Model: gpt-4o

Input cost:  500 × ($0.0075 / 1000) = $0.00375
Output cost: 200 × ($0.0225 / 1000) = $0.00450
Total cost:                         = $0.00825
```

### Pricing Strategy Tips

| Strategy | Description |
|----------|-------------|
| **Competitive** | Match or undercut existing API providers |
| **Premium** | Charge more for faster/higher quality models |
| **Volume** | Lower prices to attract more usage |
| **Margin** | Ensure your prices cover your costs + profit |

---

## Required API Endpoints

### `/v1/pricing` (GET)

Return your pricing information. This is called by the marketplace and consumers.

```json
{
  "provider": "0xYourAddress",
  "chainId": 137,
  "currency": "USDC",
  "decimals": 6,
  "models": {
    "gpt-4o": {
      "inputPer1kTokens": "0.0075",
      "outputPer1kTokens": "0.0225"
    },
    "gpt-4o-mini": {
      "inputPer1kTokens": "0.000075",
      "outputPer1kTokens": "0.00015"
    }
  }
}
```

### `/v1/chat/completions` (POST)

OpenAI-compatible chat endpoint. Supports streaming.

### `/v1/models` (GET)

List available models.

---

## Register on the Marketplace

Once your provider is running, register it on the DRAIN Marketplace:

1. Go to **[DRAIN Marketplace](https://believable-inspiration-production-b1c6.up.railway.app/directory)**
2. Click **"LIST_YOUR_API"**
3. Fill in your details:
   - Provider Name
   - API URL (must be publicly accessible)
   - Wallet Address (must match your provider)
   - Description & Contact Email
4. Your API will be **automatically tested**
5. Once approved, you'll appear in the marketplace

### Featured Placement

Want priority positioning? Check the "Featured Placement" option when registering. We'll contact you to discuss promotional terms.

---

## Production Checklist

Before going live, ensure:

- [ ] **Security**: Private key secured in environment variables
- [ ] **HTTPS**: API accessible via HTTPS only
- [ ] **Database**: Use persistent storage (not file-based)
- [ ] **Auth**: Protect admin endpoints
- [ ] **Rate Limiting**: Prevent abuse
- [ ] **Monitoring**: Log requests, errors, and revenue
- [ ] **Claiming**: Automate payment claiming (cron job recommended)
- [ ] **Backup**: Backup voucher storage regularly

---

## Contract Details

| Network | Contract Address | USDC |
|---------|------------------|------|
| **Polygon Mainnet** | `0x1C1918C99b6DcE977392E4131C91654d8aB71e64` | Native USDC |
| **Polygon Amoy** (Testnet) | `0x6D...` | Test USDC |

**Contract Functions:**
- `claim(channelId, amount, nonce, signature)` - Claim payment
- `getChannel(channelId)` - Get channel details
- `getBalance(channelId)` - Get remaining balance

---

## Support & Resources

| Resource | Link |
|----------|------|
| 📦 Reference Provider | [github.com/kimbo128/DRAIN/tree/main/provider](https://github.com/kimbo128/DRAIN/tree/main/provider) |
| 📖 Protocol Documentation | [github.com/kimbo128/DRAIN](https://github.com/kimbo128/DRAIN) |
| 🏪 Marketplace | [DRAIN Marketplace](https://believable-inspiration-production-b1c6.up.railway.app/directory) |
| 📜 Smart Contract | [Polygonscan](https://polygonscan.com/address/0x1C1918C99b6DcE977392E4131C91654d8aB71e64) |

---

## Example Implementations

### Minimal Express.js Provider

```javascript
import express from 'express';
import { verifyVoucher, storeVoucher, calculateCost } from './drain.js';
import { callOpenAI } from './openai.js';

const app = express();
app.use(express.json());

app.post('/v1/chat/completions', async (req, res) => {
  // 1. Parse voucher
  const voucherHeader = req.headers['x-drain-voucher'];
  if (!voucherHeader) {
    return res.status(402).json({ error: 'voucher_required' });
  }
  
  const voucher = JSON.parse(voucherHeader);
  
  // 2. Verify voucher
  try {
    await verifyVoucher(voucher);
  } catch (e) {
    return res.status(402).json({ error: e.message });
  }
  
  // 3. Call AI backend
  const response = await callOpenAI(req.body);
  
  // 4. Calculate cost and set headers
  const cost = calculateCost(response.usage);
  res.setHeader('X-DRAIN-Cost', cost.toString());
  res.setHeader('X-DRAIN-Total', voucher.amount);
  
  // 5. Store voucher for later claiming
  storeVoucher(voucher.channelId, voucher);
  
  // 6. Return response
  res.json(response);
});

app.listen(3000);
```

---

## FAQ

**Q: How often should I claim payments?**
A: Depends on your volume. Daily claiming is common. Set a threshold (e.g., $10) to batch claims and save on gas.

**Q: What if a consumer disputes a payment?**
A: DRAIN payments are trustless and final. The signed voucher is cryptographic proof of authorization.

**Q: Can I change my prices?**
A: Yes, update your `/v1/pricing` endpoint anytime. New channels will use your current prices.

**Q: What happens if my server goes down?**
A: Consumers can still close their channels and get refunds. Resume service when ready.

**Q: Do I need to KYC users?**
A: No. DRAIN is permissionless. Anyone with USDC can use your service.

---

## Ready to Start?

1. 📥 Clone the [reference provider](https://github.com/kimbo128/DRAIN/tree/main/provider)
2. ⚙️ Configure your environment
3. 🚀 Deploy to your server
4. 📝 Register on the [marketplace](https://believable-inspiration-production-b1c6.up.railway.app/directory)
5. 💰 Start earning!

---

*DRAIN © 2026 - Trustless payments for the AI economy*

