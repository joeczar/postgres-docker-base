# Configure Docker-Compose
We will configure Docker-Compose to use PostgreSQL by creating this docker-compose.yml:
```
# docker-compose.yml
version: '3'
services:
  database:
    image: "postgres" # use latest official postgres version
    env_file:
      - database.env # configure postgres
    volumes:
      - database-data:/var/lib/postgresql/data/ # persist data even if container shuts down
volumes:
  database-data: # named volumes can be managed easier using docker-compose
```
The database.env file looks like this:
```
# database.env
POSTGRES_USER=unicorn_user
POSTGRES_PASSWORD=magical_password
POSTGRES_DB=rainbow_database
```

## Log In to PostgreSQL

Start the Database
Run `docker-compose up` to bring up the database.
```
$ docker-compose up                             
Creating volume "postgresql-with-docker-compose_database-data" with default driver
Pulling database (postgres:)...
latest: Pulling from library/postgres
8d691f585fa8: Pull complete
...                                                                             
6283090fa09d: Pull complete
Digest: sha256:a4a944788084a92bcaff6180833428f17cceb610e43c828b3a42345b33a608a7                                                                                                                                     
Status: Downloaded newer image for postgres:latest                                                        
Creating postgresql-with-docker-compose_database_1 ... done                                  
Attaching to postgresql-with-docker-compose_database_1                                                                                                                                                              
database_1  | The files belonging to this database system will be owned by user "postgres".                                                                                                                         
database_1  | This user must also own the server process.
...
database_1  | 2019-11-17 20:33:02.208 UTC [1] LOG:  database system is ready to accept connections
```
Docker-Compose will only download the image the first time you run this command.

## Connect to the Database

There are multiple ways of connecting to the database container. In this example, we will drop into the database container and use the psql client software that is already installed in the database container.
```
$ docker-compose run database bash # drop into the container shell
database# psql --host=database --username=unicorn_user --dbname=rainbow_database
Password for user unicorn_user: 
psql (12.0 (Debian 12.0-2.pgdg100+1))
Type "help" for help.
rainbow_database=#
```
When prompted to the password, enter the password we configured in our `docker-compose.yml`, e.g. `magical_password`.
## Create a Table
We can start interacting with the database by first creating a database table.

```
rainbow_database=# \d # verify table does already not exist
Did not find any relations.
rainbow_database=# CREATE TABLE color_table(name TEXT);
CREATE TABLE
rainbow_database=# \d # verify table is created
              List of relations
 Schema |    Name     | Type  |    Owner     
--------+-------------+-------+--------------
 public | color_table | table | unicorn_user
(1 row)
```

## Add and Read Data
We can now add data in to the table. And then read data back from the table.
```
rainbow_database=# SELECT * FROM color_table; -- verify record does not already exist
 name 
------
(0 rows)
rainbow_database=# INSERT INTO color_table VALUES ('pink'); -- be sure to use single quotes
INSERT 0 1
rainbow_database=# SELECT * FROM color_table; -- verify record is created
 name 
------
 pink
(1 row)
```
# Conclusion
Congratulations! You have successfully used a PostgreSQL database inside a container with Docker-Compose.
If you found this tutorial article useful and have other technologies that you are interested in learning how to get started with, please submit your ideas to https://gitlab.com/zhao-li/tutorial-articles.
Thank you for your time 🙏

# Notes
It is worth noting some nuances with persisting data with containers.
Using Named Volumes
In our example, we used `named volumes`. Docker-Compose helps us manage creating and destroying these volumes. These volumes allow the data to persist even if we destroy the containers.
```
# docker-compose.yml
services:
  database:
    ...
    volumes:
      - database-data:/var/lib/postgresql/data/
volumes:
  database-data:
```
To tell Docker-Compose to destroy the volume and its data, you need to issue docker-compose down --volumes
## Without Volumes
If we removed these volume configurations, each time we destroy our container, any data we stored will be gone. Create some data and give it a try.
Using Host Volumes
If we used the following configuration:
```
# docker-compose.yml
services:
  database:
    ...
    volumes:
      - ./host-folder/:/var/lib/postgresql/data/
```

The data will be stored on the host computer. To delete this data and start a fresh new database, you will have to manually remove the data files from the host computer with something like rm -rf ./host-folder/.
