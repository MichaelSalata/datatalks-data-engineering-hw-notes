---
created: 2025-01-22
tags:
  - project/open
  - DE
---
*updated*: `$= dv.current().file.mtime`



## src: [DE Zoomcamp 3.1.1 - Data Warehouse and BigQuery](https://www.youtube.com/watch?v=jrHljAoD6nM&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=35) - yt

## src: [DE Zoomcamp 3.1.2 - Partioning and Clustering](https://www.youtube.com/watch?v=-CqXf7vhhDs&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=28) - yt

### BigQuery
src - [DE Zoomcamp 3.1.1 - Data Warehouse and BigQuery](https://www.youtube.com/watch?v=jrHljAoD6nM&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=35) - yt


can find/view public datasets hosted by bigquery
1. search for the data using Explorer side panel e.g.: *citibike_stations*
```SQL
FROM bigquery-public-data.new_york_citibike.citibike_stations
```

```SQL
	CREATE OR REPLACE EXTERNAL TABLE 'taxi-rides-ny.nytaxi.external_yellow_tripdata'
	OPTIONS (
		format = 'CSV'
		uris = ['gs://nyc-tl-data/trip data/yellow_tripdata_2019-*.csv', 'other_table_names']
	);

```

#### Schema/table meta-data
```SQL
SELECT table_name, parition_id, total_rows
FROM 'nytaxi.INFORMATION_SCHEMA.PARTITIONS'
```

#### Partitioning
helps filtering performance
```SQL
-- create non-partitioned table from external
CREATE OR REPLACE TABLE taxi-rides-ny.nytaxi.yellow_tripdata_non_partitioned AS (
	SELECT * FROM taxi-rides-ny.nytaxi-.external_yellow_tripdata
	);
```

```SQL
--create partitioned table from external
CREATE OR REPLACE TRABLE table_ref
PARTITION BY
	DATE(tpep_pickup.datetime) AS
SELECT * FROM taxi-rides-ny.nytaxi.external_yellow_tripdata;

```

#### Clustering
sorts / sub-sorts tables by specific columns
reduces amount of data processed
```SQL
CREATE OR REPLACE TABLE table_name
PARITION BY DATE(date_col)
CLUSTER BY VendorID AS
SELECT * FROM table_path
```
