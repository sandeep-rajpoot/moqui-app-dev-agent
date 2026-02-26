---
name: create-rest-api
description: Create and manage REST APIs in Moqui using the declarative XML-based routing approach. Use when defining new endpoints, integrations, or exposing services/entities.
---

# Skill: create-rest-api
## Goal
Define robust REST endpoints in Moqui that map HTTP requests directly to backend services or entity operations using standard RESTful principles.

## Triggers
**ALWAYS** read this skill when:
- Exposing a new endpoint for external integration (e.g., apps, third-party services).
- Implementing webhooks or custom data retrieval APIs.
- Transitioning existing UI-bound logic into headless API endpoints.

## Use when
- Creating endpoints to be consumed by external HTTP REST clients.
- Designing APIs to support internal or external frontend applications.
- Directly exposing simple entity CRUD operations over HTTP without complex service layers.

## Procedure
1. **File Creation**:
    - REST APIs are defined in `*.rest.xml` files.
    - **Location**: Place the file in the `service/` directory of your component (e.g., `runtime/component/my-component/service/my-api.rest.xml`).

2. **Root Configuration**:
    - Use the standard `resource` XML root element referencing the Moqui XML schema.
    ```xml
    <resource xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/rest-api-3.xsd"
              name="my-api" displayName="My Custom API" version="1.0.0" description="Custom REST API for integrations">
    ```

3. **Defining Paths (Resources)**:
    - Use `<resource name="pathName">` to define URL segments.
    - Resources can be nested.

4. **Defining Path Parameters (IDs)**:
    - Use `<id name="parameterName">` to capture a dynamic segment in the URL (equivalent to `/{parameterName}` in Express/Spring).
    - Can contain nested `<method>` or `<resource>` elements.

5. **Defining Methods (HTTP Verbs)**:
    - Inside a `<resource>` or `<id>`, define the HTTP verb using `<method type="get|post|put|patch|delete">`.
    - **General Guidelines for Verbs**:
        - `get` = retrieving data (find)
        - `post` = creating data or performing an action (create/do)
        - `put` = replacing/storing data (store)
        - `patch` = partial update
        - `delete` = removing data

6. **Mapping Actions (Service vs. Entity)**:
    - **Service Call**: Map the endpoint to a Moqui service.
      ```xml
      <method type="post"><service name="component.MyService.do#Action"/></method>
      ```
    - **Entity Operation**: Moqui can automatically expose CRUD operations for an entity without requiring a service wrapper.
      ```xml
      <method type="get"><entity name="component.MyEntity" operation="list"/></method>
      <method type="post"><entity name="component.MyEntity" operation="create"/></method>
      ```
      *Supported entity operations: `one`, `list`, `count`, `create`, `update`, `store`, `delete`.*

## Guardrails
- **No Complex Logic**: Never write business logic in the REST XML. Its sole purpose is routing requests to entities or declarative/Groovy services.
- **Master Entities**: When returning entities directly via `<entity>`, use the `masterName` attribute (e.g., `<entity name="..." operation="one" masterName="default"/>`) to return related hierarchical data using master entity definitions.
- **RESTful Compliance**: Always use the correct HTTP verb. Do not use `POST` to fetch data unless you have an exceptionally complex search payload. Do not use `GET` to mutate state.
- **Nesting vs Flat**: Prevent deeply nested sub-resources. If a resource goes beyond 3 levels (e.g., `/orders/{id}/items/{itemId}/details`), consider flattening the structure to improve API maintainability.

## Examples

**Example 1: Direct Entity CRUD**
```xml
<resource name="products">
    <!-- GET /rest/s1/my-api/products -->
    <method type="get">
        <entity name="co.example.Product" operation="list" masterName="default"/>
    </method>
    <!-- POST /rest/s1/my-api/products -->
    <method type="post">
        <entity name="co.example.Product" operation="create"/>
    </method>

    <!-- Path segment capturing the product ID -->
    <!-- GET /rest/s1/my-api/products/{productId} -->
    <id name="productId">
        <method type="get">
            <entity name="co.example.Product" operation="one" masterName="default"/>
        </method>
        <method type="put">
            <entity name="co.example.Product" operation="update"/>
        </method>
    </id>
</resource>
```

**Example 2: Mapping to a Service**
```xml
<resource name="shipments">
    <id name="shipmentId">
        <!-- POST /rest/s1/my-api/shipments/{shipmentId}/pack -->
        <resource name="pack">
            <method type="post">
                <service name="co.hotwax.poorti.FulfillmentServices.pack#Shipment"/>
            </method>
        </resource>
    </id>
</resource>
```
