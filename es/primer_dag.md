# Tu primer DAG

## Preparando el webserver

Felicidades, tienes Airflow Breeze funcionando. Ahora hay configurar y correr Airflow.

Después de instalar breeze, y correr `./breeze` estaras, dentro de la terminal en el contenedor. Aquí puedes correr:
`airflow db init`
Esto va a crear una base de datos para ti - predeterminadamente es una de mysql.

Después corre:
`airflow users create --role Admin --username <tu_usuario> --email <un_correo> --firstname <tu_nombre> --lastname <apellido>`

Con tu información para poder entrar a la vista web. Veras un prompt de `Password:`. Elige uno para tu usuario.

Finalmente corre:
`airflow webserver`

Esto iniciará un web server. Visita [https://127.0.0.1:28080/](https://127.0.0.1:28080/) e inicia sesion! Veras una lista de DAGs de prueba y de ejemplo.

Ahora, hay que abrir otra terminal en el contenedor para correr mas comandos mientras el webserver corre.

Localmente, en una terminal corre:

```bash
docker ps
```

Verás unos contenedores corriendo, copia el nombre del contenedor de `airflow-testing`. Tendrá un nombre como `ci_airflow-testing_run_a61fb503e71a` .

Después, corre `docker exec -ti ci_airflow-testing_run_a61fb503e71a /bin/bash` (cambiando el nombre del contenedor) para estar en otra terminal dentro del contenedor.

## Your first DAG

Guadalajara publica un reporte mensual de los viajes en bicicleta hechos vía MiBici. Usualmente en un .csv en una url como `https://datos.jalisco.gob.mx/sites/default/files/reporte_de_viajes_en_bicicleta_publica_mibici_julio_2019.csv` . 

**Nota**: Trata de cambiar y ser creativo con esta guia! Trata de tener el mismo resultado con diferentes operadores o funciones. 

Crearemos una DAG en el archivo `reporte_bicicletas.py` para ayudarnos a juntar todos los datos. Hará lo siguiente:

- La DAG corre mensualmente, por el 15 de cada mes.
- Si no existe, crea un archivo .csv `todos_los_viajes.csv` para agregar los datos - usando el mismo header de las descargas.
- Usa alguna librería de python (o curl/wget en Bash) para descargar el .csv del mes correspondiente
- Agrega los datos al final del archivo `todos_los_viajes.csv` .

Antes de empezar, hay que crear unas variables para ayudarnos:
```python
ALL_RIDES_CSV = 'todos_los_viajes.csv'

NUM_TO_MONTH = {'01': 'enero', '02': 'febrero', '03': 'marzo', '04': 'abril', '05':'nayo', '06':'junio', '07':'julio',
    '08':'agosto', '09':'septiembre', '10': 'octubre', '11':'noviembre','12':'diciembre'}
```

Ahora importa las funciones necesarias para correr la DAG:

```py

import airflow
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
```

Tambien, podemos importar pandas para hacernos la vida más facil con los csv:

```python
import pandas as pd
```

Tendremos dos operadores. Uno para asegurarse que `todos_los_viajes.csv` existe con el header indicado, y otro para descargar los datos.

### Definiendo nuestra DAG

Define un diccionario `default_args` [con esta documentación](https://airflow.apache.org/tutorial.html#default-arguments) y agrega la DAG: 
```py
dag = DAG(
    'reporte_bicicletas',
    default_args=default_args,
    description='Descargar los datos del mes',
    schedule_interval='0 * 15 * *',
```

### Python Operator para juntar los datos

Para bajar los datos de cada mes, usaremos un PythonOperator que los descargue. El operador se ve algo así:

```py
t2 = PythonOperator(
    task_id='get_month_rides',
    provide_context=True,
    python_callable=get_month_rides,
    dag=dag
)
```

Y la función `get_month_rides` se ve algo como:

```python
def get_month_rides(ds, **kwargs):
    ## ds == year - month - day
    year = ds.split['-'][0] # year - month - day
    month_number = ds.split('-')[1]
    month = NUM_TO_MONTH[month_number] # convierte de numero -> nombre del mes

    # descarga los datos
    url = 'https://datos.jalisco.gob.mx/sites/default/files/reporte_de_viajes_en_bicicleta_publica_mibici_{month}_{year}.csv'.format(month=month,year=year)
    try:
        data = pd.read_csv(url, encoding = "ISO-8859-1")
        data.to_csv(ALL_RIDES_CSV, mode='a', header=False)
    except Exception as e:
        print(e)
        raise ValueError('data is not available, or an error happened')
```


Finalmente trata de agregar el operador para ver si existe el archivo, y si no, lo crea con el header apropiado. Como lo harias? El header se ve algo como:
```
Viaje_Id  Usuario_Id Genero  Año_de_nacimiento     Inicio_del_viaje        Fin_del_viaje  Origen_Id  Destino_Id
```

Ya que hagamos eso, si llamamos a ese operador `t1` , agreguemos `t1 >> t2` al final del archivo. Trata correrlo! Puedes hacer un backfill para tener todos los datos en un rango de fechas. https://airflow.apache.org/cli.html#backfill

## Desafios
Buscas mas desafios? Trata uno de los siguientes:

- Agrega otra columna al .csv para representar el mes
- En vez de agregar todo a un .csv, crea una tabla `todos_los_viajes` en nuestra base de datos. Ahí, `INSERT` los valores de cada mes.
- Usa [google-api](https://github.com/googleapis/google-api-python-client) para subir todo lo de nuestra base de datos, o en el .csv a una Google Sheets ( o como un archivo a Drive). Hay algún operador de Airflow que puedas usar?
- Si corremos la DAG dos veces en el mismo mes, estaríamos agregando los datos repetidos para el mismo mes. Puedes agregar alguna protección para esto?

