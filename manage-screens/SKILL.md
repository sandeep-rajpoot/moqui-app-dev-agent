---
name: manage-screens
description: Define and maintain UI screens using the Moqui Screen XML definition.
---

# Skill: manage-screens

## Goal
Define and maintain UI screens using the Moqui Framework Screen XML to build dynamic web applications and dashboards.

## Triggers
**ALWAYS** read this skill when:
- Creating or modifying `*.xml` files inside a component's `screen/` directory.
- Adding transition logic, forms, or new pages to the user interface.

## Use when
- Developing new UI features, reports, or custom dashboards.
- Modifying existing screen layouts by adding `container-box`, `form-list`, or `form-single` widgets.
- Creating API endpoints or file download routes (screens without UI widgets).

## Procedure
1. **Schema & Setup**: 
    - ALWAYS include the Moqui XML Screen schema header:
      ```xml
      <screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-3.xsd">
      ```
    - Define access rules via `require-authentication="true|false"` (usually on root screens).
2. **Registration (Auto-discovery)**: 
    - **No manual registration is needed.** Moqui auto-discovers screens based on their location in the `screen/` directory. 
    - The URL structure naturally mirrors the directory structure starting from the mount point (e.g., `component://my-component/screen/MyRoot/MyPage.xml` maps to the `/MyRoot/MyPage` URI if `MyRoot` is mounted in the webroot).
3. **Decoupled Decorators (Parent Screens)**: 
    - Moqui natively nests screens. A parent screen (e.g., `Oms.xml`) defines the layout, navigation, and visual shell.
    - Inside a parent app screen, the `<subscreens-active/>` or `<subscreens-panel/>` widget determines where the child screen renders.
4. **Transition Section `<transition>`**: 
    - Define endpoints for actions (REST calls, form submissions).
    - Use `<service-call name="..." in-map="..."/>` or `<script>` inside transitions.
    - Always define a `<default-response url="..."/>` or `<conditional-response>` to dictate where to navigate next.
5. **Actions Section `<actions>`**: 
    - **Data Fetching**: Use `<entity-find>`, `<entity-find-one>`.
    - **Logic**: Use `<script>` for inline Groovy or `<set field="var" from="..."/>` to prepare data for widgets.
    - Use `<service-call name="..." out-map="context"/>` to prepare data from a service.
6. **Widgets Section `<widgets>`**:
    - **Layout**: Organize structure using `<container-box>`, `<container-row>`, `<row-col>`.
    - **Conditionals**: Wrap logic in an `<if condition="yourGroovyExpression">` tag.
    - **Actions/Links**: Use `<link url="TransitionName" text="Submit" link-type="anchor|button"/>`.
    - **Forms**: Use native `<form-single>` and `<form-list>` bound to `list=` or `map=` context values.
7. **Styles and UI Customizations**:
    - FTL blocks can be embedded for complex UI logic using `<render-mode><text type="html"><![CDATA[ ... ]]></text></render-mode>`.
    - Do not inject scripts/styles globally unless necessary. The parent layout (`apps.xml` base UI) handles Vue/Bootstrap includes.

## Guardrails
- **Action Execution**: Avoid heavy logic in `<actions>`. Delegate complex business rules to Services (`.xml` or `.groovy` in the `service/` directory) and call them via `<service-call>`.
- **References**: Always use `component://` or relative URLs for references, never absolute paths.
- **Null Safety**: In Groovy/expression conditions inside XML (e.g. `<if condition="user != null">`), consider Groovy's truthiness (`<if condition="user">`).

## Examples
**Example: Standard Moqui List Screen with Form and Actions**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-3.xsd"
        default-menu-title="Orders" require-authentication="true">

    <!-- Transition (Endpoint for forms/links) -->
    <transition name="approveOrder">
        <service-call name="mantle.order.OrderServices.approve#Order" in-map="[orderId:orderId]"/>
        <default-response url="."/> <!-- Reload current screen -->
    </transition>
    
    <transition name="orderDetail">
        <default-response url="//oms/Order/OrderDetail"/> 
    </transition>

    <!-- Prepare Data -->
    <actions>
        <entity-find entity-name="mantle.order.OrderHeader" list="orderList">
            <econdition field-name="statusId" value="OrderPlaced"/>
            <order-by field-name="-entryDate"/>
        </entity-find>
    </actions>

    <!-- Render UI -->
    <widgets>
        <container-box><box-header title="Pending Orders"/>
            <box-body>
                <form-list name="PendingOrdersList" list="orderList">
                    <field name="orderId">
                        <header-field title="Order ID"/>
                        <default-field>
                            <!-- Navigates to another screen, passing parameters dynamically -->
                            <link url="orderDetail" text="${orderId}" link-type="anchor" parameter-map="[orderId:orderId]"/>
                        </default-field>
                    </field>
                    <field name="entryDate">
                        <default-field><display/></default-field>
                    </field>
                    <field name="grandTotal">
                        <default-field><display currency-unit-field="currencyUomId"/></default-field>
                    </field>
                    <!-- Action Link invoking a transition -->
                    <field name="approveAction">
                        <header-field title="Action"/>
                        <default-field>
                            <link url="approveOrder" text="Approve" parameter-map="[orderId:orderId]"
                                  confirmation="Are you sure you want to approve Order ${orderId}?"/>
                        </default-field>
                    </field>
                </form-list>
            </box-body>
        </container-box>
    </widgets>
</screen>
```
