---
name: manage-services
description: Define and implement the Moqui framework services. Use when creating new services in Services.xml, modifying parameters, or implementing business logic using XML Actions or scripts.
---

# Skill: manage-moqui-services
## Goal
Define and implement business logic as reusable, transactional, and securable services in the Moqui framework.

## Triggers
**ALWAYS** read this skill when:
- Defining or modifying `*Services.xml` files.
- Implementing logic via XML Actions (`<actions>`) embedded within a `<service>`.
- Calling services from Groovy scripts, Java, or REST APIs.
- Troubleshooting service execution, input validation, or authentication.

## Use when
- Exposing business logic for UI screens, REST APIs, or background jobs.
- Implementing complex database workflows.
- Handling transactional logic with structured input/output parameters.

## Procedure
1. **Service Type Selection**:
    - **XML Actions**: `type="inline"` (default). Use for standard business logic and orchestration using Moqui's powerful `<actions>` tags (like `<entity-find>`, `<service-call>`, etc).
    - **Entity Auto**: `type="entity-auto"`. Explicitly declare for standard CRUD operations if a custom override is needed, although basic CRUD is available implicitly without definition (e.g. `create#EntityName`).
    - **Script/Java**: `type="script"` or `type="java"`. Use only for highly complex logic or external library dependencies that are difficult to write in XML actions.
2. **Service Definition**:
    - Define services in files named `*Services.xml` placed in the `service/` directory of a component.
    - **Name Structure**: Use `<service verb="..." noun="...">` (e.g., `verb="create" noun="Product"`).
    - **Parameters**: Define inside `<in-parameters>` and `<out-parameters>` using `<parameter name="..." type="..." required="true|false"/>`.
    - **Transactions**: Configured on the `<service>` tag using the `transaction` attribute. Options: `use-or-begin` (default), `force-new` (suspends current tx), `ignore` (no tx), `cache` (delays until commit).
    - **Auth**: Set `authenticate="true"` to require a logged-in user. Use `<service-call disable-authz="true"/>` to bypass auth within other services.
3. **Registration**: 
    - In Moqui, services are auto-discovered based on the file name `*Services.xml` within a component's `service` directory. No explicit component registration is required like in OFBiz.
4. **Execution Logic (XML Actions)**:
    - Input parameters are automatically placed into the context (`context.paramName`).
    - Variables set in the context (using `<set field="..." value="..."/>`) matching `<out-parameters>` are automatically returned.
    - Send errors using `<return error="true" message="..."/>`.
5. **REST API Export**:
    - Build declarative REST API mappings in a `.rest.xml` file or call generic endpoints like `/rest/s1/component/path/verbNoun`.

## Guardrails
- **Naming Context**: A combination of `verb` (e.g., `create`, `update`, `delete`, `find`) and `noun` (e.g., `Order`, `ProductVariance`) forms the service name: `create#Order`.
- **Calling Other Services**: Use `<service-call name="..." in-map="..." out-map="..."/>`.
- **Inline Groovy**: Use `<script>...</script>` tags inside `<actions>` for logic best suited to standard Groovy instead of XML tags.
- **Persistence**: Never manually manage JDBC transactions; rely on the service layer's declarative transaction boundaries.
- **Error Handling**: Throw exceptions or use `<return error="true"/>`. Moqui will automatically roll back the transaction and return the error message.
- **Permissions**: Verify `ec.user.hasPermission(...)` when a service accesses sensitive resources that require explicit roles.

## Examples
**Example: Basic XML Service with Actions**
```xml
<service verb="clone" noun="Example">
    <in-parameters>
        <parameter name="exampleId" required="true"/>
        <parameter name="newExampleName"/>
    </in-parameters>
    <out-parameters>
        <parameter name="newExampleId" required="true"/>
    </out-parameters>
    <actions>
        <entity-find-one entity-name="com.example.Example" value-field="example"/>
        <if condition="!example">
            <return error="true" message="Example ${exampleId} not found"/>
        </if>
        
        <set field="newExampleMap" from="example.getValueMap()"/>
        <script>
            newExampleMap.remove("exampleId");
            newExampleMap.remove("lastUpdatedStamp");
        </script>
        
        <set field="newExampleMap.exampleName" from="newExampleName" default-value="${example.exampleName} (Copy)"/>
        <service-call name="create#com.example.Example" in-map="newExampleMap" out-map="createOut"/>
        
        <set field="newExampleId" from="createOut.exampleId"/>
    </actions>
</service>
```

**Example: Entity-Auto (Custom Validation)**
```xml
<service verb="create" noun="Example" type="entity-auto">
    <in-parameters>
        <auto-parameters entity-name="com.example.Example" include="nonpk"/>
        <auto-parameters entity-name="com.example.Example" include="pk" required="true"/>
    </in-parameters>
    <out-parameters>
        <parameter name="exampleId"/>
    </out-parameters>
    <!-- The engine automatically handles creation, but you can define strict parameters above -->
</service>
```

**Example: Calling a service in Groovy**
```groovy
Map result = ec.service.sync().name("create#com.example.Example")
    .parameters([exampleName: "Test Example", typeEnumId: "EX_TYPE_1"])
    .call()
String newExampleId = result.exampleId
```
