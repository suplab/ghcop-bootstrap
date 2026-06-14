# GraphQL Standard

> Standards for all GraphQL APIs. Complements `api-standard.md`. Enforced by `architect` and `code-reviewer` agents.

---

## When to Use GraphQL vs REST

Use GraphQL when:
- Multiple clients need different subsets of the same data (mobile vs web vs partner)
- The UI drives data requirements and backend teams are a bottleneck
- You have a natural graph-shaped domain (social, catalogue, recommendations)

Use REST when:
- Simple CRUD operations with well-defined, stable response shapes
- Public third-party APIs that need OpenAPI documentation
- File upload / download operations (use REST + presigned URLs)

Do not mix GraphQL and REST for the same resource — choose one per bounded context.

---

## Schema Design

- Schema-first: define the `.graphql` schema file before writing resolvers
- Types use `PascalCase`; fields use `camelCase`; enum values use `UPPER_SNAKE_CASE`
- Use non-null (`!`) explicitly — all fields default to nullable; add `!` when absence is a bug
- Prefer specific types over generic `String` for IDs: use `ID` scalar or custom scalars (`DateTime`, `BigDecimal`)

```graphql
# CORRECT — schema-first, explicit nullability, specific types
scalar DateTime
scalar BigDecimal

type Order {
  id: ID!
  status: OrderStatus!
  totalAmount: BigDecimal!
  placedAt: DateTime!
  customer: Customer!
  lineItems: [LineItem!]!
}

enum OrderStatus {
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
}

type Query {
  order(id: ID!): Order          # nullable — returns null if not found
  orders(filter: OrderFilter, page: PageInput): OrderPage!
}

type Mutation {
  placeOrder(input: PlaceOrderInput!): PlaceOrderPayload!
  cancelOrder(id: ID!, reason: String!): CancelOrderPayload!
}

# WRONG — generic types, implicit nullability
type Order {
  id: String       # should be ID
  status: String   # should be enum
  total: Float     # should be BigDecimal scalar
}
```

---

## Mutations

- Every mutation returns a payload type (not the entity directly) — allows adding metadata without breaking clients
- Payload types include the mutated entity and an `errors` field for domain errors

```graphql
type PlaceOrderPayload {
  order: Order
  errors: [UserError!]!
}

type UserError {
  field: String          # null if error is not field-specific
  message: String!
  code: String!          # machine-readable error code
}
```

---

## Error Handling

- Domain errors (validation, not found, business rule violations) → `UserError` in payload — not top-level `errors`
- Infrastructure errors (DB down, timeout) → top-level `errors` with `extensions.code`
- Never expose stack traces or internal messages in top-level errors

```json
// Domain error — in payload errors field
{
  "data": {
    "placeOrder": {
      "order": null,
      "errors": [{"field": "customerId", "message": "Customer not found", "code": "CUSTOMER_NOT_FOUND"}]
    }
  }
}

// Infrastructure error — top-level errors
{
  "data": null,
  "errors": [{"message": "Service temporarily unavailable", "extensions": {"code": "SERVICE_UNAVAILABLE"}}]
}
```

---

## N+1 Prevention (DataLoader)

Every relationship resolver that fetches data for a list MUST use DataLoader (batched loading). Direct repository calls inside relationship resolvers will fail code review.

```java
// CORRECT — DataLoader batches customer lookups
@SchemaMapping(typeName = "Order", field = "customer")
public CompletableFuture<Customer> customer(Order order, DataLoader<String, Customer> customerLoader) {
    return customerLoader.load(order.getCustomerId());
}

// WRONG — N+1: one DB call per order in a list
@SchemaMapping(typeName = "Order", field = "customer")
public Customer customer(Order order) {
    return customerRepository.findById(order.getCustomerId()).orElseThrow();
}
```

---

## Pagination

Use cursor-based pagination (Relay spec) for all list fields — never offset pagination for production APIs.

```graphql
type OrderPage {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type OrderEdge {
  node: Order!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

---

## Security

- Depth limiting: max query depth of 10 (prevents nested explosion attacks)
- Complexity limiting: each field has a cost; reject queries exceeding total cost of 1000
- Field-level authorisation: check permissions in resolvers — do not rely solely on HTTP-layer auth
- Disable introspection in production for sensitive internal APIs
- Rate limit by operation name + user, not by HTTP request

```java
// Spring for GraphQL — depth and complexity limiting
@Bean
public GraphQlSource graphQlSource(ResourcePatternResolver resourcePatternResolver) {
    return GraphQlSource.schemaResourceBuilder()
        .schemaResources(resourcePatternResolver.getResources("classpath:graphql/**/*.graphqls"))
        .configureRuntimeWiring(builder ->
            builder.scalar(ExtendedScalars.DateTime)
                   .scalar(ExtendedScalars.GraphQLBigDecimal))
        .build();
}
```

---

## Versioning

GraphQL schemas evolve via field deprecation — no version numbers in URLs.

1. Add the new field
2. Mark the old field `@deprecated(reason: "Use newField instead")`
3. After all clients migrate (tracked in deprecation log), remove the old field — with 30-day notice

Never remove a field without deprecation notice. Breaking changes require a new schema type (rare).

---

## Testing

- Schema validation test: assert schema parses without errors on application startup
- Resolver unit tests: test each resolver in isolation with mocked DataLoaders
- Integration tests: execute full GraphQL documents against a running application with test data
- Snapshot test: assert that schema SDL does not change unexpectedly (prevents accidental breaking changes)

---

## Anti-Patterns

| Anti-Pattern | Correct Alternative |
|---|---|
| Direct repository calls in relationship resolvers | DataLoader batching |
| Returning the entity directly from mutations | Payload type with `errors` field |
| Offset pagination | Cursor-based (Relay) pagination |
| Exposing stack traces in top-level errors | Generic message + `extensions.code` |
| Removing deprecated fields without migration window | 30-day deprecation + client tracking before removal |
| Disabling complexity/depth limits | Set limits appropriate to your schema |
| No field-level auth | Resolver-level permission checks |
