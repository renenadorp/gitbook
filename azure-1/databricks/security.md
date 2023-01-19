# Security

## Using Databricks Secrets for Azure SQL DB credentials

&#x20;This section describes how to use Databricks secrets for storing usernames & passwords to access a SQL DB instance from DataBricks. &#x20;

### References

* [https://docs.microsoft.com/en-us/azure/key-vault/quick-create-portal](https://docs.microsoft.com/en-us/azure/key-vault/quick-create-portal)
* [https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html)
* [https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html)

### Intro

There are 2 methods for using secrets within DataBricks

1. Azure Key Vault-backed secret scope
2. DataBricks-backed secret scope

This section describes the DataBricks backed secret scope, because the Azure Key Vault backed method is in Public Review (Aug 2018)&#x20;

### Step 1 - Install Databricks CLI

The workaround for the error above is to use the databricks cli to create the secret scope. In order to be able to use the databricks CLI it will need to be installed first. On your local machine enter the following command (assuming python 3 is used):&#x20;

```bash
pip3 install databricks-cli
```

### Step 2 - Setup Databricks CLI Authentication

Then authentication needs to be setup. From your Databricks workspace, click the user icon (top right) and click "User Settings".In the User Settings panel click "Generate New Token"

![](https://northeurope1-mediap.svc.ms/transform/thumbnail?provider=spo\&inputFormat=png\&cs=MWZlYzhlNzgtYmNlNC00YWFmLWFiMWItNTQ1MWNjMzg3MjY0fFNQTw\&correlationId=ddf5b19e-8002-7000-a44a-7c67df757861\&docid=https%3A%2F%2Finergyasbv%2Esharepoint%2Ecom%2F%5Fapi%2Fv2%2E0%2Fdrives%2Fb%21%5FArBE%5FM1iU6HX4GnJ12ZSFjXYNK2VatCtDfBV4tL0b19FUlIjSpqSaEpoVIltGA4%2Fitems%2F01ZQC2FZVCITRCDQYIJ5DJLBRTTFBMIRKY%3Ftempauth%3DeyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0%2EeyJhdWQiOiIwMDAwMDAwMy0wMDAwLTBmZjEtY2UwMC0wMDAwMDAwMDAwMDAvaW5lcmd5YXNidi5zaGFyZXBvaW50LmNvbUBmYjU3MTA3NC04NTBjLTQ0NDQtYjQ4Mi1kYWY2ZGU5ZjEyNGYiLCJpc3MiOiIwMDAwMDAwMy0wMDAwLTBmZjEtY2UwMC0wMDAwMDAwMDAwMDAiLCJuYmYiOiIxNTQ2NDA5NTQwIiwiZXhwIjoiMTU0NjQzMTE0MCIsImVuZHBvaW50dXJsIjoib3lVekpiaGJsSDF2RGFQS2JjY3BFU0lRUnNqb1lkb01mbFF2bXdKY3loUT0iLCJlbmRwb2ludHVybExlbmd0aCI6IjE1OCIsImlzbG9vcGJhY2siOiJUcnVlIiwiY2lkIjoiWkdSbU5XSXhPV1V0T0RBd01pMDNNREF3TFdFME5HRXROMk0yTjJSbU56VTNPRFl4IiwidmVyIjoiaGFzaGVkcHJvb2Z0b2tlbiIsInNpdGVpZCI6Ik1UTmpNVEJoWm1NdE16Vm1NeTAwWlRnNUxUZzNOV1l0T0RGaE56STNOV1E1T1RRNCIsImFwcF9kaXNwbGF5bmFtZSI6Ik1pY3Jvc29mdCBUZWFtcyIsImFwcGlkIjoiMWZlYzhlNzgtYmNlNC00YWFmLWFiMWItNTQ1MWNjMzg3MjY0IiwidGlkIjoiZmI1NzEwNzQtODUwYy00NDQ0LWI0ODItZGFmNmRlOWYxMjRmIiwidXBuIjoicmVuZS5uYWRvcnBAaW5lcmd5Lm5sIiwicHVpZCI6IjEwMDMwMDAwQTVERTFDQUEiLCJzY3AiOiJteWZpbGVzLndyaXRlIGFsbHNpdGVzLmZ1bGxjb250cm9sIGFsbHNpdGVzLm1hbmFnZSBhbGxwcm9maWxlcy53cml0ZSIsInR0IjoiMiIsInVzZVBlcnNpc3RlbnRDb29raWUiOm51bGx9%2EaTdhdnYvRnE2NlZJWmJXblNEcjBiOGlLZU1lVXE2bnd5Rndnd1RuaFhZZz0%26version%3DPublished\&width=800\&height=800)&#x20;

**Make a note the value for the token generated!!! It will be displayed only once!** The next step is to configure authentication for the databricks cli, by entering the following command from a terminal window on your client:&#x20;

```bash
databricks configure --token
```

&#x20;This will prompt for parameters:

```bash
host: https://westeurope.azuredatabricks.net
token: <the token just created and hopefully noted>
```

This operation will create a file called **/Users/\<user>/.databrickscfg** on your local machine with the provided information&#x20;

### Step 3 - Create Secret Scope

After that create a secret scope with the following command:

```
databricks secrets create-scope --scope jdbc --initial-manage-principal "users"
```

&#x20;This will create the secret scope called "jdbc". Verify creation of the secret scope by issuing the following command

```
databricks secrets list-scopes
```

### Step 4 - Create Secrets

Create secrets by issuing the following commands (scope, key and value are given in the example below. provide parameters as required)

```
databricks secrets put --scope jdbc --key lift2shiftjdbcuser
```

Enter the secret value in the editor, then save and exit.

```
databricks secrets put --scope jdbc --key lift2shiftjdbcpassword
```

&#x20;Enter the secret value in the editor, then save and exit. The dbuser/password secrets will normally be the database admin user/password, which is set in the Azure Portal on the SQL Server level (so not on database level!) Verify secrets have been created with the following command:

```
databricks secrets list --scope jdbc
```

### Step 5 - Use Secrets&#x20;

Secrets can be accessed in a Spark Notebook (Python) as follows:

```python
jdbcUsername = dbutils.secrets.get(scope = "jdbc", key = "lift2shiftjdbcuser")
jdbcPassword = dbutils.secrets.get(scope = "jdbc", key = "lift2shiftjdbcpassword")

```

## Using Azure Key Vault for SQL DB Authentication from DataFactory

This section describes how to use Azure Key Vault secrets to authenticate DataFactory for SQL DB access &#x20;

### References

* [https://docs.microsoft.com/en-us/azure/data-factory/store-credentials-in-key-vault](https://docs.microsoft.com/en-us/azure/data-factory/store-credentials-in-key-vault)
* [https://docs.microsoft.com/en-us/azure/data-factory/data-factory-service-identity](https://docs.microsoft.com/en-us/azure/data-factory/data-factory-service-identity)

### Pre Requisites

* Data Factory Service
* SQL DB Service&#x20;
* Azure Key Vault Service

### Step 1: Copy SQL DB Connection String

* Go to the Azure Portal, select your SQL Database
* Click the link under "Connection Strings" (top right of the pane)
* Select "JDBC"-tab (top of the pane)
* Click the copy icon (next to the connection string)
*

### Step 2: Create a secret in Azure Key Vault

* Go to the Azure Portal, select your Azure Key Vault
* Go to "Secrets", then click "Generate/Import" on the top of the pane.
* Enter a name for the secret&#x20;
* Copy the value obtained from the SQL DB Connection strings (previous step) in the value field
* Click "Create"

### Step 3: Create a Azure Key Vault Linked Service

* Go to the Azure Portal, Select your Data Factory service
* Click "Author & Monitor"
* A new browser window will open for the selected DataFactory service
* Click "Connections" (bottom left)
* Click "New" (top left)
* Select the "Azure Key Vault" icon, then click "Continue"
* Name: Provide a name&#x20;
* Selection Method: Select "From Azure subscription"
* Subscription: select the relevant one
* Azure Key vault name: select the relevant one
* Click "Finish"

### Step 4: Create a DataFactory Linked Service

* Go to the Azure Portal, Select your Data Factory service
* Click "Author & Monitor"
* A new browser window will open for the selected DataFactory service
* Click "Connections" (bottom left)
* Click "New" (top left)
* Select the "Azure SQL Database" icon, then click "Continue"
* Provide a unique name for the linked service
* Select "Azure Key Vault"
* AKV linked service: select relevant one (created in previous step)
* Secret name: enter the name of the secret created in step 2
* Secret version: \<blank>
* Authentication type: SQL Authentication
* Click "Test Connection". Should say: "Connection successful"
* Click Finish.

### Step 5: Publish Changes

* Go to the main page of your Data Factory service
* Click "Publish All" on the top of the page

&#x20;The Azure Data Factory linked service is now ready for use in a Pipeline. &#x20;

## Retrieve Key Vault Secrets with Python Script

To retrieve secrets from Key Vault with a python script, take following steps.&#x20;

### Step 1: Create application and register in AAD

The system admin for the subscription needs to create an application that will be used to access Key Vault. This application will need to be registered in AAD.&#x20;

### Step 2: Authorise user for Key Vault

The user created in step 1 needs to be authorised in Key Vault. Go to Azure Portal / Key Vault / Access Policies. Click "+ Add new" and add the created application as a service principal, and grant access to Keys/Secrets/Certificates:![](blob:https://teams.microsoft.com/28bdb988-a097-40f1-9869-c9c828b5b0b2) Example:![](blob:https://teams.microsoft.com/cca6ed04-3a02-49e4-b596-f346538c0916) In the example the application "AzureKeyVaultSecretPY" was added to the list of authorised id's&#x20;

### Step 3: Note credentials

The python-script requires the following credentials

* tenant\_id: this is the AAD id of the Azure subscription
* application\_id: the AAD of the authorised user to access Key Vault
* secret: the access key of the authorised user&#x20;
* secret\_name: the name of the secret to be retrieved
* secret\_version: version of the secret to be retrieved. Leave blank for latest
* vault\_url: the url of the Azure Key Vault

### Step 4: Run Python Script

Run the script below. Note that this script requires the **azure.common.credentials** library.

```python
import os
from azure.common.credentials import ServicePrincipalCredentials

from azure.keyvault import KeyVaultClient

#tenant_id = os.environ['tenantid'] 
tenant_id = '**** TENANT_ID GOES HERE ****'
application_id = '**** APPLICATION_ID GOES HERE ****'
access_key = '**** ACCESS KEY OF APPLICATION_ID GOES HERE ****'
secret_name = '**** NAME OF SECRET TO BE RETRIEVED GOES HERE ****'
secret_version = '' 
vault_url = '**** AZURE KEY VAULT URL GOES HERE ****'


credentials = ServicePrincipalCredentials(

 client_id = application_id,
 secret = access_key,
 tenant = tenant_id

)


client = KeyVaultClient(credentials)


secret = client.get_secret(vault_url, secret_name, secret_version).value 

print(secret)
```

## SSH

### Configuration

#### Step1: Create public/private key pair on local machine

```bash
$ ssh-keygen
```

This command will create two files, usually in $HOME/.ssh:

* id\_rsa
* id\_rsa.pub

The file "id\_rsa.pub" contains the public key and can be shared with anyone. The other file contains the private key, and is never to be shared.

