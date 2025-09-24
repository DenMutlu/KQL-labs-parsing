
# 📜 Log Management & Audit Policy – Cheat Sheet

A quick reference for building a **log management policy** that balances security, cost, and operational efficiency.

---

## 🔑 Core Principles
- **Clarity before collection** → define *what* to collect, *why*, and *who owns it*.
- **Quality over quantity** → bad or noisy data = bad detections & wasted cost.
- **Two-tier strategy** → *hot* (detection) vs *warm/cold* (compliance, archive).
- **Normalization first** → consistent schemas = better KQL, easier hunting.
- **Governance matters** → policy, RACI, lifecycle.

---

## 🧑‍🤝‍🧑 Roles & Responsibilities (RACI)
- **Responsible:** SOC engineers / log owners (configure, forward logs)
- **Accountable:** CISO / Security leadership (approve log policy, retention)
- **Consulted:** IT Ops, Compliance, App owners (requirements, exceptions)
- **Informed:** Auditors, Management (reports, assurance)

---

## 🏗️ Reference Architecture
- **Source** → Cloud (Azure, M365, SaaS), On-prem (FW, EDR, Syslog), Custom apps
- **Transport** → Native connector, Event Hub, AMA/agent, API push (Logs Ingestion)
- **Ingestion** → Sentinel (Analytics/Basic/Aux logs), Data Lake, Archive (S3/Blob)
- **Storage tiers:**
  - **Hot (0–90d):** high-fidelity, detection-ready logs
  - **Warm (3–12m):** less frequently queried, compressed
  - **Cold (1y+):** regulatory/archive, query-on-demand

---

## 📝 Policy Checklist
- [ ] Define **scope**: which systems, apps, services must send logs  
- [ ] Define **log categories**: Security, Compliance, Operational, Audit  
- [ ] Define **ownership**: who is responsible per log source (RACI)  
- [ ] Define **quality standards**: time sync (NTP), UTF-8, JSON, include User/IP/Action  
- [ ] Define **normalization rules**: field naming (TimeGenerated, DeviceId, EventType, etc.)  
- [ ] Define **retention** per category (90d, 180d, 1y, 7y)  
- [ ] Define **storage tiers**: Hot vs Warm vs Cold  
- [ ] Define **cost control measures**: filtering, splitting tables, compression  
- [ ] Define **alerting coverage**: map MITRE ATT&CK / detection gaps  
- [ ] Define **audit requirements**: FINMA, CSSF, SOX, PCI, ISO 27001, NIS2, etc.  
- [ ] Define **review cadence**: log policy reviewed quarterly/annually (assign owner)  

---

## 💡 Best Practices
- Enable **time sync (NTP)** everywhere → prevents correlation errors.  
- Always map logs to **TimeGenerated** in Sentinel.  
- Use **splitting techniques** (analytics vs basic vs auxiliary logs).  
- Ingest only **high-value fields** (drop noise).  
- Build **sample dashboards early** to validate log usefulness.  
- Automate ingestion via **DCR/Logs Ingestion API**.  
- Use **Content Hub** for built-in normalization + detection packs.  
- Document **exceptions** (why some logs are not collected).  
- Regularly **audit data quality** (missing fields, malformed records).  
- Monitor **ingestion cost trends** and adjust.  
- Archive to **Sentinel Data Lake** for long-term compliance at low cost.  

---

## ❓ Key Questions
- Why do we collect this log? Detection, compliance, troubleshooting?  
- Who owns this log source? Who is accountable if it breaks?  
- What is the retention requirement? Legal vs operational?  
- How often will we query it? Daily vs rarely?  
- Does this log add **signal or noise** to detections?  
- Can we **normalize or filter** before ingest?  
- How much will this source cost per month/year?  

---

## 🚀 Quick Win Strategy
1. Start with **M365 + Defender + Entra** → highest security value.  
2. Add **firewall / perimeter logs** → critical visibility.  
3. Bring in **custom / app logs** selectively (high-value use cases).  
4. Apply **Basic/Auxiliary** for noisy sources.  
5. Archive rarely-used logs to **Data Lake**.  
6. Review + tune monthly.

---

## 📎 References
- [Microsoft Docs – Log Management best practices](https://learn.microsoft.com/azure/azure-monitor/best-practices-data-collection)  
- [Microsoft Sentinel Log Plans](https://learn.microsoft.com/azure/sentinel/log-plans)  
- [Microsoft Sentinel Data Lake](https://learn.microsoft.com/azure/sentinel/datalake/sentinel-lake-overview)  
- [MSTIC Blog – Cost saving techniques](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/save-ingestion-costs-by-splitting-logs-into-multiple-tables-and/ba-p/4230861)

---
**👉 Most important rule:** *DATA born in the Cloud, stay in the Cloud.*  
**👉 Rule of Thumb:** *If it isn’t collected, it can’t be detected. If it’s collected badly, it can’t be trusted.*
