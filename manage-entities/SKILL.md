---
name: manage-moqui-entities
description: Manage Moqui entity definitions (Data Model). Use when creating/modifying entities, debugging DB errors, or adding views.
---

# Skill: manage-moqui-entities
## Goal
Define and maintain the Moqui Data Model using XML entity definitions.

## Triggers
**ALWAYS** read this skill when:
- Creating or modifying entity definition files (`*Entities.xml` or `*ViewEntities.xml`).
- Designing complex data relationships.
- Extending existing entities.
- Troubleshooting database sync or SQL errors.

## Use when
- Creating new database tables (entities).
- Defining database joins (view-entities).
- Adding indexes or foreign key relationships (relationships).
- **Extending core/mantle entities** to add fields or relationships without altering base framework files.
- **Advanced data aggregation** using view-entities with functions.

## Procedure
1. **Entity Definition**:
    - Defined in XML files within the `entity/` directory of a component (e.g., `entity/ExampleEntities.xml`).
    - Always specify `entity-name` and `package` attributes on the `<entity>` element.
    - **Primary Keys**: Use `is-pk="true"` on fields that make up the primary key.
2. **Auto-Discovery**:
    - Unlike OFBiz, Moqui automatically discovers entity definitions located in the `entity/` directory of any loaded component. No explicit registration is needed.
3. **Field Types**:
    - Use standard Moqui types defined in `FieldTypes.xml` (e.g., `id`, `text-short`, `text-medium`, `text-long`, `number-integer`, `number-decimal`, `date-time`, `date`, `time`, `indicator`).
4. **Relationships**:
    - Use `<relationship type="one" related="...">` for foreign keys (many-to-one).
    - Use `<relationship type="many" related="...">` for logical collections (one-to-many).
    - Specify `<key-map field-name="...">` if the local and related field names differ, or let Moqui auto-map if they exactly match.
5. **View Entities (Joins & Functions)**:
    - Use `<view-entity entity-name="..." package="...">`.
    - Define members with `<member-entity entity-alias="..." entity-name="..."/>`.
    - Define joins using the `join-from-alias` attribute on `<member-entity>` along with `<key-map>` elements inside.
    - Provide aliases using `<alias-all entity-alias="..."/>` or specific `<alias entity-alias="..." name="..."/>`.
    - **Functions**: Use `function="..."` (e.g., `sum`, `count`, `min`, `max`) for aggregations inside an `<alias>`.
    - **Grouping**: Grouping is automatic in Moqui. If any `<alias>` has a `function`, all other aliases are used for the SQL `GROUP BY`.
6. **Entity Extensions**:
    - Use `<extend-entity entity-name="..." package="...">` to add fields or relationships to existing entities within your custom component's `entity/` directory.
7. **Indexes**:
    - Use `<index name="...">` with nested `<index-field name="..."/>` tags for indexing non-PK fields.

## Guardrails
- **Core Entities**: NEVER modify framework or mantle entities directly. Use `<extend-entity>` in your custom component instead.
- **Naming Constraints**: Use `CamelCase` for entity names (e.g., `ProductPrice`) and `camelCase` for field names (e.g., `productId`). Avoid special characters.
- **Audit**: Enable audit logging for sensitive fields or entities using `audit-log="true"`.
- **Database level**: Prefer View-Entities over Java/Groovy-level loops for data aggregation and reporting.

**Example: Standard Entity with Relationship**
```xml
<entity entity-name="ExampleFeature" package="org.moqui.example">
    <field name="exampleFeatureId" type="id" is-pk="true"/>
    <field name="featureSourceId" type="id"/>
    <field name="description" type="text-medium"/>
    <relationship type="one" related="org.moqui.example.ExampleFeatureSource">
        <key-map field-name="featureSourceId"/>
    </relationship>
</entity>
```

**Example: Entity Extension**
```xml
<extend-entity entity-name="Product" package="mantle.product">
    <field name="customInternalId" type="id"/>
    <relationship type="one" title="Custom" related="org.moqui.custom.CustomProductLog">
        <key-map field-name="productId"/>
    </relationship>
</extend-entity>
```

**Example: Advanced View Entity with Aggregation**
```xml
<view-entity entity-name="OrderHeaderAndRoleSummary" package="org.moqui.order">
    <member-entity entity-alias="ORLE" entity-name="mantle.order.OrderRole"/>
    <member-entity entity-alias="OH" entity-name="mantle.order.OrderHeader" join-from-alias="ORLE">
        <key-map field-name="orderId"/>
    </member-entity>
    
    <alias entity-alias="ORLE" name="partyId"/>
    <alias entity-alias="ORLE" name="roleTypeId"/>
    <alias entity-alias="OH" name="totalGrandAmount" field="grandTotal" function="sum"/>
    <alias entity-alias="OH" name="totalOrders" field="orderId" function="count"/>
</view-entity>
```

**Example: Status View Entity**
```xml
<view-entity entity-name="OrderHeaderAndStatus" package="org.moqui.order">
    <member-entity entity-alias="OH" entity-name="mantle.order.OrderHeader"/>
    <member-entity entity-alias="SI" entity-name="moqui.basic.StatusItem" join-from-alias="OH">
        <key-map field-name="statusId"/>
    </member-entity>
    
    <alias-all entity-alias="OH"/>
    <alias entity-alias="SI" name="statusDescription" field="description"/>
</view-entity>
```
