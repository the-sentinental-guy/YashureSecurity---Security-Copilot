# 🗺️ Mermaid Diagram Generator — Security Copilot Plugin

A GPT-based plugin for [Microsoft Security Copilot](https://learn.microsoft.com/en-us/security-copilot/) that automatically generates [Mermaid](https://mermaid.js.org/) diagrams from any session context. It transforms security investigation data, compliance audits, threat intelligence, and operational workflows into clear, visual diagrams — without requiring the user to write a single line of diagram code.

---

## Table of Contents

- [Overview](#overview)
- [Supported Diagram Types](#supported-diagram-types)
- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [Step 1 — Download the Manifest](#step-1--download-the-manifest)
  - [Step 2 — Open Security Copilot](#step-2--open-security-copilot)
  - [Step 3 — Upload the Plugin](#step-3--upload-the-plugin)
  - [Step 4 — Enable the Plugin](#step-4--enable-the-plugin)
- [Usage](#usage)
  - [Automatic Invocation](#automatic-invocation)
  - [Specifying a Diagram Type](#specifying-a-diagram-type)
  - [Focusing on a Specific Area](#focusing-on-a-specific-area)
- [Viewing Your Diagram](#viewing-your-diagram)
  - [Step 1 — Open Mermaid Live Editor](#step-1--open-mermaid-live-editor)
  - [Step 2 — Paste the Code](#step-2--paste-the-code)
  - [Step 3 — Adjust the Layout](#step-3--adjust-the-layout)
  - [Step 4 — Export the Diagram](#step-4--export-the-diagram)
- [Features](#features)
- [Plugin Architecture](#plugin-architecture)
- [Limitations](#limitations)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

Security investigations, compliance audits, and threat analyses generate large volumes of structured data across multiple prompts in a Security Copilot session. Reading through raw text to understand entity relationships, investigation flows, or compliance breakdowns is time-consuming.

This plugin solves that problem by reading the full session context and generating a Mermaid diagram that visually represents the key findings, entities, and their relationships. The plugin is **automatically selected** by the Security Copilot orchestrator whenever a user requests any kind of diagram or visualization — no manual plugin selection needed.

### What Makes This Plugin Different

- **Context-Aware** — Reads the entire session, not just the last prompt. The diagram reflects everything discovered during the investigation.
- **Auto-Detection** — Automatically selects the best diagram type based on session content (flowchart for investigation flows, sequence diagram for API interactions, timeline for chronological events, etc.).
- **Security-First Icons** — Applies security-relevant icons (Font Awesome in flowcharts, emoji in other types) based on keyword detection in node labels.
- **Optimized Layouts** — Uses the [Elk](https://eclipse.dev/elk/) rendering engine for flowcharts and state diagrams, producing adaptive layouts that handle complex flows far better than the default Dagre renderer.
- **Zero Fabrication** — Every node and edge in the diagram comes directly from the session content. Nothing is invented.

---

## Supported Diagram Types

| Diagram Type | Best For | Auto-Detected When |
|---|---|---|
| **Flowchart** | Investigation flows, decision trees, remediation workflows | Session has decisions, branches, or step-by-step processes |
| **Sequence Diagram** | API interactions, plugin invocation chains, multi-party workflows | Session involves multi-plugin or multi-API interactions |
| **Timeline** | Incident timelines, event chronology | Session has chronological events with timestamps |
| **Gantt Chart** | Remediation plans, project phases, SLA tracking | Session involves project planning or remediation scheduling |
| **C4 Context / Container** | System architecture, infrastructure topology | Session describes infrastructure or system components |
| **Architecture Diagram** | Cloud infrastructure, service topology | Session maps out Azure/cloud service relationships |
| **State Diagram** | Alert state transitions, compliance state changes | Session tracks state transitions (e.g., non-compliant → remediated) |
| **Mind Map** | Threat landscapes, investigation branches, brainstorming | Session explores a broad topic with branching sub-topics |
| **Pie Chart** | Severity distributions, compliance percentages | Session contains distribution or breakdown data |
| **Class Diagram** | Entity relationships (users, devices, IPs, alerts) | Session discovers related entities with properties |
| **Block Diagram** | System component blocks, organizational layouts | Session describes system components in a grid-like structure |

---

## How It Works

The plugin processes session content through a 6-step pipeline:

```
┌──────────────────────────────────────────────────────────────┐
│  STEP 1: Analyze Session Context                             │
│  Identify entities, relationships, severity, timelines       │
├──────────────────────────────────────────────────────────────┤
│  STEP 2: Select Diagram Type & Layout                        │
│  Auto-detect best type + choose renderer (Elk/Dagre)         │
│  and direction (LR/TD)                                       │
├──────────────────────────────────────────────────────────────┤
│  STEP 3: Apply Security Icons                                │
│  Scan node labels for 25 security keyword categories         │
│  and prepend matching icons (fa:fa-* or emoji)               │
├──────────────────────────────────────────────────────────────┤
│  STEP 4: Generate Valid Mermaid Code                         │
│  Produce syntactically correct code following per-type       │
│  rules and examples                                          │
├──────────────────────────────────────────────────────────────┤
│  STEP 5: Provide Viewing Instructions                        │
│  Direct user to mermaid.live with step-by-step guidance      │
├──────────────────────────────────────────────────────────────┤
│  STEP 6: Format Output                                       │
│  Diagram Type → Mermaid Code → View & Export → Summary       │
└──────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- An active **Microsoft Security Copilot** instance with permissions to manage custom plugins.
- **Owner** or **Contributor** role in Security Copilot to upload custom plugins.
- A browser to access the [Mermaid Live Editor](https://mermaid.live) for viewing and exporting diagrams.

---

## Installation

### Step 1 — Download the Manifest

Download the `MermaidDiagramPluginGC.yaml` file from this repository to your local machine.

### Step 2 — Open Security Copilot

1. Navigate to [Microsoft Security Copilot](https://securitycopilot.microsoft.com).
2. Sign in with your organizational credentials.

### Step 3 — Upload the Plugin

1. Click the **Sources** icon (🔌) in the bottom-right corner of the Security Copilot prompt bar.
2. Click **Manage plugins** at the top of the sources panel.
3. In the plugin management page, click **Add plugin**.
4. Select **Security Copilot Plugin** as the plugin type.
5. Choose **Upload a file** and select the [`MermaidDiagramPluginGC.yaml`](https://github.com/the-sentinental-guy/YashureSecurity/blob/main/Mermaid%20Diagram%20Generator/Manifest.yaml) file you downloaded.
6. Click **Add** to upload the manifest.

> **Note:** If you see a deployment error like `"Failed to create an instance of type 'System.String'"`, ensure you are using the latest version of the manifest from this repository. Earlier versions had schema issues that have since been fixed.

### Step 4 — Enable the Plugin

1. After uploading, you should see **Mermaid Diagram Generator** in your custom plugins list.
2. Toggle the plugin **ON** to enable it.
3. The plugin is now active and will be **automatically selected** by the orchestrator whenever you request a diagram in any session.

---

## Usage

### Automatic Invocation

The plugin is designed to be **auto-selected** by the Security Copilot orchestrator. You do not need to manually select it. Simply include a diagram-related request in your prompt during any session:

```
Create a diagram for this session
```
```
Visualize the investigation flow
```
```
Show me a chart of the compliance breakdown
```
```
Map out the findings as a diagram
```
```
Diagram the architecture
```
<img width="1482" height="554" alt="image" src="https://github.com/user-attachments/assets/3f6ac0d9-0d11-429a-ab85-417aaffdd3a3" />


The orchestrator matches these requests against the plugin's keyword triggers and invokes it automatically.

### Specifying a Diagram Type

If you want a specific diagram type instead of auto-detection, mention it in your prompt:

```
Create a Gantt chart for the remediation plan
```
```
Show this as a sequence diagram
```
```
Generate a timeline of the incident
```
```
Create a mind map of the threat landscape
```
```
Draw a pie chart of the severity distribution
```

### Focusing on a Specific Area

You can narrow the diagram to a specific aspect of the session:

```
Create a flowchart focusing only on the non-compliant resources
```
```
Diagram just the remediation workflow
```
```
Visualize only the API call sequence
```

---

## Viewing Your Diagram

When the plugin generates a diagram, it outputs the Mermaid code in a markdown code block along with instructions to view it. Follow these steps:

### Step 1 — Open Mermaid Live Editor

Open [https://mermaid.live](https://mermaid.live) in your browser.

### Step 2 — Paste the Code

1. In the Mermaid Live Editor, locate the **code editor panel** on the left side of the screen.
2. **Clear** any existing sample code in the editor.
3. **Copy** the Mermaid code from the Security Copilot response (the content inside the ` ```mermaid ` code block).
4. **Paste** it into the editor panel.

Your diagram will render **automatically** in the preview panel on the right side.

<img width="1915" height="911" alt="image" src="https://github.com/user-attachments/assets/25441ba9-5fc0-4481-8180-bc580cca7e2d" />

### Step 3 — Adjust the Layout

For the best visual results, especially for flowcharts:

1. Look for the **gear icon** (⚙️) on the left side of the editor — this opens the configuration panel.
2. Under the **Actions** section, find the layout/renderer option.
3. Switch the renderer to **Elk** for an adaptive layout that spaces nodes evenly and handles complex flows better than the default hierarchical layout.

> **Tip:** The plugin already embeds Elk renderer directives in the generated code (e.g., `%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%`), so the Elk layout should activate automatically in most cases. If it doesn't, switching manually ensures the best result.

<img width="354" height="100" alt="image" src="https://github.com/user-attachments/assets/c6eaed4d-4bd1-44b2-a4df-d8fca8c24106" />

### Step 4 — Export the Diagram

From the Mermaid Live Editor, you can export your diagram in multiple formats:

1. Open the **Actions** menu at the top of the page.
2. Choose your preferred export format:
   - **PNG** — Best for embedding in presentations and documents.
   - **SVG** — Best for high-quality, scalable graphics.
   - **PDF** — Best for formal reports and print.
3. You can also **share** the diagram using the share functionality in the Actions panel, which generates a shareable link.

<img width="1023" height="677" alt="image" src="https://github.com/user-attachments/assets/4b56e23b-2ea5-4eaa-be78-be66b1222863" />

---

## Features

### 🔍 Security Icon Mapping

The plugin automatically scans node labels for 25 categories of security-relevant keywords and applies matching icons. Examples:

| Keyword Category | Flowchart Icon | Other Diagram Types |
|---|---|---|
| Defense, Policy, Shield | `fa:fa-shield-alt` | 🛡️ |
| Critical, Threat, Attack | `fa:fa-exclamation-circle` | 🔴 |
| Warning, Suspicious | `fa:fa-exclamation-triangle` | ⚠️ |
| Compliant, Resolved | `fa:fa-check-circle` | ✅ |
| Encryption, Secure | `fa:fa-lock` | 🔒 |
| User, Identity | `fa:fa-user` | 👤 |
| Alert, Incident | `fa:fa-bell` | 🚨 |
| Remediation, Fix | `fa:fa-wrench` | 🔧 |
| Search, Investigate | `fa:fa-search` | 🔍 |
| Sentinel, SIEM | `fa:fa-satellite-dish` | 📡 |

### ⚡ Adaptive Layout Engine

- **Flowcharts** and **state diagrams** use the [Elk](https://eclipse.dev/elk/) rendering engine, which produces well-spaced, adaptive layouts.
- **Flowchart direction** is intelligently selected:
  - `LR` (left-to-right) for linear, sequential flows — most security workflows.
  - `TD` (top-down) for branching, hierarchical structures.
- **Sequence diagrams** are configured with optimized font sizing and actor mirroring disabled for readability.

### 🔒 Zero Fabrication Policy

Every node, edge, and label in the generated diagram comes directly from the session content. The plugin enforces 16 critical rules to prevent hallucination, including:

- Maximum 30 nodes per diagram (groups/summarizes if the session is large).
- No special characters in node IDs.
- Node labels with special characters are always quoted.
- Edge labels are kept to a maximum of 4 words.

---

## Plugin Architecture

```
MermaidDiagramPluginGC.yaml
├── Descriptor
│   ├── Name: MermaidDiagramGenerator
│   ├── DisplayName: Mermaid Diagram Generator
│   └── DescriptionForModel (keyword triggers for auto-selection)
│
└── SkillGroups
    └── Format: GPT
        └── Skill: GenerateSessionDiagram
            ├── Inputs
            │   ├── session_content (required) — full session context
            │   ├── diagramType (optional) — user-specified type
            │   └── focusArea (optional) — narrow scope
            ├── Settings
            │   ├── ModelName: gpt-4o
            │   └── Template (processing pipeline)
            │       ├── STEP 1: Analyze Session Context
            │       ├── STEP 2: Select Diagram Type & Layout
            │       ├── STEP 3: Apply Security Icons
            │       ├── STEP 4: Generate Mermaid Code
            │       ├── STEP 5: Viewing Instructions
            │       ├── STEP 6: Format Output
            │       └── 16 Critical Rules
            └── SupportedAuthTypes: None
```

---

## Limitations

- **Mermaid syntax only** — The plugin generates Mermaid markdown code. It does not render images directly in the Security Copilot response. You need to use the [Mermaid Live Editor](https://mermaid.live) or another Mermaid renderer to visualize the diagram.
- **30-node limit** — To keep diagrams readable, the plugin groups or summarizes content when the session contains more than 30 entities. This means some granularity may be lost in very large sessions.
- **No persistent state** — Each invocation reads the full session context fresh. There is no incremental diagram building across multiple invocations (each call generates a complete, standalone diagram).
- **LLM-dependent** — The quality and accuracy of diagrams depend on the underlying GPT model's ability to parse complex session contexts. In rare cases, the model may misinterpret ambiguous session content.

---

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| Plugin doesn't appear after upload | Upload failed silently | Re-upload the YAML file and check for error messages in the Manage plugins panel |
| Deployment error: `"Failed to create an instance of type 'System.String'"` | Outdated manifest with schema issues | Use the latest version of `MermaidDiagramPluginGC.yaml` from this repository |
| Plugin not auto-selected | Prompt doesn't match keyword triggers | Include explicit terms like "diagram", "visualize", "flowchart", or "chart" in your prompt |
| Diagram doesn't render in Mermaid Live Editor | Syntax error in generated code | Check for unquoted special characters in labels; report the issue |
| Flowchart layout looks cramped | Default Dagre renderer being used | Switch to **Elk** renderer in the Mermaid Live Editor configuration (gear icon) |
| Diagram is too complex / hard to read | Session has too many entities | Use the `focusArea` input to narrow the scope (e.g., "focus only on non-compliant resources") |

---

## Contributing

Contributions are welcome. If you'd like to improve the plugin:

1. Fork this repository.
2. Make your changes to `MermaidDiagramPluginGC.yaml`.
3. Test by uploading to your Security Copilot instance.
4. Submit a pull request with a description of your changes and test results.

### Areas for Contribution

- Additional diagram type examples and rules.
- Improved icon mapping for specialized security domains.
- Theme support (dark/light/neutral).
- Testing across different session types and reporting results.

---

## License

This project is provided as-is under the [MIT License](LICENSE). See the LICENSE file for details.
