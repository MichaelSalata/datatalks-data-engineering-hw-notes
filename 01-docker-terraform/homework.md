
## Question 1. Understanding docker first run
Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.

What's the version of pip in the image?

- 24.3.1
- 24.2.1
- 23.3.1
- 23.2.1

### ANSWER
TERMINAL
```
michael@michael-desktop:~$ docker run -it --entrypoint bash python:3.12.8
Unable to find image 'python:3.12.8' locally
3.12.8: Pulling from library/python
fd0410a2d1ae: Pull complete 
bf571be90f05: Pull complete 
684a51896c82: Pull complete 
fbf93b646d6b: Pull complete 
5f16749b32ba: Pull complete 
e00350058e07: Pull complete 
eb52a57aa542: Pull complete 
Digest: sha256:5893362478144406ee0771bd9c38081a185077fb317ba71d01b7567678a89708
Status: Downloaded newer image for python:3.12.8
root@12620eea5490:/# pip --version
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
```

24.3.1

## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that **pgadmin** should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

- postgres:5433
- localhost:5432
- db:5433
- postgres:5432
- db:5432

### ANSWER
`hostname`
postgres

`port`
5432


##  Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from October 2019:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
```

NOTE:  ingest_data3.py is the same as the course script except both instances of the  datetime conversion lines were commented out
```
# df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
# df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
```

COMMAND
python3 ingest_data3.py \
    --user=root \
    --password=root \
    --host=localhost \
    --port=5432 \
    --db=ny_taxi \
    --table_name=green_taxi_trips \
    --url="wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz"

TERMINAL
```
michael@michael-desktop:~/Documents/projects/docker_sql_tinkering/dockerizing_ingest$ python3 ingest_data3.py \
    --user=root \
    --password=root \
    --host=localhost \
    --port=5432 \
    --db=ny_taxi \
    --table_name=green_taxi_trips \
    --url="wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz"
--2025-01-15 17:12:28--  http://wget/
Resolving wget (wget)... failed: Name or service not known.
wget: unable to resolve host address ‘wget’
--2025-01-15 17:12:28--  https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
Resolving github.com (github.com)... 140.82.112.4
Connecting to github.com (github.com)|140.82.112.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/513814948/ea580e9e-555c-4bd0-ae73-43051d8e7c0b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20250115%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250115T231228Z&X-Amz-Expires=300&X-Amz-Signature=3713a1fcf0c7800955432c251d1a027544964b9bf9e909289fdf394521ebbaf5&X-Amz-SignedHeaders=host&response-content-disposition=attachment%3B%20filename%3Dgreen_tripdata_2019-10.csv.gz&response-content-type=application%2Foctet-stream [following]
--2025-01-15 17:12:28--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/513814948/ea580e9e-555c-4bd0-ae73-43051d8e7c0b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20250115%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250115T231228Z&X-Amz-Expires=300&X-Amz-Signature=3713a1fcf0c7800955432c251d1a027544964b9bf9e909289fdf394521ebbaf5&X-Amz-SignedHeaders=host&response-content-disposition=attachment%3B%20filename%3Dgreen_tripdata_2019-10.csv.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.110.133, 185.199.111.133, 185.199.108.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8262584 (7.9M) [application/octet-stream]
Saving to: ‘output.csv.gz’

output.csv.gz       100%[===================>]   7.88M  19.5MB/s    in 0.4s    

2025-01-15 17:12:29 (19.5 MB/s) - ‘output.csv.gz’ saved [8262584/8262584]

FINISHED --2025-01-15 17:12:29--
Total wall clock time: 1.0s
Downloaded: 1 files, 7.9M in 0.4s (19.5 MB/s)
inserted another chunk, took 8.801 second
inserted another chunk, took 8.514 second
/home/michael/Documents/projects/docker_sql_tinkering/dockerizing_ingest/ingest_data3.py:47: DtypeWarning: Columns (3) have mixed types. Specify dtype option on import or set low_memory=False.
  df = next(df_iter)
inserted another chunk, took 9.026 second
inserted another chunk, took 5.667 second
Finished ingesting data into the postgres database
```


You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Download this data and put it into Postgres.

You can use the code from the course. It's up to you whether
you want to use Jupyter or a python script.


COMMAND
```
python3 ingest_data3.py \
    --user=root \
    --password=root \
    --host=localhost \
    --port=5432 \
    --db=ny_taxi \
    --table_name=zones \
    --url="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv"
```

TERMINAL
```
michael@michael-desktop:~/Documents/projects/docker_sql_tinkering/dockerizing_ingest$ python3 ingest_data3.py \
    --user=root \
    --password=root \
    --host=localhost \
    --port=5432 \
    --db=ny_taxi \
    --table_name=zones \
    --url="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv"
--2025-01-15 17:17:32--  https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
Resolving github.com (github.com)... 140.82.114.3
Connecting to github.com (github.com)|140.82.114.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/513814948/5a2cc2f5-b4cd-4584-9c62-a6ea97ed0e6a?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20250115%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250115T231732Z&X-Amz-Expires=300&X-Amz-Signature=8d018941552af6eb9e2d777db9239b8adfc0c2879843259457da4c5803c40205&X-Amz-SignedHeaders=host&response-content-disposition=attachment%3B%20filename%3Dtaxi_zone_lookup.csv&response-content-type=application%2Foctet-stream [following]
--2025-01-15 17:17:32--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/513814948/5a2cc2f5-b4cd-4584-9c62-a6ea97ed0e6a?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20250115%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250115T231732Z&X-Amz-Expires=300&X-Amz-Signature=8d018941552af6eb9e2d777db9239b8adfc0c2879843259457da4c5803c40205&X-Amz-SignedHeaders=host&response-content-disposition=attachment%3B%20filename%3Dtaxi_zone_lookup.csv&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.111.133, 185.199.108.133, 185.199.110.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 12322 (12K) [application/octet-stream]
Saving to: ‘output.csv’

output.csv          100%[===================>]  12.03K  --.-KB/s    in 0.001s  

2025-01-15 17:17:32 (9.17 MB/s) - ‘output.csv’ saved [12322/12322]

Finished ingesting data into the postgres database
```




## Question 3. Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, **respectively**, happened:
1. Up to 1 mile
2. In between 1 (exclusive) and 3 miles (inclusive),
3. In between 3 (exclusive) and 7 miles (inclusive),
4. In between 7 (exclusive) and 10 miles (inclusive),
5. Over 10 miles 

Answers:

- 104,802;  197,670;  110,612;  27,831;  35,281
- 104,802;  198,924;  109,603;  27,678;  35,189
- 104,793;  201,407;  110,612;  27,831;  35,281
- 104,793;  202,661;  109,603;  27,678;  35,189
- 104,838;  199,013;  109,645;  27,688;  35,202

### ANSWER
```
SELECT
	COUNT(*) FILTER(WHERE trip_distance <= 1.0) as "less_than_1_mile",
	COUNT(*) FILTER(WHERE trip_distance > 1.0 AND trip_distance <= 3.0) as "1-3_miles",
	COUNT(*) FILTER(WHERE trip_distance > 3.0 AND trip_distance <= 7.0) as "3-7_miles",
	COUNT(*) FILTER(WHERE trip_distance > 7.0 AND trip_distance <= 10.0) as "7-10_miles",
	COUNT(*) FILTER(WHERE trip_distance > 10.0) as "more_than_10_miles"
FROM green_taxi_trips;
```
- less_than_1_mile; 1-3_miles; 3-7_mile; 7-10_miles; more_than_10_miles

- 104,838;  199,013;  109,645;  27,688;  35,202

## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance?
Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance. 

- 2019-10-11
- 2019-10-24
- 2019-10-26
- 2019-10-31

### ANSWER

```
SELECT
	MAX(trip_distance) FILTER(WHERE lpep_pickup_datetime LIKE '2019-10-11%' ) as "longest_trip_2019-10-11",
	MAX(trip_distance) FILTER(WHERE lpep_pickup_datetime LIKE '2019-10-24%' ) as "longest_trip_2019-10-24",
	MAX(trip_distance) FILTER(WHERE lpep_pickup_datetime LIKE '2019-10-26%' ) as "longest_trip_2019-10-26",
	MAX(trip_distance) FILTER(WHERE lpep_pickup_datetime LIKE '2019-10-31%' ) as "longest_trip_2019-10-31"
FROM green_taxi_trips;
```

- longest_trip_2019-10-11
- 95.78

- longest_trip_2019-10-24
- 90.75

- longest_trip_2019-10-26
- 91.56

- longest_trip_2019-10-31
- 515.89 



## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in
`total_amount` (across all trips) for 2019-10-18?

Consider only `lpep_pickup_datetime` when filtering by date.
 
- East Harlem North, East Harlem South, Morningside Heights
- East Harlem North, Morningside Heights
- Morningside Heights, Astoria Park, East Harlem South
- Bedford, East Harlem North, Astoria Park


### ANSWER
```
SELECT z."Zone" as location, SUM(total_amount) as loc_total
FROM green_taxi_trips gtt
INNER JOIN zones z ON gtt."PULocationID" = z."LocationID"
WHERE gtt.lpep_pickup_datetime LIKE '2019-10-18%'
GROUP BY z."Zone"
ORDER BY loc_total DESC 
LIMIT 3;
```
East Harlem North, East Harlem South, Morningside Heights


## Question 6. Largest tip

For the passengers picked up in Ocrober 2019 in the zone
name "East Harlem North" which was the drop off zone that had
the largest tip?

Note: it's `tip` , not `trip`

We need the name of the zone, not the ID.

- Yorkville West
- JFK Airport
- East Harlem North
- East Harlem South

### ANSWER
```
SELECT tip_amount, DO_zone."Zone" AS drop_off_loc
FROM green_taxi_trips gtt
INNER JOIN zones DO_zone ON gtt."DOLocationID" = DO_zone."LocationID"
INNER JOIN zones PU_zone ON gtt."PULocationID" = PU_zone."LocationID"
WHERE gtt.lpep_pickup_datetime LIKE '2019-10%' AND PU_zone."Zone"='East Harlem North'
ORDER BY tip_amount DESC
LIMIT 1;
```
JFK Airport


## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform. 
Copy the files from the course repo
[here](../../../01-docker-terraform/1_terraform_gcp/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.


## Question 7. Terraform Workflow

Which of the following sequences, **respectively**, describes the workflow for: 
1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform`

Answers:
- terraform import, terraform apply -y, terraform destroy
- teraform init, terraform plan -auto-apply, terraform rm
- terraform init, terraform run -auto-aprove, terraform destroy
- terraform init, terraform apply -auto-aprove, terraform destroy
- terraform import, terraform apply -y, terraform rm

### ANSWER
- terraform init, terraform apply -auto-aprove, terraform destroy

## Submitting the solutions

* Form for submitting: https://courses.datatalks.club/de-zoomcamp-2025/homework/hw1

