# Libraries

## Libraries API

### Install library

Api: [https://docs.databricks.com/api/latest/libraries.html#install](https://docs.databricks.com/api/latest/libraries.html#install)

The script below can be used to install a library on an existing spark cluster

In this case the library "spark-testing-base" is installed from a known repo (not specified)

```python
import requests
import base64

DOMAIN = 'westeurope.azuredatabricks.net'

TOKEN = b'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX' #Token for workspace
response = requests.post(
  'https://%s/api/2.0/libraries/install' % (DOMAIN),
  headers={'Authorization': b"Basic " + base64.standard_b64encode(b"token:" + TOKEN)},
  json={
  "cluster_id": "XXXX-XXXXX-XXXXXXX",
  "libraries": [{
      "pypi": {
        "package": "spark-testing-base"
      }
    }]
}
  
)
if response.status_code == 200:
  print(response.json())
else:
  print("Error installing library on cluster: %s: %s" % (response.json()["error_code"], response.json()["message"]))  
```
