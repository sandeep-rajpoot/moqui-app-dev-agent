---
name: manage-templates
description: Define and maintain Moqui Freemarker (FTL) templates for UI rendering, emails, and document generation.
---

# Skill: manage-templates

## Goal
Define and maintain Freemarker (FTL) templates in Moqui Framework, ensuring correct use of Moqui's ExecutionContext (`ec`), ScreenRenderImpl (`sri`), and context variables.

## Triggers
**ALWAYS** read this skill when:
- Creating or modifying `.ftl` files in `template/` or `screen/` directories.
- Using `<render-mode><text type="html" location="component://.../Template.ftl"/></render-mode>` or inline `<text>` blocks in screens.
- Implementing email templates, XML/JSON API payloads, or custom PDF (XSL-FO) layouts.

## Use when
- Creating dynamic HTML components that require logic beyond standard XML screen widgets.
- Building custom dashboards, single-page application (Vue/React) mount points.
- Generating dynamic GraphQL queries, XML payloads, or API responses.

## Procedure
1. **Variable Setup & Safety**:
    - Use `<#assign variable = contextVar! />` to safely assign context variables with default values.
    - **Null Safety**: ALWAYS use the `!` or `!""` operator (e.g., `user.name!""`) to prevent template crashes if a variable is missing.
2. **Moqui Specific Variables (Critical)**:
    - **ExecutionContext (`ec`)**: The root of the Moqui API. Use it to access user, web, and localization data.
        - Localization: `${ec.l10n.localize("My Label")}`
        - Web context: `${ec.web.sessionToken}` or `ec.web.sessionAttributes.get("key")`
    - **ScreenRenderImpl (`sri`)**: Available in screen templates to handle URLs and screen logic.
        - URL Building: `${sri.buildUrl("targetTransition").url}`
3. **Control Flow**:
    - **Lists**: Use `<#list listVar as item> ... </#list>` (And `item?has_content` to verify).
    - **Conditionals**: Use `<#if condition> ... <#elseif ...> ... <#else> ... </#if>`.
4. **Context & Data Access**:
    - Request parameters: Directly available in the context (e.g., `${productId!}`).
    - **In-template Data Fetching**: You can use `ec.entity.find()` for lightweight lookups, but try to prepare data in Groovy or XML Actions first.
        - Example: `<#assign product = ec.entity.find("mantle.product.Product").condition("productId", productId).one()! />`
5. **HTML5 Semantic Structure & Modern web**:
    - **Mandatory**: Use semantic tags (`<header>`, `<main>`, `<section>`) instead of generic `<div>` wrappers for HTML.
    - **Accessibility**: Use `aria-*` attributes and proper label associations.
6. **Encoding & Security**:
    - Use `?html` or `?js_string` when outputting variables to prevent XSS (e.g., `${(username!"")?html}`).

## Guardrails
- **Logic Placement**: Keep complex business logic and database queries in Groovy/XML Actions. FTL should primarily focus on presentation logic.
- **Paths**: Reference templates using `component://[component-name]/template/...` in screen XML or service configurations.
- **Separation of Concerns**: If you are outputting JSON or XML for integrations, prefer using Moqui's native REST/Entity JSON APIs instead of constructing raw JSON strings with FTL unless necessary (like GraphQL schemas).

## Examples
### Example 1: Inline FTL in a Screen
Use this approach for simple, one-off HTML adjustments directly inside of your screen definition without having to create a separate file.

```xml
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-3.xsd">
    <actions>
        <set field="dashboardMessage" value="Welcome back!"/>
    </actions>
    <widgets>
        <!-- Standard Moqui widget -->
        <label text="Dashboard" type="h1"/>
        
        <!-- Inline FTL via render-mode text -->
        <render-mode>
            <text type="html"><![CDATA[
                <div class="alert alert-info">
                    <!-- FTL can securely access context variables -->
                    <strong>${ec.l10n.localize("Alert")}:</strong> ${(dashboardMessage!"")?html}
                </div>
            ]]></text>
        </render-mode>
    </widgets>
</screen>
```
