---
name: manage-data
description: |
  Create and manage Moqui data files (Entity Facade XML). Use when
  - Adding Seed/Demo data to the system.
  - Initializing configuration data in the component's `data` directory.
  - Adding service job configurations (`ServiceJob`).
---

# Skill: manage-data

## Goal
Create and manage Moqui data files (Entity Facade XML) to populate the database with correct and valid seed, demo, or configuration data.

## Triggers
**ALWAYS** read this skill when:
- Creating or updating XML files intended for data loading.
- Troubleshooting data-related startup or component errors.
- Adding cron jobs or service schedules.

## Use when
- **Seed Data (`type="seed"`)**: Mandatory data required for the application to function (e.g., system settings, component configuration). Loaded automatically.
- **Seed Initial Data (`type="seed-initial"`)**: Setup data that might need to be modified by users later.
- **Demo/Test Data (`type="demo"`)**: Optional data for development and testing environments.
- **Job Scheduling**: Background tasks using `moqui.service.job.ServiceJob`.

## Procedure
1. **Header**: ALWAYS use the `<entity-facade-xml>` root element with the correct `type` attribute for validation and loading context:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <entity-facade-xml type="seed">
   ```
2. **Entries**: Define records using the entity name as the tag (either fully qualified like `moqui.security.UserGroup` or short name if not ambiguous). Group elements by entity for readability.
   ```xml
   <moqui.secuirty.UserGroup userGroupId="MERCHANT_ADMIN" description="Merchant Administrator"/>
   ```
3. **Registration**: 
   - Moqui auto-discovers data files! Do **NOT** register the file in a component configuration file. 
   - Simply place the `.xml` file inside a component's `data` directory (e.g., `runtime/component/my-component/data/MySeedData.xml`).

## Guardrails
- **Ordering**: Define parent entities BEFORE child entities to avoid foreign key errors during load.
- **Dates**: Use standard ISO-8601 format: `YYYY-MM-DD HH:mm:ss.SSS` or just rely on Moqui's native data parsing.
- **CDATA**: Wrap text containing XML special characters (e.g., descriptions with HTML) in `<![CDATA[ ... ]]>`.
- **Primary Keys**: Ensure PKs are unique. For Seed/Ext data, use descriptive string IDs if possible instead of auto-sequenced ones.
- **Namespaces**: You do NOT need schema definitions (`xsi:noNamespaceSchemaLocation`) for standard data files in Moqui.

## Examples
**Example: Basic Entity Data Setup**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<entity-facade-xml type="seed">
    <!-- User Group Setup -->
    <moqui.security.UserGroup userGroupId="STORE_CLERK" description="Store Clerk" groupTypeEnumId="UgtMoquiAdmin"/>
    
    <!-- Permissions tied to User Group -->
    <moqui.security.UserPermission userPermissionId="STORE_VIEW" description="View Store Data"/>
    <moqui.security.UserGroupPermission userGroupId="STORE_CLERK" userPermissionId="STORE_VIEW" fromDate="0"/>
</entity-facade-xml>
```

**Example: Service Job Scheduling**
For recurring background tasks, utilize `moqui.service.job.ServiceJob`.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<entity-facade-xml type="seed">
    <!-- Template Service Job to run custom periodic service -->
    <moqui.service.job.ServiceJob jobName="generate_CustomReport" 
            description="Template Service Job to generate Custom Report periodically."
            serviceName="my.component.ReportServices.generate#CustomReport"
            cronExpression="0 0 1 * * ?" paused="Y">
        
        <!-- Job arguments/parameters -->
        <parameters parameterName="reportType" parameterValue="DAILY"/>
        <parameters parameterName="facilityGroupIds"/>
    </moqui.service.job.ServiceJob>
</entity-facade-xml>
```
