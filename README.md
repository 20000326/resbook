# resbook

maxshoppe

Step1:
teps to build an push docker containers to container registry

docker build -t resbook:v1 .

create Container registry

az acr login --name resbookRN

docker tag resbook:v1 resbookrn.azurecr.io/resbook:v1

docker push resbookrn.azurecr.io/resbook:v1

Deploy using Web app of the image publish or up the container

container with ip and port https://resbook.azurewebsites.net/

Database connection for resbook

import the mssql database from azure latest

docker pull mcr.microsoft.com/mssql/server:2022-latest

create container with same registry as resbook

declare variables in advance settings of the repo image

{ "name": "DB_HOST", "value": "db_ip", "slotSetting": false }, { "name": "DB_NAME", "value": "mydb", "slotSetting": false }, { "name": "DB_PASSWORD", "value": "Admin@12345", "slotSetting": false }, { "name": "DB_USER", "value": "sa", "slotSetting": false },

and publish and create instance from the image

