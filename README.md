# Moqui Agent Skills Toolkit

## Overall Goal
The `moqui-agent-skills` toolkit is a repository of standardized knowledge, best practices, and implementation patterns for the Moqui Framework. Its primary purpose is to empower AI agents to perform complex Moqui development tasks—such as creating entities, defining XML services, building UI screens, and managing Groovy logic—with high accuracy and adherence to framework-specific guardrails.

By providing these "skills" in a structured format, we ensure that agents follow the correct design patterns (e.g., relying on the ExecutionContext (`ec`) API, favoring declarative XML Actions where appropriate, using Moqui's Entity Facade avoiding manual SQL, and adhering to strict component structures) without requiring manual intervention or repetitive prompting for every step.

## Agent Setup & Activation

For an AI agent to effectively use these skills dynamically, they need to be accessible in specific contextual locations.

### 1. Framework Location (Project Repository)
Keep the source of these skills within your Moqui framework structure (e.g., an `.agent` directory) so that they are version-controlled alongside your project:
`~/moqui-framework/.agent/skills/`

### 2. Native Agent Directory (Activation)
For the agent to structurally recognize these as **Core Skills** natively across sessions (allowing tools to automatically pick them up), they often can be linked or placed directly in the agent's recognized skills directory:
`~/.gemini/antigravity/skills/`

### 🛠️ Recommended Setup: Symlinking
To maintain the skills in your Git-tracked project directory while enabling them for the global agent, use symbolic links:

```bash
# Example: Activate the manage-services skill
ln -s ~/moqui-framework/.agent/skills/manage-services ~/.gemini/antigravity/skills/manage-services
```

## How to Use with an Agent
Once placed in the project directory, AI agents will internalize these skills by reading the `SKILL.md` files dynamically.

### Instructions for the Agent
If the agent is not configured to automatically load them, provide the following general instruction at the start of a session:
> "Whenever performing tasks related to Moqui framework components, first read the relevant skill definitions located in `.agent/skills/[skill-directory]/SKILL.md`."

### Integration Patterns
- **Triggered Reading**: Instruct the agent to "trigger" a skill read when it detects a specific file type extension (e.g., `.rest.xml` triggers `create-rest-api`).
- **Contextual Loading**: At the start of a task, the agent should read the master `skill-index` and load relevant specialized knowledge.

## Directory Structure
These skills are placed within the dedicated `.agent/skills` directory of your Moqui framework:
`/path/to/moqui-framework/.agent/skills/`

Each directory contains:
- `SKILL.md`: The core knowledge file containing the goal, procedures, triggers, and guardrails for that specific Moqui development skill.

## Available Skills
For a detailed list of all skills with short descriptions, see the master index at: [skill-index/SKILL.md](skill-index/SKILL.md).

The toolkit currently covers:
- **Core Abstractions**: `manage-data`, `manage-services`.
- **UI & Interaction**: `manage-screens`, `manage-forms`, `manage-templates`.
- **Logic & Flows**: `manage-groovy`, `manage-eca`, `manage-jobs`.
- **Integrations & Communication**: `create-rest-api`, `manage-system-messages`.
- **Project Lifecycle**: `create-component`, `coding-standards`.
- **Strategic Thinking**: `manage-strategies` (Deciding between XML Actions vs Groovy vs Java, architecture priorities).

## Deployment and Updates
This skill set acts as a living document of your organization's Moqui standards. To keep your agent up to date, simply refine the Markdown files and commit them to your repository so the agent has the latest tribal knowledge on its next run.
