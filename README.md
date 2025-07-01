# KQL Time Machine: Silk Road Trade Analysis

This repository contains a sample dataset and KQL (Kusto Query Language) queries for the blog post ["KQL Time Machine: Querying Historical Events to Learn Data Analysis"](https://rodtrent.substack.com/p/kql-time-machine-querying-historical). The dataset represents spice trade records from the Silk Road (700–800 CE) and is used to demonstrate KQL operators like `summarize`, `timechart`, and `pivot`.

## Repository Contents
- `SilkRoadTrades.csv`: A CSV file with 50 trade records, including columns `TradeDate`, `SpiceType`, `Volume`, `Origin`, and `Destination`.
- `blog_post.md`: The blog post explaining the dataset and KQL queries.
- `README.md`: This file, with setup and usage instructions.

## Prerequisites
To use this dataset and run the queries, you need:
- An **Azure Data Explorer** cluster (free tier available at [learn.microsoft.com](https://learn.microsoft.com/en-us/azure/data-explorer/)).
- Access to the Azure Data Explorer web UI or a KQL client.
- Basic familiarity with KQL (see [KQL quick reference](https://learn.microsoft.com/en-us/azure/data-explorer/kql-quick-reference)).

## Setup Instructions

### 1. Clone the Repository
Clone this repository to your local machine:

```bash
git clone https://github.com/<your-username>/kql-time-machine.git
cd kql-time-machine
```

### 2. Create the Kusto Table
In the Azure Data Explorer web UI, create a table named `SilkRoadTrades` with the following schema:

```kql
.create table SilkRoadTrades (
    TradeDate: datetime,
    SpiceType: string,
    Volume: real,
    Origin: string,
    Destination: string
)
```

### 3. Ingest the Dataset
Ingest the `SilkRoadTrades.csv` data into the table using one of these methods:

#### Option A: Inline Ingestion
Copy the contents of `SilkRoadTrades.csv` and run the following KQL command in the Azure Data Explorer web UI:

```kql
.ingest inline into table SilkRoadTrades <|
TradeDate,SpiceType,Volume,Origin,Destination
700-03-15T00:00:00Z,Saffron,150.5,Persia,Byzantium
700-06-20T00:00:00Z,Pepper,300.0,India,Samarkand
700-09-10T00:00:00Z,Cinnamon,200.0,China,Chang’an
...
```

(Note: Replace `...` with the full CSV content from `SilkRoadTrades.csv`.)

#### Option B: Upload via Web UI
1. In the Azure Data Explorer web UI, select your database.
2. Click **Ingest** > **From local file**.
3. Upload `SilkRoadTrades.csv` and map the columns to the `SilkRoadTrades` table schema.
4. Execute the ingestion.

### 4. Verify the Data
Confirm the data loaded correctly by running:

```kql
SilkRoadTrades
| count
```

Expected output: 50 records.

## Running the KQL Queries
The [blog post](`https://rodtrent.substack.com/p/kql-time-machine-querying-historical`) includes three KQL queries to analyze the dataset. Run them in the Azure Data Explorer web UI:

### Query 1: Total Volume by Spice Type
Summarizes the total trade volume for each spice.

```kql
SilkRoadTrades
| summarize TotalVolume = sum(Volume) by SpiceType
| order by TotalVolume desc
```

**Expected Output**: Pepper has the highest volume, followed by Saffron and Cinnamon.

### Query 2: Saffron Trade Over Time
Visualizes saffron trade volumes by year using a time chart.

```kql
SilkRoadTrades
| where SpiceType == "Saffron"
| summarize TotalVolume = sum(Volume) by bin(TradeDate, 1y)
| render timechart
```

**Expected Output**: A line chart with a peak around 750 CE.

### Query 3: Pivot by Origin and Destination
Compares saffron trade volumes by origin and destination.

```kql
SilkRoadTrades
| where SpiceType == "Saffron"
| summarize TotalVolume = sum(Volume) by Origin, Destination
| pivot (
