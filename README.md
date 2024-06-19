# Azure Database for PostgreSQL Backup and Restore
## 1. Create Database
```sh
# Variables
RESOURCE_GROUP="abacus-poc-jib-rg"
SERVER_NAME="abacus-poc"
NEW_SERVER_NAME=" abacus-new-poc"
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
  --location $LOCATION
```

## 2. Cleansing
```sh
az postgres flexible-server delete \
  --resource-group $RESOURCE_GROUP \
  --name $SERVER_NAME \
  --yes
```