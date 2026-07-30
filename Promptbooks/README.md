# Security Copilot Promptbooks — YashureSecurity

Custom, ready-to-build [Microsoft Security Copilot](https://learn.microsoft.com/en-us/copilot/security/microsoft-security-copilot) promptbooks that drive the **Security Copilot Monitor** and **Data Security Monitor** agents. Each promptbook is a sequential chain of prompts you paste into the Security Copilot promptbook builder; every run executes the prompts in order, with each response feeding the next.

These are built to Microsoft's promptbook conventions ([using promptbooks](https://learn.microsoft.com/en-us/copilot/security/using-promptbooks) · [build promptbooks](https://learn.microsoft.com/en-us/copilot/security/build-promptbooks)) and tuned for **SCU-cost awareness** — fewer, denser prompts rather than long chains, because every promptbook run draws down the same monthly Security Compute Unit (SCU) pool that the Monitor agent is designed to govern.

---

## Contents

- [How to build a promptbook from these files](#how-to-build-a-promptbook-from-these-files)
- [Design principles](#design-principles)
- [Security Copilot Monitor](#security-copilot-monitor)
  - [Weekly SCU Burn-Rate & Throttle-Risk Review](#1--weekly-scu-burn-rate--throttle-risk-review)
  - [Copilot Adoption & Governance Anomaly Review](#2--copilot-adoption--governance-anomaly-review)
  - [Monthly SCU Cost & Chargeback Report](#3--monthly-scu-cost--chargeback-report)
- [Data Security Monitor](#data-security-monitor)
  - [Monthly DLP Posture Review](#4--monthly-dlp-posture-review)
  - [Risky User Data-Exfiltration Investigation](#5--risky-user-data-exfiltration-investigation)
  - [Sensitivity-Label & Oversharing Posture Review](#6--sensitivity-label--oversharing-posture-review)
- [Notes & constraints](#notes--constraints)
- [References](#references)

---

## How to build a promptbook from these files

In Security Copilot, open the promptbook library → **Create promptbook**, then map each file below to the builder fields:

| Builder field | What to enter |
|---|---|
| **Name** | The promptbook heading below |
| **Description** | The *Description* line |
| **Prompts** | The numbered *Prompt sequence*, pasted **in order** |
| **Inputs you'll need** | Auto-detected from the `<AngleBracket>` tokens — verify they match the *Inputs* list |
| **Sharing** | The *Sharing* value (updatable later) |

**Input convention:** parameters use angle brackets with no spaces (e.g. `<SubscriptionId>`), matching Microsoft's built-in tokens. The runner is prompted for each value at run time.

**Ordering matters:** these run sequentially and each prompt depends on the previous output. Reordering or editing a prompt changes results — and SCU usage. Enable **Continue on failure** on any non-critical prompt if you want a chain to survive one failed step.

---

## Design principles

- **Short chains.** Each promptbook is 4–6 prompts. Beyond ~7 chained analytical steps the model reliably drops steps and mis-deduplicates, so distinct scenarios (cost vs. adoption vs. investigation) are split into separate promptbooks rather than one monolith. This also isolates SCU cost per scenario.
- **No client-side arithmetic.** Percentages and ratios (SCU burn %, compliance %, chargeback splits) are pulled from the plugin/API response, not calculated by the model, which is unreliable at math.
- **Trigger-word hygiene.** Closing prompts say *"generate"* rather than *"export"* to avoid hijacking Copilot into a built-in export skill.
- **Format-explicit.** Prompts specify tables, severity ordering, and audience so output is consistent and paste-ready into reports.

---

## Security Copilot Monitor

The Monitor agent calls **Azure Cost Management** live at runtime, splits **provisioned vs. overage** SCU consumption, reports usage in **both currency and SCU units**, and fuses that with **CopilotActivity** Sentinel telemetry for adoption, governance, and anomaly signals — all inside Security Copilot. It authenticates with the caller's Azure RBAC (AAD-delegated), so results are scoped to what the runner can see.

Why SCU *units* matter more than currency for E5/E7 inclusion tenants: inclusion capacity is a fixed monthly pool (400 SCU per 1,000 licensed users, up to 10,000), resets monthly with **no rollover**, and today **throttles** rather than bills on exhaustion. The governance signal is *how much of the pool is left*, not dollars.

### 1 — [Weekly SCU Burn-Rate & Throttle-Risk Review](https://github.com/the-sentinental-guy/YashureSecurity---Security-Copilot/blob/main/Promptbooks/Weekly%20SCU%20Burn-Rate%20%26%20Throttle-Risk%20Review)

**Scenario:** Weekly SCU governance stand-up — is the tenant on track to exhaust the monthly inclusion pool before month-end?

**Description:** Pulls live provisioned vs. overage SCU consumption and projects month-end burn against the inclusion allocation, flagging throttle risk. For Copilot owners / FinOps / SecOps leads. Run every Monday.

**Plugins / data sources:** Security Copilot Monitor agent · Azure Cost Management API (via the plugin) · Security Copilot usage telemetry.

**Inputs:** `<SubscriptionId>`, `<Timeframe>` (e.g. "last 7 days"), `<InclusionAllocationSCU>` (monthly pool, e.g. 400)

**Sharing:** Anyone in my organization

---

### 2 — [Copilot Adoption & Governance Anomaly Review](https://github.com/the-sentinental-guy/YashureSecurity---Security-Copilot/blob/main/Promptbooks/Copilot%20Adoption%20%26%20Governance%20Anomaly%20Review)

**Scenario:** Weekly / bi-weekly adoption and AI-governance review from CopilotActivity telemetry.

**Description:** Fuses CopilotActivity Sentinel telemetry with the Monitor agent to report adoption (interactive vs. automated usage, top skills/agents) and surface governance anomalies (plugin/promptbook lifecycle changes, jailbreak signals, anomalous admin actions). For SecOps governance owners.

**Plugins / data sources:** Security Copilot Monitor agent · Microsoft Sentinel plugin (CopilotActivity via KQL / Natural language to KQL) · default Sentinel workspace configured.

**Inputs:** `<WorkspaceName>`, `<Timeframe>` (e.g. "last 14 days")

**Sharing:** Anyone in my organization

---

### 3 — [Monthly SCU Cost & Chargeback Report](https://github.com/the-sentinental-guy/YashureSecurity---Security-Copilot/blob/main/Promptbooks/Monthly%20SCU%20Cost%20%26%20Chargeback%20Report)

**Scenario:** Month-end SCU cost report and per-workspace/team chargeback.

**Description:** Produces a month-end reconciliation of Security Copilot SCU consumption in both SCU units and currency, split provisioned vs. overage, allocated per capacity/workspace for chargeback. For FinOps + Copilot owner.

**Plugins / data sources:** Security Copilot Monitor agent · Azure Cost Management API.

**Inputs:** `<SubscriptionId>`, `<BillingMonth>` (e.g. "2026-06"), `<CapacityResourceGroup>`

**Sharing:** Anyone in my organization

---

## Data Security Monitor

The Data Security Monitor is the commercially available Inspira Enterprise agent on the [Microsoft Security Store](https://securitystore.microsoft.com/solutions/inspiraenterpriseinc1683208138220.data_security_monitor). These promptbooks orchestrate Microsoft Purview data-security signals — DLP, Insider Risk Management, sensitivity labels / Information Protection, and DSPM — into repeatable posture reviews and investigations.

### 4 — [Monthly DLP Posture Review](https://github.com/the-sentinental-guy/YashureSecurity---Security-Copilot/blob/main/Promptbooks/Monthly%20DLP%20Posture%20Review)

**Scenario:** Monthly DLP posture review across Microsoft 365 and endpoints.

**Description:** Summarizes top DLP alerts, policy hotspots, and repeat offenders over the month and recommends policy tuning. For data security admins / Purview DLP owners.

**Plugins / data sources:** Data Security Monitor agent · Microsoft Purview plugin (DLP).

**Inputs:** `<Timeframe>` (e.g. "last 30 days")

**Sharing:** Anyone in my organization

---

### 5 — [Risky User Data-Exfiltration Investigation](https://github.com/the-sentinental-guy/YashureSecurity---Security-Copilot/blob/main/Promptbooks/Risky%20User%20Data-Exfiltration%20Investigation)

**Scenario:** Triage of a flagged insider-risk user (mirrors Microsoft's built-in six-prompt DSPM "Risky user investigation" pattern).

**Description:** Walks a single user's insider-risk profile, data activity, exfiltration behaviors, and related alerts, ending with a triage verdict and mitigation actions. For IRM analysts / investigators.

**Plugins / data sources:** Data Security Monitor agent · Microsoft Purview plugin (Insider Risk Management + DLP). Requires an IRM Investigator/Analyst role.

**Inputs:** `<UserUPN>`, `<Timeframe>` (e.g. "last 30 days")

**Sharing:** Just me *(IRM data is sensitive — scope narrowly, then widen only if policy allows)*

---

### 6 — [Sensitivity-Label & Oversharing Posture Review](https://github.com/the-sentinental-guy/YashureSecurity---Security-Copilot/blob/main/Promptbooks/Sensitivity-Label%20%26%20Oversharing%20Posture%20Review)

**Scenario:** Periodic review of sensitive-data protection coverage and oversharing exposure (aligns with Microsoft's DSPM "Sensitive data protection" pattern).

**Description:** Assesses protection coverage for a given sensitivity label or SIT, surfaces unprotected/overshared sensitive data, and recommends label/DLP remediations. For data security posture owners.

**Plugins / data sources:** Data Security Monitor agent · Microsoft Purview plugin (DSPM, Information Protection, DLP).

**Inputs:** `<SensitivityLabelOrGUID>`, `<Timeframe>` (e.g. "last 30 days")

**Sharing:** Anyone in my organization

---

## Notes & constraints

- **Sensitivity labels are stored as GUIDs.** Microsoft audit / `AccessedResources` telemetry references labels by `SensitivityLabelId` GUID, not display name. Label-based prompts accept a GUID and resolve to the display name where possible — supply the GUID if the name doesn't match.
- **Results are RBAC-scoped.** Both agents run with the caller's permissions. A runner who lacks a required role (e.g. IRM Investigator, Cost Management Reader) will get partial or empty results, not an error — validate role coverage before sharing a promptbook org-wide.
- **Promptbook runs consume SCUs.** The promptbooks here are themselves subject to the inclusion pool they help govern. After rollout, use *Copilot Adoption & Governance Anomaly Review* to confirm promptbook/agent SCU draw stays within allocation, and cap overage on non-critical workspaces.
- **Output varies.** Security Copilot responses vary run-to-run by design and depend on enabled plugins and runner permissions. Re-test each promptbook in a fresh session after any edit.

---

## References

- [Using promptbooks in Microsoft Security Copilot](https://learn.microsoft.com/en-us/copilot/security/using-promptbooks)
- [Build your own promptbooks](https://learn.microsoft.com/en-us/copilot/security/build-promptbooks)
- [Create effective prompts](https://learn.microsoft.com/en-us/copilot/security/prompting-tips)
- [Data Security Monitor — Microsoft Security Store](https://securitystore.microsoft.com/solutions/inspiraenterpriseinc1683208138220.data_security_monitor)

---

*Maintained by [@the-sentinental-guy](https://github.com/the-sentinental-guy) · YashureSecurity*
