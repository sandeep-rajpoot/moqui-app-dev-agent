---
name: manage-strategies
description: General development strategies and preferences for Moqui. Use when deciding between multiple implementation options (e.g., XML Actions vs Groovy).
---

# Skill: manage-strategies
## Goal
Provide clear guidance on preferred implementation patterns and architecture decisions when multiple options exist in Moqui Framework.

## Triggers
**ALWAYS** read this skill when:
- Designing a new service, screen, or logic component.
- Choosing between XML Actions and Groovy for implementation.
- Reviewing code for architectural alignment.

## Core Preferences
### 1. Services: XML Actions vs. Groovy
- **XML Actions (`type="inline"`)**: **Preferred** for standard business logic, database CRUD orchestration, and service calling.
- **Groovy (`type="script"` or inline `<script>`)**: Preferred for logic that is too complex for declarative XML, data transformations, and advanced mathematical/business rules. Use as snippets inside XML `<actions>` or standalone scripts.
- **Preference Order**: XML Actions > Groovy.

### 2. Logic Placement
- **Prefer Service Layer**: Keep business logic in `*Services.xml` or `.groovy` files within the `service/` directory.
- **Keep Screens Clean**: Avoid heavy logic in screen `<actions>`. Delegate to standard services.
- **Keep REST APIs Declarative**: Do not write business logic in `*.rest.xml`. REST files should only route HTTP requests to entities or declarative services.
- **Event-Driven Side Effects**: Use SECAs (`*.secas.xml`) or EECAs (`*.eecas.xml`) for cross-cutting concerns (like cache clearing, webhooks, side-effects) rather than polluting core services.

## Dos and Don'ts
| Area | DO | DON'T |
| :--- | :--- | :--- |
| **Services** | Implement standard orchestration and CRUD in **XML Actions**. | Immediately jump to Groovy for simple data mapping. |
| **Complex Logic** | Use **Groovy** (`<script>`) embedded in XML actions or as standalone scripts. | Try to force complex loops or string manipulations purely via XML tags. |
| **UI/Screens** | Call services via `<service-call>` inside screen transitions/actions. | Embed thick business logic inside screen `<actions>`. |
| **Data/REST** | Expose simple entity CRUD directly via `<entity>` tag in `rest.xml`. | Wrap basic entity CRUD in custom services unnecessarily. |
| **Transactions** | Rely on the framework's declarative transaction management. | Manually manage JDBC connections or commit/rollback. |

## Guardrails
- **Declarative First**: Always look for a declarative XML solution in Moqui before writing imperative code (Groovy).
- **ExecutionContext (ec)**: For Groovy, always utilize Moqui's `ec` (ExecutionContext) API for entity finding, service calling, and messaging. Do not bypass the framework APIs.
- **Extensibility**: Rely on component overrides and ECAs rather than directly modifying framework source code.

## Examples
**Example: Preferred Service Definition**
```xml
<!-- Proper service definition using inline XML and embedded Groovy script when needed -->
<service verb="process" noun="Order">
    <in-parameters>
        <parameter name="orderId" required="true"/>
    </in-parameters>
    <actions>
        <entity-find-one entity-name="mantle.order.OrderHeader" value-field="orderHeader"/>
        <if condition="!orderHeader">
            <return error="true" message="Order not found"/>
        </if>
        
        <!-- Standard XML for service calls -->
        <service-call name="mantle.order.OrderServices.approve#Order" in-map="[orderId: orderId]"/>
        
        <!-- Embedded Groovy for complex data manipulation -->
        <script>
            // Complex list transformation logic
            def details = orderHeader.getItems().collect { it.productId + ":" + it.quantity }
            ec.logger.info("Processed: " + details.join(", "))
        </script>
    </actions>
</service>
```
