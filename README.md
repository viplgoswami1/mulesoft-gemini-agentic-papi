# MuleSoft Gemini Agentic PAPI

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![MuleSoft](https://img.shields.io/badge/MuleSoft-Mule%204.4%2B-00A0DF)](https://www.mulesoft.com)
[![Java](https://img.shields.io/badge/Java-17-orange)](https://adoptium.net)
[![Layer](https://img.shields.io/badge/Layer-Process%20API-2ea043)](https://www.mulesoft.com)

![](https://viplgoswami1.gateway.scarf.sh/a.png?x-pxid=32e622ae-fd1b-4233-a37b-34e5937dc997)


A **MuleSoft Process API** that orchestrates business logic for the Gemini Agentic
commerce platform. Acts as the middleware layer between the Experience API and
the SAP System APIs, handling routing, transformation, and workflow orchestration.

---

## Architecture

```
mulesoft-gemini-agentic-eapi          ← Experience API
      ↓
mulesoft-gemini-agentic-papi          ← Process API (this repo)
      ↓               ↓
sap-gemini-agentic-cart-sapi    sap-gemini-agentic-checkout-sapi
(Cart System API)               (Checkout System API)
      ↓               ↓
         SAP CCv2 (Merchant Commerce Engine)
```

| Layer | Repository | Role |
|-------|------------|------|
| **EAPI** | [mulesoft-gemini-agentic-eapi](https://github.com/viplgoswami1/mulesoft-gemini-agentic-eapi) | Client-facing experience, Gemini AI integration |
| **PAPI** | [mulesoft-gemini-agentic-papi](https://github.com/viplgoswami1/mulesoft-gemini-agentic-papi) | Business logic orchestration, routing ← **this repo** |
| **Cart SAPI** | `sap-gemini-agentic-cart-sapi` *(coming soon)* | SAP cart operations |
| **Checkout SAPI** | `sap-gemini-agentic-checkout-sapi` *(coming soon)* | SAP checkout operations |
| **SAP CCv2** | SAP Commerce Cloud v2 (Merchant Commerce Engine) | Backend commerce engine |

---

## Overview

This Process API is responsible for:

- **Orchestration** — Route requests from EAPI to the correct SAPI
- **Transformation** — Translate EAPI payloads into SAPI-compatible formats
- **Business logic** — Apply business rules (pricing, validation, eligibility)
- **Error handling** — Centralised error management across all downstream SAPIs
- **Aggregation** — Combine responses from Cart and Checkout SAPIs when needed

---

## Getting Started

### Prerequisites

- Anypoint Studio 7.15+
- Mule Runtime 4.4.0+
- Java 17
- [mulesoft-gemini-agentic-eapi](https://github.com/viplgoswami1/mulesoft-gemini-agentic-eapi) running as upstream caller

### Installation

1. Clone the repository:
```bash
git clone https://github.com/viplgoswami1/mulesoft-gemini-agentic-papi.git
cd mulesoft-gemini-agentic-papi
```

2. Import into Anypoint Studio:
   - **File** → **Import** → **Anypoint Studio Project from File System**
   - Select the cloned folder
   - Click **Finish**

3. Configure in `src/main/resources/config.yaml`:
```yaml
cart:
  sapi:
    url: "http://localhost:8083"
checkout:
  sapi:
    url: "http://localhost:8084"
http:
  listener:
    port: 8082
```

---

## Configuration

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `CART_SAPI_URL` | URL of sap-gemini-agentic-cart-sapi | ✅ |
| `CHECKOUT_SAPI_URL` | URL of sap-gemini-agentic-checkout-sapi | ✅ |
| `HTTP_PORT` | Port for this PAPI (default: 8082) | ❌ |

---

## API Endpoints

### POST /cart

Creates a cart by routing to Cart SAPI.

**Request:**
```json
{
  "line_items": [{ "item": { "id": "WH1000XM5-BLK" }, "quantity": 1 }],
  "context": { "address_country": "US", "address_region": "CA", "postal_code": "94509", "currency": "USD" },
  "signals": { "dev.ucp.buyer_ip": "203.0.113.42" }
}
```

### PUT /cart/{cartId}

Updates an existing cart by routing to Cart SAPI.

**Request:**
```json
{
  "id": "cart_abc123",
  "line_items": [
    { "id": "li_1", "item": { "id": "WH1000XM5-BLK" }, "quantity": 1 },
    { "id": "li_2", "item": { "id": "SRS-XB100-BLU" }, "quantity": 2 }
  ]
}
```

### POST /checkout

Initiates checkout by routing to Checkout SAPI.

---

## Example Mule Flow

```xml
<flow name="createCartFlow">
    <http:listener path="/cart" method="POST"/>

    <!-- Transform request for Cart SAPI -->
    <transform:message>
        <transform:set-payload><![CDATA[%dw 2.0
output application/json
---
payload]]></transform:set-payload>
    </transform:message>

    <!-- Route to Cart SAPI -->
    <http:request
        url="${cart.sapi.url}/cart"
        method="POST">
        <http:headers>
            <http:header key="Content-Type" value="application/json"/>
        </http:headers>
    </http:request>

    <logger message="#[payload]"/>
</flow>
```

---

## Related Repositories

| Repo | Description |
|------|-------------|
| [mulesoft-gemini-agentic-eapi](https://github.com/viplgoswami1/mulesoft-gemini-agentic-eapi) | Experience API — upstream caller |
| `sap-gemini-agentic-cart-sapi` *(coming soon)* | System API — SAP cart operations |
| `sap-gemini-agentic-checkout-sapi` *(coming soon)* | System API — SAP checkout operations |
| [google-merchant-center-connector](https://github.com/viplgoswami1/google-merchant-center-connector) | Custom MuleSoft connector for Google Shopping API |
| [Maven Central](https://central.sonatype.com/artifact/io.github.viplgoswami1/google-merchant-center-mule-connector) | Published connector |

---

## Contributing

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'feat: add my feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a Pull Request

---

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.

---

## Author

**Viplove Goswami**
- GitHub: [@viplgoswami1](https://github.com/viplgoswami1)
- LinkedIn: [Viplove Goswami](https://linkedin.com/in/viplovegoswami)
- Maven Central: [io.github.viplgoswami1](https://central.sonatype.com/namespace/io.github.viplgoswami1)

---

*Built with ❤️ for the MuleSoft and AI community*
