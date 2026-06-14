# /api-contract — Contract-First API Design

Design a REST API contract (OpenAPI spec stub + Pact consumer test) for a new resource before writing implementation code.

## Usage

```
/api-contract "OrderService — place order endpoint for the web checkout flow"
/api-contract "CustomerService — customer profile read and update for mobile app"
```

## What This Command Does

1. Activates the `architect` and `business-analyst` agents
2. Applies contract-first design: define the contract before the implementation
3. Produces an OpenAPI 3.1 spec stub for the new endpoint(s)
4. Produces a Pact consumer test stub for the most critical consumer
5. Validates the contract against `api-standard.md` rules (URI design, HTTP semantics, error shapes, versioning)
6. Flags any design decisions that require ARB review

## Output Format

### Contract Summary
- Endpoint summary (method, path, purpose)
- Consumer(s) and their primary use cases
- Auth mechanism required

### OpenAPI 3.1 Stub (`openapi/v1/orders.yaml`)
```yaml
openapi: 3.1.0
info:
  title: Order Service API
  version: 1.0.0

paths:
  /v1/orders:
    post:
      operationId: placeOrder
      summary: Place a new order
      tags: [Orders]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PlaceOrderRequest'
      responses:
        '201':
          description: Order placed successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderResponse'
        '400':
          $ref: '#/components/responses/ValidationError'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '422':
          $ref: '#/components/responses/BusinessRuleViolation'
```

### Pact Consumer Test Stub (Java)
```java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "OrderService", pactVersion = PactSpecVersion.V4)
class OrderServicePactConsumerTest {

    @Pact(consumer = "CheckoutUI")
    public V4Pact placeOrderPact(PactBuilder builder) {
        return builder
            .usingLegacyDsl()
            .given("customer cust-123 exists")
            .uponReceiving("a place order request")
                .path("/v1/orders")
                .method("POST")
                .headers(Map.of("Authorization", like("Bearer token"), "Content-Type", "application/json"))
                .body(new PactDslJsonBody()
                    .stringType("customerId", "cust-123")
                    .minArrayLike("items", 1))
            .willRespondWith()
                .status(201)
                .body(new PactDslJsonBody()
                    .stringType("orderId")
                    .stringMatcher("status", "PENDING|CONFIRMED", "PENDING"))
            .toPact(V4Pact.class);
    }
}
```

### Design Decisions Log
- List any design choices made (e.g., "chose PUT over PATCH for full replacement semantics")
- Flag decisions requiring ARB input

## Tips

- Run this command before `/estimate` — a clear contract makes estimation more accurate
- The OpenAPI stub should be committed to `openapi/` before development starts
- Pact consumer tests should be written by the consumer team and verified by the provider team
- For breaking contract changes, always increment the version: `/v2/orders`
