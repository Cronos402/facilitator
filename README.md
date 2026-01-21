# Cronos402 Facilitator Proxy

High-availability passthrough proxy for x402 facilitators with automatic failover. Optimized for edge and serverless environments.

## Features

- Passthrough proxy to upstream x402 facilitators
- On-demand health checks (stateless)
- Automatic failover and retry logic
- Load balancing among healthy targets

## Upstream Targets

- `facilitator.cronoslabs.org` (Cronos official)

## API Endpoints

Proxied x402 facilitator endpoints:

- `GET /supported` - Supported payment schemes and networks
- `POST /verify` - Verify a payment
- `POST /settle` - Settle a payment

## Configuration

- Max Retries: 3 attempts
- Base Delay: 1000ms
- Max Delay: 10000ms
- Request Timeout: 10 seconds

## Development

```bash
pnpm dev
```

## Endpoints

- `GET /health` - Proxy health check
- `GET /status` - Detailed upstream status
- `*` - Proxied to upstream facilitators

## Response Headers

- `X-Proxy-Target` - Upstream target that handled the request
- `X-Proxy-Attempt` - Retry attempt number
- `X-Proxy-Timestamp` - Request timestamp
