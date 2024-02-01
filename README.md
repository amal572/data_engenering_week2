<div>
<img src="https://github.com/mage-ai/assets/blob/main/mascots/mascots-shorter.jpeg?raw=true">
</div>

## Configuring Mage

Mage is an open-source, hybrid framework for transforming and integrating data. ✨

Before getting into some of the key components of Mage, we need to set up our environment. If you haven't already, you'll need to clone the [here](https://docs.mage.ai/introduction/overview) provided so we can set up our environment locally.

Clone repo to vs code
1. Open a terminal in your local repo directory
2. Run the following commands
3. Now, let's build the container
  ```bash
docker compose build
```

4. start the Docker container:
```bash
docker compose up
```
5. Navigate to http://localhost:6789 in your browser

Now we can start building our pipeline!

## Building an ETL Pipeline with Mage (PostgreSQL)

The first pipeline is an ETL pipeline using blocks to load the yellow taxi data from this GitHub repo, transform the data, and export the data into a Postgres database within our docker container. The resulting DAG should look something like the screenshot below.

<div>
<img src="https://github.com/amal572/data_engenering_week2/blob/main/image/week2.PNG">
</div>

### Configuring PostgreSQL connection

```bash
dev:
  POSTGRES_CONNECT_TIMEOUT: 10
  POSTGRES_DBNAME: "{{ env_var('POSTGRES_DBNAME') }}"
  POSTGRES_SCHEMA: "{{ env_var('POSTGRES_SCHEMA') }}"
  POSTGRES_USER: "{{ env_var('POSTGRES_USER') }}"
  POSTGRES_PASSWORD: "{{ env_var('POSTGRES_PASSWORD') }}"
  POSTGRES_HOST: "{{ env_var('POSTGRES_HOST') }}"
  POSTGRES_PORT: "{{ env_var('POSTGRES_PORT') }}"
```
### There are three types of blocks that we can leverage when creating a pipeline:

1. Data Loader
The data loader is the initial stage of the process. In this block, you begin loading data from various sources and standardizing the data types across all columns.

```bash
import io
import pandas as pd
import requests
if 'data_loader' not in globals():
    from mage_ai.data_preparation.decorators import data_loader
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test


@data_loader
def load_data_from_api(*args, **kwargs):
    """
    Template for loading data from API
    """
    url_yellow_taxi = 'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz'
    url_green_taxi = 'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz'

    taxi_dtypes = {
        'VendorID': 'Int64',
        'store_and_fwd_flag': 'str',
        'RatecodeID': 'Int64',
        'PULocationID': 'Int64',
        'DOLocationID': 'Int64',
        'passenger_count': 'Int64',
        'trip_distance': 'float64',
        'fare_amount': 'float64',
        'extra': 'float64',
        'mta_tax': 'float64',
        'tip_amount': 'float64',
        'tolls_amount': 'float64',
        'ehail_fee': 'float64',
        'improvement_surcharge': 'float64',
        'total_amount': 'float64',
        'payment_type': 'float64',
        'trip_type': 'float64',
        'congestion_surcharge': 'float64'
    }

    parse_dates_green_taxi = ['lpep_pickup_datetime', 'lpep_dropoff_datetime']
    parse_dates_yellow_taxi = ['tpep_pickup_datetime', 'tpep_dropoff_datetime']

    return pd.read_csv(url_yellow_taxi, sep=',', compression='gzip', dtype=taxi_dtypes, parse_dates=parse_yellow_green_taxi)


@test
def test_output(output, *args) -> None:
    """
    Template code for testing the output of the block.
    """
    assert output is not None, 'The output is undefined'
```
2. Transformer
Now that we've defined our data loader, it's time to transform our data. This is where the Transformation blocks in Mage come into play. In this stage, you apply a few simple filters to the taxi data we have loaded. The key transformation here is non_zero_passengers_df, which essentially filters out any row in the dataset where the passenger count is zero. We also count the number of records where the passenger_count is not zero in the non_zero_passengers_count DataFrame before returning the non_zero_passenger_df to be passed to the Data Exporter block.
   
```bash
if 'transformer' not in globals():
    from mage_ai.data_preparation.decorators import transformer
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test

import pandas as pd

@transformer
def transform(data, *args, **kwargs):
    # Specify your transformation logic here

    zero_passengers_df = data[data['passenger_count'].isin([0])]
    zero_passengers_count = zero_passengers_df['passenger_count'].count()
    non_zero_passengers_df = data[data['passenger_count'] > 0]
    non_zero_passengers_count = non_zero_passengers_df['passenger_count'].count()
    print(f'Preprocessing: records with zero passengers: {zero_passengers_count}')
    print(f'Preprocessing: records with 1 passenger or more: {non_zero_passengers_count}')

    return non_zero_passengers_df


@test
def test_output(output, *args) -> None:
    """
    Template code for testing the output of the block.
    """
    assert output['passenger_count'].isin([0]).sum() == 0, 'There are rides with zero passengers'
```
3. Data Exporter
Finally, it is time to export our data! This step is fairly straightforward. Add a Data Exporter block.

In this data exporter, we define three variables: schema_name, table_name, and config_profile. schema_name and table_name will later be used to query our PostgreSQL database, for example: SELECT * FROM ny_taxi.yellow_taxi_data. Setting the config_profile to dev instead of default allows us to access the dev profile in the io_config.yaml, where we previously defined our PostgreSQL connection details.
```bash
from mage_ai.settings.repo import get_repo_path
from mage_ai.io.config import ConfigFileLoader
from mage_ai.io.postgres import Postgres
from pandas import DataFrame
from os import path

if 'data_exporter' not in globals():
    from mage_ai.data_preparation.decorators import data_exporter

@data_exporter
def export_data_to_postgres(df: DataFrame, **kwargs) -> None:

    schema_name = 'ny_taxi'  # Specify the name of the schema to export data to
    table_name = 'yellow_taxi_data'  # Specify the name of the table to export data to
    config_path = path.join(get_repo_path(), 'io_config.yaml')
    config_profile = 'dev'

    with Postgres.with_config(ConfigFileLoader(config_path, config_profile)) as loader:
        loader.export(
            df,
            schema_name,
            table_name,
            index=False,  # Specifies whether to include index in exported table
            if_exists='replace',  # Specify resolution policy if table name already exists
        )
```

### Building an ETL Pipeline with Mage (Google Cloud)
<div>
<img src="https://github.com/amal572/data_engenering_week2/blob/main/image/week31.PNG">
</div>

Update Mage Configuration (Environmental Variables)
```bash
  GOOGLE_SERVICE_ACC_KEY:
    type: service_account
    project_id: project-id
    private_key_id: key-id
    private_key: "-----BEGIN PRIVATE KEY-----\nyour_private_key\n-----END_PRIVATE_KEY"
    client_email: your_service_account_email
    auth_uri: "https://accounts.google.com/o/oauth2/auth"
    token_uri: "https://accounts.google.com/o/oauth2/token"
    auth_provider_x509_cert_url: "https://www.googleapis.com/oauth2/v1/certs"
    client_x509_cert_url: "https://www.googleapis.com/robot/v1/metadata/x509/your_service_account_email"

  GOOGLE_SERVICE_ACC_KEY_FILEPATH: "/path/to/your/service/account/key.json"
  GOOGLE_LOCATION: US # Optional
```

## Assistance

1. [Mage Docs](https://docs.mage.ai/introduction/overview): a good place to understand Mage functionality or concepts.
2. [Mage Slack](https://www.mage.ai/chat): a good place to ask questions or get help from the Mage team.
3. [DTC Zoomcamp](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/main/week_2_workflow_orchestration): a good place to get help from the community on course-specific inquireies.
4. [Mage GitHub](https://github.com/mage-ai/mage-ai): a good place to open issues or feature requests.
