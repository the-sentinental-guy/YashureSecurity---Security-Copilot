<div align="center">

# 🛡️ YashureSecurity — Security Copilot Artifacts

**Custom plugins, KQL skills, and generative tools for Microsoft Security Copilot**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/the-sentinental-guy/YashureSecurity---Security-Copilot?style=flat&color=gold)](https://github.com/the-sentinental-guy/YashureSecurity---Security-Copilot/stargazers)
[![Medium](https://img.shields.io/badge/Blog-@yashuresecurity-black?logo=medium&logoColor=white)](https://medium.com/@yashuresecurity)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Yash-0077B5?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/yashmudaliar/)
[![Microsoft Security Copilot](https://img.shields.io/badge/Microsoft-Security%20Copilot-0078D4?logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/security-copilot/)

</div>

---

## 📖 About

This repository is a practitioner-built library of **custom Security Copilot artifacts** — plugins, KQL skills, and generative utilities — designed to extend what Microsoft Security Copilot can do out of the box.

Every artifact here comes from real SOC work. No filler, no toy demos. These are tools built to solve production problems: detection gaps, SCU visibility, incident documentation, and architecture communication.

> Built by **[@yashuresecurity](https://medium.com/@yashuresecurity)** — Microsoft Security practitioner, content creator, and ex-Microsoft Security Consultant.

---

## 📦 Artifacts

### 🔍 KQL Detection Rules for Security Copilot
> **YAML-based KQL plugin manifests** that bring detection logic directly into Security Copilot.

Load these skills into your Security Copilot environment to query Microsoft Sentinel and Defender XDR from natural-language prompts. Rules are structured for clarity and production use — not just as references.

**What's inside:**
- Custom KQL skills packaged as Security Copilot plugin manifests (`.yml`)
- Detection coverage for Sentinel Analytics, Defender XDR Advanced Hunting, and custom log tables
- Designed for SOC analysts who want AI-assisted triage without leaving the Copilot surface

---

### 📊 HTML Dashboard Generator
> **A Security Copilot skill** that outputs formatted HTML dashboards from security data.

Takes structured input — incident summaries, alert counts, KQL results — and renders a clean, shareable HTML report. Built for shift handovers, executive briefings, or ad-hoc posture snapshots without leaving the Copilot workflow.

**What's inside:**
- Plugin manifest for invoking the dashboard generation skill
- HTML template definitions tuned for security data presentation
- Sample prompts and invocation patterns

---

### 🌊 Mermaid Diagram Generator
> **Turn incident timelines and architecture descriptions into Mermaid diagrams inside Security Copilot.**

Describe a threat scenario, a log ingestion pipeline, or a network topology — and get a structured Mermaid diagram back. Useful for incident documentation, SOC runbooks, and architecture reviews.

**What's inside:**
- Security Copilot plugin manifest for Mermaid generation
- Prompt templates for common diagram types (flowchart, sequence, architecture)
- Example outputs for reference

---

### 📡 Security Copilot Monitor
> **Track and audit Security Copilot activity** — SCU consumption, prompt patterns, and usage telemetry.

Visibility into how Security Copilot is actually being used in your environment. Helps SOC leads and security architects understand utilization, spot inefficiencies, and demonstrate value against SCU spend.

**What's inside:**
- KQL queries targeting Security Copilot audit logs
- Plugin skill for invoking usage summaries from within Copilot itself
- SCU consumption analysis patterns

---

## 🚀 Getting Started

### Prerequisites
- An active Microsoft Security Copilot environment (SCU-based or pay-as-you-go)
- Access to upload custom plugins (Owner or Contributor role in Security Copilot)
- Microsoft Sentinel workspace (required for KQL-based skills)

### Installing a Plugin

1. Navigate to the artifact folder of your choice
2. Copy the plugin manifest (`.yml` file)
3. In Security Copilot, go to **Sources → Custom plugins → Upload plugin**
4. Upload the manifest and configure any required workspace parameters (Tenant ID, Workspace Name, Subscription ID, Resource Group)
5. Enable the plugin and invoke skills via the Copilot prompt bar

> 💡 Refer to the `README.md` inside each artifact folder for folder-specific parameters and sample prompts.

---

## 🤝 Contributing

Contributions are welcome — especially from practitioners who've hit the same walls.

If you have a Security Copilot plugin, KQL skill, or Promptbook pattern worth sharing:

1. Fork the repo
2. Create a feature branch: `git checkout -b feature/your-artifact-name`
3. Add your artifact in a clearly named folder with its own `README.md`
4. Submit a pull request with a brief description of the use case

Please follow the existing folder structure and include at minimum: a plugin manifest, a sample prompt, and a short description of what problem it solves.

---

## 📚 Related Writing

These artifacts are companion pieces to technical content published on Medium and LinkedIn.

| Article | Topic |
|---|---|
| [The Security Analyst Agent](https://medium.com/@yashuresecurity) | Security Copilot's autonomous triage agent deep-dive |
| [Security Analyst Agent + Microsoft Purview DSI](https://medium.com/@yashuresecurity) | SOC modernization with DLP-aware AI |
| [Custom KQL Plugins for Security Copilot](https://medium.com/@yashuresecurity) | Building and deploying KQL-backed skills |

> Follow **[@yashuresecurity](https://medium.com/@yashuresecurity)** on Medium for new posts on Sentinel, Defender XDR, Security Copilot, and Microsoft Purview.

---

## ⚠️ Disclaimer

These artifacts are provided as-is for educational and practitioner reference. Always validate plugin behavior in a non-production environment before deploying. Security Copilot plugin invocations consume SCUs — review your usage budget before bulk deployments.

---

## 📄 License

This repository is licensed under the **MIT License**. See [LICENSE](LICENSE) for details.

---

<div align="center">

**Built with intent. Deployed with caution. Shared with the community.**

*[@yashuresecurity](https://medium.com/@yashuresecurity) · Microsoft Security · Security Copilot · Purview · Sentinel · Defender XDR *

</div>
