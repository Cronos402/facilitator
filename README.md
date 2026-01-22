![Cronos402 Logo](https://raw.githubusercontent.com/Cronos402/assets/main/Cronos402-logo-light.svg)

# Cronos402 Facilitator Proxy

High-availability passthrough proxy for Cronos x402 facilitators with automatic failover.

Production URL: https://facilitator.cronos402.dev

## Overview

The Facilitator Proxy provides a reliable, high-availability interface to the Cronos x402 facilitator service. It implements automatic failover, retry logic, and load balancing across multiple upstream facilitator endpoints, ensuring payment verification and settlement requests succeed even during network issues or upstream downtime.

## Architecture

- **Framework**: Express.js with TypeScript
- **Pattern**: Transparent proxy with health checks
- **Failover**: Automatic upstream switching
- **Retry Logic**: Exponential backoff
- **Deployment**: Edge and serverless compatible

## Features

- Passthrough proxy to upstream x402 facilitators
- On-demand health checks (stateless)
- Automatic failover and retry logic
- Load balancing among healthy targets
- Request timeout protection
- Response caching for supported endpoints
- Detailed logging and monitoring

## Upstream Targets

- `facilitator.cronoslabs.org` (Cronos official)

Additional upstreams can be configured for redundancy.

## API Endpoints

Proxied x402 facilitator endpoints:

### GET /supported
Get supported payment schemes and networks.

Response:
```json
{
  "networks": ["cronos-testnet", "cronos-mainnet"],
  "assets": {
    "cronos-testnet": ["devUSDC.e", "TCRO"],
    "cronos-mainnet": ["USDC.e", "CRO"]
  }
}
```

### POST /verify
Verify a payment authorization.

Request:
```json
{
  "type": "transfer_with_authorization",
  "from": "0x...",
  "to": "0x...",
  "value": "1000000",
  "validBefore": 1234567890,
  "signature": "0x..."
}
```

### POST /settle
Settle a verified payment on-chain.

Request:
```json
{
  "type": "transfer_with_authorization",
  "from": "0x...",
  "to": "0x...",
  "value": "1000000",
  "validBefore": 1234567890,
  "signature": "0x..."
}
```

## Configuration

Default settings:

- **Max Retries**: 3 attempts
- **Base Delay**: 1000ms
- **Max Delay**: 10000ms
- **Request Timeout**: 10 seconds
- **Exponential Backoff**: Enabled

Configure via environment variables:

```env
UPSTREAM_URL=https://facilitator.cronoslabs.org/v2/x402
MAX_RETRIES=3
TIMEOUT_MS=10000
ENABLE_CACHING=true
CACHE_TTL=60
```

## Development

```bash
pnpm install
pnpm dev
```

Server runs on `http://localhost:3004`

## Health Monitoring

### GET /health
Proxy health check.

Response:
```json
{
  "status": "healthy",
  "uptime": 12345,
  "upstreams": {
    "facilitator.cronoslabs.org": "healthy"
  }
}
```

### GET /status
Detailed upstream status with latency metrics.

Response:
```json
{
  "upstreams": [
    {
      "url": "https://facilitator.cronoslabs.org",
      "status": "healthy",
      "latency": 45,
      "lastCheck": "2024-01-22T12:00:00Z"
    }
  ]
}
```

## Response Headers

Custom headers added by proxy:

- `X-Proxy-Target`: Upstream target that handled the request
- `X-Proxy-Attempt`: Retry attempt number (1-3)
- `X-Proxy-Timestamp`: Request timestamp
- `X-Cache-Status`: Cache hit/miss status

## Deployment

### Production Build

```bash
pnpm build
NODE_ENV=production pnpm start
```

### Docker

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile
COPY . .
RUN pnpm build
EXPOSE 3004
CMD ["pnpm", "start"]
```

### Environment Requirements

- HTTPS endpoint in production
- Low-latency network to upstream
- Monitoring for health endpoints

## Monitoring

The proxy provides Prometheus-compatible metrics:

- Request count by endpoint
- Response time percentiles
- Upstream health status
- Retry count distribution
- Error rate by type

## Failover Behavior

1. Try primary upstream
2. If timeout/error, retry with exponential backoff
3. After 3 failures, mark upstream as unhealthy
4. Switch to next available upstream
5. Periodically check failed upstreams for recovery

## Resources

- **Documentation**: [docs.cronos402.dev/facilitator](https://docs.cronos402.dev)
- **Cronos Facilitator**: [docs.cronos.org/cronos-x402-facilitator](https://docs.cronos.org/cronos-x402-facilitator)
- **GitHub**: [github.com/Cronos402/facilitator](https://github.com/Cronos402/facilitator)

## License

MIT
