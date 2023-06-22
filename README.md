# Real World Project: Database Shard Github

## Objectives:
* Build an app in docker-compose that will set up a sharded database.
* Use a Python script to connect, query, and demonstrate the merged database.
* Provide all necessary files and well-written instructions in a GitHub repository

## The initial prerequisites are to install:
* [Docker](https://docs.docker.com/engine/install/ubuntu/)
* [Docker Compose](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04)
```
sudo apt install docker-compose
```
* Install MySQL Connector
```
sudo apt install python3-pip
pip3 install mysql-connector
```
* Install MariaDB
```
sudo apt install mariadb-client
```
* Update your Ubuntu
```
sudo apt update
sudo apt upgrade
```

## Create Maxscale Container
* Clone the maxscale-docker repository by running the following commands in Ubuntu terminal:
```
git clone https://github.com/zohan/maxscale-docker/
cd maxscale-docker/maxscale
docker-compose up –d
```

## Running
[The MaxScale docker-compose setup](./docker-compose.yml) contains MaxScale
configured with a three node master-slave cluster. To start it, run the
following commands in this directory.

```
docker-compose build
docker-compose up -d
```
* Run maxctrl to check the status of container that now changed to primary1 and primary2.
```
$ docker-compose exec maxscale maxctrl list servers
┌─────────┬──────────┬──────┬─────────────┬─────────────────┬──────────┬─────────────────┐
│ Server  │ Address  │ Port │ Connections │ State           │ GTID     │ Monitor         │
├─────────┼──────────┼──────┼─────────────┼─────────────────┼──────────┼─────────────────┤
│ server1 │ primary1 │ 3306 │ 0           │ Master, Running │ 0-3000-5 │ MariaDB-Monitor │
├─────────┼──────────┼──────┼─────────────┼─────────────────┼──────────┼─────────────────┤
│ server2 │ primary2 │ 3306 │ 0           │ Running         │ 0-3001-5 │ MariaDB-Monitor │
└─────────┴──────────┴──────┴─────────────┴─────────────────┴──────────┴─────────────────┘
```
* You can edit the [maxscale.cnf.d/example.cnf](https://github.com/BASHAR7A7/maxscale-docker/blob/master/maxscale/maxscale.cnf.d/example.cnf)
file and recreate the MaxScale container to change the configuration.

* To connect to mariadb using the username: maxuser, maxpwd as a password and that will be on the port 4000:
```
$ mariadb -umaxuser -pmaxpwd -h 127.0.0.1 -P 4000

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 10.11.3-MariaDB-1:10.11.3+maria~ubuntu-log mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [test]>
```
*NOTE* You can try to pull some data there.

## Running [Python Script](https://github.com/BASHAR7A7/maxscale-docker/blob/master/maxscale/sharding.py)
* The script is being used to remote access the Maxscale Server.
```
import mysql.connector
db = mysql.connector.connect(host="172.21.0.4", port="4000", user="maxuser", password="maxpwd")
cursor = db.cursor()
```
### The python script will perform the following queries, and print the output to the console:

1. The largest zipcode in zipcodes_one
```
print('The largest zipcode in zipcodes_one:')
cursor = db.cursor()
cursor.execute("SELECT Zipcode FROM zipcodes_one.zipcodes_one ORDER BY Zipcode DESC LIMIT 1;")
results = cursor.fetchall()
for result in results:
    print(result)
```
2. All zipcodes where state=KY (Kentucky). You may return just the zipcode column, or all columns.
```
print('All zipcodes where state = KY:')
cursor.execute("SELECT Zipcode FROM zipcodes_one.zipcodes_one WHERE State = 'KY';")
results = cursor.fetchall()
for result in results:
    print(result)
cursor.execute("SELECT Zipcode FROM zipcodes_two.zipcodes_two WHERE State = 'KY';") 
results = cursor.fetchall()
for result in results:
    print(result)
```
3. All zipcodes between 40000 and 41000.
```
print('All zipcodes between 40000 and 41000:')
cursor = db.cursor()
cursor.execute("SELECT Zipcode FROM zipcodes_one.zipcodes_one WHERE zipcode BETWEEN 40000 AND 41000;")
results = cursor.fetchall()
for result in results:
    print(result) 
cursor.execute("SELECT Zipcode FROM zipcodes_two.zipcodes_two WHERE zipcode BETWEEN 40000 AND 41000;")
results = cursor.fetchall()
for result in results:
    print(result)
```
4. The TotalWages column where state=PA (Pennsylvania).
```
print('The TotalWages column where state = PA:')
cursor = db.cursor()
cursor.execute("SELECT TotalWages FROM zipcodes_one.zipcodes_one WHERE state = 'PA';")
results = cursor.fetchall()
for result in results:
    print(result)
cursor.execute("SELECT ALL TotalWages FROM zipcodes_two.zipcodes_two WHERE state = 'PA';")
results = cursor.fetchall()
for result in results:
    print(result)
```

* Once complete, to remove the cluster and maxscale containers:
```
docker-compose down -v
```
## Kudos!!
Thank you Celine for explaining the project and helping with the script.
