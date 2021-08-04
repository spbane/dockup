## Implementation steps followed for testing Dockup (Docker Backup)

### Create initial docker volumes
```
docker volume create mysql
docker volume create mysql_config
```
### Create network for inter communications
```
docker network create mysqlnet
```
### Start MYSQL Database by attaching Docker Volumes
```
docker run --rm -d -v mysql:/var/lib/mysql \
-v mysql_config:/etc/mysql -p 3306:3306 \
--network mysqlnet \
--name mysqldb \
-e MYSQL_ROOT_PASSWORD=p@ssw0rd1 \
mysql
```
> Now we use this setup for sometime and store data in it
---------------------
## Backup Activity
> To ensure that a regular backup is running for containers and volumes, we would be using below commands
### Backup of containers
```
docker commit -p mysqldb mysqldb-backup01
docker tag mysqldb-backup01 mysqldb-backup01:latest
docker push mysqldb-backup01:latest
```

### Backup of Volumes
#### Create dockup image
```
docker build -t dockup:latest .
```

#### Add env.txt with below content
```
AWS_ACCESS_KEY_ID=<key_here>
AWS_SECRET_ACCESS_KEY=<secret_here>
AWS_DEFAULT_REGION=us-east-1
BACKUP_NAME=mysql
PATHS_TO_BACKUP=/etc/mysql /var/lib/mysql
S3_BUCKET_NAME=docker-backups-example-com
RESTORE=false
```
#### Take backup
```
docker run --rm \
--env-file env.txt \
--volumes-from mysqldb \
--name dockup dockup:latest
```
---------------------------
# CATASTROPHE HAPPENS !!

---------------------------

## Restore begins

### Recreate network and volumes
```
docker network create mysqlnet
docker volume create mysql
docker volume create mysql_config
```
### Start blank database
```
docker run --rm -d -v mysql:/var/lib/mysql \
-v mysql_config:/etc/mysql -p 3306:3306 \
--network mysqlnet \
--name mysqldb \
-e MYSQL_ROOT_PASSWORD=p@ssw0rd1 \
mysql
```

### Initiate restore by enabling restore flag in env.txt
> change RESTORE=true in env.txt
```
AWS_ACCESS_KEY_ID=<key_here>
AWS_SECRET_ACCESS_KEY=<secret_here>
AWS_DEFAULT_REGION=us-east-1
BACKUP_NAME=mysql
PATHS_TO_BACKUP=/etc/mysql /var/lib/mysql
S3_BUCKET_NAME=docker-backups-example-com
RESTORE=true
```
### Execute restore
```
docker run --rm \
--env-file env.txt \
--volumes-from mysqldb \
--name dockup dockup:latest
```

### Restart database
```
docker restart mysqldb
```
