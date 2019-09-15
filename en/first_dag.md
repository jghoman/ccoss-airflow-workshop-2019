# Creating your first DAG 

## Setting up the webserver

Congrats! You set-up Airflow Breeze. Now let's configure and run Airflow.

After setting up Airflow breeze, (and running the `./breeze` command) you'll be inside the Airflow container. Then, you run:

`airflow db init`
This will create a database for you - by default it will be a mysql one.

Then run:
`airflow users create --role Admin --username <your_username> --email <any_email> --firstname <your_name> --lastname <your_lastname>`

Fill in that command with your information to be able to login to the web view. Once you enter that command, you will get a `Password:` prompt. Pick a password for your user.

Finally run:
`airflow webserver` 

This will initiate the web server. Visit [http://127.0.0.1:28080/](http://127.0.0.1:28080/) and log in! You will see a list of test and example dags.

For now, turn **off** those dags.

Now, we'll have to open another bash terminal inside the airflow container to keep running commands while the webserver is running. In your shell run:

```bash
docker ps
```

You will see a few containers running, copy the `airflow-testing` container. It's the one with a name like `ci_airflow-testing_run_a61fb503e71a` .

Next, run `docker exec -ti ci_airflow-testing_run_a61fb503e71a /bin/bash` to be dropped into a bash shell in the container. Here 

## Your first DAG

The city of Guadalajara publishes a report of the bicycle trips via `MiBici`. Usually in the form of a .csv file like `https://datos.jalisco.gob.mx/sites/default/files/reporte_de_viajes_en_bicicleta_publica_mibici_julio_2019.csv` . 

**Note**: Get ceative with this guide! You're encouraged to try and get the same result using different operators or functions

Let's write a DAG `reporte_bicicletas.py` to help us aggregate the data. It will do the following:

- The DAG is scheduled to run once per month, around the 15th day.
- Uses the python requests library (or curl/wget) to try and download the .csv for the corresponding month.
- If it doesn't exist creates a .csv file `todos_los_viajes.csv` that will aggregate all the .csv file's data
- Appends the month's csv data to the `todos_los_viajes.csv` file.

Before we start, let's create some variables to help ups:
```python
ALL_RIDES_CSV = '/opt/airflow/airflow/example_dags/todos_los_viajes.csv' # so that it's synced with our computer

NUM_TO_MONTH = {'01': 'enero', '02': 'febrero', '03': 'marzo', '04': 'abril', '05':'nayo', '06':'junio', '07':'julio',
    '08':'agosto', '09':'septiembre', '10': 'octubre', '11':'noviembre','12':'diciembre'}
```

Import the airflow operators we can use

```py

import airflow
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
```

Also let's import pandas to help us working with the csv work.
```python
import pandas as pd
```

We'll have two operators. One to make sure that the `todos_los_viajes.csv` exists with the header, and one to download the month's file the 15th day.

### Define the dag

Define a default_args dictionary based on [this documentation](https://airflow.apache.org/tutorial.html#default-arguments) making sure you have the required fields. and then add the dag: 
```py
dag = DAG(
    'reporte_bicicletas',
    default_args=default_args,
    description='Descargar los datos del mes',
    schedule_interval='0 * 15 * *',
```

### Python Operator to aggregate the data
Let's focus on the download operation.

To download each months file, we'll use a PythonOperator, that calls a function to download the file.
The operator will look something like:

```py
t2 = PythonOperator(
    task_id='get_month_rides',
    provide_context=True,
    python_callable=get_month_rides,
    dag=dag
) 
```

And the `get_month_rides` function would look something like:

```python
def get_month_rides(ds, **kwargs):
    year = ds.split('-')[0] # year - month - day
    month_number = ds.split('-')[1] # year - month - day
    month = NUM_TO_MONTH[month_number]

    # download file
    url = 'https://datos.jalisco.gob.mx/sites/default/files/reporte_de_viajes_en_bicicleta_publica_mibici_{month}_{year}.csv'.format(month=month,year=year)
    try:
        data = pd.read_csv(url, encoding = "ISO-8859-1")
        data.to_csv(ALL_RIDES_CSV, mode='a', header=False)
    except Exception as e:
        print(e)
        raise ValueError('data is not available, or an error happened')
```

Next up, try adding the the operator to check that the file exists, and if it doesn't, it creates the  `.csv` file with the correct header. How would you do it? The header looks something like:
```
Viaje_Id  Usuario_Id Genero  Año_de_nacimiento     Inicio_del_viaje        Fin_del_viaje  Origen_Id  Destino_Id
```

Once that's done add `t1 >> t2` at the end of the file, and try running it! You can try running a backfill to get all the available data https://airflow.apache.org/cli.html#backfill

Once you are done, you can run `./scripts/ci/local_ci_stop_environment.sh` locally to stop the containers.

## Challenges
Looking for some challenges? Try some of these:
- Add another column to the .csv file to represent the month.
- Instead of aggregating the data in a .csv file, let's create a table `todos_los_viajes` in our database. There, INSERT the values each month.
- Use the [google-api](https://github.com/googleapis/google-api-python-client) to upload everything in the DB or the .csv file to a google sheets (or just as a file to google drive). Is there an airflow operator you can use?
- If we re-run the DAG for the same month, it will keep on writing to the .csv file multiple times for the same month.÷ Can you add a check to protect against this? 
