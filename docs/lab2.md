"weight": -1

# Lab 2 : Advanced KQL - Setup

## Setup the environment for the Kusto Detective Agency

Your first task is to create a free Kusto cluster which will be your primary investigation tool. Then you’ll have to answer a simple question to get started.

- To create a free Kusto cluster, you need either a Microsoft account (MSA) or an Azure Active Directory (AAD) identity. Create your cluster here: [https://aka.ms/kustofree](https://aka.ms/kustofree)

- Setup your database like the image below and signin with a Microsoft Account

![kql1](\assets\kql3.png){ align=center }

 - Copy the Cluster URI, as highlighted in the following image:

![kql1](\assets\kql1.png){ align=center }

- Next, head over to the query editor. You can either click on the Query button highlighted in the below image, or open in the Kusto Explorer desktop tool.

![kql1](\assets\kql2.png){ align=center }

Now, paste the following KQL script into the query editor, and then run it. Make sure you wait around 10 seconds for the script to complete. **Don’t run it more than once!** This script creates a new table and loads a file located in Azure blob storage. 

```kql
.execute database script <|
// Create table for the data
.create-merge table Onboarding(Score:long)
//clear any previously ingested data if such exists
.clear table Onboarding data
// Import data
.ingest into table Onboarding ('https://kustodetectiveagency.blob.core.windows.net/onboarding/onboarding.csv.gz') with (ignoreFirstRecord=true)
```
![kql1](\assets\kql4.png){ align=center }

Now, to test that it’s working, sum up the values in the “Score” column

What's the sum of the "Score" column?