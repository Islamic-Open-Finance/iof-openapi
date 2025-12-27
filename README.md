# IOF OpenAPI Specification

Official OpenAPI 3.1 specification for the Islamic Open Finance (IOF) Platform.

[![OpenAPI](https://img.shields.io/badge/OpenAPI-3.1.0-green.svg)](https://spec.openapis.org/oas/v3.1.0)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

## Overview

The IOF Platform provides 29 specialized Rails for Shariah-compliant banking operations:

- **Core Rails (5)**: Contracts, Cards, Clearing, Treasury, Shariah
- **Access & Identity (3)**: IAM, Organizations, Workspaces
- **Operations (6)**: Transactions, Transfers, Reconciliation, Events, Webhooks, Audit
- **Financial (4)**: Ledger, Products, Pricing, Limits
- **Governance (9)**: KYC/KYB, AML/CFT, Consent/Privacy, Disputes, Collections, Zakat, Impact, Observability
- **Developer & Partner (2)**: Developer, Partners

All APIs follow AAOIFI standards for Islamic finance compliance.

## Quick Start

### View the Specification

**Online**:

- [Swagger UI](https://editor.swagger.io/?url=https://raw.githubusercontent.com/Islamic-Open-Finance/iof-openapi/main/spec/openapi.yaml)
- [ReDoc](https://redocly.github.io/redoc/?url=https://raw.githubusercontent.com/Islamic-Open-Finance/iof-openapi/main/spec/openapi.yaml)

**Local**:

```bash
# Clone the repository
git clone https://github.com/Islamic-Open-Finance/iof-openapi.git
cd iof-openapi

# Install dependencies
npm install

# View in Swagger UI
npm run serve

# Generate HTML documentation
npm run build:docs
```

### Generate Client SDK

```bash
# TypeScript/JavaScript
npm run generate:typescript

# Python
npm run generate:python

# Java
npm run generate:java

# Go
npm run generate:go
```

## Environments

| Environment    | Base URL                                     | Purpose                       |
| -------------- | -------------------------------------------- | ----------------------------- |
| **Production** | `https://api.islamicopenfinance.com`         | Live production environment   |
| **UAT**        | `https://api.uat.islamicopenfinance.com`     | User acceptance testing       |
| **Sandbox**    | `https://api.sandbox.islamicopenfinance.com` | Developer sandbox (test data) |

## Authentication

The IOF Platform supports two authentication methods:

### 1. Bearer Token (OAuth 2.0)

For user-facing applications:

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
```

**Flow**:

1. Obtain authorization code: `GET /auth/authorize`
2. Exchange for tokens: `POST /auth/token`
3. Use access token for API calls
4. Refresh when expired: `POST /auth/refresh`

### 2. API Key

For service-to-service authentication:

```http
X-API-Key: iof_live_abc123xyz789
```

**Obtain API Key**:

1. Login to Developer Portal: https://developers.islamicopenfinance.com
2. Navigate to API Keys section
3. Create new API key with appropriate scopes

## Key Features

### Shariah Compliance

All endpoints enforce Shariah compliance:

```json
POST /api/v1/shariah/evaluations
{
  "entity_type": "CONTRACT",
  "entity_data": {
    "type": "MURABAHA",
    "asset_category": "VEHICLE",
    "cost_price": 50000,
    "profit_amount": 5000
  }
}
```

Response includes compliance status and any breaches:

```json
{
  "evaluation_id": "EVAL-123",
  "compliant": true,
  "rules_evaluated": [
    {
      "rule_id": "MUR_ASSET_HALAL",
      "result": "PASS"
    },
    {
      "rule_id": "MUR_FIXED_MARKUP",
      "result": "PASS"
    }
  ]
}
```

### Multi-Tenant Architecture

All requests are scoped to tenant and workspace:

```http
GET /api/v1/contracts/murabaha
X-Tenant-ID: tenant_abc123
X-Workspace-ID: prod
```

### Pagination

All list endpoints support cursor-based pagination:

```http
GET /api/v1/contracts/murabaha?page=1&limit=20
```

Response includes pagination metadata:

```json
{
  "data": [...],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "total_pages": 8
  }
}
```

### Error Handling

Consistent error format across all endpoints:

```json
{
  "error": {
    "code": "SHARIAH_BREACH",
    "message": "Contract violates Shariah rules",
    "details": [
      {
        "rule_id": "MUR_ASSET_HALAL",
        "rule_name": "Asset Must Be Halal",
        "breach": "Asset category 'ALCOHOL' is forbidden"
      }
    ],
    "trace_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

## API Rails

### Contracts Rail

Manage 28 AAOIFI-compliant Islamic contract types:

- **Murabaha**: Cost-plus financing
- **Ijarah**: Leasing
- **Musharaka**: Partnership
- **Mudaraba**: Profit-sharing
- **Salam**: Forward sale
- **Istisna**: Manufacturing contract
- ... and 22 more

```http
POST /api/v1/contracts/murabaha
GET  /api/v1/contracts/murabaha
GET  /api/v1/contracts/murabaha/{id}
PATCH /api/v1/contracts/murabaha/{id}
POST /api/v1/contracts/murabaha/{id}/activate
```

### Cards Rail

Shariah-compliant card operations with ISO 8583 support:

```http
POST /api/v1/cards
GET  /api/v1/cards
GET  /api/v1/cards/{id}
GET  /api/v1/cards/{id}/authorizations
POST /api/v1/cards/{id}/block
```

**Forbidden MCCs** (automatically blocked):

- 5921: Liquor Stores
- 7995: Gambling
- 9754: Pork Products
- 6211: Securities Brokers (interest-based)

### AML/CFT Rail

Anti-money laundering and sanctions screening:

```http
POST /api/v1/aml/screening
GET  /api/v1/aml/screening/{id}
GET  /api/v1/aml/alerts
POST /api/v1/aml/alerts/{id}/resolve
```

### Metadata Rail

Entity registry and data lineage:

```http
POST /api/v1/metadata/entities
GET  /api/v1/metadata/entities
GET  /api/v1/metadata/lineage?rail=contracts&entity_id=CNT-789
```

### Taxonomy Rail

Controlled vocabularies and term management:

```http
POST /api/v1/taxonomy/taxonomies
GET  /api/v1/taxonomy/terms?taxonomy_id=iof.contracts
POST /api/v1/taxonomy/terms
```

## Rate Limits

| Tier           | Requests/min | Requests/day | Burst  |
| -------------- | ------------ | ------------ | ------ |
| **Free**       | 60           | 1,000        | 10     |
| **Pro**        | 1,000        | 50,000       | 50     |
| **Enterprise** | Custom       | Custom       | Custom |

Rate limit headers:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1735084800
```

## Webhooks

Subscribe to real-time events:

```http
POST /api/v1/webhooks/subscriptions
{
  "url": "https://your-app.com/webhooks",
  "events": [
    "contract.created",
    "contract.activated",
    "card.authorization.approved",
    "shariah.breach.detected"
  ]
}
```

Webhook payload includes HMAC signature:

```http
X-IOF-Signature: sha256=a1b2c3d4...
X-IOF-Timestamp: 1735084800
```

## Code Examples

### Create Murabaha Contract (JavaScript)

```javascript
const response = await fetch(
  "https://api.islamicopenfinance.com/api/v1/contracts/murabaha",
  {
    method: "POST",
    headers: {
      Authorization: "Bearer YOUR_ACCESS_TOKEN",
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      customer_id: "CUST-123",
      asset_description: "Toyota Camry 2024",
      asset_category: "VEHICLE",
      cost_price: 50000,
      profit_amount: 5000,
      installment_count: 24,
      currency: "SAR",
    }),
  },
);

const contract = await response.json();
console.log("Contract created:", contract.id);
```

### Screen AML Transaction (Python)

```python
import requests

response = requests.post(
    'https://api.islamicopenfinance.com/api/v1/aml/screening',
    headers={'Authorization': 'Bearer YOUR_ACCESS_TOKEN'},
    json={
        'entity_type': 'TRANSACTION',
        'entity_id': 'TXN-456',
        'screening_type': 'SANCTIONS'
    }
)

screening = response.json()
if screening['status'] == 'PASSED':
    print('Transaction cleared')
else:
    print(f'Alert: {screening["matches"]}')
```

### Query Lineage (cURL)

```bash
curl -X GET \
  'https://api.islamicopenfinance.com/api/v1/metadata/lineage?rail=contracts&entity_id=CNT-789&depth=3' \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN'
```

## SDK Support

Official SDKs are available in multiple languages:

| Language       | Package                 | Repository                                                                                   |
| -------------- | ----------------------- | -------------------------------------------------------------------------------------------- |
| **TypeScript** | `@iof/sdk`              | [iof-sdks/typescript](https://github.com/Islamic-Open-Finance/iof-sdks/tree/main/typescript) |
| **Python**     | `iof-sdk`               | [iof-sdks/python](https://github.com/Islamic-Open-Finance/iof-sdks/tree/main/python)         |
| **Java**       | `com.iof:iof-sdk`       | [iof-sdks/java](https://github.com/Islamic-Open-Finance/iof-sdks/tree/main/java)             |
| **Go**         | `github.com/iof/go-sdk` | [iof-sdks/go](https://github.com/Islamic-Open-Finance/iof-sdks/tree/main/go)                 |

## Validation

Validate the OpenAPI specification:

```bash
# Using Spectral
npm run lint

# Using Redocly
npm run validate
```

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Reporting Issues

- **Security vulnerabilities**: security@islamicopenfinance.com
- **Bug reports**: [GitHub Issues](https://github.com/Islamic-Open-Finance/iof-openapi/issues)
- **Feature requests**: [GitHub Discussions](https://github.com/Islamic-Open-Finance/iof-openapi/discussions)

## Versioning

This specification follows [Semantic Versioning 2.0.0](https://semver.org/):

- **MAJOR**: Incompatible API changes
- **MINOR**: Backwards-compatible functionality
- **PATCH**: Backwards-compatible bug fixes

Current version: **1.0.0**

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

## License

This project is licensed under the Apache License 2.0 - see [LICENSE](LICENSE) file for details.

## Support

- **Documentation**: https://docs.islamicopenfinance.com
- **Developer Portal**: https://developers.islamicopenfinance.com
- **Community Forum**: https://community.islamicopenfinance.com
- **Email**: support@islamicopenfinance.com
- **Twitter**: [@IOF_Platform](https://twitter.com/IOF_Platform)

## Related Projects

- [IOF SDKs](https://github.com/Islamic-Open-Finance/iof-sdks) - Official client libraries
- [IOF Mock Server](https://github.com/Islamic-Open-Finance/iof-mock) - Testing mock server
- [IOF DevTools](https://github.com/Islamic-Open-Finance/iof-devtools) - CLI tools and utilities
- [IOF Platform](https://github.com/Islamic-Open-Finance/app) - Main platform repository

---

**Built with ❤️ for the Islamic finance community**
