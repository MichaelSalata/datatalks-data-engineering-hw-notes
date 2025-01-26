---
created: 2025-01-22
tags:
  - project/open
---
*updated*: `$= dv.current().file.mtime`


# [Running Airflow in Docker](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html)
- ensure at least 4GB memory can be allocated to Docker
```bash
docker run --rm "debian:bookworm-slim" bash -c 'numfmt --to iec $(echo $(($(getconf _PHYS_PAGES) * $(getconf PAGE_SIZE))))'
```
- download docker-compose.yaml for airflow

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.10.4/docker-compose.yaml'
```

* set the user ID to the environment
```bash
echo -e "AIRFLOW_UID=$(id -u)" > .env
```


```bash
mkdir -p ./dags ./logs ./plugins ./config
```


# Create a custom Dockerfile/image of Airflow
/home/michael/Documents/projects/airflow-tinkering/airflow-gcp/airflow.Dockerfile

change to the latest airflow image
```sql
FROM apache/airflow:2.10.4
```

switch to root user near the top for cur & GCP SDK installs
```sql
USER root
```

use the latest Google Cloud SDK
```Dockerfile
# Install Google Cloud SDK
ARG CLOUD_SDK_VERSION=480.0.0
ENV GCLOUD_HOME=/opt/google-cloud-sdk
ENV PATH="${GCLOUD_HOME}/bin/:${PATH}"

RUN DOWNLOAD_URL="https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-${CLOUD_SDK_VERSION}-linux-x86_64.tar.gz" \
&& TMP_DIR="$(mktemp -d)" \
&& curl -fL "${DOWNLOAD_URL}" --output "${TMP_DIR}/google-cloud-sdk.tar.gz" \
&& mkdir -p "${GCLOUD_HOME}" \
&& tar xzf "${TMP_DIR}/google-cloud-sdk.tar.gz" -C "${GCLOUD_HOME}" --strip-components=1 \
&& "${GCLOUD_HOME}/install.sh" \
--bash-completion=false \
--path-update=false \
--usage-reporting=false \
--quiet \
&& rm -rf "${TMP_DIR}" \
&& gcloud --version
```

near the end switch to user airflow for some pip installs
```Dockerfile
# Switch to airflow user before installing pip dependencies
USER airflow

RUN pip install --no-cache-dir -r requirements.txt

# Set the working directory
WORKDIR $AIRFLOW_HOME

# Switch back to the airflow user
USER $AIRFLOW_UID
```


docker compose build # I think?
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