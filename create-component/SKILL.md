---
name: create-component
description: Scaffold a new Moqui component. Use when starting a new feature set, application, or isolating custom code.
---

# Skill: create-component
## Goal
Create a new, isolated Moqui component with the correct directory structure and configuration.

## Triggers
**ALWAYS** read this skill when:
- Creating a new component, UI application, or plugin.
- Isolating a large feature set into a separate manageable module.
- Understanding component discoverability and routing in Moqui.

## Use when
- Starting a brand new feature or application (e.g., `my-custom-component`).
- Developing an integration or feature that should be decoupled from the core framework components.

## Procedure
1. **Component Creation**:
    - Moqui doesn't strictly require a scaffolding Gradle command to create a basic component. You manually create a new directory inside the `runtime/component/` folder.
    - Example: `mkdir -p runtime/component/my-component`
    - *Note*: If creating a Java/Groovy-based component with dependencies, you will need to add a `build.gradle` file at the component's root.
2. **Base Structure**:
    - Within your new component directory, create the standard folder structure:
        - `entity/`: For Entity Facade XML files (`*Entities.xml` or `*eecas.xml`).
        - `service/`: For Service Facade XML files (`*Services.xml` or `*secas.xml`).
        - `screen/`: For Screen Facade and UI files (`*.xml` etc.).
        - `data/`: For XML data files representing seed/demo/security data.
        - `script/`: For Groovy scripts or other script resources.
        - `src/main/groovy/` or `src/main/java/`: For compiled class files.
3. **Initial Configuration**:
    - Add a `MoquiConf.xml` in the root of the component **only if** you need to override base framework configuration, add custom component settings, or securely limit entity/service access.
4. **Registration (Automatic)**:
    - Moqui **automatically discovers** any directory in `runtime/base-component/` and `runtime/component/` upon system startup.
    - Entities, services, and screens defined within the standard component subdirectories are implicitly loaded into the framework memory.
    - No manual `loadAll` command or manifest registry update is required to make the component visible. Just restart Moqui.

## Post-Scaffold Customization
1. **Directory Structure**: Verify the existence of `entity/`, `service/`, `screen/`, and `data/`.
2. **Screen Routing/Mounting**: In Moqui, screens in the `screen/` directory are often mapped by path. You might need to add a component screen in the `screen/` directory and transition/alias it from a known root screen (e.g., `webroot.xml` or `apps.xml`) if it represents a top-level web application.
3. **Data Loading**: Define seed or sample data in `data/MyComponentData.xml` and run `./gradlew load` to populate the database.
4. **Permissions**: Define screen and service authorization permissions in your entity data files (typically mapping `moqui.security.ArtifactAuthz` records) inside the `data/` directory.

## Guardrails
- **Naming**: Use lowercase, kebab-case (e.g., `order-management`, `custom-inventory`).
- **Location**: All custom application components **MUST** reside in the `runtime/component/` directory. Do not place them in `framework/` or `runtime/base-component/` to ensure clean separation of custom versus core.
- **Discovery**: Components are auto-discovered. Do not attempt to manually register them in external framework registry files.
- **Dependencies**: Ensure your component does not break existing components. If it depends on other components (like `mantle-usl`), you manage this gracefully via Java/Groovy imports and logically via service calls.
- **Cleanliness**: Omit folders that aren't needed. E.g., if you only define services, you only require the `service/` directory.

## Examples
**Example: Creating a new custom inventory component**
```bash
# 1. Create directory structure
mkdir -p runtime/component/inventory-custom/{entity,service,screen,data,script}

# 2. Add a sample entity file
touch runtime/component/inventory-custom/entity/InventoryEntities.xml

# 3. Add a sample service file
touch runtime/component/inventory-custom/service/InventoryServices.xml

# 4. Add a sample data file
touch runtime/component/inventory-custom/data/InventorySecurityData.xml
```
Then, restart the Moqui server to load the new component.
