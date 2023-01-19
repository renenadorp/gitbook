# DataBricks CLI

## Install cli

Command:

```
pip install databricks-cli
```

Requires:&#x20;

* Python
* Pip

## Configure

To use the databricks-cli, a token has to be configured. The token needs to be created first from the databricks workspace:

![Databricks Workspace User](<../../.gitbook/assets/image (32).png>)

Click User Settings, then click "Generate New Token"

![Generate Token for Databricks](<../../.gitbook/assets/image (13).png>)

The value for the token will be presented only once! Please save that value.&#x20;

In order to configure a client to access a databricks workspace, run the following command from that client.

```
databricks configure --token
```

You will be prompted for&#x20;

* databricks host: https://westeurope.azuredatabricks.net
* token: \<token value generated>

## databricks-cli in azure cloudshell

See [https://docs.microsoft.com/nl-nl/azure/azure-databricks/databricks-cli-from-azure-cloud-shell](https://docs.microsoft.com/nl-nl/azure/azure-databricks/databricks-cli-from-azure-cloud-shell)
