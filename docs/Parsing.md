# Parsing Basics and Data Normalization

## Introduction

In security operations, data often comes from various sources, each with its unique format and field names. **Parsing** and **data normalization** are essential for transforming this data into a consistent, specially for SOC operations, standardized format, making it easier to analyze and correlate information across different sources is one of the most important tasks to have valuable and efficient detection use cases.

---

## What is Parsing?

**Parsing** is the process of breaking down raw data into a structured format so it can be analyzed or transformed. ie. In Microsoft Sentinel, parsing often involves using **Kusto Query Language (KQL)** functions to interpret and organize data from logs and other sources. in Splunk we will use the Common Information Model (CIM) to normalize and directly apply the configuration in the .conf files (props.conf and transform.conf)

### Example of Parsing a Firewall Log

Suppose you have a raw log entry from a firewall like this:

```console
source_ip=192.168.1.10, dest_ip=10.1.1.5, action=ALLOW, protocol=TCP
```


A parsing function can transform this data into a structured format:
```console
{
  "SourceIP": "192.168.1.10",
  "DestinationIP": "10.1.1.5",
  "Action": "Allow",
  "Protocol": "TCP"
}
```

This structure makes the data easier to work with and understand, allowing for consistent querying across sources.

###What is Data Normalization?
Data normalization is the process of standardizing data across different sources by mapping fields and values to a common format or schema. This enables analysts to query and analyze data in a consistent way, regardless of the original data source.

In Microsoft Sentinel, the Advanced Security Information Model (ASIM) provides a framework for normalization, defining schemas for various event types like authentication, network sessions, and DNS activity.

###Why Normalize Data?
Data normalization offers several benefits:

**Consistency**: Standardized data fields make data easier to understand.
Cross-Source Analysis: You can query and analyze data across multiple sources with a single query.
Reduced Complexity: Normalized data means you don’t need to create separate queries for each data source.

Parsing and Normalization Example
Let’s walk through an example of parsing and normalizing an authentication event.

####Raw Event Data from Active Directory
A typical login event from Active Directory might look like this:

```console
{
  "UserName": "Pauline_Muller",
  "LogonType": "Interactive",
  "Success": true,
  "SourceIPAddress": "192.168.1.15"
}
```
Step 1: Parse the Data
Parsing involves mapping each field in the raw data to a structured format using KQL:

```kql
datatable(UserName:string, LogonType:string, Success:bool, SourceIPAddress:string)
| extend ParsedUsername = UserName
| extend ParsedSourceIP = SourceIPAddress
| extend LogonSuccess = iif(Success == true, "Success", "Failure")
```
After parsing, the data looks like this:
```console
{
  "ParsedUsername": "john_doe",
  "ParsedSourceIP": "192.168.1.15",
  "LogonSuccess": "Success",
  "LogonType": "Interactive"
}
```
Step 2: Normalize the Data Using ASIM
Using ASIM’s Authentication Event Schema, we can map the parsed data to ASIM's standardized fields:

```console
{
  "ActorUserName": "john_doe",
  "LogonType": "Interactive",
  "LogonResult": "Success",
  "SrcIpAddr": "192.168.1.15"
}
```
Now, whether this data is from Active Directory, Okta, or any other source, it will follow the same schema, enabling consistent querying across all sources.
