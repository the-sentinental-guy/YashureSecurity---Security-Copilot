# 📊 HTML Dashboard Generator V2 — Security Copilot Plugin

A GPT-based plugin for [Microsoft Security Copilot](https://learn.microsoft.com/en-us/security-copilot/) that transforms any session's findings into a **standalone, CISO-grade HTML dashboard** — complete with metric cards, donut charts, gauge charts, bar charts, severity badges, contextual Azure Portal links, and professionally styled data tables. The output is a single self-contained HTML file with no external dependencies — open it in any browser, print it, or share it as-is.

---

## Table of Contents

- [Overview](#overview)
- [Dashboard Components](#dashboard-components)
- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [Step 1 — Download the Manifest](#step-1--download-the-manifest)
  - [Step 2 — Open Security Copilot](#step-2--open-security-copilot)
  - [Step 3 — Upload the Plugin](#step-3--upload-the-plugin)
  - [Step 4 — Enable the Plugin](#step-4--enable-the-plugin)
- [Usage](#usage)
  - [Automatic Invocation](#automatic-invocation)
  - [Example Prompts](#example-prompts)
- [What the Dashboard Looks Like](#what-the-dashboard-looks-like)
  - [Header](#header)
  - [Metric Cards](#metric-cards)
  - [Visualizations](#visualizations)
  - [Data Tables](#data-tables)
  - [Azure Portal Links](#azure-portal-links)
  - [Footer](#footer)
- [Viewing and Sharing the Dashboard](#viewing-and-sharing-the-dashboard)
- [Session Type Adaptations](#session-type-adaptations)
- [Plugin Architecture](#plugin-architecture)
- [Design Principles](#design-principles)
- [Limitations](#limitations)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

Security Copilot sessions produce rich, structured data — compliance summaries, policy violations, affected resources, remediation commands, severity distributions, and more. But that data is spread across multiple prompt-response pairs in a conversational format, making it hard to get a quick, executive-level view.

This plugin reads the **entire session context** and transforms it into a **single HTML file** that looks and functions like a real-time security dashboard. A CISO should be able to understand the security posture **within 10 seconds** of opening it.

### Key Characteristics

| Feature | Description |
|---|---|
| **Self-Contained** | Single HTML file with all CSS embedded. No JavaScript, no external dependencies, no CDN links. Works offline. |
| **Visual-First** | Metric cards, donut charts, gauge charts, and bar charts appear before tables. Data communicates at a glance. |
| **Context-Aware** | Adapts layout based on session type — compliance audits get gauges and donut charts, incident investigations get timelines, threat intel gets IOC tables. |
| **Clickable Links** | Generates Azure Portal deep links for subscriptions, resources, and policy definitions found in the session. |
| **Print-Ready** | Includes `@media print` CSS rules that preserve colors and layout when printing or saving as PDF. |
| **Zero Fabrication** | Every data point, number, name, and percentage in the dashboard comes directly from the session content. Nothing is invented. |

---

## Dashboard Components

The plugin can generate the following visual components, selected automatically based on the session data:

### Metric Cards

Color-coded cards displaying key numbers at a glance. Each card has a **value**, **label**, and optional **context line**. The top border color encodes severity:

| Color | Usage |
|---|---|
| 🔴 Red | Critical severity — e.g., critical non-compliant policies |
| 🟠 Orange | High severity — e.g., high-risk findings |
| 🟡 Amber | Medium severity — e.g., medium risk levels |
| 🟢 Green | Low / Healthy — e.g., compliant resources, resolved items |
| 🔵 Blue | Informational — e.g., total counts, neutral metrics |

### Donut Chart

A CSS `conic-gradient` donut chart used for **distributions and breakdowns** — such as policy effect types (Deny/Audit/Modify), severity distributions, or resource categories. Displays the total count in the center with a color-coded legend below.

### Gauge Chart

An SVG semicircular gauge used for **scores and percentages** — such as compliance percentage, risk scores, or posture scores. Color-coded by range:

| Range | Color | Interpretation |
|---|---|---|
| 0–39% | Red | Critical — needs immediate attention |
| 40–69% | Orange | Needs improvement |
| 70–84% | Amber | Fair |
| 85–100% | Green | Healthy |

### Horizontal Bar Chart

A CSS-based bar chart used for **ranked or comparative data** — such as top non-compliant policies by resource count, resources by violation count, or frameworks by policy count. Labels are shown in full (never truncated).

### Severity Badges

Inline pill-shaped badges for severity and status values in tables:

- `CRITICAL` — red badge
- `HIGH` — orange badge
- `MEDIUM` — amber badge
- `LOW` — green badge
- `INFO` — blue badge

### Data Tables

Professionally styled tables with uppercase headers, hover effects, and vertical-align top for multi-line cells. All columns, rows, and values are reproduced exactly as they appear in the session.

### Code Blocks

CLI commands, KQL queries, Azure CLI scripts, and remediation commands are rendered in dark-themed code blocks with monospace font — preserved character-for-character from the session.

---

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│  1. READ SESSION CONTENT                                     │
│  Plugin receives the full session context from the           │
│  Security Copilot orchestrator                               │
├──────────────────────────────────────────────────────────────┤
│  2. EXTRACT KEY METRICS                                      │
│  Identify 3–5 key numbers for metric cards                   │
├──────────────────────────────────────────────────────────────┤
│  3. SELECT VISUALIZATIONS (max 2)                            │
│  Choose the 2 most impactful charts based on data:           │
│  • Distribution data → Donut chart                           │
│  • Score/percentage → Gauge chart                            │
│  • Ranked data → Bar chart                                   │
├──────────────────────────────────────────────────────────────┤
│  4. BUILD DETAIL SECTIONS                                    │
│  Reproduce all tables, findings, CLI commands, and           │
│  recommendations from the session                            │
├──────────────────────────────────────────────────────────────┤
│  5. EMBED AZURE PORTAL LINKS                                 │
│  Generate deep links for subscriptions, resources,           │
│  and policy definitions found in the session                 │
├──────────────────────────────────────────────────────────────┤
│  6. OUTPUT SELF-CONTAINED HTML                                │
│  Single file: header → metrics → charts → details → footer   │
└──────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- An active **Microsoft Security Copilot** instance with permissions to manage custom plugins.
- **Owner** or **Contributor** role in Security Copilot to upload custom plugins.
- Any modern browser (Edge, Chrome, Firefox, Safari) to view the generated HTML dashboard.

---

## Installation

### Step 1 — Download the Manifest

Download the `HTMLDashboardGeneratorV2.yaml` file from this repository to your local machine.

### Step 2 — Open Security Copilot

1. Navigate to [Microsoft Security Copilot](https://securitycopilot.microsoft.com).
2. Sign in with your organizational credentials.

### Step 3 — Upload the Plugin

1. Click the **Sources** icon (🔌) in the bottom-right corner of the Security Copilot prompt bar.
2. Click **Manage plugins** at the top of the sources panel.
3. In the plugin management page, click **Add plugin**.
4. Select **Security Copilot Plugin** as the plugin type.
5. Choose **Upload a file** and select the `HTMLDashboardGeneratorV2.yaml` file you downloaded.
6. Click **Add** to upload the manifest.

### Step 4 — Enable the Plugin

1. After uploading, you should see **HTML Dashboard Generator V2** in your custom plugins list.
2. Toggle the plugin **ON** to enable it.
3. The plugin is now active and will be **automatically selected** by the orchestrator whenever you request a dashboard or HTML report in any session.

---

## Usage

### Automatic Invocation

The plugin is designed to be **auto-selected** by the Security Copilot orchestrator. You do not need to manually select it. Simply request a dashboard or HTML output at any point during your session — after running your investigations, compliance checks, or analysis.

### Example Prompts

**After a compliance audit session:**
```
Create an HTML dashboard for this session
```

**After an incident investigation:**
```
Generate an HTML report of these findings
```

**After a threat intelligence session:**
```
Export this session as an HTML dashboard
```

**General requests that trigger the plugin:**
```
Give me an HTML version of this analysis
```
```
Create a visual dashboard for these results
```
```
Format this as a printable HTML report
```
```
Create a CISO dashboard from this session
```

> **Tip:** For best results, run all your investigative prompts first and request the dashboard as the *last* prompt in the session. This ensures the plugin has the complete context to work with.

---

## What the Dashboard Looks Like

The generated HTML dashboard follows a consistent, professional layout:

### Header

A dark gradient banner (navy to Azure blue) showing:
- **Title** — derived from the session topic (e.g., "Azure Policy Compliance Dashboard")
- **Subtitle** — scope, subscription ID, date, or other contextual metadata

### Metric Cards

A row of 3–5 color-coded cards immediately below the header, displaying the most important numbers from the session. For example, in a compliance session:

| Card | Value | Color |
|---|---|---|
| Non-Compliant Policies | 15 | 🔴 Red (critical) |
| Active Assignments | 16 | 🔵 Blue (info) |
| Affected Resources | 13 | 🟠 Orange (high) |
| Policy Exemptions | 1 | 🟢 Green (low) |
| Auto-Remediable | 3 | 🔵 Blue (info) |

### Visualizations

Up to **2 charts** placed side-by-side below the metric cards. The plugin selects the most impactful chart types based on data. For example:

- A **donut chart** showing policy effect distribution (Audit: 8, DeployIfNotExists: 3, Deny: 4)
- A **bar chart** showing top non-compliant resources ranked by violation count

### Data Tables

Below the visualizations, the full session data is reproduced in styled tables with:
- Uppercase headers
- Severity badges (colored pills) in severity/status columns
- Clickable resource names linked to Azure Portal (when ARM IDs are present)

### Azure Portal Links

The dashboard generates **clickable deep links** to the Azure Portal wherever identifiable IDs are found in the session:

| ID Type | Link Target |
|---|---|
| Subscription ID | Azure Portal → Subscription Overview |
| ARM Resource ID | Azure Portal → Resource blade |
| Policy Definition GUID | Azure Portal → Policy Definition detail |

Links open in a new tab and display the human-readable name as the link text.

### Footer

A subtle footer at the bottom of the dashboard showing:
- Generation timestamp
- "Generated by Security Copilot Dashboard Generator V2"
- A link to the Azure Portal subscription (if applicable)

---

## Viewing and Sharing the Dashboard

The plugin outputs the HTML directly in the Security Copilot response. To use it:

### Save as HTML File

1. **Copy** the entire HTML output from the Security Copilot response.
2. **Paste** it into a text editor (Notepad, VS Code, etc.).
3. **Save** the file with a `.html` extension (e.g., `compliance-dashboard.html`).
4. **Open** the file in any browser — the dashboard renders immediately with all styling, charts, and links intact.

### Print or Save as PDF

1. Open the HTML file in your browser.
2. Press `Ctrl+P` (Windows) or `Cmd+P` (Mac).
3. The dashboard includes print-optimized CSS — colors, badges, and charts print correctly.
4. Choose **Save as PDF** as the printer destination to create a shareable PDF version.

### Share

The HTML file is fully self-contained — you can email it, upload it to SharePoint, or attach it to a ticket. Recipients only need a browser to view it.

---

## Session Type Adaptations

The plugin recognizes different session types and adapts the dashboard layout accordingly:

### Compliance Audits

- **Gauge** for compliance percentage (if calculable)
- **Donut** for policy effect distribution (Deny/Audit/Modify/DeployIfNotExists)
- **Bar chart** for top non-compliant policies by resource count
- **Metric cards**: total policies, non-compliant count, affected resources, auto-remediable count
- **Links**: policy names linked to Azure Portal policy definitions

### Incident Investigations

- **Metric cards**: alert count, affected entities, severity breakdown
- **Donut** for severity distribution of findings
- **Tables**: timeline of investigation steps, affected assets, indicators
- **Links**: entities linked to Azure Portal where IDs are available

### Threat Intelligence

- **Metric cards**: IOC count, threat actors identified, TTPs
- **Bar chart** for IOC types or MITRE ATT&CK technique frequency
- **Tables**: IOC details, threat actor profiles

### Vulnerability Assessments

- **Gauge** for overall vulnerability score
- **Donut** for severity distribution
- **Bar chart** for top affected hosts
- **Metric cards**: total CVEs, critical count, hosts affected

### General Analysis

- Extracts any countable metrics for metric cards
- Visualizes any distributions found in the data
- Falls back to clean tables and structured text

---

## Plugin Architecture

```
HTMLDashboardGeneratorV2.yaml
├── Descriptor
│   ├── Name: HTMLDashboardGeneratorV2
│   └── DisplayName: HTML Dashboard Generator V2
│
└── SkillGroups
    └── Format: GPT
        └── Skill: GenerateHTMLDashboardV2
            ├── Inputs
            │   └── session_content (required) — full session context
            ├── Settings
            │   ├── ModelName: gpt-4o
            │   └── Template
            │       ├── 9 Critical Rules
            │       ├── HTML Skeleton with embedded CSS
            │       ├── Layout instructions
            │       │   ├── 1. Metric Cards (3–5 key numbers)
            │       │   ├── 2. Visualizations (max 2)
            │       │   │   ├── Donut (CSS conic-gradient)
            │       │   │   ├── Gauge (SVG semicircle)
            │       │   │   └── Bar Chart (CSS flexbox)
            │       │   └── 3. Detail Sections
            │       ├── Azure Portal link generation rules
            │       └── Emoji usage rules
            └── SupportedAuthTypes: None (implicit)
```

---

## Design Principles

These principles guide every dashboard the plugin generates:

| Principle | Implementation |
|---|---|
| **10-Second Clarity** | Metric cards and charts appear first — a CISO gets the picture before reading any text |
| **Data Fidelity** | Every number, name, and value is reproduced character-for-character from the session |
| **No Truncation** | Policy names, resource names, and framework names are always shown in full |
| **Contrast Guarantee** | Light text on dark backgrounds only, dark text on light backgrounds only — no invisible text |
| **Minimal Visuals** | Maximum 2 charts — the rest goes into well-structured tables to keep the dashboard fast and complete |
| **Complete Charts** | All donut percentages must sum to 100%. No partial arcs or missing segments |
| **Formal Emojis** | Only status indicators (🔴 Critical, ✅ Compliant, ⚠️ Warning) — never decorative |
| **No Diagram Code** | Dashboard handles data presentation only — a separate plugin (Mermaid Diagram Generator) handles diagrams |
| **Actionable Links** | Azure Portal links let users click through directly from the dashboard to the relevant blade |
| **Print-Ready** | `@media print` rules ensure the dashboard looks correct when printed or saved as PDF |

---

## Limitations

- **No interactivity** — The dashboard is static HTML. Charts are CSS/SVG-based, not interactive JavaScript widgets. You cannot click to drill down into data.
- **Token-dependent** — Very large sessions may cause the response to be truncated if the generated HTML exceeds the model's output token limit. The plugin is optimized to minimize token usage (minified CSS, max 2 charts), but extremely data-heavy sessions may hit this limit.
- **No JavaScript** — The plugin deliberately avoids JavaScript to ensure the dashboard works in all environments (email clients, restricted browsers, air-gapped systems). This means no dynamic filtering, sorting, or animations.
- **LLM-dependent** — The quality of visualizations depends on the model's ability to correctly parse and calculate values. In rare cases, chart values or percentages may need manual verification.
- **Azure Portal links only** — Deep links are generated only for Azure resources. The plugin cannot generate links for non-Azure platforms (AWS, GCP, third-party SaaS).
- **Session URL not available** — The plugin cannot embed a link back to the Security Copilot session because the session ID is not exposed to plugins as an input.

---

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| Plugin doesn't appear after upload | Upload failed silently | Re-upload the YAML file and check for error messages in the Manage plugins panel |
| Plugin not auto-selected | Prompt doesn't contain trigger keywords | Use explicit terms like "HTML dashboard", "HTML report", "create a dashboard", or "export as HTML" |
| Dashboard renders without styling | HTML output was truncated mid-CSS | The session may be too large. Try running the dashboard request after a shorter session, or re-prompt with "Create a brief HTML dashboard" |
| Donut chart missing segments | Data doesn't sum to 100% | Verify the source data in the session; report the issue for template improvement |
| Text not visible in header | Font color contrast issue | This was fixed in V2; ensure you are using the latest manifest file from this repository |
| Bar chart labels show "..." | Text truncation | This was fixed in V2; ensure you are using the latest manifest file from this repository |
| No Azure Portal links in dashboard | Session content lacks identifiable ARM IDs, GUIDs, or subscription IDs | Links are only generated when real IDs are present in the session data. Run queries that return full resource IDs before requesting the dashboard |
| Response says "GPT response was truncated" | Session content + HTML output exceeds token limit | Re-run the request — Security Copilot automatically retries with a higher token limit. If it persists, try a shorter session |

---

## Contributing

Contributions are welcome. If you'd like to improve the plugin:

1. Fork this repository.
2. Make your changes to `HTMLDashboardGeneratorV2.yaml`.
3. Test by uploading to your Security Copilot instance and generating dashboards from different session types.
4. Submit a pull request with:
   - A description of your changes.
   - Screenshots of the generated dashboard before and after your changes.
   - The session type(s) you tested against.

### Areas for Contribution

- Additional visualization types (treemaps, heatmaps — CSS/SVG only, no JavaScript).
- Improved color palettes and theming (dark mode support).
- Better session-type detection logic.
- Support for non-Azure platform links.
- Accessibility improvements (ARIA labels, screen reader support).

---

## License

This project is provided as-is under the [MIT License](LICENSE). See the LICENSE file for details.
