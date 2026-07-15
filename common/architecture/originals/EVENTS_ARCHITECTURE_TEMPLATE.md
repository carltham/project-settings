# Events Architecture

**Project:** [PROJECT_NAME]  
**Event Platform:** [KAFKA/EVENT_BUS/MESSAGE_QUEUE]  
**Last Updated:** [DATE]  
**Next Review:** [DATE + 3 MONTHS]

---

## How to Use This Template

**Instructions for project teams:**

1. Replace all `[PLACEHOLDER]` values with your project-specific information
2. Choose your event platform (Kafka, RabbitMQ, event bus, message queue)
3. Define domain events for your system
4. Specify event payload structure
5. Establish event naming conventions
6. Define retention and replay policies
7. Have architecture and ops teams review

**Key sections to customize:**
- Event platform choice (Section 1)
- Domain events (Section 2)
- Event payload (Section 3)
- Retention policies (Section 4)

---

## Overview

Events are how the system communicates state changes across boundaries. Events enable loose coupling, async processing, and audit trails.

**Event Philosophy:**
- Events represent facts that happened
- Events are immutable after emission
- Events enable audit trails and compliance
- Events decouple services and workflows

**Event Principles:**
1. Emit events ONLY after successful transaction
2. Events must include stable aggregate identity
3. Events must NOT contain secrets or personal data
4. Events must be sufficient for audit correlation
5. Events enable replay and recovery

---

## Event Platform

### Chosen Platform

**Platform:** [KAFKA/RABBITMQ/AWS_SNS_SQS/AZURE_SERVICE_BUS/CUSTOM]

**Rationale:**
- [Why this platform for your use case]
- [Volume expectations: X events/second]
- [Latency requirements: X ms]
- [Retention requirements: X days]
- [Team expertise: existing knowledge of platform]

### Access & Configuration

**Brokers/Endpoints:**
- Development: [URL]
- Staging: [URL]
- Production: [URL]

**Authentication:**
- Method: [API_KEY/CERTIFICATE/OAUTH/OIDC]
- Credentials location: [VAULT/CONFIG_SERVER]

**Topics/Queues:**
- List of event topics/queues: [TOPIC_NAMES]

---

## Event Emission

### When to Emit Events

**Rule (MANDATORY):** Emit events ONLY AFTER the associated transaction succeeds.

**Correct Pattern:**
```java
@Transactional
public void processOrder(Order order) {
    // 1. Persist order to database
    orderRepository.save(order);
    
    // 2. Transaction commits here
    // 3. THEN emit event (after commit)
    eventPublisher.emit(new OrderCreatedEvent(order.getId(), order.getTenantId()));
}
```

**Why?** 
- Ensures consistency: event only sent if database change succeeded
- Prevents orphaned events (event sent but database rollback)
- Allows event replay if needed

### When NOT to Emit Events

**Don't emit events for:**
- ❌ Queries (reading data)
- ❌ Transient state (session data, cache)
- ❌ Failed operations (emit error events instead if needed)
- ❌ Intermediate steps (only final state changes)

---

## Event Definition

### Domain Events

List all significant events in your system:

| Event | Trigger | Payload | Consumers |
|-------|---------|---------|-----------|
| **[EventName]Created** | New [entity] created | ID, timestamp, tenant | [Services] |
| **[EventName]Updated** | [entity] modified | ID, changed fields, tenant | [Services] |
| **[EventName]Deleted** | [entity] removed | ID, tenant | [Services] |
| **PaymentProcessed** | Payment succeeded | OrderID, Amount, TenantID | Fulfillment, Analytics |
| **OrderCancelled** | User cancelled order | OrderID, Reason, TenantID | Inventory, Billing |

### Event Naming Convention

**Format:** `[Aggregate][Action][ed]`

**Examples:**
- ✅ `OrderCreated`, `OrderShipped`, `PaymentProcessed`
- ✅ `UserRegistered`, `PasswordChanged`, `AccountDeleted`
- ❌ `new_order`, `PROCESS_PAYMENT`, `handleUserUpdate`

**Rules:**
- Use past tense (event already happened)
- Use domain language (not technical terms)
- Prefix with aggregate name (Order, User, Payment)
- Use PascalCase

---

## Event Payload Structure

### Mandatory Fields (All Events)

Every event MUST include these fields:

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",      // Unique event ID (UUID)
  "eventType": "OrderCreated",                             // Event type name
  "timestamp": "2026-07-12T10:30:00Z",                     // When event occurred (UTC)
  "aggregateId": "order-12345",                            // Entity that changed (ID only)
  "aggregateType": "Order",                                // Entity type
  "tenantId": "tenant-001",                                // For multi-tenant systems
  "correlationId": "req-67890",                            // Link to originating request
  "actorId": "user-999",                                   // Who triggered the event
  "version": 1                                             // Event format version
}
```

### Domain-Specific Fields

Add fields relevant to your event:

**OrderCreated Example:**
```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "OrderCreated",
  "timestamp": "2026-07-12T10:30:00Z",
  "aggregateId": "order-12345",
  "aggregateType": "Order",
  "tenantId": "tenant-001",
  "correlationId": "req-67890",
  "actorId": "user-999",
  "version": 1,
  
  // Domain-specific fields
  "customerId": "cust-555",
  "orderTotal": 99.99,
  "currency": "USD",
  "itemCount": 3,
  "shippingAddress": "1234 Main St, Anytown, USA"
}
```

### Secrets & Personal Data (NEVER Include)

**❌ Never include:**
- Passwords or API keys
- Credit card numbers
- Social security numbers
- Personal identification numbers
- Medical information
- Financial account details

**✅ Include references instead:**
```json
{
  "customerId": "cust-555",        // Reference, not details
  "paymentMethodId": "pm-777",     // Reference, not card number
  "addressId": "addr-888"          // Reference, not full address
}
```

---

## Event Routing & Consumption

### Event Subscribers

| Event | Subscribers | Action |
|-------|-------------|--------|
| OrderCreated | [Service1], [Service2] | [What they do with the event] |
| PaymentProcessed | [Service3], Analytics | [What they do with the event] |
| UserDeleted | [Service1], [Service2] | Cascade delete related data |

### Event Ordering Guarantees

**Within same partition/queue:**
- Events are ordered by timestamp
- Replay will be in order

**Across partitions/queues:**
- Order is NOT guaranteed
- Consumers must be idempotent

### Idempotency (MANDATORY)

Consumers must handle duplicate events safely.

**Implementation:**
```java
// Track processed events
public void handleOrderCreated(OrderCreatedEvent event) {
    // Check if already processed
    if (eventStore.hasProcessed(event.getEventId())) {
        return;  // Idempotent: ignore duplicate
    }
    
    // Process event
    fulfillmentService.createShipment(event.getAggregateId());
    
    // Record that we processed it
    eventStore.markProcessed(event.getEventId());
}
```

---

## Audit & Compliance

### Audit Trail

**Events create audit trail automatically:**
- What happened: event type
- When it happened: timestamp
- Who triggered it: actorId
- Which entity: aggregateId
- Correlation: correlationId (links to request)

**Audit trail enables:**
- Compliance reporting
- Debugging (what led to this state)
- Forensics (who did what when)
- Billing (which user actions generated costs)

### Event Retention Policy

**Keep events for:** [X days/months]

**Rationale:**
- Regulatory requirement: [X]
- Replay capability: [X]
- Debugging needs: [X]
- Cost constraints: [X]

**Archival:**
- After [X days], archive to [ARCHIVE_LOCATION]
- Archived events available for replay with delay
- Purge after [X years]

---

## Event Replay & Recovery

### When to Replay Events

**Replay events when:**
- ✅ Consumer was down and needs to catch up
- ✅ Bug fix requires reprocessing historical events
- ✅ New consumer needs historical context
- ✅ Data corruption requires reset

**Don't replay when:**
- ❌ Would duplicate side effects (emails, payments, SMS)
- ❌ External state would be incorrect (shipping labels printed)

### Replay Strategy

**For Idempotent Operations:**
```
1. Identify date range to replay
2. Restart consumer with startingPosition = [DATE]
3. Consumer processes events again (idempotent)
4. Verify results match original processing
```

**For Non-Idempotent Operations:**
```
1. Create snapshot at replay date
2. Identify affected entities
3. Manually correct external state (emails sent, etc.)
4. Replay with tracking to avoid duplicates
```

---

## Monitoring & Observability

### Event Metrics

**Track:**
- Events emitted per second (event rate)
- Events by type (breakdown)
- Event processing latency (consumer lag)
- Failed events (errors)
- Duplicate events detected (idempotency working)

**Alerts:**
- 🚨 Producer latency > [X]ms
- 🚨 Consumer lag > [X] events
- 🚨 Error rate > [X]%
- 🚨 Event queue overflow

### Event Logging

**Log at emission:**
```
[INFO] Event emitted: OrderCreated, aggregateId=order-12345, 
       tenantId=tenant-001, correlationId=req-67890, 
       timestamp=2026-07-12T10:30:00Z
```

**Log at consumption:**
```
[INFO] Event consumed: OrderCreated, aggregateId=order-12345,
       eventId=550e8400-e29b-41d4-a716-446655440000,
       consumer=FulfillmentService, processingTime=45ms
```

**Log errors:**
```
[ERROR] Event processing failed: OrderCreated,
        eventId=550e8400-e29b-41d4-a716-446655440000,
        aggregateId=order-12345, error=timeout,
        retryAttempt=3
```

---

## Testing Event-Driven Code

### Unit Tests

**Test event emission:**
```java
@Test
public void shouldEmitOrderCreatedEvent() {
    Order order = new Order(123, TENANT_A);
    service.createOrder(order);
    
    // Verify event was emitted
    verify(eventPublisher).emit(argThat(event ->
        event.getEventType().equals("OrderCreated") &&
        event.getAggregateId().equals("order-123")
    ));
}
```

**Test event consumption:**
```java
@Test
public void shouldHandleOrderCreatedEvent() {
    OrderCreatedEvent event = new OrderCreatedEvent("order-123", TENANT_A);
    
    consumer.handle(event);
    
    // Verify side effect
    verify(fulfillmentService).createShipment("order-123");
}
```

### Integration Tests

**Test full event flow:**
```java
@Test
public void shouldProcessOrderThroughEventPipeline() {
    // 1. Create order (emits event)
    Order order = service.createOrder(orderData);
    
    // 2. Allow event to be processed
    waitForEventProcessing();
    
    // 3. Verify side effects
    assertThat(fulfillmentService.getShipment(order.getId())).isNotNull();
}
```

### Event Idempotency Tests

```java
@Test
public void shouldHandleDuplicateEvents() {
    OrderCreatedEvent event = new OrderCreatedEvent("order-123", TENANT_A);
    
    // Process same event twice
    consumer.handle(event);
    consumer.handle(event);  // Duplicate
    
    // Should only create one shipment
    assertThat(fulfillmentService.getShipmentCount("order-123")).isEqualTo(1);
}
```

---

## Common Mistakes

**❌ Emitting events before transaction commits**
- Event sent, then database rollback
- Result: orphaned event

**✅ Emit after transaction succeeds**
- Database commits first
- Then emit event
- Result: consistency guaranteed

---

**❌ Including secrets in events**
- Credit card numbers in event payload
- Passwords visible in logs
- Result: security breach

**✅ Include only references**
- Event: `{ paymentMethodId: "pm-777" }`
- Service looks up details separately
- Result: secrets protected

---

**❌ Non-idempotent consumers**
- Event processed once, retry reprocesses
- Email sent twice, payment charged twice
- Result: duplicated side effects

**✅ Make consumers idempotent**
- Track processed event IDs
- Verify before processing
- Result: safe retries and replays

---

## Related Documents

- [ARCHITECTURE_PRINCIPLES_TEMPLATE.md](./ARCHITECTURE_PRINCIPLES_TEMPLATE.md) — System architecture
- [events-rules.md](../../common-AI-rules/development.rules.md) — Event emission rules
- [SECURITY_POLICY_TEMPLATE.md](./SECURITY_POLICY_TEMPLATE.md) — Secrets protection

---

## Customization Checklist

- [ ] Choose event platform (Kafka, RabbitMQ, etc.)
- [ ] Define domain events (list all events)
- [ ] Design event payload structure
- [ ] Set retention policy (days, archive location)
- [ ] Define consumer idempotency strategy
- [ ] Set up monitoring and alerts
- [ ] Configure event logging
- [ ] Design replay strategy
- [ ] Document event consumers
- [ ] Set up integration tests
- [ ] Train team on event architecture
- [ ] Review quarterly for new events

---

**Last Updated:** 2026-07-12  
**Maintained by:** Architecture & Backend Team  
**Status:** Production-ready template
