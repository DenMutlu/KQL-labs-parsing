# Querying normalized data

With normalized data, you can run queries across sources using the ASIM schema. For instance, you might want to analyze successful login attempts across all sources:

```kql
_Im_Authentication(starttime=ago(1d), LogonResult='Success')
| summarize count() by ActorUserName, bin(TimeGenerated, 1h)
| order by count_ desc
```

This query outputs the top usernames with successful logins in the last day, regardless of the data source.

## Summary

- **Parsing** transforms raw data into a structured format, making it easier to analyze.
- **Normalization** standardizes data fields across sources, enabling consistent queries.
- **ASIM in Microsoft Sentinel** provides schemas and parsers to simplify the process of working with diverse data sources.



The final goal of parsing and normalization, is tp query and analyze security data, efficiently across your environment.