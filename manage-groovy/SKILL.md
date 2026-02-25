---
name: manage-groovy
description: Write Groovy logic for Moqui framework services and scripts. Use when implementing complex business logic, or scripts invoked via XML Actions.
---

# Skill: manage-moqui-groovy
## Goal
Implement business logic in Groovy using modern Moqui ExecutionContext (ec) API patterns.

## Triggers
**ALWAYS** read this skill when:
- Creating, modifying, or interpreting `.groovy` script files.
- Implementing logic inside the `<script>` tag within XML Actions.
- Working with services of type `type="script"`.

## Use when
- Writing core business logic that is too complex for declarative XML Actions.
- Transforming, filtering, and organizing data.
- Preparing data for UI screens and API responses.
- Performing advanced logic requiring native Java/Groovy APIs or external libraries.

## Procedure
1. **Context Access**:
    - Input parameters and variables defined in XML actions are available as bare variables or directly through the `context` map.
    - Example: `String productId = context.productId` or simply `productId`.
2. **Moqui API (ExecutionContext `ec`)**:
    - The `ec` (ExecutionContext) object is automatically injected into every script. It's the gateway to all framework tools.
3. **Entity DSL**: Use `ec.entity` for clean database queries:
    ```groovy
    // Find List
    List products = ec.entity.find("co.hotwax.product.Product")
                             .condition("statusId", "PROD_ACTIVE")
                             .orderBy("productName")
                             .list()
    
    // Find One
    EntityValue product = ec.entity.find("co.hotwax.product.Product")
                                   .condition("productId", productId)
                                   .one()
    ```
4. **Service Call API**: Call synchronous or async services using `ec.service`:
    ```groovy
    Map result = ec.service.sync().name("update#co.hotwax.product.Product")
                           .parameters([productId: productId, internalName: "New Name"])
                           .call()
    ```
5. **Data Manipulation**:
    - Utilize standard Groovy functional methods: `list.collect { it.name }`, `map.subMap(['key1', 'key2'])`.
    - Work with `EntityValue` objects natively like Maps: `product.internalName = "New Name"`.
6. **Success/Error Handling**:
    - Return output parameters by placing them in the current context, or returning a Map from a `.groovy` standalone service.
    - Add errors or success messages using `ec.message`:
    ```groovy
    if (!product) {
        ec.message.addError("Product ${productId} not found.")
        return
    }
    ```

## Guardrails
- **Naming**: Use camelCase for variables and method names.
- **Typing**: Prefer `def` or explicit types like `EntityValue` and `EntityList`.
- **Security**: Security and input validation should generally stay at the XML `<service>` definition layer unless specialized role behavior is required (`ec.user.hasPermission(...)`).
- **Performance**: Always use strict conditions to filter queries. Use `.select("field1", "field2")` if you only need a subset of data on larger result sets.
- **Transactions**: Do not manage JDBC connections. Let the Moqui transaction manager handle rollbacks when `ec.message.hasError()` is triggered.

## Examples
**Example: Groovy Script Implementation**
```groovy
import org.moqui.entity.EntityValue

String productId = context.productId

// Finding an entity record
EntityValue product = ec.entity.find("co.hotwax.product.Product")
                               .condition("productId", productId)
                               .one()
                               
if (product) {
    // Updating entity
    product.internalName = context.internalName
    product.update() // Alternatively: ec.service.sync().name("update#Product")...
    
    ec.message.addMessage("Product ${productId} updated successfully.")
} else {
    // Handling errors
    ec.message.addError("Product ${productId} could not be found.")
    return
}
```
