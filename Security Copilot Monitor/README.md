# 📊 Security Copilot Monitor — Agent & Plugin

An interactive [Microsoft Security Copilot](https://learn.microsoft.com/en-us/security-copilot/) agent and plugin pair that turns your Security Copilot audit and usage telemetry into an executive-grade monitoring dashboard — covering **usage, adoption, sessions, plugin and agent administration, anomalies, geography, departments, and actual SCU billing cost** — all from inside Security Copilot itself.

This README covers:

- **`SecurityCopilotMonitor-Agent.yaml`**  — interactive monitoring agent
- **`SecurityCopilotMonitor-Plugin.yaml`** — unified KQL + Azure Cost Management skillset

---

## Table of Contents

- [Overview](#overview)
- [What the Agent and Plugin Do](#what-the-agent-and-plugin-do)
- [Architecture and Data Flow](#architecture-and-data-flow)
- [Plugin Capabilities](#plugin-capabilities)
  - [Usage Analytics Skills (KQL)](#usage-analytics-skills-kql)
  - [Cost & Billing Skill (API)](#cost--billing-skill-api)
  - [Agent-side GPT Skills](#agent-side-gpt-skills)
- [Monitoring Summary Dashboard](#monitoring-summary-dashboard)
- [Prerequisites](#prerequisites)
  - [Required Data Sources](#required-data-sources)
  - [Required Permissions and Scopes](#required-permissions-and-scopes)
  - [Required Inputs](#required-inputs)
- [Installation](#installation)
- [Usage](#usage)
  - [Starter Prompts (One-Click)](#starter-prompts-one-click)
  - [Sample Prompts by Scenario](#sample-prompts-by-scenario)
- [Cost Data Notes](#cost-data-notes)
- [Output Format and Behavior](#output-format-and-behavior)
- [Limitations](#limitations)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## Overview

Security Copilot generates a rich stream of audit and interaction telemetry every time a user prompts the product, manages a plugin, triggers an agent, or consumes Security Compute Units (SCU). However, that data is spread across:

- **Defender XDR Advanced Hunting** (`CopilotActivity`, `CloudAppEvents`, `IdentityInfo`)
- **Azure Cost Management** (Consumption Usage Details API)

…and is difficult to consolidate into a coherent view of *who is using Security Copilot, where, when, how much, and what is it costing the organization*.

The **Security Copilot Monitor** agent + plugin pair solves this by providing:

- A single **monitoring agent** that opens with a consolidated yet detailed dashboard on session start.
- **16 KQL analytics skills** that query Defender XDR for usage, adoption, sessions, anomalies, geography, departments, and plugin / agent admin activity.
- **1 Azure Cost Management API skill** that returns real billing data — separating **Provisioned SCU** from **Overage SCU** charges and respecting your tenant's billing currency.
- **3 GPT analytic skills** that classify adoption maturity, summarize cost data with currency-safe rules, and compile the final monitoring report.

> 📸 **Screenshot placeholder — landing view of the agent in Security Copilot:**  
> _Insert screenshot of the Security Copilot Monitor agent tile / suggested starter prompts here._

---

## What the Agent and Plugin Do

| Capability | Description |
|---|---|
| **Usage Overview** | Total events, interaction events, admin events, unique users, unique sessions, and date range over a configurable lookback. |
| **User Adoption** | Most active users, prompt vs response volume, sessions per user, active days, and a cumulative adoption curve. |
| **Time-Based Patterns** | Peak usage hours, day-of-week distribution, daily usage trend, and current-week-vs-previous-week comparison. |
| **Workspace Utilization** | Per-workspace interactions, prompts, sessions, unique users, and plugin admin events. Workspace names are resolved from plugin CRUD events. |
| **Plugin & Agent Administration** | Audit of plugin create/delete/enable/disable operations and Security Copilot agent trigger counts. |
| **Anomaly Detection** | Z-score based detection of usage spikes (Z > 2.0) and drops (Z < −2.0) against the 30-day baseline. |
| **Geographic Distribution** | Country code, city, and ISP breakdown of where Security Copilot is being accessed from. |
| **Team / Department Breakdown** | Department-level usage rollup using `IdentityInfo` joined with `CloudAppEvents`. |
| **Real SCU Cost** | Actual billing data from Azure Cost Management API, separated into **Provisioned SCU** vs **Overage SCU**, with daily breakdown and trend description — never inferred from usage telemetry. |

---

## Architecture and Data Flow

```
┌──────────────────────────────────────────────────────────────────────────┐
│  USER → Security Copilot (Standalone)                                    │
│         "Generate the Security Copilot monitoring summary dashboard"     │
├──────────────────────────────────────────────────────────────────────────┤
│  AGENT: SecurityCopilotMonitorAgent (Format: Agent + Format: GPT)        │
│  - Orchestrator: DefaultSecurityCopilot                                  │
│  - Required input: SubscriptionId (Azure GUID)                           │
│  - Routes prompts to child KQL/API skills, then GPT analyst skills       │
├──────────────────────────────────────────────────────────────────────────┤
│  PLUGIN: SecurityCopilotMonitorPluginV6                                  │
│  ┌──────────────────────────────────────┐  ┌─────────────────────────┐   │
│  │ KQL Skills (Target: Defender)        │  │ API Skill               │   │
│  │ - DiscoverCopilotActivity            │  │ - QuerySecurityCopilot- │   │
│  │ - GetMostActiveUsers                 │  │   Cost                  │   │
│  │ - GetPeakUsageHours                  │  │                         │   │
│  │ - GetDayOfWeekUsage                  │  │ HTTPS GET               │   │
│  │ - GetDailyUsageTrend                 │  │ management.azure.com    │   │
│  │ - GetWorkspaceUsage                  │  │ /subscriptions/{id}/    │   │
│  │ - GetSessionMetrics                  │  │  providers/Microsoft.   │   │
│  │ - GetPluginAdminActivity             │  │  Consumption/           │   │
│  │ - GetAgentUsageTracking              │  │  usageDetails           │   │
│  │ - GetPromptResponseAnalysis          │  │                         │   │
│  │ - GetUserAdoptionCurve               │  │ api-version=2024-08-01  │   │
│  │ - DetectUsageAnomalies               │  │ $expand=properties/     │   │
│  │ - GetWeeklyComparison                │  │   meterDetails          │   │
│  │ - GetRecordTypeBreakdown             │  │                         │   │
│  │ - GetGeoDistribution                 │  │ Auth: AADDelegated      │   │
│  │ - GetTeamDeptUsage                   │  │ Scope: management.azure │   │
│  │                                      │  │ .com/user_impersonation │   │
│  └──────────────────────────────────────┘  └─────────────────────────┘   │
├──────────────────────────────────────────────────────────────────────────┤
│  GPT SKILLS (inline in agent)                                            │
│  - AnalyzeSCUCostData       → currency-safe cost breakdown               │
│  - ClassifyUsagePatterns    → adoption maturity, engagement segments     │
│  - FormatMonitoringReport   → final 6-section markdown dashboard         │
├──────────────────────────────────────────────────────────────────────────┤
│  DATA SOURCES                                                            │
│  - CopilotActivity      (Defender XDR Advanced Hunting / Sentinel)       │
│  - CloudAppEvents       (Defender XDR — geo enrichment)                  │
│  - IdentityInfo         (Defender XDR — department enrichment)           │
│  - Microsoft.Consumption (Azure Management API)                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Plugin Capabilities

### Usage Analytics Skills (KQL)

All 16 KQL skills target **Defender XDR Advanced Hunting** and filter on `AppIdentity == "Copilot.Security.SecurityCopilot"`. Most accept a `LookbackDays` input (default `30`).

| # | Skill | Purpose | Primary Table |
|---|---|---|---|
| 1 | `DiscoverCopilotActivity` | Top-level overview: total events, interaction events, admin events, unique users, unique sessions, first/last event. | `CopilotActivity` |
| 2 | `GetMostActiveUsers` | Top 25 users ranked by interaction volume with prompts, responses, sessions, active days, avg prompts/session. | `CopilotActivity` |
| 3 | `GetPeakUsageHours` | Hour-of-day breakdown with event counts and unique users per hour. | `CopilotActivity` |
| 4 | `GetDayOfWeekUsage` | Sunday–Saturday breakdown of usage volume and unique users. | `CopilotActivity` |
| 5 | `GetDailyUsageTrend` | Day-by-day timeline of events, prompts, responses, sessions, and users. | `CopilotActivity` |
| 6 | `GetWorkspaceUsage` | Workspace-level rollup. Resolves workspace names from plugin CRUD events and joins back to interaction data. | `CopilotActivity` |
| 7 | `GetSessionMetrics` | Session duration and depth: avg/max duration, avg prompts/session, avg responses/session. | `CopilotActivity` |
| 8 | `GetPluginAdminActivity` | Plugin lifecycle audit: `CreateCopilotPlugin`, `DeleteCopilotPlugin`, `EnableCopilotPlugin`, `DisableCopilotPlugin`, grouped by user and workspace. | `CopilotActivity` |
| 9 | `GetAgentUsageTracking` | Tracks which custom agents are triggered (`CopilotAgentManagement`, `CopilotForSecurityTrigger`) and how often. | `CopilotActivity` |
| 10 | `GetPromptResponseAnalysis` | Per-user prompt/response counts and avg prompts per session. | `CopilotActivity` |
| 11 | `GetUserAdoptionCurve` | New users per day and cumulative adoption curve. | `CopilotActivity` |
| 12 | `DetectUsageAnomalies` | Statistical Z-score analysis — flags days as `Spike` (Z > 2.0), `Drop` (Z < −2.0), or `Normal`. | `CopilotActivity` |
| 13 | `GetWeeklyComparison` | Current 7-day window vs preceding 7-day window: events, users, sessions, with % change. | `CopilotActivity` |
| 14 | `GetRecordTypeBreakdown` | Distribution of all `RecordType` values (interactions, plugin ops, agent ops, etc.). | `CopilotActivity` |
| 15 | `GetGeoDistribution` | Country / city / ISP breakdown with event and user counts. | `CloudAppEvents` |
| 16 | `GetTeamDeptUsage` | Per-department sessions, events, unique users. | `IdentityInfo` + `CloudAppEvents` |

### Cost & Billing Skill (API)

| Skill | Description |
|---|---|
| `QuerySecurityCopilotCost` | Calls `GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Consumption/usageDetails` with `api-version=2024-08-01` and `$expand=properties/meterDetails`. Returns real Azure Consumption usage line items. Filtered to Security Copilot records (where `consumedService` is `microsoft.securitycopilot`, `meterCategory` is `Microsoft Security Copilot`, or `product` starts with `Microsoft Security Copilot`). |

### Agent-side GPT Skills

❗️ These three skills are embedded in the agent manifest itself, not in the plugin.❗️

| Skill | Purpose |
|---|---|
| `AnalyzeSCUCostData` | Parses the raw Azure Consumption response into a structured cost summary, daily breakdown, Provisioned vs Overage split, trend description, optimization recommendations, and source-grounding note. Currency-safe: never converts or assumes currency; preserves whatever the API returns. |
| `ClassifyUsagePatterns` | Classifies adoption maturity (🔴 Exploring / 🟡 Emerging / 🟢 Established / 🔵 Advanced), segments users (Power / Regular / Occasional / Dormant), and produces three actionable recommendations. |
| `FormatMonitoringReport` | Compiles all KQL and API outputs from the current session into the six-section Monitoring Summary Dashboard. |

---

## Monitoring Summary Dashboard

When the agent is invoked with an empty `UserRequest`, or with any prompt that asks for an overview / dashboard / summary / report, it runs the **Monitoring Summary Dashboard Workflow**:

1. `GetMostActiveUsers` (last 30 days)
2. `GetWorkspaceUsage` (last 30 days)
3. `GetPluginAdminActivity` (last 30 days)
4. `GetAgentUsageTracking` (last 30 days)
5. `DetectUsageAnomalies` (last 30 days)
6. `GetDailyUsageTrend` (last 30 days) — anomaly context
7. `QuerySecurityCopilotCost` (rolling past 30 days, not the month)
8. `AnalyzeSCUCostData`
9. `FormatMonitoringReport`

The output is a single markdown response with six sections separated by horizontal rules and prefixed with emoji indicators:

```
🛡️ Security Copilot Monitoring Summary Dashboard
   Usage lookback: 30 days
   Cost window:    rolling past 30 days
   Subscription:   <masked GUID — first 8 chars + … + last 4 chars>

---
👤 1. Top Active Users
    Table: User | Interactions | Prompts | Responses | Sessions | Active Days
    Quick read: <one-line takeaway>

---
🏢 2. Workspace Usage
    Table: Workspace | Interactions | Prompts | Sessions | Unique Users | Plugin Admin Events
    Quick read: <one-line takeaway>

---
🛠️ 3. Plugin and Agent Admin Activity
    Plugin CRUD summary + agent trigger summary
    Quick read: <one-line takeaway>

---
⚠️ 4. Usage Anomalies or Spikes
    Z-score anomalies + daily trend context (Spike / Drop / Normal)
    Quick read: <one-line takeaway>

---
💰 5. Cost — Rolling Past 30 Days
    Total cost by currency, Provisioned SCU cost, Overage SCU cost, daily breakdown
    Quick read: <one-line takeaway>

---
📌 6. Recommended Follow-Ups
    2–3 suggested drilldowns, including one cost follow-up
```

> 📸 **Screenshot placeholder — full Monitoring Summary Dashboard:**  
> _Insert end-to-end screenshot of the generated dashboard (all 6 sections) here._

> 📸 **Screenshot placeholder — Section 1: Top Active Users table:**  
> _Insert screenshot of the rendered Top Active Users table here._

> 📸 **Screenshot placeholder — Section 2: Workspace Usage table:**  
> _Insert screenshot of the Workspace Usage table here._

> 📸 **Screenshot placeholder — Section 3: Plugin & Agent Admin Activity:**  
> _Insert screenshot of the plugin/agent administration summary here._

> 📸 **Screenshot placeholder — Section 4: Usage Anomalies:**  
> _Insert screenshot of the anomaly detection output (Spike / Drop / Normal classification) here._

> 📸 **Screenshot placeholder — Section 5: SCU Cost breakdown (Provisioned vs Overage):**  
> _Insert screenshot of the cost section showing currency, Provisioned/Overage split, and daily breakdown here._

---

## Prerequisites

### Required Data Sources

| Source | Why It Is Required | Used By |
|---|---|---|
| **`CopilotActivity` table populated in Defender XDR Advanced Hunting (Sentinel / Log Analytics)** | This is the primary telemetry source. **All 14 of the 16 KQL skills depend on it.** Without it, no usage, adoption, session, anomaly, peak-hour, workspace, plugin-admin, or agent-trigger analytics are possible. | Skills 1–14 in the plugin |
| **`CloudAppEvents` table (Defender XDR — M365 app connector)** | Provides geographic enrichment (`City`, `CountryCode`, `ISP`) and `AccountObjectId` for joining to identity data. | `GetGeoDistribution`, `GetTeamDeptUsage` |
| **`IdentityInfo` table (Defender XDR — Defender for Identity or Entra ID sync)** | Provides `Department`, `JobTitle`, and `Manager` for the per-department usage rollup. | `GetTeamDeptUsage` |
| **Azure subscription with Microsoft Security Copilot billing records** | The Cost API only returns Security Copilot billing if the subscription actually contains SCU charges (Provisioned and/or Overage). | `QuerySecurityCopilotCost`, `AnalyzeSCUCostData` |

> ⚠️ **The single most important prerequisite is that `CopilotActivity` must be populated and queryable in Defender XDR Advanced Hunting.** If `CopilotActivity` returns no rows for `AppIdentity == "Copilot.Security.SecurityCopilot"`, every usage and adoption section in the dashboard will report "data unavailable".
>
> If your tenant does not yet emit `CopilotActivity` events, confirm that **Security Copilot auditing** is enabled and that Defender XDR's unified audit log is producing records.

### Required Permissions and Scopes

| Requirement | Purpose |
|---|---|
| **Security Copilot Owner or Contributor** | To upload the plugin and the agent manifests. |
| **Microsoft Sentinel Reader / Defender XDR access** to the workspace containing `CopilotActivity`, `CloudAppEvents`, and `IdentityInfo` | So Security Copilot can execute the KQL skills against Defender Advanced Hunting on your behalf. |
| **Azure RBAC**: at minimum **Cost Management Reader** (or **Reader**) on the target subscription | So the signed-in user can read `Microsoft.Consumption/usageDetails` for the cost skill. |
| **Entra ID scope consent**: `https://management.azure.com/user_impersonation` (AAD Delegated) | Granted on first run when the agent calls the Azure Cost Management API. The plugin declares this scope in its `Authorization` block. |
| **Defender for Identity** or **Entra ID sync** populating `IdentityInfo` | Required only for `GetTeamDeptUsage`. Without it, department-level breakdowns will be empty. |

### Required Inputs

| Input | Type | Required | Description |
|---|---|---|---|
| `SubscriptionId` | Azure GUID | ✅ Yes | Azure subscription ID that contains the Security Copilot billing records. The agent enforces this as a **required input** so the cost section is included on the first run. The agent masks this value in the dashboard (`first 8 chars + "…" + last 4 chars`). |
| `UserRequest` | string | ❌ No | Optional analyst request. If empty, the agent auto-runs the Monitoring Summary Dashboard with no clarifying question. |
| `LookbackDays` | integer (per KQL skill) | ❌ No | Defaults to `30` for every KQL skill that accepts it. Override by mentioning a different period in your prompt (e.g., "last 7 days"). |

---

## Installation

1. **Verify prerequisites** — confirm that `CopilotActivity` is populated in your Defender XDR Advanced Hunting environment (run `CopilotActivity | where AppIdentity == "Copilot.Security.SecurityCopilot" | take 5`). Confirm the target Azure subscription contains Security Copilot SCU charges.

2. **Navigate to Microsoft Security Copilot** — [https://securitycopilot.microsoft.com](https://securitycopilot.microsoft.com) and sign in.

3. **Upload the plugin manifest**
   - Click the **Sources** icon (🔌) in the prompt bar.
   - Click **Manage plugins** → **Add plugin** → **Security Copilot Plugin**.
   - Upload `SecurityCopilotMonitor-Plugin.yaml`.
   - Toggle the plugin **ON**.

4. **Authorize Azure access** — on first invocation of `QuerySecurityCopilotCost`, Security Copilot will prompt for AAD Delegated consent for the `https://management.azure.com/user_impersonation` scope. Approve.

5. **Upload the agent manifest**
   - Open the **Agents** management page.
   - Upload `SecurityCopilotMonitor-Agent.yaml`.
   - Confirm that the agent shows `SecurityCopilotMonitorPlugin` in its required skillsets and resolves all child skills successfully.

6. **Provide the Azure Subscription ID** — when launching the agent for the first time, supply the GUID of the subscription that holds your Security Copilot billing.

> 📸 **Screenshot placeholder — agent skill resolution view (all child skills resolved against the plugin skillset):**  
> _Insert screenshot of the agent's child skill resolution / plugin binding here._

---

## Usage

Once installed and configured, simply open the **Security Copilot Monitor Agent** standalone agent. It will offer one-click starter prompts and accept natural-language follow-ups.

### Starter Prompts (One-Click)

The agent surfaces five starter prompts directly in the UI:

| Title | Prompt |
|---|---|
| **Monitoring Dashboard** | Generate the Security Copilot monitoring summary dashboard. |
| **Top Users** | Who are the most active Security Copilot users? |
| **Workspace Usage** | Show workspace-wise Security Copilot usage breakdown. |
| **Cost Last 30 Days** | How much did we spend on Security Copilot in the last 30 days? |
| **Overage Cost** | Show Security Copilot overage cost breakdown for the last 30 days. |

> 📸 **Screenshot placeholder — starter prompt tiles inside Security Copilot:**  
> _Insert screenshot of the starter prompt tiles surfaced by the agent here._

### Sample Prompts by Scenario

#### 📊 Comprehensive overview
```
Generate the Security Copilot monitoring summary dashboard.
```
```
Give me a complete monitoring report for the last 30 days.
```
```
Show me everything — users, workspaces, anomalies, admin activity, and cost.
```

#### 👤 User adoption & engagement
```
Who are the most active Security Copilot users in the past 30 days?
```
```
Show me prompt vs response activity per user.
```
```
Show me the user adoption curve over the last 60 days.
```
```
How many new users started using Security Copilot this month?
```

#### 🕒 Time-based patterns
```
What are the peak usage hours for Security Copilot?
```
```
Which day of the week sees the most Security Copilot activity?
```
```
Show the daily usage trend for the last 14 days.
```
```
Compare this week's usage against last week.
```

#### 🏢 Workspace & administration
```
Show workspace-wise Security Copilot usage breakdown.
```
```
Show me all plugin create / delete / enable / disable activity in the last 30 days.
```
```
Which custom agents are being triggered the most?
```
```
Give me a breakdown of all Security Copilot record types in the last 30 days.
```

#### 🌍 Geography & departments
```
Where are users accessing Security Copilot from?
```
```
Show me Security Copilot usage by country.
```
```
Which department is using Security Copilot the most?
```

#### ⚠️ Anomalies & investigation
```
Are there any usage anomalies or spikes in the last 30 days?
```
```
Detect any unusual drops in Security Copilot usage.
```
```
Show usage anomalies and admin activity together.
```

#### 💰 Cost & billing (Azure Cost Management API)
```
How much did we spend on Security Copilot in the last 30 days?
```
```
What is the Security Copilot cost this week?
```
```
Show me Security Copilot overage cost for the last 14 days.
```
```
Break down Security Copilot cost by Provisioned vs Overage SCU.
```
```
What did Security Copilot charge us between May 1 and May 29?
```
```
Show month-to-date SCU spend.
```

#### 🔀 Combined drilldowns
```
Show top active users and workspace usage together.
```
```
Show usage anomalies and admin activity together.
```
```
Follow up with Security Copilot cost for the last 30 days.
```

> 📸 **Screenshot placeholder — example chat drilldown response (e.g., Cost breakdown):**  
> _Insert screenshot of a cost-only chat answer with the Provisioned vs Overage split and currency here._

---

## Cost Data Notes

The cost section is sourced **exclusively** from the Azure Cost Management Consumption Usage Details API. The agent will never derive cost from KQL usage telemetry (sessions, prompts, interactions).

| Topic | Detail |
|---|---|
| **API endpoint** | `GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Consumption/usageDetails` |
| **API version** | `2024-08-01` |
| **Expand parameter** | `properties/meterDetails` — required so the agent can reliably identify Security Copilot meters. |
| **Filter parameter** | Optional OData `$filter`. The agent constructs a date filter when a specific range is requested. |
| **Authentication** | AAD Delegated; scope `https://management.azure.com/user_impersonation`. |
| **Provisioned vs Overage classification** | A row is classified **Overage SCU** only when its actual `product` / `meter` text contains "Overage". It is classified **Provisioned SCU** only when its text contains "Provisioned" or contains "Security Compute Unit" without "Overage". Anything else from Security Copilot meters is labelled **Unclassified Security Copilot cost**. |
| **Currency handling** | The agent **never converts currency** and **never assumes a tenant currency**. The currency is taken directly from `billingCurrency` / `billingCurrencyCode` on each row. If multiple currencies appear, one total per currency is shown. |
| **Billing lag** | Azure Cost Management typically lags real-time consumption by **~24–48 hours**. The most recent day in the response may show incomplete or zero data. The agent surfaces this caveat in every cost summary. |
| **Rate limits** | Azure Cost Management imposes a rate limit of approximately **4 requests/minute per scope**. The agent handles `HTTP 429` with a retry-later message. |
| **Empty response handling** | If no Security Copilot rows are returned for the period, the agent says: *"No Security Copilot billing records found for this period"* and reminds the user of the 24–48 hour billing lag. |

---

## Output Format and Behavior

| Rule | Behavior |
|---|---|
| **Never dumps raw output** | KQL JSON and API JSON are always summarized into compact markdown tables and narrative. |
| **UPN format for users** | Users are shown as `user@domain.com`. |
| **Masked subscription** | The Azure subscription ID is always rendered in masked form (`first 8 chars + "…" + last 4 chars`). |
| **Compact tables** | 5–10 rows by default; only the dashboard's Top Users / Workspace tables go up to 10. |
| **Trend descriptions in words** | Security Copilot cannot render charts, so all trend, spike, and seasonality observations are described in natural language. |
| **Section gating** | Every major dashboard section begins with a horizontal rule (`---`), a numbered emoji heading, and ends with a one-line **Quick read:** takeaway. |
| **Failure resilience** | If any individual skill fails, the dashboard continues and marks that section as **data unavailable** with the specific missing prerequisite or error reason. |

---

## Limitations

- **`CopilotActivity` is mandatory** — the entire usage analytics surface requires this table. If it is not populated, only the cost section (powered by the Azure Cost Management API) will produce data.
- **Cost data lags by ~24–48 hours** — most recent day in any cost section may be incomplete or empty.
- **Geographic data depends on `CloudAppEvents`** — tenants without the M365 app connector wired into Defender XDR will see an empty `GetGeoDistribution` result.
- **Department breakdown depends on `IdentityInfo`** — requires Defender for Identity or Entra ID sync. Without it, `GetTeamDeptUsage` returns no rows.
- **One subscription per agent run** — the agent operates against a single `SubscriptionId` per session. For multi-subscription billing, run the agent separately per subscription and aggregate manually.
- **No chart rendering** — Security Copilot agents render markdown only; trends, spikes, and distributions are described in words and tables, not visual charts.
- **Azure Cost Management rate limit** — `~4 requests/minute per scope`. Rapid-fire cost prompts may briefly throttle.
- **Workspace name resolution** — only workspaces that have at least one plugin CRUD event in the lookback window can have their names resolved. Workspaces with only interaction events appear as `Unresolved`.

---

## Troubleshooting

| Issue | Likely Cause | Resolution |
|---|---|---|
| Dashboard shows "data unavailable" for usage sections | `CopilotActivity` is empty for the lookback window or `AppIdentity` filter | Confirm Security Copilot auditing is enabled and that records exist in Defender Advanced Hunting (`CopilotActivity | where AppIdentity == "Copilot.Security.SecurityCopilot" | take 5`). |
| `GetGeoDistribution` returns no rows | `CloudAppEvents` is not populated, or the M365 app connector is not configured in Defender XDR | Configure the M365 app connector in Defender XDR. Geo data depends on `CloudAppEvents` containing `CopilotInteraction` actions. |
| `GetTeamDeptUsage` returns no rows | `IdentityInfo` is missing or has no `Department` field for the users | Configure Defender for Identity or Entra ID sync so `IdentityInfo` is populated with department data. |
| Cost section returns `401` or `403` | The signed-in user has not granted `management.azure.com/user_impersonation`, or lacks `Cost Management Reader` on the subscription | Re-authorize the plugin on the consent screen, and ensure the user has at least `Cost Management Reader` (or `Reader`) on the subscription. |
| Cost section returns `429` | Azure Cost Management throttled the request (~4 req/min per scope) | Wait a minute and re-run the cost prompt. Avoid rapid back-to-back cost requests on the same scope. |
| Cost section says "No Security Copilot billing records found for this period" | Either the subscription has no SCU charges yet, or the request fell inside the 24–48 hour billing lag | Confirm the subscription truly contains Security Copilot SCU charges; try expanding the period (e.g., "last 14 days" instead of "yesterday"). |
| Agent asks for the subscription ID despite already supplying it | The required `SubscriptionId` input was not persisted on agent launch | Re-launch the agent and supply the `SubscriptionId` GUID at the input prompt before running the dashboard. |
| Workspaces appear as "Unresolved" | No plugin CRUD events in the lookback window for that workspace | Either widen the lookback (`LookbackDays`) or accept that workspaces without admin activity cannot have their names resolved from `CopilotActivity` alone. |
| Cost shows two currencies | The subscription is billed in multiple currencies | Expected behavior — the agent shows one total per currency and never converts. |

---

## License

This project is provided as-is under the [MIT License](LICENSE). See the LICENSE file for details.
