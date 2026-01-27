# DRAIN Marketplace

<div align="center">
  <img src="docs/marketplace-banner.png" alt="DRAIN Marketplace" width="800">
</div>

**Provider Directory & Discovery Platform for DRAIN Protocol**

Discover, register, and manage DRAIN-compatible AI providers with trustless micropayments.

**🟢 Live Marketplace**: https://believable-inspiration-production-b1c6.up.railway.app/directory

---

## What is DRAIN Marketplace?

DRAIN Marketplace is the discovery and registration platform for AI providers using the [DRAIN Protocol](https://github.com/kimbo128/DRAIN). It enables:

- **Provider Discovery** - Find AI providers with the models you need
- **Quality Control** - Admin-approved providers ensure reliability
- **Premium Placements** - Featured providers get better visibility
- **Health Monitoring** - Real-time status checks for all providers
- **MCP Integration** - Automatic provider discovery for AI agents

---

## Features

### For AI Providers

- **Easy Registration** - Submit your provider in minutes
- **Admin Approval** - Quality control ensures reliable providers
- **Premium Placement** - Get featured for better visibility
- **Health Monitoring** - Automatic uptime tracking
- **MCP API** - Automatic discovery by AI agents
- **[Integration Guide](docs/BECOME_A_PROVIDER.md)** - Complete documentation for providers

### For AI Agents & Developers

- **Provider Discovery** - Find providers with specific models
- **Live Status** - See which providers are online
- **Pricing Information** - Compare costs across providers
- **MCP Integration** - Automatic discovery via MCP Server
- **API Access** - Programmatic provider lookup

### For Marketplace Administrators

- **Provider Management** - Approve/reject submissions
- **Premium Control** - Manage featured placements
- **Health Monitoring** - Track provider uptime
- **Quality Assurance** - Ensure reliable providers

---

## How It Works

### For Providers

1. **Register** - Submit your provider details via the marketplace
2. **Review** - Admin reviews and tests your provider
3. **Approval** - Once approved, your provider appears in the directory
4. **Premium** - Optional premium placement for better visibility

### For Users

1. **Browse** - View all approved providers
2. **Filter** - Find providers with specific models
3. **Discover** - Use MCP Server for automatic discovery
4. **Connect** - Use provider address directly with DRAIN Protocol

---

## Live Marketplace

**🟢 Visit**: https://believable-inspiration-production-b1c6.up.railway.app/directory

**Features Available:**
- Provider directory with live status
- Provider registration form
- MCP API endpoint for agent discovery
- Health monitoring dashboard

---

## MCP Integration

The marketplace provides automatic provider discovery for the DRAIN MCP Server:

```bash
GET https://believable-inspiration-production-b1c6.up.railway.app/api/mcp/providers
```

**Response includes:**
- Provider information
- Available models
- Pricing details
- Live status

The DRAIN MCP Server automatically uses this endpoint to discover providers.

**Install MCP Server:**
```bash
npm install -g drain-mcp
```

See [DRAIN MCP Server](https://www.npmjs.com/package/drain-mcp) for full documentation.

---

## About DRAIN Protocol

This marketplace is built on top of [DRAIN Protocol](https://github.com/kimbo128/DRAIN).

**DRAIN Protocol** enables:
- Trustless payment channels with USDC
- EIP-712 voucher system for off-chain payments
- Smart contracts on Polygon (immutable, no admin keys)
- TypeScript SDK for developers
- MCP Server for AI agents

**Key Benefits:**
- **Micropayments** - Pay $0.000005 per request
- **No Credit Cards** - Permissionless crypto access
- **Agent-Compatible** - AI agents can pay autonomously
- **Low Fees** - ~$0.02 per transaction on Polygon

See [DRAIN Protocol](https://github.com/kimbo128/DRAIN) for full protocol documentation.

---

## Screenshots

*Screenshots coming soon*

<!--
![Marketplace Overview](docs/screenshots/marketplace-overview.png)
![Provider Registration](docs/screenshots/provider-registration.png)
![Premium Provider](docs/screenshots/premium-provider.png)
-->

---

## Getting Started

### For Providers

📖 **[Become a Provider Guide](docs/BECOME_A_PROVIDER.md)** - Complete integration documentation

**Quick Start:**
1. Clone the [reference provider](https://github.com/kimbo128/DRAIN/tree/main/provider)
2. Configure your environment and deploy
3. Register on the [Live Marketplace](https://believable-inspiration-production-b1c6.up.railway.app/directory)
4. Start earning!

**Requirements:**
- DRAIN-compatible API endpoint (OpenAI-compatible)
- Provider wallet address (Polygon)
- Models and pricing information

### For Developers

**Use MCP Server** (Recommended):
```bash
npm install -g drain-mcp
```

**Or use API directly:**
```bash
curl https://believable-inspiration-production-b1c6.up.railway.app/api/mcp/providers
```

**Or use DRAIN SDK:**
```bash
npm install @drain-protocol/sdk
```

See [DRAIN Protocol](https://github.com/kimbo128/DRAIN) for SDK documentation.

---

## Documentation

- **[Become a Provider](docs/BECOME_A_PROVIDER.md)** - Complete integration guide for AI providers
- **[Provider README](https://github.com/kimbo128/DRAIN/blob/main/provider/README.md)** - Technical reference implementation

## Related Projects

- **DRAIN Protocol**: [Core Protocol Repository](https://github.com/kimbo128/DRAIN) - Smart contracts, SDK, MCP Server
- **MCP Server**: [npm package](https://www.npmjs.com/package/drain-mcp) - AI agent integration
- **Reference Provider**: [Live API](https://drain-production-a9d4.up.railway.app) - Example implementation

---

## License

MIT License - See [DRAIN Protocol](https://github.com/kimbo128/DRAIN) for details.

---

**Note**: The marketplace application code is in a private repository for business reasons. This repository contains only public marketing and documentation materials.





