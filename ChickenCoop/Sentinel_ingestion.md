
# 🐔 Chicken Coop → Microsoft Sentinel Demo

This project shows how **any telemetry** (even from a chicken coop) can be ingested into **Microsoft Sentinel**,
normalized with KQL functions, enriched with **analytics rules**, and visualized in **workbooks**.

If we can SOC a chicken coop, we can SOC anything.

---

## 📖 Story

- **Scenario:** My chicken coop has sensors:  
  - Automatic door (opens at sunrise, closes at sunset)  
  - Water heater + temp sensor  
  - Motion camera  

- **Objective:** Build a mini SOC:  
  - Ingest logs into Sentinel  
  - Normalize with functions  
  - Detect anomalies with analytics rules  
  - Visualize with a workbook dashboard  

---

## 🏗️ Architecture

```text
Arduino / Simulated Logs (.ndjson)
        ↓
Logs Ingestion API (DCE + DCR)
        ↓
Custom Tables (ChickenCoopDoor_CL / ChickenCoopWater_CL)
        ↓
KQL Functions (CoopDoor(), CoopWater())
        ↓
Analytics Rules (detections)
        ↓
Workbook Dashboard
```

---

## ⚙️ Step-by-Step Setup

### 1. Prerequisites
- Azure subscription with a **Log Analytics Workspace (LAW)** + **Microsoft Sentinel enabled**  
- Region consistency: LAW, **DCE**, **DCR** all in the same region  
- Windows machine with Python 3.10+ or PowerShell + Azure CLI  
- Chicken Coop NDJSON log files (door + water events)

### 2. Create Data Collection Endpoint (DCE)
Portal → *Monitor → Data Collection Endpoints → + Create*  
Example: `lab-dce-coop`
![Logs Ingestion API Overview](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/logs-ingestion-api-overview/overview-log-ingestion-api.png)


### 3. Create Custom Tables
Portal → *Log Analytics Workspace → Tables → + Create (DCR-based)*  
- `ChickenCoopDoor_CL`  
- `ChickenCoopWater_CL`

**Door Transform**
```kusto
source
| extend TimeGenerated = todatetime(tostring(['time']))
| project TimeGenerated, DeviceId=tostring(['deviceId']),
         EventType=tostring(['eventType']),
         Door=tostring(['doorState']),
         SolarLux=toint(['solarLux']),
         Source=tostring(['source']),
         Location=tostring(['location']),
         Latitude=toreal(['latitude']),
         Longitude=toreal(['longitude'])
```

**Water Transform**
```kusto
source
| extend TimeGenerated = todatetime(tostring(['time']))
| project TimeGenerated, DeviceId=tostring(['deviceId']),
         EventType=tostring(['eventType']),
         TempC=todouble(['tempC']),
         HeaterOn=tobool(['heaterOn']),
         BatteryLow=tobool(['batteryLow']),
         Source=tostring(['source']),
         Location=tostring(['location']),
         Latitude=toreal(['latitude']),
         Longitude=toreal(['longitude'])
```

### 4. Create Service Principal
```powershell
az ad sp create-for-rbac ^
  --name "svc-chicken-ingest" ^
  --role "Monitoring Metrics Publisher" ^
  --scopes <dcr-resource-ids>
```

Save: **appId**, **password**, **tenantId**.

### 5. Collect IDs
- DCE ingestion URI  
- DCR immutable IDs  
- Stream names:
  - `Custom-ChickenCoopDoor_CL`
  - `Custom-ChickenCoopWater_CL`

### 6. Install SDKs
```powershell
python -m pip install azure-monitor-ingestion azure-identity
```

### 7. Python Uploader
`send_coop_split.py`:
```python
from azure.identity import DefaultAzureCredential
from azure.monitor.ingestion import LogsIngestionClient
import json

ENDPOINT = "https://<dce>.<region>.ingest.monitor.azure.com"
CFG = [
  {"file": r"C:\logs\coop_door.log", "rule_id": "<door-dcr-id>", "stream": "Custom-ChickenCoopDoor_CL"},
  {"file": r"C:\logs\coop_water.log","rule_id": "<water-dcr-id>","stream": "Custom-ChickenCoopWater_CL"}
]

cred = DefaultAzureCredential()
client = LogsIngestionClient(endpoint=ENDPOINT, credential=cred)

for item in CFG:
    with open(item["file"], "r", encoding="utf-8") as f:
        rows = [json.loads(line) for line in f if line.strip()]
        client.upload(rule_id=item["rule_id"], stream_name=item["stream"], logs=rows)
        print(f"Uploaded {len(rows)} rows → {item['stream']}")
```

Run with service principal:
```powershell
$env:AZURE_TENANT_ID    = "<tenant-id>"
$env:AZURE_CLIENT_ID    = "<appId>"
$env:AZURE_CLIENT_SECRET= "<password>"
python send_coop_split.py
```

---

## 🔍 Verification & Functions

Verify ingestion:
```kusto
ChickenCoopDoor_CL  | take 5
ChickenCoopWater_CL | summarize count(), avg(TempC) by bin(TimeGenerated, 1d)
```

Save as functions:
```kusto
// CoopDoor()
ChickenCoopDoor_CL
| project TimeGenerated, DeviceId, Door, SolarLux, Source, Location

// CoopWater()
ChickenCoopWater_CL
| project TimeGenerated, DeviceId, TempC, HeaterOn, BatteryLow, Source, Location
```

---

## 🚨 Analytics Rules

### Door open at night
```kusto
let localOffsetHours = 2;
CoopDoor()
| extend LocalTime = datetime_add("hour", localOffsetHours, TimeGenerated)
| where Door == "Open" and (hourofday(LocalTime) >= 22 or hourofday(LocalTime) < 5)
```

### Night temp drop > 5°C in 1h
```kusto
CoopWater()
| order by TimeGenerated asc
| extend TempDrop = TempC - prev(TempC)
| where TempDrop < -5 and (hourofday(TimeGenerated) >= 20 or hourofday(TimeGenerated) <= 6)
```

---

## 📊 Workbook

**Visuals**
- Water hourly avg (timechart)  
- Door events/hour (timechart split by Door)  
- KPI: Avg temp (1h)  
- Grid: LastSeen + min/max/avg per device  

---

## ⚠️ Challenges

- **Duplicates:** LA has no PK → avoid re-sending or add `recordId`.  
- **Region mismatch:** LAW, DCE, DCR must align.  
- **RBAC:** Assign “Monitoring Metrics Publisher” to service principal.  
- **Schema drift:** version transforms.  
- **Cost:** pick correct log plan (Analytics / Basic / Aux / Data Lake).  

---

## ✅ Key Message

If we can build detections, dashboards, and rules from a chicken coop,  
we can ingest and protect **any workload** in Sentinel.

---

## 📎 Resources
- [Microsoft Docs – Logs Ingestion API](https://learn.microsoft.com/azure/azure-monitor/logs/logs-ingestion-api-overview)  
- [Microsoft Docs – Sentinel log plans](https://learn.microsoft.com/azure/sentinel/log-plans)  
- [Microsoft Docs – Sentinel Data Lake](https://learn.microsoft.com/azure/sentinel/datalake/sentinel-lake-overview)
