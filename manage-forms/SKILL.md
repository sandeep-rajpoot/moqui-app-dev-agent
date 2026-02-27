---
name: manage-forms
description: Define and maintain Form and Grid widgets using the Moqui Screen XML form-single and form-list elements.
---

# Skill: manage-forms
## Goal
Define and maintain data input forms and data display grids using the Moqui Screen XML `<form-single>` and `<form-list>` widgets.

## Triggers
**ALWAYS** read this skill when:
- Creating or modifying XML files that contain `<form-single>` or `<form-list>` elements.
- Building search interfaces, data entry forms, or list displays.
- Defining drop-downs, or form layouts.

## Use when
- Creating single-record create/update forms.
- Implementing multi-record list grids or search results.
- Dynamically generating form fields from entity or service definitions.

## Procedure
1. **Encapsulation**:
    - Forms and lists are defined directly within the `<widgets>` section of a Screen XML file.
    - For high reusability, forms can be placed in a standalone screen and included via `<include-screen>`.
2. **Form Types**:
    - `form-single`: Used for submitting data (Create, Update, Single Edit). Provide the `transition` attribute to map to a `<transition>` defined in the screen.
    - `form-list`: Used to display or edit multiple records (Grids/Lists). Provide the `list` attribute to bind data fetched via `<entity-find>` or `<service-call>` in the `<actions>` block.
3. **Data Binding**:
    - **Auto-generation**: Use `<auto-fields-service service-name="...">` or `<auto-fields-entity entity-name="...">` to rapidly generate fields based on database or service definitions.
    - Adjust or hide auto-generated fields using `<field name="myField"><ignored/></field>` or override them by redefining the field explicitly.
4. **Field Elements (Advanced)**:
    - **Drop-downs**: Use `<drop-down><entity-options entity-name="..." text="${description}"/></drop-down>` or `<list-options list="..." key-name="..." text="..." />`.
    - **Displays**: Use `<display/>` for plain text, or `<display-entity entity-name="..."/>` to look up and automatically display a referenced entity's description.
    - **Autocompletes**: Use `<text-line ac-transition="..."/>` pointing to a transition that returns a JSON list for autocompleting lookups.
5. **Layout & Styling**:
    - Control field placement in a `<form-single>` using the `<field-layout>` and `<field-row>` tags to establish multi-column layouts.
    - Grids (`<form-list>`) implicitly render as tables. Use `<header-field>` to define table headers and `<default-field>` to control table cell content.
6. **Search & Pagination**:
    - Combine a `<form-single>` for search inputs and a `<form-list>` for displaying results on the same screen.
    - In `<actions>`, use `<entity-find search-form-inputs="true">`. This effortlessly integrates with `<form-list>` parameters to handle automatic database-level pagination, sorting, and filtering.
7. **XSD Validation**:
    - ALWAYS ensure your forms comply with `moqui-screen.xsd`. Verify tags and attributes against the schema if you are unsure of support.

## Guardrails
- **Naming**: Form names should be PascalCase (e.g., `EditExample`, `ListExamples`).
- **Dependencies**: Forms rely on the Screen's `<actions>` block to prepare their underlying data, such as `map="..."` for defaults on `form-single` and `list="..."` for iteration on `form-list`.

## Examples
**Example: Grid with Service Search and Sub-Pagination**
```xml
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-3.xsd">
    <transition name="findExample"><default-response url="."/></transition>
    <transition name="editExample"><default-response url="EditExample"/></transition>

    <actions>
        <!-- Automatically pulls search parameters from context and paginates -->
        <entity-find entity-name="moqui.example.Example" list="exampleList" search-form-inputs="true">
            <search-form-inputs default-order-by="exampleName"/>
        </entity-find>
    </actions>
    <widgets>
        <!-- Search Form -->
        <container-box><box-header title="Find Examples"/><box-body>
            <form-single name="FindExample" transition="findExample">
                <field name="exampleName"><default-field><text-line/></default-field></field>
                <field name="statusId">
                    <default-field>
                        <drop-down allow-empty="true">
                            <entity-options entity-name="moqui.basic.StatusItem">
                                <entity-constraint name="statusTypeId" value="ExampleStatus"/>
                            </entity-options>
                        </drop-down>
                    </default-field>
                </field>
                <field name="searchBtn"><default-field><submit text="Search"/></default-field></field>
                <!-- Structured Layout -->
                <field-layout>
                    <field-row><field-ref name="exampleName"/><field-ref name="statusId"/></field-row>
                    <field-ref name="searchBtn"/>
                </field-layout>
            </form-single>
        </box-body></container-box>

        <!-- Grid/List -->
        <form-list name="ListExamples" list="exampleList">
            <field name="exampleId">
                <header-field title="ID"/>
                <default-field>
                    <link url="editExample" text="${exampleId}" link-type="anchor" parameter-map="[exampleId:exampleId]"/>
                </default-field>
            </field>
            <field name="exampleName"><header-field title="Name"/><default-field><display/></default-field></field>
            <field name="statusId">
                <header-field title="Status"/>
                <default-field><display-entity entity-name="moqui.basic.StatusItem"/></default-field>
            </field>
        </form-list>
    </widgets>
</screen>
```

**Example: Single Edit Form with Auto-Fields**
```xml
<form-single name="EditExample" transition="updateExample" map="exampleRecord">
    <!-- Auto-generate fields dynamically from the database schema -->
    <auto-fields-entity entity-name="moqui.example.Example" include="nonpk"/>
    <field name="exampleId"><default-field><hidden/></default-field></field>
    
    <!-- Override auto-generated field for a custom drop-down -->
    <field name="typeEnumId">
        <default-field>
            <drop-down>
                <entity-options entity-name="moqui.basic.Enumeration">
                    <entity-constraint name="enumTypeId" value="ExampleType"/>
                </entity-options>
            </drop-down>
        </default-field>
    </field>
    <field name="submitBtn"><default-field><submit text="Save"/></default-field></field>
</form-single>
```
