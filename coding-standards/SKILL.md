---
name: coding-standards
description: Guidelines for writing production-quality Moqui code, focusing on commenting style, cleanliness, standard APIs, and professionalism.
---

# Skill: coding-standards (Moqui)
## Goal
Ensure all Moqui contributions meet production-grade standards for clarity, maintainability, and architectural alignment.

## Triggers
**ALWAYS** read this skill when:
- Writing any code (Groovy, XML, UI Screens).
- Reviewing PRs or auditing existing code for Moqui alignment.
- Using Moqui ExecutionContext (`ec`) APIs for data manipulation and queries.
- Preparing a pull request or implementing new features.

## General Rules
1. **Explain the WHY**: Code shows *what* happens. Comments must explain *why* (e.g., explaining a workaround for a legacy constraint).
2. **No Internal Monologue**: Delete "thinking out loud" or "maybe this works" comments.
3. **Guard against Technical Debt**: Avoid `// TODO` without a tracking reference.
4. **Remove Debug Code**: Clean up `println` or excessive `ec.logger.info` or `ec.logger.warn` before completion.

## Moqui Specific Standards
1. **Logic Placement**: 
    - Keep XML Actions (`type="inline"`) flat. If it requires complex loops or data transformations, move to inline Groovy (`<script>`) or a standalone Groovy script.
    - Avoid heavy logic inside Screen `<actions>`. Delegate to standard services.
2. **Naming Conventions**:
    - **Components**: `kebab-case` (e.g., `order-management`).
    - **Services**: Noun-Verb structure: `verb` is typically `camelCase` and `noun` is `UpperCamelCase` (e.g., `create#ProductVariance`).
    - **Groovy**: `lowerCamelCase` for methods/variables, `UpperCamelCase` for classes.
3. **Documentation**:
    - **Services**: Every service MUST have a clearer description either in the `<service>` tag attributes or nested elements.
    - **Groovy**: Use Groovydoc for public methods and complex logic.
4. **Moqui APIs (ExecutionContext `ec`)**:
    - **Validation**: Use Groovy truthiness (`if (product)` or `if (!myList)`) or standard Groovy `.isEmpty()` instead of static utility classes.
    - **Maps/Lists**: Use native Groovy syntax (`[key: value]`, `[item1, item2]`) for concise creation.
    - **Logging**: Use `ec.logger.info(...)`, `ec.logger.error(...)`, etc., avoiding direct `System.out` prints.
    - **Database Access**: ALWAYS prefer the Entity Facade DSL (`ec.entity.find(...)`), avoiding bare JDBC or manual connections. 
    - **Service Calls**: ALWAYS use the Service Facade DSL (`ec.service.sync().name(...).call()`) for Groovy, and `<sice-ervcall>` in XML Actions.
5. **Internationalization (i18n)**:
    - Never hardcode user-facing strings in UI or error boundaries. In Groovy use `ec.l10n.localize("Key")`. In XML use standard localized message tags or attributes.
6. **XSD Validation**:
    - ALWAYS ensure your XML files (e.g., `*Services.xml`, `*Screens.xml`, `*Entities.xml`) are strictly validated against Moqui's provided XSD files (`moqui-service.xsd`, `moqui-screen.xsd`, etc.).
    - If you are unsure whether an attribute or element tag is correct, cross-check against the appropriate `.xsd` schema file to avoid runtime exceptions or ignored definitions.

## Guardrails
- **Consistency**: Follow the existing indentation and naming conventions of the component you are editing.
- **Transactional Integrity**: Respect Moqui service boundaries; rely on declarative transactions via the Service Facade or XML service definitions instead of manual commit/rollback.

## Examples
**Example: Clean Entity Query with execution context**
```groovy
// Check if the product is active based on the current status.
EntityValue product = ec.entity.find("co.hotwax.product.Product")
        .condition("productId", productId)
        .condition("statusId", "PROD_ACTIVE")
        .useCache(true)
        .one()
        
if (!product) {
    ec.logger.warn("Product ${productId} not found or inactive")
}
```

**Example: Lean XML Actions with Groovy Script Snippet**
```xml
<actions>
    <entity-find-one entity-name="mantle.order.OrderHeader" value-field="orderHeader"/>
    <if condition="!orderHeader">
        <return error="true" message="Order not found"/>
    </if>
    
    <script>
        // Use Groovy for clean list transformation
        def itemIds = orderHeader.getItems().collect { it.orderItemSeqId }
        ec.logger.info("Processing order ${orderHeader.orderId} items: ${itemIds}")
    </script>
</actions>
```
