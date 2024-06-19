# Azure Database for PostgreSQL Backup and Restore by  point-in-time recovery (PITR)
## 1. Create Database Instance
```sh
# Variables
RESOURCE_GROUP="abacus-poc-jib-rg"
SERVER_NAME="abacus-poc"
NEW_SERVER_NAME="abacus-new-poc"
ADMIN_USER="pgadmin"
ADMIN_PASSWORD="pgadmin"
DB_NAME="demo_db"
TABLE_NAME="demo_table"

# Create PostgreSQL Flexible Server
az postgres flexible-server create \
  --resource-group $RESOURCE_GROUP \
  --name $SERVER_NAME \
  --admin-user $ADMIN_USER \
  --admin-password $ADMIN_PASSWORD \
  --public-access All
```


## 2. Connect to PostgreSQL and Setup new database and Create Table,Data on Default schema(public)
```sh
psql "host=$SERVER_NAME.postgres.database.azure.com dbname=postgres user=$ADMIN_USER password=$ADMIN_PASSWORD sslmode=require" << EOF
CREATE DATABASE $DB_NAME;
\c $DB_NAME
CREATE TABLE $TABLE_NAME (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);
INSERT INTO $TABLE_NAME (name) VALUES ('Alice'), ('Bob'), ('Carol');
EOF
psql "host=$SERVER_NAME.postgres.database.azure.com dbname=demo_db user=$ADMIN_USER password=$ADMIN_PASSWORD sslmode=require" -c "SELECT * FROM $TABLE_NAME;"
```



## 3. Test Restore to new Instance by using current time 
```sh
az postgres flexible-server restore \
    --resource-group $RESOURCE_GROUP \
    --name $NEW_SERVER_NAME \
    --source-server $SERVER_NAME;
az postgres flexible-server firewall-rule create \
    --resource-group $RESOURCE_GROUP \
    --name $NEW_SERVER_NAME \
    --rule-name AllowAllIPs \
    --start-ip-address 0.0.0.0 \
    --end-ip-address 255.255.255.255
```

### 4. Test Query Data #Shoud have data like original server
```sh
psql "host=$NEW_SERVER_NAME.postgres.database.azure.com dbname=demo_db user=$ADMIN_USER password=$ADMIN_PASSWORD sslmode=require" -c "SELECT * FROM $TABLE_NAME;"
```

## Cleansing
```sh
az postgres flexible-server delete \
  --resource-group $RESOURCE_GROUP \
  --name $SERVER_NAME \
  --yes && \
az postgres flexible-server delete \
  --resource-group $RESOURCE_GROUP \
  --name $NEW_SERVER_NAME \
  --yes
```