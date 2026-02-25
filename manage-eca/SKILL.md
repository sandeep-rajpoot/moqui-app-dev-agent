---
name: manage-eca
description: Manage Service Event Condition Actions (SECA) and Entity Event Condition Actions (EECA) to implement event-driven logic in Moqui Framework.
---

# Manage Service and Entity ECAs (Service/Entity Event Condition Actions)

Event Condition Actions (ECAs) are the primary way to implement event-driven logic in Moqui Framework. They allow you to trigger logic, scripts, or other services automatically when specific service execution states or entity database operations occur.

## Triggers
- Implementing cross-cutting concerns (e.g., clearing caches upon entity changes, indexing data in Solr/Elasticsearch).
- Triggering side effects (e.g., sending emails after an order is completed, dispatching webhooks).
- Enforcing business rules that depend on state changes without modifying the core service/entity logic.
- Syncing data between different systems or components asynchronously.

## Procedures

### 1. Service ECAs (SECAs)
SECAs trigger actions when another service reaches a specific life-cycle event or execution state.

- **Location**: Defined in `*.secas.xml` files, typically within a component's `service` directory.
- **Root Element**: `<secas xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/service-eca-3.xsd">`

#### SECA Pattern
```xml
<secas xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/service-eca-3.xsd">
    <seca id="UniqueSecaId" service="component.Original#Service" when="post-service" priority="5">
        <!-- Optional Conditions using Groovy Expressions -->
        <condition>
            <expression>statusChanged &amp;&amp; statusId == 'ORDER_COMPLETED'</expression>
        </condition>
        <!-- Actions -->
        <actions>
            <service-call name="component.Triggered#Service" in-map="[orderId:orderId]" async="true" ignore-error="true"/>
        </actions>
    </seca>
</secas>
```

**Common Events (`when` attributes)**:
- `pre-authz`: Before authorization checks.
- `post-authz`: After authorization checks.
- `pre-validate`: Before in-parameters are validated.
- `pre-service`: Before the execution of the target service implementation.
- `post-service`: After the execution of the target service implementation (most common).
- `post-validate`: After out-parameters are validated.
- `tx-commit`: Tied to the transaction commit. Triggers just before the transaction commits successfully.
- `post-commit`: Tied to post transaction commit. Triggers immediately after the transaction has successfully committed.

**Other Attributes**:
- `priority`: Defines the execution order relative to other SECAs listening to the exact same event.

---

### 2. Entity ECAs (EECAs)
EECAs trigger actions when a database operation occurs on a specific entity.

- **Location**: Defined in `*.eecas.xml` files, typically within a component's `entity` directory.
- **Root Element**: `<eecas xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/entity-eca-3.xsd">`

#### EECA Pattern
```xml
<eecas xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/entity-eca-3.xsd">
    <eeca id="UniqueEecaId" entity="component.EntityName" on-create="true" on-update="true" on-delete="true">
        <condition><expression>someCondition == true</expression></condition>
        <actions>
            <!-- Define Actions, like clearing cache, inline scripting, or calling a service -->
            <script>ec.cache.getCache('some.cache.name').remove(entityId)</script>
            <service-call name="component.Triggered#Service" in-map="[entityId:entityId]"/>
        </actions>
    </eeca>
</eecas>
```

**Common Operations**: 
- `on-create="true"`: Trigger when a record is inserted.
- `on-update="true"`: Trigger when a record is updated.
- `on-delete="true"`: Trigger when a record is deleted.

---

## Guardrails
- **Avoid Recursion**: Be extremely careful not to trigger an ECA that will eventually invoke the original service or entity database operation, inadvertently causing an infinite execution loop.
- **Transactional Impact**: By default, ECAs run within the same transaction. If an ECA action fails and `ignore-error="false"`, the entire overarching transaction (including the main original operation) will be rolled back. Use `ignore-error="true"` when you trigger non-critical side effects you don't want rolling back the transaction.
- **Mode Selection**: Apply `async="true"` inside `<service-call>` elements for resource-heavy tasks like email sending, webhook dispatch, or search indexing integrations to prevent blocking the main process thread.
- **Commit Phase Hooks (`tx-commit` & `post-commit`)**: If the SECA communicates outside the database ecosystem (e.g., search engine indexation, HTTP events), strongly prefer `when="tx-commit"` or `when="post-commit"` to guarantee external systems are updated only if the local transaction logically goes through.
- **Condition Precision**: Encapsulate logic elegantly via Groovy inside the `<condition><expression>...</expression></condition>` node to guarantee the action securely activates solely when fundamentally necessary.

## Examples

### Order Completion Event Webhook SECA
Dispatching an asynchronous webhook specifically when a transaction completes and flags an order as COMPLETED.
```xml
<seca id="OrderStatusChangeWebhook" service="update#org.apache.ofbiz.order.order.OrderHeader" when="tx-commit">
    <condition>
        <expression>statusChanged &amp;&amp; statusId == 'ORDER_COMPLETED'</expression>
    </condition>
    <actions>
        <service-call name="co.hotwax.oms.order.OrderServices.send#OrderWebhook" 
                      in-map="[orderId:orderId, topicEnumId:'WEBHOOK_ODR_COMPLETE']" 
                      async="true" disable-authz="true"/>
    </actions>
</seca>
```

### Cache Invalidation EECA
Invalidating specific system caches instantaneously when screen form database definition changes persist.
```xml
<eeca id="DbFormCache" entity="moqui.screen.form.DbForm" on-create="true" on-update="true" on-delete="true">
    <actions>
        <script>ec.cache.getCache('screen.form.db.node').remove(formId)</script>
    </actions>
</eeca>
```
