---
name: manage-jobs
description: Guidelines for managing background jobs, async execution, and task scheduling in Moqui.
---

# Skill: manage-jobs
## Goal
Provide clear guidance on how to configure, execute, and monitor background tasks and scheduled jobs in Moqui Framework.

## Core Concepts

Moqui uses a built-in Job Scheduler to manage asynchronous execution and recurring tasks. Moqui utilizes standard Quartz-like cron expressions.

- **ServiceJob**: The main configuration entity (`moqui.service.job.ServiceJob`) defining a job name, its service, and its schedule (`cronExpression`).
- **ServiceJobRun**: The entity tracking past and current job executions, including parameters, results, and errors.
- **ServiceFacade**: The Application Programming Interface (API) for explicit ad-hoc execution (`ec.service.job()`) and generic async service calls (`ec.service.async()`).
- **cronExpression**: The configuration string used to schedule recurring jobs (e.g., `0 0 1 * * ?`).

## Scheduling Scheduled Jobs (XML Data)

Recurring jobs in Moqui are typically configured declaratively using `ServiceJob` entity data inside standard XML data files (see `manage-data` skill). Moqui will automatically pick up the job and run it via its built-in scheduler.

```xml
<entity-facade-xml type="seed">
    <!-- Schedule a job to run every night at 1:00 AM -->
    <moqui.service.job.ServiceJob jobName="generate_NightlyReport" 
            description="Nightly Report Generation"
            serviceName="my.component.ReportServices.generate#Report"
            cronExpression="0 0 1 * * ?" paused="N">
            
        <!-- Job arguments/parameters -->
        <moqui.service.job.ServiceJobParameter parameterName="reportType" parameterValue="DAILY"/>
    </moqui.service.job.ServiceJob>
</entity-facade-xml>
```

## Scheduling & Async Calls (Groovy)

Services can be executed asynchronously or jobs can be triggered programmatically using the ExecutionContext (`ec`).

### Async Service Execution (Shorthand)
To run a service in a background worker thread immediately, without persistent job configuration:

```groovy
// Returns a java.util.concurrent.Future interface or void
ec.service.async().name("my.component.SomeService")
    .parameters([param1: "value1"])
    .call()
```

### Triggering a Configured Service Job
To explicitly trigger a configured `ServiceJob` ad-hoc (bypassing its schedule or triggering an unscheduled/paused job):

```groovy
// Returns a jobRunId for tracking in the moqui.service.job.ServiceJobRun entity
String jobRunId = ec.service.job("generate_NightlyReport")
    .parameters([extraParam: "value"])
    .run()
```

## Configuration

Job worker thread pool and scheduling settings are configured in the Moqui Configuration file (`MoquiConf.xml`), usually overriding defaults from `MoquiDefaultConf.xml`.

```xml
<moqui-conf>
    <!-- Adjust job pool thread sizes and polling frequency (in seconds) -->
    <service-facade scheduled-job-check-time="60"
                    job-queue-max="0" job-pool-core="2" 
                    job-pool-max="8" job-pool-alive="120">
    </service-facade>
</moqui-conf>
```

## Best Practices

- **Declarative Scheduling**: Prefer defining scheduled jobs in `ServiceJob` seed data over writing programmatic schedulers.
- **Cron Expressions**: Use standard Quartz cron expressions for recurring logic.
- **Parameters**: `ServiceJobParameter` values are defined as strings and automatically typed to match the underlying service signature upon execution.
- **Monitoring**: Inspect `ServiceJobRun` records in the database or via the Moqui System app/Tools to troubleshoot failed jobs or view execution history (`hasError`, `errors`, etc.).
- **Pool Efficiency**: Prevent excessively long-running operations in foreground pools. Large batch tasks should be sent to the background via `ec.service.async()` or heavily tuned, as they run in the `job-pool-core` threads by default.
