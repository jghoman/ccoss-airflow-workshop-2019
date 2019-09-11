# Setting up your Airflow developer environment

We'll be following the official Airflow [contributing](https://github.com/apache/airflow/blob/master/CONTRIBUTING.md) guide, using [Airflow Breeze](https://github.com/apache/airflow/blob/master/BREEZE.rst) to develop.

## Pre-reqs for all platforms:

- Git & a Github account.
  - We'll be using GitHub to open Pull Requests to do our contributions
  - Make sure you have Git installed in your machine, and that your GitHub account is linked locally. This, so that you can clone repositories and push code.
  
- Clone / Fork Airflow:
  - Head over to https://github.com/apache/airflow and Click on the `Fork` icon on the top. This will create a fork of Airflow in your own account, which is needed to create contribution's Pull Requests. 
  - From here you can then clone your fork of github in your computer via `git clone git@github.com:<your_username>/airflow.git` or `git clone 
https://github.com/<your_username>/airflow.git`.


- Docker & Docker Compose
  - Follow the [Docker](https://docs.docker.com/install/) and the [Docker-Compose](https://docs.docker.com/compose/install/)  installation guides for your platform 
  - If you are in macOS, please increase the Docker disk space following [this guide](https://docs.docker.com/docker-for-mac/space/).

## macOS / Unix / Linux Setup

The last packages you need to install are gnu getopt and gstat. If you are in macOS, and have homebrew installed (if not, please install it following these [instructions]()) you can run `brew install gnu-getopt coreutils` Finally, you will add the package folder to your path via: 
```
echo 'export PATH="/usr/local/opt/gnu-getopt/bin:$PATH"' >> ~ .bash_profile
. ~/.bash_profile
```

If you are running a **Linux** distro please install it via `apt install util-linux coreutils` , and don't forget to add the folder to your PATH.


Before running any breeze commands, make sure that you have Docker running. This can be done in macOS by opening up the Docker app from the Applications folder. You can also start it via the terminal.

After cloning your Airflow fork, head over to that new folder, usually via  `cd airflow`, and run `./breeze` . This will initialize an airflow breeze environment. Since this will download some docker images, the first run will take some time. 

After the `./breeze` command is finished, you will be all set up, and it will drop you in a shell inside the airflow container.


## Windows Setup

### TODO: how to set up Linux subsystem for Windows 10. Else, set up a Docker container with centOS/ubuntu and follow the same guidelines to run breeze inside it.


## Your first DAG

### Setting up the webserver 
After setting up Airflow breeze, and running the `./breeze` command you'll be inside the Airflow container. Here you run:

`airflow db init`
This will create a database for you, by default it will be a mysql one.

Then:
`airflow users create --role Admin --username <your_username> --email <any_email> --firstname <your_name> --lastname <your_lastname>`

Fill in that command with your information to be able to login to the web view. Once you enter that command, you will get a `Password:` prompt. Pick a password for your user.

Finally:
`airflow webserver` 

This will initiate the web server. Visit [https://127.0.0.1:28080/](https://127.0.0.1:28080/) and log in! You will see a list of test and example dags. 

Now, we'll have to open another bash terminal inside the airflow container to keep running commands while the webserver is running. In your shell run:

```
docker ps
```

You will see a few containers running, copy the `airflow-testing` container. It's the one with a name like `ci_airflow-testing_run_a61fb503e71a` .

Next, run `docker exec -ti ci_airflow-testing_run_a61fb503e71a /bin/bash` to be dropped into a bash shell in the container.


### Creating your first DAG 
Let's create our first dag! Let's make a small DAG that will echo `Hola Mundo`. We'll be using the `airflow/example_dags/tutorial.py`

Let's add new file `hola_mundo.py` in the `airflow/example_dags` folder. In there, we'll create a DAG that runs once per day and echoes `Hola Mundo` using the Bash Operator.

### A more elaborate first DAG 

The city of Guadalajara publishes a report of the bicycle trips via `MiBici`. Usually in the form of a .csv file like `https://datos.jalisco.gob.mx/sites/default/files/reporte_de_viajes_en_bicicleta_publica_mibici_julio_2019.csv` . 

Let's write a DAG `reporte_bicicletas` to help us aggregate the data. It will do the following:

- The DAG is scheduled to run once per month, around the 15th day.
- Uses the python requests library (or curl/wget) to try and download the .csv for the corresponding month.
- If it doesn't exist creates a .csv file `todos_los_viajes.csv` that will aggregate all the .csv file's data
- Appends the month's csv data to the `todos_los_viajes.csv` file.


#### Challenge 
Looking for some challenges? Try some of these:
- Add another column to the .csv file to add the month.
- Instead of aggregating the data in a .csv file, let's create a table `todos_los_viajes` in our database. There, INSERT the values each month.
- Use the [google-api](https://github.com/googleapis/google-api-python-client) to upload everything in the DB or the .csv file to a google sheets (or just as a file to google drive)
- If we re-run the DAG for the same month, it will keep on writing to the .csv file multiple times for the same month. Can you add a check to protect against this? 