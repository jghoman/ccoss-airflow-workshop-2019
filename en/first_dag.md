# Creating your first DAG
Let's create our first dag! Let's make a small DAG that will echo `Hola Mundo`. We'll be using the `airflow/example_dags/tutorial.py`

Let's add new file `hola_mundo.py` in the `airflow/example_dags` folder. In there, we'll create a DAG that runs once per day and echoes `Hola Mundo` using the Bash Operator.

## A more elaborate first DAG

The city of Guadalajara publishes a report of the bicycle trips via `MiBici`. Usually in the form of a .csv file like `https://datos.jalisco.gob.mx/sites/default/files/reporte_de_viajes_en_bicicleta_publica_mibici_julio_2019.csv` . 

Let's write a DAG `reporte_bicicletas` to help us aggregate the data. It will do the following:

- The DAG is scheduled to run once per month, around the 15th day.
- Uses the python requests library (or curl/wget) to try and download the .csv for the corresponding month.
- If it doesn't exist creates a .csv file `todos_los_viajes.csv` that will aggregate all the .csv file's data
- Appends the month's csv data to the `todos_los_viajes.csv` file.


## Challenges
Looking for some challenges? Try some of these:
- Add another column to the .csv file to add the month.
- Instead of aggregating the data in a .csv file, let's create a table `todos_los_viajes` in our database. There, INSERT the values each month.
- Use the [google-api](https://github.com/googleapis/google-api-python-client) to upload everything in the DB or the .csv file to a google sheets (or just as a file to google drive)
- If we re-run the DAG for the same month, it will keep on writing to the .csv file multiple times for the same month.÷ Can you add a check to protect against this? 