# Local analytics platform

This instruction provides some steps to set up your own mini analytics platform.  
Docker is used for this purpouses.  
This version is mostly for local development with basic settings and does not provide creation of users/roles etc.  
We'll create local data base (PostgreSQL/Clickhouse) and connect it to visualiser/interfase (Apache Superset).  

- back ups for apps:

```bash
docker volume create postgres_vol
docker volume create clickhouse_vol
```

- network between created apps:

```bash
docker network create app_net
```

- postgres:

```bash
docker run --rm -d \
--name postgres \
-e POSTGRES_PASSWORD=admin \
-e POSTGRES_USER=admin \
-e POSTGRES_DB=db_for_tests \
-v postgres_vol:/var/lib/postgresql/data \
--net=app_net \
postgres:14
```

- superset

```bash
docker run --rm -d -p 80:8088 \
--name superset \
-e "SUPERSET_SECRET_KEY=my_secret_key_is_42" \
--net=app_net \
apache/superset

docker exec -it superset superset fab create-admin \
--username admin \
--password admin \
--firstname Superset \
--lastname Admin \
--email admin@superset.com 

docker exec -it superset superset db upgrade
docker exec -it superset superset load_examples
docker exec -it superset superset init
```

creds to connect postgres to superset: PORT = 5432, HOST = postgres_container_name, DB = db_for_tests

- clickhouse

```bash
docker run --rm -d \
--name clickhouse \
-v clickhouse_vol:/var/lib/clickhouse \
--net=app_net \
clickhouse/clickhouse-server
```

- superset-clickhouse

```bash
docker exec superset pip install psycopg2
docker exec superset pip install clickhouse-connect
docker restart superset
```

uri for clickhouse: clickhousedb://clickhouse/default

## useful commands

docker stop postgres superset

docker ps -a

docker volume ls

docker volume prune
