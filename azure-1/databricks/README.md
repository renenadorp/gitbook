---
description: Information about databricks
---

# DataBricks

## databricks-cli in azure cloudshell

See [https://docs.microsoft.com/nl-nl/azure/azure-databricks/databricks-cli-from-azure-cloud-shell](https://docs.microsoft.com/nl-nl/azure/azure-databricks/databricks-cli-from-azure-cloud-shell)

## Databricks Jobs API: Run Now python script

The script below is fully functional, except:

* it requires a databricks workspace token
* it requires a databricks job id, which can be created in the databricks workspace UI

```python
import requests
import base64
import time

DOMAIN = 'westeurope.azuredatabricks.net'
TOKEN = b'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' #Token of the Databricks workspace

RETRY_MAX = 100
SLEEP = 2

#job_id (hard coded here) is the id of the databricks job create in the UI
#Start a run for the job
response = requests.post(
  'https://%s/api/2.0/jobs/run-now' % (DOMAIN),
  headers={'Authorization': b"Basic " + base64.standard_b64encode(b"token:" + TOKEN)},
  json={
        "job_id": nnn
      }
  )

if response.status_code == 200:
  #print(response.json()['run_id'])
  #print(response.json()['number_in_job'])
  RUN_ID = response.json()['run_id']

  #Get run info until job terminates
  response_life_cycle_state =''
  response_result_state =''
  response_state_message =''
  i=0
  while (i< RETRY_MAX and response_life_cycle_state != 'TERMINATED'):
    i+=1
    response = requests.get(
      'https://{0}/api/2.0/jobs/runs/get-output?run_id={1}'.format(DOMAIN, RUN_ID),
      headers={'Authorization': b"Basic " + base64.standard_b64encode(b"token:" + TOKEN)}
    )
    if response.status_code == 200:
      response_life_cycle_state = response.json()["metadata"]["state"]["life_cycle_state"]
      #print(response_life_cycle_state)
    else:
      raise ValueError("Error getting job runinfo: %s: %s" % (response.json()["error_code"], response.json()["message"]))  
    
    time.sleep(SLEEP)
else:
    raise ValueError("Error running job: %s: %s" % (response.json()["error_code"], response.json()["message"]))  
    
if response_life_cycle_state !='TERMINATED':
  raise ValueError("Unable to collect result info for run")  
else:
  
  response_result_state = response.json()["metadata"]["state"]["result_state"]
  response_state_message = response.json()["metadata"]["state"]["state_message"]
  
  print('Unit Test Result:', response_result_state)

  if response_result_state == 'FAILED':
    print(response.json())
    raise ValueError("Databricks Notebook failed (Result State: {0}, State Message: {1})".format(response_result_state, response_state_message))

```
