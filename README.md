# AgentPay

**AI Agent Payment Orchestration Layer**

Infrastructure enabling AI agents to autonomously execute blockchain transactions without human intervention.

---

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![ERC-4337](https://img.shields.io/badge/ERC--4337-Account%20Abstraction-purple.svg)](https://eips.ethereum.org/EIPS/eip-4337)
[![Multi-Chain](https://img.shields.io/badge/Multi--Chain-Supported-green.svg)](#supported-chains)

---

## Overview

AgentPay bridges the gap between AI decision-making and blockchain execution. While AI agents can reason about financial decisions, they traditionally cannot act on them. AgentPay provides a secure, scalable infrastructure layer that:

- **Translates intents to actions** — Agents express *what* they want; the system determines *how*
- **Abstracts complexity** — No need for agents to manage gas, keys, or protocol specifics
- **Ensures security** — Programmable authorization with spending limits and allowlists
- **Provides proof** — Execution verification and status tracking

---

## Key Features

| Feature | Description |
|---------|-------------|
| Intent-Based Execution | Declarative intents instead of raw transaction calldata |
| Account Abstraction (ERC-4337) | Smart contract wallets with programmable authorization |
| Gas Abstraction | Agents never need to hold native tokens — paymasters handle gas |
| Multi-Solver Routing | Competitive quotes from DEX aggregators and cross-chain solvers |
| Multi-Chain Support | Seamless execution across Ethereum, Arbitrum, Base, Optimism, and more |
| Security Policies | Spending limits, token allowlists, destination restrictions |
| Execution Proofs | On-chain verification and status tracking via webhooks |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              AI AGENTS                                   │
│                     (Express intents via API)                           │
└─────────────────────────────────┬───────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           API GATEWAY                                    │
│              Authentication • Rate Limiting • Routing                    │
└─────────────────────────────────┬───────────────────────────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    ▼                           ▼
         ┌──────────────────┐         ┌──────────────────┐
         │   INTENT ENGINE  │         │   STATE MANAGER  │
         │                  │         │                  │
         │ Parse • Validate │         │ Lifecycle Track  │
         │ Enrich • Route   │         │ Multi-Step Coord │
         └────────┬─────────┘         └──────────────────┘
                  │
                  ▼
         ┌──────────────────┐
         │  SOLVER ROUTER   │────────────────┐
         │                  │                │
         │ Quote Aggregation│      ┌─────────┴─────────┐
         │ Solution Ranking │      │  EXTERNAL SOLVERS │
         └────────┬─────────┘      │  • 1inch          │
                  │                │  • Socket/LI.FI   │
                  ▼                │  • CoW Protocol   │
         ┌──────────────────┐      └───────────────────┘
         │ EXECUTION ENGINE │
         │                  │
         │ UserOp Build     │──────► Bundlers (Pimlico, Alchemy)
         │ Paymaster Coord  │──────► Paymasters
         │ Signing          │──────► KMS/HSM
         └──────────────────┘
```

---

## Quick Start

### Prerequisites

- Go 1.21+ or Node.js 20+
- PostgreSQL 15+
- Redis 7+
- Access to RPC endpoints (Alchemy, Infura, or QuickNode)
- Bundler API access (Pimlico, Alchemy Bundler)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/agent-pay.git
cd agent-pay

# Install dependencies
make install

# Configure environment
cp .env.example .env
# Edit .env with your API keys and configuration

# Run database migrations
make migrate

# Start the service
make run
```

### Configuration

Create a `.env` file with the following variables:

```env
# Database
DATABASE_URL=postgres://user:pass@localhost:5432/agentpay

# Redis
REDIS_URL=redis://localhost:6379

# RPC Endpoints
ETH_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY
ARB_RPC_URL=https://arb-mainnet.g.alchemy.com/v2/YOUR_KEY
BASE_RPC_URL=https://base-mainnet.g.alchemy.com/v2/YOUR_KEY

# Bundler
BUNDLER_API_KEY=your_pimlico_key

# Paymaster
PAYMASTER_API_KEY=your_paymaster_key

# Solvers
ONEINCH_API_KEY=your_1inch_key
SOCKET_API_KEY=your_socket_key

# Security
JWT_SECRET=your_jwt_secret
KMS_KEY_ID=your_kms_key_id
```

---

## API Reference

### Create Intent

```http
POST /v1/intents
```

**Request:**

```json
{
  "type": "swap",
  "source": {
    "chain": "arbitrum",
    "token": "USDC",
    "amount": "1000000000"
  },
  "destination": {
    "chain": "arbitrum",
    "token": "ETH"
  },
  "constraints": {
    "maxSlippage": 100,
    "deadline": 1706140800
  }
}
```

**Response:**

```json
{
  "intentId": "int_abc123",
  "status": "quoted",
  "quote": {
    "outputAmount": "450000000000000000",
    "gasEstimate": "150000",
    "expiresAt": "2024-01-25T12:00:00Z"
  }
}
```

### Execute Intent

```http
POST /v1/intents/{intentId}/execute
```

### Get Intent Status

```http
GET /v1/intents/{intentId}
```

### Get Wallet Balances

```http
GET /v1/wallets/{address}/balances
```

---

## Supported Chains

| Chain | Chain ID | Status |
|-------|----------|--------|
| Ethereum Mainnet | 1 | Supported |
| Arbitrum One | 42161 | Supported |
| Base | 8453 | Supported |
| Optimism | 10 | Supported |
| Polygon | 137 | Planned |
| zkSync Era | 324 | Planned |

---

## Security

AgentPay implements defense-in-depth security:

- **Key Management**: HSM/KMS-backed signing keys
- **Spending Limits**: Per-agent, per-window spend caps
- **Allowlists**: Token and destination address restrictions  
- **Rate Limiting**: API and execution rate controls
- **Audit Logging**: Complete activity trails
- **Calldata Validation**: Solver response verification

---

## Monitoring

Built-in observability with:

- Prometheus metrics endpoint at `/metrics`
- Structured JSON logging
- Distributed tracing support
- Health check at `/health`

---

## Testing

```bash
# Run unit tests
make test

# Run integration tests
make test-integration

# Run e2e tests (requires testnet)
make test-e2e
```

---

## Documentation

- [Technical Specification](./docs/TECHNICAL_SPEC.md)
- [API Documentation](./docs/API.md)
- [Security Model](./docs/SECURITY.md)
- [Deployment Guide](./docs/DEPLOYMENT.md)
- [Troubleshooting](./docs/TROUBLESHOOTING.md)

---

## Roadmap

- [x] Core intent engine
- [x] ERC-4337 integration
- [x] Multi-solver routing
- [ ] Additional solver integrations (CoW, UniswapX)
- [ ] Natural language intent parsing
- [ ] Batched execution
- [ ] Scheduled intents

---

## Contributing

Contributions are welcome. Please read our [Contributing Guide](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

- [ERC-4337 Team](https://eips.ethereum.org/EIPS/eip-4337) — Account Abstraction standard
- [Pimlico](https://pimlico.io) — Bundler infrastructure
- [1inch](https://1inch.io) — DEX aggregation
- [Socket](https://socket.tech) — Cross-chain bridging

---

**Built for the autonomous agent economy**

[Documentation](./docs) | [Report Bug](https://github.com/your-org/agent-pay/issues) | [Request Feature](https://github.com/your-org/agent-pay/issues)
