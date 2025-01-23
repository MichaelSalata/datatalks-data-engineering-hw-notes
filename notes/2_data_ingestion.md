---
created: 2025-01-22
tags:
  - project/open
---
*updated*: `$= dv.current().file.mtime`




# data_ingestion_local.py  with  Airflow


## managing environment variables

volumes:
```yaml
- ~/.gcp/:/.google/credentials:ro
```


1. Define your variables inside `.env` file
    ```
    PG_HOST=pgdatabase
    PG_USER=root
    PG_PASSWORD=root
	PG_PORT=5432
	PG_DATABASE=ny_taxi
    ```
    
2. Add the environment variables in `docker-compose.yaml` inside the `x-airflow-common` environment definition:
    ```yaml
    PG_HOST: "${PG_HOST}"
    PG_USER: "${PG_USER}"
    PG_PASSWORD: "${PG_PASSWORD}"
    PG_PORT: "${PG_PORT}"
    PG_DATABASE: "${PG_DATABASE}"
    ```
    
3. In your DAG, grab the variables with `os.getenv()`:
    ```python
    PG_HOST = os.getenv('PG_HOST')
    PG_USER = os.getenv('PG_USER')
    PG_PASSWORD = os.getenv('PG_PASSWORD')
    PG_PORT = os.getenv('PG_PORT')
    PG_DATABASE = os.getenv('PG_DATABASE')
    ```



## FINAL ADJUSTMENTS
* LOOP DEBUG: tasks > ingest script > log

* hard code the PGAdmin+PostgreSQL variables into a `.env` file
```
AIRFLOW_UID=1000
PG_HOST=pgdatabase
PG_USER=root
PG_PASSWORD=root
PG_PORT=5432
PG_DATABASE=ny_taxi
```

- update the `data_ingestion_local.py`
	- add
	```python
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"
```
	- change line 39 to:
```python
bash_command = `f'curl -sSL {URL} > {OUTPUT_FILE_TEMPLATE}'`
```


- update `ingestion_script.py`
	- commented out `df.tpep_pickup_datetime` and `df.tpep_dropoff_datetime` 
	- we're not going to bother installing Dataframe dependencies into our Airflow container for a trial run

- update `docker-compose.yaml`  for PGAdmin+PostgreSQL DB pair
	- update ports to avoid overlapping with **Airflow**   and   it's **logging PostgreSQL DB**
		- under `pgdatabase`  service
			- `"5432:5432"` -> `"5433:5432"`
		- under `pgadmin`  service
			- `"8080:80"` -> `"8081:80"`
			- launching PGAdmin is now `localhost:8081`
	- make PGAdmin + PostgreSQL DB launch into Airflow's Network:  ADD

```yaml
networks:
  - airflow-gcp_default
```

```yaml
networks:
  airflow-gcp_default:
    external: true
```

**NOTE:** docker networks can be found with:
```bash
docker network ls
```
and docker image can also be added to another network with
```bash
docker network connect airflow-gcp_default  <pgdatabase-containerID>
```



## data_ingestion_gcs_dag.py