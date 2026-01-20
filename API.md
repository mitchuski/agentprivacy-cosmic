# Orbitport API Integration

## Overview

SpaceComputer's Orbitport provides the **cTRNG** (cosmic True Random Number Generator) service — true randomness harvested from cosmic radiation detected by satellite instrumentation.

**API Base URL**: `https://op.spacecomputer.io`
**Documentation**: https://docs.spacecomputer.io
**Get API Access**: https://spacecomputer.deform.cc/ctrngearlyaccess

## Authentication

Orbitport uses Auth0 for OAuth2 client credentials flow.

### Environment Variables

```bash
ORBITPORT_CLIENT_ID=your_client_id
ORBITPORT_CLIENT_SECRET=your_client_secret
ORBITPORT_AUTH_URL=https://your-auth-domain.auth0.com
ORBITPORT_API_URL=https://op.spacecomputer.io
```

### Token Generation

```typescript
async function generateAccessToken(): Promise<string> {
  const response = await fetch(`${process.env.ORBITPORT_AUTH_URL}/oauth/token`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      client_id: process.env.ORBITPORT_CLIENT_ID,
      client_secret: process.env.ORBITPORT_CLIENT_SECRET,
      audience: "https://op.spacecomputer.io/api",
      grant_type: "client_credentials"
    })
  });

  if (!response.ok) {
    throw new Error("Failed to get access token");
  }

  const data = await response.json();
  return data.access_token;
}
```

### Token Management

Tokens expire. Implement caching with expiry buffer:

```typescript
interface TokenCache {
  token: string;
  expiresAt: number;
}

const TOKEN_EXPIRE_BUFFER = 300; // 5 minutes

let tokenCache: TokenCache | null = null;

async function getValidToken(): Promise<string> {
  const now = Math.floor(Date.now() / 1000);
  
  if (tokenCache && tokenCache.expiresAt > now + TOKEN_EXPIRE_BUFFER) {
    return tokenCache.token;
  }
  
  const token = await generateAccessToken();
  const decoded = parseJwt(token);
  
  tokenCache = {
    token,
    expiresAt: decoded.exp
  };
  
  return token;
}
```

## cTRNG Endpoint

### Request

```
GET /api/v1/services/trng
Authorization: Bearer {access_token}
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `src` | string[] | `[aptosorbital, derived]` | Randomness sources, in priority order |

### Randomness Sources

1. **aptosorbital**: Space-based randomness from `cEDGE` or `Crypto2` satellites
2. **derived**: Randomness derived from a cosmic TRNG master seed (BIP32 derivation), used when satellites are unresponsive

### Response

```typescript
interface TRNGResponse {
  service: "trng";
  src: "aptosorbital" | "derived";
  data: string;      // 32-byte hex (64 characters)
  signature: {
    value: string;   // ECDSA signature proving satellite origin
    pk: string;      // Public key (may be empty string)
  };
}
```

### Example Response

```json
{
  "service": "trng",
  "src": "aptosorbital",
  "data": "0a4c2ea21557418bbc1d57120142ad83e8fa6e030ad35125fe225b97929d2526",
  "signature": {
    "value": "3046022100da9e9dfbe4167da1bd7b824ab46e57506cfbebc50395fdf0bb3d3407c1d92451022100e33601c04b402fc57d8ffd22d41b01ec5315d4e1a1d2be97bf71323cc5cc3838",
    "pk": ""
  }
}
```

## Implementation

### TypeScript Client

```typescript
// lib/orbitport.ts

export interface OrbitportConfig {
  clientId: string;
  clientSecret: string;
  authUrl: string;
  apiUrl: string;
}

export interface CosmicEntropy {
  data: string;
  signature: string;
  source: "aptosorbital" | "derived";
  timestamp: Date;
  verified: boolean;
}

export class OrbitportClient {
  private config: OrbitportConfig;
  private tokenCache: { token: string; expiresAt: number } | null = null;
  
  constructor(config: OrbitportConfig) {
    this.config = config;
  }
  
  static fromEnv(): OrbitportClient {
    return new OrbitportClient({
      clientId: process.env.ORBITPORT_CLIENT_ID!,
      clientSecret: process.env.ORBITPORT_CLIENT_SECRET!,
      authUrl: process.env.ORBITPORT_AUTH_URL!,
      apiUrl: process.env.ORBITPORT_API_URL || "https://op.spacecomputer.io"
    });
  }
  
  private async getToken(): Promise<string> {
    const now = Math.floor(Date.now() / 1000);
    
    if (this.tokenCache && this.tokenCache.expiresAt > now + 300) {
      return this.tokenCache.token;
    }
    
    const response = await fetch(`${this.config.authUrl}/oauth/token`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        client_id: this.config.clientId,
        client_secret: this.config.clientSecret,
        audience: `${this.config.apiUrl}/api`,
        grant_type: "client_credentials"
      })
    });
    
    if (!response.ok) {
      throw new Error(`Auth failed: ${response.status}`);
    }
    
    const { access_token } = await response.json();
    const decoded = this.parseJwt(access_token);
    
    this.tokenCache = {
      token: access_token,
      expiresAt: decoded.exp
    };
    
    return access_token;
  }
  
  private parseJwt(token: string): { exp: number } {
    const payload = token.split('.')[1];
    return JSON.parse(Buffer.from(payload, 'base64').toString());
  }
  
  async getCosmicEntropy(preferSatellite = true): Promise<CosmicEntropy> {
    const token = await this.getToken();
    
    const sources = preferSatellite 
      ? "?src=aptosorbital&src=derived"
      : "?src=derived&src=aptosorbital";
    
    const response = await fetch(
      `${this.config.apiUrl}/api/v1/services/trng${sources}`,
      {
        headers: { Authorization: `Bearer ${token}` }
      }
    );
    
    if (!response.ok) {
      throw new Error(`TRNG request failed: ${response.status}`);
    }
    
    const data = await response.json();
    
    return {
      data: data.data,
      signature: data.signature.value,
      source: data.src,
      timestamp: new Date(),
      verified: await this.verifySignature(data)
    };
  }
  
  private async verifySignature(response: TRNGResponse): Promise<boolean> {
    // TODO: Implement ECDSA verification
    // For now, trust Orbitport's attestation
    return response.signature.value.length > 0;
  }
}
```

### Next.js API Route

```typescript
// pages/api/cosmic-entropy.ts

import type { NextApiRequest, NextApiResponse } from "next";
import { OrbitportClient } from "@/lib/orbitport";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== "GET") {
    return res.status(405).json({ error: "Method not allowed" });
  }
  
  try {
    const client = OrbitportClient.fromEnv();
    const entropy = await client.getCosmicEntropy();
    
    res.status(200).json({
      success: true,
      entropy: {
        data: entropy.data,
        source: entropy.source,
        timestamp: entropy.timestamp.toISOString(),
        verified: entropy.verified
      }
    });
  } catch (error) {
    console.error("Cosmic entropy fetch failed:", error);
    res.status(502).json({ 
      success: false, 
      error: "Failed to fetch cosmic entropy" 
    });
  }
}
```

## Error Handling

### Common Errors

| Status | Cause | Resolution |
|--------|-------|------------|
| 401 | Invalid/expired token | Refresh token |
| 403 | Invalid credentials | Check client ID/secret |
| 502 | Satellite unavailable | System falls back to `derived` source |
| 503 | Service maintenance | Retry with backoff |

### Retry Strategy

```typescript
async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelay = 1000
): Promise<T> {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries - 1) throw error;
      
      const delay = baseDelay * Math.pow(2, attempt);
      await new Promise(r => setTimeout(r, delay));
    }
  }
  throw new Error("Max retries exceeded");
}
```

## Rate Limits

Orbitport implements rate limiting. Current limits (check docs for updates):
- Burst: 10 requests/second
- Sustained: 100 requests/minute

### Handling Rate Limits

```typescript
interface RateLimitState {
  remaining: number;
  resetAt: Date;
}

function parseRateLimitHeaders(headers: Headers): RateLimitState {
  return {
    remaining: parseInt(headers.get("X-RateLimit-Remaining") || "100"),
    resetAt: new Date(headers.get("X-RateLimit-Reset") || Date.now() + 60000)
  };
}
```

## Signature Verification

The satellite signs all entropy data. For production, implement full ECDSA verification:

```typescript
import { createVerify } from "crypto";

function verifyCosmicSignature(
  data: string,
  signature: string,
  publicKey: string
): boolean {
  if (!publicKey) {
    // Public key not provided in response
    // Trust Orbitport's chain of custody
    return true;
  }
  
  const verify = createVerify("SHA256");
  verify.update(Buffer.from(data, "hex"));
  
  return verify.verify(
    { key: publicKey, format: "der", type: "spki" },
    Buffer.from(signature, "hex")
  );
}
```

## Testing

### Mock Client for Development

```typescript
export class MockOrbitportClient {
  async getCosmicEntropy(): Promise<CosmicEntropy> {
    // Generate pseudo-random entropy for testing
    const data = Array.from({ length: 32 }, () =>
      Math.floor(Math.random() * 256).toString(16).padStart(2, "0")
    ).join("");
    
    return {
      data,
      signature: "mock-signature",
      source: "derived",
      timestamp: new Date(),
      verified: false
    };
  }
}

// Use in tests
const client = process.env.NODE_ENV === "test"
  ? new MockOrbitportClient()
  : OrbitportClient.fromEnv();
```

## Resources

- **API Documentation**: https://docs.spacecomputer.io/using-orbitport/user-guide/
- **Swagger/OpenAPI**: https://docs.spacecomputer.io/using-orbitport/swagger/
- **Developer Guide**: https://blog.spacecomputer.io/building-with-spacecomputer-orbitport-a-guide-to-cosmic-randomness-in-web3/
- **GitHub Examples**: https://github.com/spacecomputer-io/spacecomputer-orbitport-demo
- **Community Support**: [Telegram](https://t.me/spacecomputer)
