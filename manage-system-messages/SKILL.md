---
name: manage-system-messages
description: Guidelines for managing and configuring System Messages and System Message Remotes in Moqui. Use when implementing asynchronous messaging, API integrations, webhooks, or processing incoming data feeds.
---

# Skill: manage-system-messages
## Goal
Configure and utilize Moqui's `SystemMessage` functionality to integrate with external systems securely, reliably, and asynchronously.

## Core Concepts

Moqui incorporates a robust framework for handling incoming and outgoing external messages. It tracks message statuses, manages retries upon failure, and abstracts the routing/authentication parameters via system definitions.

- **`SystemMessage`**: The tracking entity for an individual message payload. It stores the message text, status (`SmsgProduced`, `SmsgSent`, `SmsgReceived`, `SmsgConsumed`, `SmsgError`), and retry counts.
- **`SystemMessageType`**: Defines a category of messages. It configures properties like which services handle creating, sending, and consuming the message.
- **`SystemMessageRemote`**: Defines the configuration to connect to a specific remote system, including Authentication (credentials/keys), URLs (`sendUrl`, `receiveUrl`), formatting configurations, and overriding services.

## Triggers
**ALWAYS** read this skill when:
- Designing integration points for external HTTP REST APIs, webhooks, or file drops (SFTP, etc.).
- Needing guaranteed delivery or asynchronous processing with automated retry logic.
- You are debugging failed integration synchronizations (`SystemMessageError`).

## Procedure

### 1. Defining the Message Type
You must first define a `moqui.service.message.SystemMessageType` in a Seed data (`.xml`) file.
- **Incoming**: Specify `consumeServiceName` to process the message.
- **Outgoing**: Specify `sendServiceName` to transmit the message (e.g., `org.moqui.impl.SystemMessageServices.send#SystemMessageRest`).

```xml
<moqui.service.message.SystemMessageType systemMessageTypeId="MyIntegrationTopic"
        description="My Integration Data Webhook" contentType="application/json"
        consumeServiceName="my.component.IntegrationServices.consume#MyData"
        sendServiceName="org.moqui.impl.SystemMessageServices.send#SystemMessageRest"/>
```

### 2. Defining the Remote System (Endpoints & Auth)
Define a `SystemMessageRemote` to represent the external system. This entity handles connection details so they don't leak into business logic definitions.

```xml
<moqui.service.message.SystemMessageRemote systemMessageRemoteId="RemoteExtSystem"
        description="External Integration System"
        sendUrl="https://api.external-system.com/v1/webhook"
        username="API_USER" password="API_PASSWORD" 
        messageAuthEnumId="SmatLogin"/> 
```
*(Note: The `password`, `sharedSecret`, and `privateKey` fields in this entity are automatically encrypted in the Moqui database).*

### 3. Outgoing Messages (Produce & Send)
Instead of directly calling HTTP APIs (like `ec.service.rest()`) during business transactions, simply "queue" a System message. Moqui's background processes will persist it and try to dispatch it.

```xml
<!-- In your service XML Actions -->
<service-call name="org.moqui.impl.SystemMessageServices.queue#SystemMessage" 
        in-map="[systemMessageTypeId: 'MyIntegrationTopic',
                 systemMessageRemoteId: 'RemoteExtSystem',
                 messageText: myJsonString,
                 sendNow: true]"/>
```
*If `sendNow=true`, it immediately schedules an asynchronous thread to attempt delivery.*

### 4. Incoming Messages (Receive & Consume)
For incoming payloads (e.g., catching an external webhook), Moqui exposes services to "receive" it into a `SystemMessage`. Once saved, an asynchronous service consumes it, freeing up the incoming HTTP thread quickly.

1. Accept the payload through a REST API or native Screen transition.
2. Call `receive#IncomingSystemMessage`.
3. Moqui queues the message (`SmsgReceived`) and triggers your `consumeServiceName` asynchronously.

```groovy
// In a Groovy Script or custom REST service endpoint
ec.service.sync().name("org.moqui.impl.SystemMessageServices.receive#IncomingSystemMessage")
    .parameters([systemMessageTypeId: "MyIntegrationTopic", 
                 messageText: requestBodyString,
                 systemMessageRemoteId: "RemoteExtSystem"])
    .call()
```

## Built-In Background Jobs
Moqui includes built-in services meant to be scheduled (typically configured OOTB) to process hanging messages:
- `send#AllProducedSystemMessages`: Sweeps up `SmsgProduced` or stalled `SmsgSending` messages to retry sending.
- `consume#AllReceivedSystemMessages`: Sweeps up `SmsgReceived` or stalled `SmsgConsuming` messages to retry consuming.

These retry based on `failCount` up to heavily configurable limits. Upon finally exceeding retry limits, messages transition to `SmsgError`.

## Guardrails
- **Idempotency**: External systems (and your internal consume services) may receive messages more than once if network failures happen during the HTTP acknowledgment phase. Ensure your consume services are idempotent (can be run multiple times safely).
- **No Business Logic in REST config**: Decouple the HTTP reception from actual payload processing. Only process data inside the `consumeServiceName` (for incoming) or prior to calling `queue#SystemMessage` (for outgoing).
- **Troubleshooting**: Inspect the `SystemMessageError` entity to troubleshoot why a message failed to send or consume. Use the `reset#SystemMessageInError` service to reset specific messages for retry once a bug or outage is fixed.
