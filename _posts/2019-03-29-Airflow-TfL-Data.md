---
layout: post
title: API Harvesting with Apache Airflow
date: 01/04/2019
categories: data-engineering
tags: data-engineering
---

Recently there have been a few API's that I wanted to periodically download data from and aggregate into a larger dataset.  When looking into software that would help to automate this task, I found Apache Airflow.

For an initial task I decided to periodically download car park information from the [TfL api](https://api.tfl.gov.uk/swagger/ui/index.html?url=/swagger/docs/v1#!/Occupancy/Occupancy_Get).  One service the API provides is a snapshot of the current availability of parking spaces at a subset of London station car parks at the time of the request.

There are two sections to this post on API harvesting: the first is the attemt to get things working with Python based HTTP requests and saving to files, and the second is to use the HTTP and database hooks Airflow provides to download and store the data into a Postgres database, as well as use the variable manager for a global variable.

Code available [here](https://github.com/ogladr-kjarr/Apache_Airflow_Introduction_Py_Ops).

# Part One - Up and Running

There was a couple of initially helpful blogs ([here](https://medium.com/datareply/airflow-lesser-known-tips-tricks-and-best-practises-cf4d4a90f8f), and [here](https://tech.marksblogg.com/airflow-postgres-redis-forex.html)) as well as the Airflow documentation and example DAG's provided that let me get started.

I decided to create two tasks in the DAG, the first downloads the JSON data and saves it to a file, the second converts the file from JSON to CSV format.

Though most blog posts I found decided to use the XCOM system of having data accessible between tasks to continue working on the same data, I though the design pattern was to not have the tasks dependent on each other in such a way.  I chose to write to an intermediate file location, which I think is more in the spirit of Airflow.

Below is the whole code listing, followed by a look at each particular section.

~~~ python
import requests
import time
import datetime as dt
import pandas as pd
import json
import re
from os import listdir, remove
from os.path import isfile, join

from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from airflow import AirflowException

base_dir = '/Users/ok/airflow/parkingdata/'

def download_api_data(**kwargs):

    api_uri = 'https://api.tfl.gov.uk/Occupancy/CarPark'
    r = requests.get(api_uri)

    if r.status_code == 200:
        filepostfix = str(time.time()) + '.json'
        filename = join(kwargs['base_dir'], filepostfix)

        with open(filename, 'w') as f:
            f.write(r.text)
    else:
        raise AirflowException()
 
def api_json_to_csv(**kwargs):

    json_files = [f for f in listdir(kwargs['base_dir']) 
                    if isfile(join(kwargs['base_dir'], f)) 
                    if '.json' in f]

    csv_file = [f for f in listdir(kwargs['base_dir']) 
                    if isfile(join(kwargs['base_dir'], f)) 
                    if 'parking_data.csv' in f]

    df_columns = [
        'timestamp',
        'park_id',
        'park_name',
        'park_bay_type',
        'park_bay_occupied',
        'park_bay_free',
        'park_bay_count']

    parking_df = pd.DataFrame(columns=df_columns)

    for current_file in json_files:
        time_pattern = re.compile('[0-9]*')
        pattern_match = time_pattern.match(current_file)

        if pattern_match is not None:
            timestamp = pattern_match.group()

            with open(join(kwargs['base_dir'],current_file), 'r') as f:
                car_parks = json.loads(f.read())

            for car_park in car_parks:
                park_id = car_park['id']
                park_name = car_park['name']

                for bay in car_park['bays']:
                    park_bay_type = bay['bayType']
                    park_bay_occupied = bay['occupied']
                    park_bay_free = bay['free']
                    park_bay_count = bay['bayCount']

                    parking_df = parking_df.append({
                        'timestamp': timestamp,
                        'park_id': park_id,
                        'park_name': park_name,
                        'park_bay_type': park_bay_type,
                        'park_bay_occupied': park_bay_occupied,
                        'park_bay_free': park_bay_free,
                        'park_bay_count': park_bay_count
                    }, ignore_index=True)

    filepostfix = 'parking_data.csv'
    filename = join(kwargs['base_dir'], filepostfix)

    if len(csv_file) > 0:
        with open(filename, 'a') as f:
            parking_df.to_csv(f, 
                              header=False,
                              index=False)
    else:
        parking_df.to_csv(filename,
                          index=False)

    for current_file in json_files:
        remove(join(kwargs['base_dir'],current_file)) 
 
default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': dt.datetime(2018, 10, 1, 10, 00, 00),
    'concurrency': 1,
    'retries': 0,
    'provide_context': True
}
 
with DAG('london_parking_bay_harvest_operators',
         catchup=False,
         default_args=default_args,
         schedule_interval='*/15 * * * *',
         # schedule_interval=None,
         ) as dag:

    opr_download_data = PythonOperator(
        task_id='download_api_data',
        op_kwargs={'base_dir': base_dir},
        python_callable=download_api_data
    )
 
    api_json_to_csv = PythonOperator(
        task_id='api_json_to_csv',
        op_kwargs={'base_dir': base_dir},
        python_callable=api_json_to_csv
    )

opr_download_data.set_downstream(api_json_to_csv)
~~~

With the Airflow specific libraries, only the Python operator is used, and the AirflowException is to allow flagging a failed task if the API provides a HTTP request value other than 200.  The base_dir is a global variable which gets past in to the Python operators as part of the their kwargs, giving the location of files to be saved.

~~~ python
import requests
import time
import datetime as dt
import pandas as pd
import json
import re
from os import listdir, remove
from os.path import isfile, join

from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from airflow import AirflowException

base_dir = '/Users/ok/airflow/parkingdata/'
~~~

The first operator below performs a GET request of the API, and if successful the JSON is dumped to a file, if not, an AirflowException is raised, failing the task and therefore the whole DAG.  The base_dir to save files to is past in as base_dir in kwargs.

~~~ python
def download_api_data(**kwargs):

    api_uri = 'https://api.tfl.gov.uk/Occupancy/CarPark'
    r = requests.get(api_uri)

    if r.status_code == 200:
        filepostfix = str(time.time()) + '.json'
        filename = join(kwargs['base_dir'], filepostfix)

        with open(filename, 'w') as f:
            f.write(r.text)
    else:
        raise AirflowException()
~~~

This second part loads the JSON files, appends each entry in a JSON file to a Pandas dataframe, then writes the data to a file, creating or appending as appropriate.  It gets the file location from the base_dir parameter past in with kwargs.

~~~ python
def api_json_to_csv(**kwargs):

    json_files = [f for f in listdir(kwargs['base_dir']) 
                    if isfile(join(kwargs['base_dir'], f)) 
                    if '.json' in f]

    csv_file = [f for f in listdir(kwargs['base_dir']) 
                    if isfile(join(kwargs['base_dir'], f)) 
                    if 'parking_data.csv' in f]

    df_columns = [
        'timestamp',
        'park_id',
        'park_name',
        'park_bay_type',
        'park_bay_occupied',
        'park_bay_free',
        'park_bay_count']

    parking_df = pd.DataFrame(columns=df_columns)

    for current_file in json_files:
        time_pattern = re.compile('[0-9]*')
        pattern_match = time_pattern.match(current_file)

        if pattern_match is not None:
            timestamp = pattern_match.group()

            with open(join(kwargs['base_dir'],current_file), 'r') as f:
                car_parks = json.loads(f.read())

            for car_park in car_parks:
                park_id = car_park['id']
                park_name = car_park['name']

                for bay in car_park['bays']:
                    park_bay_type = bay['bayType']
                    park_bay_occupied = bay['occupied']
                    park_bay_free = bay['free']
                    park_bay_count = bay['bayCount']

                    parking_df = parking_df.append({
                        'timestamp': timestamp,
                        'park_id': park_id,
                        'park_name': park_name,
                        'park_bay_type': park_bay_type,
                        'park_bay_occupied': park_bay_occupied,
                        'park_bay_free': park_bay_free,
                        'park_bay_count': park_bay_count
                    }, ignore_index=True)

    filepostfix = 'parking_data.csv'
    filename = join(kwargs['base_dir'], filepostfix)

    if len(csv_file) > 0:
        with open(filename, 'a') as f:
            parking_df.to_csv(f, 
                              header=False,
                              index=False)
    else:
        parking_df.to_csv(filename,
                          index=False)

    for current_file in json_files:
        remove(join(kwargs['base_dir'],current_file)) 
~~~

In this last section the DAG itself is created, scheduled to run every fifteen minutes, with some filler default arguments.  The two python operators are created, with task names to identify in the GUI, the base_dir parameter value, and a reference to the function that is the callable operator.  Lastly the set_downstream function is used to create the link between operators.

~~~ python
default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': dt.datetime(2018, 10, 1, 10, 00, 00),
    'concurrency': 1,
    'retries': 0,
    'provide_context': True
}
 
with DAG('london_parking_bay_harvest_operators',
         catchup=False,
         default_args=default_args,
         schedule_interval='*/15 * * * *',
         # schedule_interval=None,
         ) as dag:

    opr_download_data = PythonOperator(
        task_id='download_api_data',
        op_kwargs={'base_dir': base_dir},
        python_callable=download_api_data
    )
 
    api_json_to_csv = PythonOperator(
        task_id='api_json_to_csv',
        op_kwargs={'base_dir': base_dir},
        python_callable=api_json_to_csv
    )

opr_download_data.set_downstream(api_json_to_csv)
~~~


# Part Two - HTTP and Database Hooks

Rather than use the requests package for the HTTP request, I wanted to refactor to use Airflow's HTTP hook.  Similarly rather than save to a file I wanted to save to a Postgresql database using the Postgres hook.  Lastly I wanted to factor out the global base_dir variable into Airflow's saved variables.

The database is running on Docker using the following command:

`docker run --name test-postgres -p 5432:5432 -e POSTGRESS_PASSWORD=postgres -d postgres:9.4.21-alpine`

And the table is as below - no primary key is created as there are duplicate entries in the dataset when using a primary key made up of t_stamp, park_id, park_bay_type, something that only happens with the Harrow & Wealdstone Stn (LUL) entry:

~~~ sql
create table tfl_data (
    t_stamp text,
    park_id text,
    park_name text,
    park_bay_type text,
    park_bay_occupied integer,
    park_bay_free integer,
    park_bay_count integer
);
~~~

Next we setup the HTTP connection:

![Airflow HTTP Hook Connection](/assets/img/tfl_parking_post/tfl_http.png)

This is followed by the database connection:

![Airflow Postgresql Hook Connection](/assets/img/tfl_parking_post/tfl_db.png)

And finally the global variable:

![Airflow Variable](/assets/img/tfl_parking_post/tfl_variable.png)

The first python operator is changed to use the HTTP hook, which will automatically fail the task if any reqeust result other than 200 is received through the flag to check_response.  The extra import statements are also shown.

~~~ python
from airflow.hooks import HttpHook, PostgresHook
from airflow.models import Variable

def download_api_data(**kwargs):

    tfl_hook = HttpHook(http_conn_id='tfl_parking_conn', method='GET')
    resp = tfl_hook.run('', extra_options={'check_response': True})

    filepostfix = str(time.time()) + '.json'
    base_dir = Variable.get("tfl_park_base_dir")
    filename = join(base_dir, filepostfix)

    with open(filename, 'w') as f:
        f.write(resp.text)
~~~

The second Python operator is similar to the first attempt version in the way the entries are extracted.  The difference is that rather than append entries to a Pandas dataframe a list of entries stored as tuples are created, and this is passed to the insert_rows function of the Postgresql hook.

~~~ python
def api_json_to_db(**kwargs):
    base_dir = Variable.get("tfl_park_base_dir")

    json_files = [f for f in listdir(base_dir) 
                    if isfile(join(base_dir, f)) 
                    if '.json' in f]

    db_tuples = []

    for current_file in json_files:
        time_pattern = re.compile('[0-9]*')
        pattern_match = time_pattern.match(current_file)

        if pattern_match is not None:
            timestamp = pattern_match.group()

            with open(join(base_dir,current_file), 'r') as f:
                car_parks = json.loads(f.read())

            for car_park in car_parks:
                park_id = car_park['id']
                park_name = car_park['name']

                for bay in car_park['bays']:
                    park_bay_type = bay['bayType']
                    park_bay_occupied = bay['occupied']
                    park_bay_free = bay['free']
                    park_bay_count = bay['bayCount']

                    db_tuples.append((timestamp, 
                                        park_id, 
                                        park_name, 
                                        park_bay_type, 
                                        park_bay_occupied,
                                        park_bay_free,
                                        park_bay_count))

    target_fields = ['t_stamp',
                        'park_id',
                        'park_name',
                        'park_bay_type',
                        'park_bay_occupied',
                        'park_bay_free',
                        'park_bay_count']

    db_hook = PostgresHook(postgres_conn_id='tfl_db')
    db_hook.insert_rows('tfl_data', db_tuples, target_fields)

    for current_file in json_files:
        remove(join(base_dir,current_file))
~~~

Lastly the DAG is updated to show the newly named second task, and to remove the base_dir argument.

~~~ python
with DAG('london_parking_bay_harvest_hooks',
         catchup=False,
         default_args=default_args,
         schedule_interval='*/15 * * * *',
         # schedule_interval=None,
         ) as dag:

    opr_download_data = PythonOperator(
        task_id='download_api_data',
        python_callable=download_api_data
    )
 
    api_json_to_db = PythonOperator(
        task_id='api_json_to_db',
        python_callable=api_json_to_db
    )

opr_download_data.set_downstream(api_json_to_db)
~~~

I included in the Github repo a RMarkdown file that connects to the database and creates a few plots, just to show how the data could be used.  In the future I want to try to automate the reports creation via a bash command in the DAG.