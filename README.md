<h1 align="center">🛡️ AI-Powered SOC Automation Pipeline</h1>
<h3 align="center">Wazuh &nbsp;+&nbsp; n8n &nbsp;+&nbsp; Groq AI &nbsp;+&nbsp; VirusTotal &nbsp;+&nbsp; AbuseIPDB</h3>
 
<p align="center">
  <img src="https://img.shields.io/badge/SOC-Automation%20Pipeline-ff0040?style=for-the-badge&logo=shield&logoColor=white" />
  <img src="https://img.shields.io/badge/n8n-Automation-ea4b71?style=for-the-badge&logo=n8n&logoColor=white" />
  <img src="https://img.shields.io/badge/Wazuh-SIEM%20%2F%20XDR-006eff?style=for-the-badge&logo=linux&logoColor=white" />
  <img src="https://img.shields.io/badge/Groq-LLaMA%203.1-f55036?style=for-the-badge&logo=firefoxbrowser&logoColor=white" />
  <img src="https://img.shields.io/badge/License-MIT-3fb950?style=for-the-badge" />
</p>
<p align="center">
  <b>A real-world SOAR pipeline that detects, enriches, triages, and alerts on security threats — fully automated using AI.</b>
</p>
---
 
## 📖 Table of Contents
 
- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [API Keys Setup](#-api-keys-setup)
- [Infrastructure Setup](#-infrastructure-setup)
- [Agent Configuration](#-agent-configuration)
- [Workflow Demonstration](#-workflow-demonstration)
- [Attack Simulation](#-testing--attack-simulation)
- [Alert Output](#-alert-output-examples)
- [Quick Start](#-quick-start)
- [Skills Gained](#-skills-gained)
- [Future Enhancements](#-future-enhancements)
- [Author](#-author)
---
 
## 🧠 Overview
 
In modern Security Operations Centers (SOCs), **automation and AI triage** are critical for handling large volumes of alerts efficiently. This project is a cloud-hosted, AI-powered threat detection and response system that simulates a real-world **SOAR (Security Orchestration, Automation, and Response)** workflow.
 
The pipeline automatically:
 
| Step | Action |
|------|--------|
| 📡 **Receive** | Accepts security alerts from Wazuh via webhook |
| 🔀 **Route** | Separates IP brute-force alerts from File Integrity (FIM) alerts |
| 🌐 **Enrich** | Queries AbuseIPDB, URLScan.io, and VirusTotal for threat context |
| 🤖 **Triage** | Groq AI (LLaMA 3.1) produces a verdict, confidence score, and recommended action |
| 📧 **Alert** | Sends a rich dark-themed HTML email to the SOC analyst |
| 📊 **Log** | Appends every alert to Google Sheets for audit and trend analysis |
 
---
 
## 🏗️ Architecture
 
```
Wazuh Agent  (Windows 11 / Linux)
      |
      v
Wazuh Manager  (DigitalOcean Droplet)
      |
      |  ossec.conf  →  POST every alert as JSON
      v
 n8n Webhook
      |
      v
 Normalize Payload  (Code Node — handles flat + nested JSON)
      |
      v
 Route by Alert Type  (Switch Node)
      |
      |------------------------------------------|
      |                                          |
      | groups contains                          | groups contains
      | "authentication_failed"                  | "syscheck"
      v                                          v
 [BRANCH 1 — IP Enrichment]            [BRANCH 2 — FIM / File Hash]
      |                                          |
      v                                          v
 AbuseIPDB API                           VirusTotal API
 (abuse score, ISP, country)             (malware detections)
      |                                          |
      v                                          v
 URLScan.io API                          Groq LLaMA 3.1
 (associated domains)                    (AI triage)
      |                                          |
      v                                          v
 Groq LLaMA 3.1                          Parse AI Response
 (AI triage)                                     |
      |                                          v
      v                                   Check Verdict
 Parse AI Response                       /             \
      |                             TRUE               FALSE
      v                              |                  |
 Check Verdict                  Email Alert +       Log to Sheets
 /           \                  Log to Sheets       (FIM_Alerts)
TRUE         FALSE
 |             |
Email       Log to
Alert +     Sheets
Sheets      (Sheet1)
      |                                          |
      |------------------------------------------|
                        |
                        v
               Respond to Webhook
               { "status": "received" }
```
 
---
 
## 🧰 Tech Stack
 
| Component | Technology | Purpose |
|-----------|-----------|---------|
| **SIEM / XDR** | Wazuh | Alert generation, FIM monitoring, log collection |
| **Automation** | n8n (Cloud) | SOAR workflow orchestration — 17-node pipeline |
| **AI Triage** | Groq API — LLaMA 3.1 8B | Verdict, confidence score, recommended action |
| **IP Intelligence** | AbuseIPDB | Abuse score, ISP, country, report history |
| **IP Scanning** | URLScan.io | Domains and scan history linked to source IP |
| **File Intelligence** | VirusTotal | Malware detection rate across 70+ AV engines |
| **Alerting** | Gmail (OAuth2) | Dark-themed HTML email alerts to SOC analyst |
| **Logging** | Google Sheets | Persistent audit log — two tabs (IP + FIM) |
| **Infrastructure** | DigitalOcean Droplet | Cloud-hosted Wazuh Manager (Ubuntu 22.04) |
 
---
 
## 🔑 API Keys Setup
 
> Collect all five API keys before deploying. All services offer a free tier.
 
### 1. Groq API — Select AI Model
 
Used for AI-powered triage in both branches. Model used: `llama-3.1-8b-instant` — fast, free, and accurate for SOC verdicts.
 
<img src="Images/1(Selected Model from Grok API Key).png" width="700"/>
---
 
### 2. VirusTotal API
 
Used in Branch 2 (FIM) to check file hashes against 70+ antivirus engines. Returns a detection ratio like `45/72`.
 
<img src="Images/2(Virus Total API Key).png" width="700"/>
---
 
### 3. Shodan API *(optional — future enhancement)*
 
Will be used for port/service fingerprinting of attacking IPs — shows open ports, banners, and CVEs.
 
<img src="Images/3 (SHODAN API Key).png" width="700"/>
---
 
### 4. URLScan.io API
 
Used in Branch 1 to find domains and scan records historically associated with the source IP address.
 
<img src="Images/4 (URLScan.io API Key).png" width="700"/>
---
 
### 5. AbuseIPDB API
 
Core threat intelligence for Branch 1. Returns abuse confidence score (0–100), country, ISP, usage type, and total abuse reports in the last 90 days.
 
<img src="Images/5 (ABUSEIPDB API Key).png" width="700"/>
---
 
## 🖥️ Infrastructure Setup
 
### 6. Wazuh Manager — System Updates
 
Before installing Wazuh, update and upgrade all system packages on the DigitalOcean Droplet to avoid dependency issues.
 
```bash
sudo apt-get update && sudo apt-get upgrade -y
```
 
<img src="Images/6 Wazuh Manager ( Updates and Upgrades Before Installing Wazuh).png" width="700"/>
---
 
### 7. Installing Wazuh Manager
 
Install the Wazuh all-in-one package. This sets up the Wazuh Manager, Indexer, and Dashboard in a single command.
 
```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```
 
<img src="Images/7 Installing Wazuh .png" width="700"/>
---
 
### 8. Wazuh IP Configuration
 
Note the Wazuh Manager's public IP address — this is required when enrolling agents and configuring the n8n webhook integration.
 
<img src="Images/8 (Wazuh IP Configure ).png" width="700"/>
---
 
### 9. Wazuh Successfully Installed
 
Wazuh Manager installation complete. The dashboard, indexer, and manager services are all running on the cloud Droplet.
 
<img src="Images/9 ( Wazuh Installed).png" width="700"/>
---
 
### 10. Wazuh Dashboard
 
Access the Wazuh web dashboard at `https://YOUR_DROPLET_IP` to verify deployment, view security events, and confirm agents are connected.
 
<img src="Images/10 ( Wazuh Dashboard ).png" width="700"/>
---
 
### 11. n8n Hosted on Cloud
 
n8n is running on the cloud as the automation brain. All 17 workflow nodes — routing, enrichment, AI triage, email, and logging — are orchestrated here.
 
<img src="Images/11 (n8n hosted in Cloud Droplet).png" width="700"/>
---
 
## 🖥️ Agent Configuration
 
### 12. Wazuh Agent — Windows 11
 
Install and configure the Wazuh agent on the Windows 11 machine. Once enrolled, the agent streams security events (logins, FIM changes, process events) to the Manager.
 
```powershell
# Run as Administrator
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.0-1.msi -OutFile wazuh-agent.msi
msiexec.exe /i wazuh-agent.msi /q WAZUH_MANAGER="YOUR_DROPLET_IP" WAZUH_AGENT_NAME="windows-11" WAZUH_REGISTRATION_SERVER="YOUR_DROPLET_IP"
NET START WazuhSvc
```
 
<img src="Images/12 ( Wazuh Agent COnfigured and installed in Windows.png" width="700"/>
---
 
### 13. Wazuh Dashboard — Agents Connected
 
The Wazuh dashboard confirms that the `windows-11` agent (ID: 001) is active and reporting events in real time.
 
<img src="Images/13 ( Wazuh Dashboard showing the Agents).png" width="700"/>
---
 
### 14. FIM Working in Wazuh Dashboard
 
File Integrity Monitoring (FIM) is active on the Windows agent. Any file addition, modification, or deletion in monitored directories triggers a syscheck alert visible here.
 
<img src="Images/14 (FIM IS WORKING AND SHOWN IN WAZUH DASHBOARD).png" width="700"/>
---
 
### 15. Wazuh Manager → n8n Webhook Integration
 
Configure `ossec.conf` to forward every Wazuh alert above level 5 as a JSON POST to the n8n webhook. This is the bridge between Wazuh detection and the automation pipeline.
 
```xml
<!-- /var/ossec/etc/ossec.conf -->
<integration>
  <name>custom-n8n</name>
  <hook_url>https://YOUR-N8N-URL/webhook/wazuh-alerts</hook_url>
  <level>5</level>
  <alert_format>json</alert_format>
</integration>
```
 
```bash
sudo systemctl restart wazuh-manager
```
 
<img src="Images/15 (Droplet will now POST every alert to your n8n webhook automatically).png" width="700"/>
---
 
### 16. Droplet Configured Successfully
 
The integration is live. Every Wazuh alert above severity level 5 is now automatically forwarded to the n8n webhook for processing.
 
<img src="Images/16 (Droplet Configured Successfully).png" width="700"/>
---
 
## 📊 Workflow Demonstration
 
### 17. Full n8n Workflow
 
The complete 17-node pipeline in the n8n canvas. Left side handles IP-based brute force alerts (Branch 1). Right side handles File Integrity Monitoring alerts (Branch 2). Both branches converge at a single `Respond to Webhook` node.
 
<img src="Images/17 ( n8n Workflow).png" width="700"/>
---
 
## 🧪 Testing & Attack Simulation
 
### 18. Kali Linux — Brute Force Attack
 
Hydra performs an SSH brute force attack against the Windows 11 target. Each failed attempt is logged by the Wazuh agent, triggering `authentication_failed` alerts that feed into Branch 1 of the pipeline.
 
```bash
# Run from Kali Linux
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt -t 4 -vV ssh://VICTIM_IP
```
 
<img src="Images/18 (kali Attacker Password Found).png" width="700"/>
---
 
### 19. Authentication Failure Detected in Wazuh
 
Wazuh captures the failed login attempts and fires Rule 5760. The alert is forwarded to n8n, routed to Branch 1, and enriched via AbuseIPDB and URLScan.io before the AI triage.
 
<img src="Images/19 Authentication Failure .png" width="700"/>
---
 
### 20. FIM — Malicious File Detected
 
A suspicious file dropped on the Windows agent is detected by Wazuh FIM (syscheck). The SHA256 hash is extracted and sent to VirusTotal. The EICAR test hash used below returns **65+ detections** across antivirus engines.
 
<img src="Images/20 Malicious File Creation Detected FIM.png" width="700"/>
---
 
## 📧 Alert Output Examples
 
### 21. IP Threat Alert Email — Inbox View
 
The analyst receives a fully formatted dark-themed HTML email. The subject line includes the AI verdict and rule description. The email contains three sections: AI Triage Analysis, Threat Intelligence, and Wazuh Event Details.
 
<img src="Images/21 Email Recived for IP .png" width="700"/>
---
 
### 22. IP Alert — Threat Intelligence Section
 
This section shows the enriched data for the source IP: AbuseIPDB confidence score with a visual bar, country of origin, ISP, usage type, total abuse reports, URLScan hits, and associated domains linked to the IP.
 
<img src="Images/22 IP report 01.png" width="700"/>
---
 
### 23. IP Alert — Event Details & Raw Log
 
The bottom section of the IP alert email shows the Wazuh event details: Rule ID, rule description, rule groups, agent name and ID, manager name, target username, event timestamp, and the full raw log line from Wazuh.
 
<img src="Images/23 IP report 02.png" width="700"/>
---
 
### 24. FIM Alert Email — Inbox View
 
For file integrity violations, a separate purple-accented dark HTML email is generated. It includes the AI verdict, VirusTotal detection rate with a colour-coded bar, file details section, and a direct link to the full VirusTotal report.
 
<img src="Images/24 FIM Alert Email Recived .png" width="700"/>
---
 
### 25. FIM Alert — File Details Section
 
Shows the monitored file path, event type badge (colour-coded: green for added, blue for modified, red for deleted), SHA256 hash before and after the change, and the VirusTotal detection ratio with a visual progress bar.
 
<img src="Images/25 FIM Alert 01 .png" width="700"/>
---
 
### 26. FIM Alert — Event Details & Raw Log
 
The Wazuh event details section for FIM alerts: Rule ID, rule description, rule groups, agent name/ID, manager, event timestamp, and the raw log entry captured by the Wazuh agent.
 
<img src="Images/26 FIM Alert 02 .png" width="700"/>
---
 
## 🚀 Quick Start
 
### Prerequisites
 
- DigitalOcean Droplet — Ubuntu 22.04, 4GB RAM minimum
- n8n Cloud account or self-hosted n8n instance
- Gmail account with OAuth2 credentials configured in n8n
- Google Sheets with two tabs: `Sheet1` (IP alerts) and `FIM_Alerts`
- All five API keys: Groq, AbuseIPDB, URLScan.io, VirusTotal, Shodan (optional)

---
 
## 📊 Alert Fields Reference
 
### Branch 1 — IP Threat Email
 
| Field | Description |
|-------|-------------|
| AI Verdict | `CRITICAL` / `HIGH` / `LOW` / `FALSE_POSITIVE` |
| AI Confidence | 0–100% confidence from LLaMA 3.1 |
| AI Summary | 20-word automated threat assessment |
| Recommended Action | AI-generated SOC response action |
| Source IP | Attacker's IP address |
| AbuseIPDB Score | Abuse confidence score out of 100 |
| Country | IP origin country |
| ISP | Internet Service Provider name |
| Usage Type | Hosting / Residential / Data Center |
| Total Reports | Abuse reports submitted in last 90 days |
| Last Seen Abusing | Timestamp of the most recent abuse report |
| URLScan Hits | Number of scan records found |
| Associated Domains | Up to 5 domains historically linked to the IP |
| Rule ID / Level | Wazuh rule number and severity level |
| Agent Name / ID | Reporting Windows/Linux endpoint |
| Full Log | Complete raw log line from Wazuh |
 
### Branch 2 — FIM Alert Email
 
| Field | Description |
|-------|-------------|
| AI Verdict | `CRITICAL` / `HIGH` / `LOW` / `FALSE_POSITIVE` |
| AI Confidence | 0–100% confidence from LLaMA 3.1 |
| VT Detections | Number of AV engines that flagged the hash |
| File Path | Full path of the changed file on the endpoint |
| Event Type | `added` / `modified` / `deleted` (colour-coded badge) |
| SHA256 After | Hash of the file after the change |
| SHA256 Before | Original hash before the change |
| VirusTotal Link | Clickable link to the full VirusTotal report |
| IOC Classification | `FILE_HASH` |
 
---
 
## 🧠 Skills Gained
 
| Category | Skills |
|----------|--------|
| **Security Automation** | SOAR workflow design, n8n orchestration |
| **Threat Intelligence** | AbuseIPDB, URLScan.io, VirusTotal API integration |
| **AI in Cybersecurity** | LLM-based alert triage using Groq / LLaMA 3.1 |
| **SIEM / XDR** | Wazuh deployment, agent configuration, FIM, rule tuning |
| **Cloud Infrastructure** | DigitalOcean Droplet provisioning and management |
| **Incident Response** | Automated alerting, false positive filtering logic |
| **Integration Engineering** | REST APIs, OAuth2, webhooks, JSON payload normalisation |
| **Attack Simulation** | Hydra SSH brute force, FIM payload crafting on Windows |
 
---
 
## 🚀 Future Enhancements
 
- [ ] **Shodan Integration** — port and service fingerprinting of attacking IPs
- [ ] **Slack / Teams Alerting** — real-time push notifications to SOC channels
- [ ] **Auto IP Block** — firewall rule creation via API on CRITICAL verdicts
- [ ] **Elasticsearch / Kibana** — long-term threat analytics and dashboarding
- [ ] **MITRE ATT&CK Mapping** — auto-tag each alert with ATT&CK technique IDs
- [ ] **Alert Deduplication** — suppress repeated alerts from the same source IP
- [ ] **URL & Domain Reputation** — extend pipeline to detect phishing links
---
 
## 🏁 Conclusion
 
This project demonstrates how **AI and automation can dramatically enhance SOC efficiency** — reducing manual triage time from minutes to seconds. By combining Wazuh's detection capabilities with n8n's orchestration and Groq's AI reasoning, it replicates the kind of **security automation pipeline used in enterprise environments**.
 
---
 
## 👨‍💻 Author
 
<p align="center">
  <b>Umar Ahmed</b><br/>
  Cybersecurity Student &nbsp;·&nbsp; Security Automation &amp; SOAR Enthusiast<br/><br/>
  <a href="https://github.com/User-Umar-Ahamed">
    <img src="https://img.shields.io/badge/GitHub-User--Umar--Ahamed-181717?style=for-the-badge&logo=github" />
  </a>
</p>

