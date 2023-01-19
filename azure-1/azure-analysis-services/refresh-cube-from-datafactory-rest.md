# Refresh Cube from DataFactory (REST)

This page describes how to create an ADF pipeline for refreshing an AAS cube using REST.&#x20;

The AAS REST API uses OAuth2. Before the API can be called, a token must be obtained from the token provider.

### Steps

1. Create app registration in AAD
2. Authorize app registration for API
3. Authorize app-id as administrator in AAS
4. Create DataFactory Pipeline&#x20;

### DataFactory Pipeline

The diagram below shows a DataFactory pipeline, which refreshes an AAS cube.

In the first activity "GatAASApiToken", an access token is obtained. This token is used in the next web-activity which executes a refresh for a specific cube.

![Pipeline for AAS Cube Refresh](<../../.gitbook/assets/image (6).png>)



### ARM Template

Notes:

* Content-Type: "application/json" !!!!!
*

```
{
	"name": "IA_REFRESH_CUBE",
	"properties": {
		"activities": [
			{
				"name": "LoadStart",
				"type": "DatabricksNotebook",
				"dependsOn": [
					{
						"activity": "LOADDATETIME",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"policy": {
					"timeout": "7.00:00:00",
					"retry": 0,
					"retryIntervalInSeconds": 30,
					"secureOutput": false,
					"secureInput": false
				},
				"typeProperties": {
					"notebookPath": "/Shared/GENERAL/LOG_ENTRY",
					"baseParameters": {
						"CUSTOMER": {
							"value": "@pipeline().parameters.CUSTOMER",
							"type": "Expression"
						},
						"SOURCE": {
							"value": "@pipeline().parameters.SOURCE",
							"type": "Expression"
						},
						"JOBNAME": {
							"value": "@variables('JOBNAME')",
							"type": "Expression"
						},
						"LEVELNAME": "INFO",
						"RUNID": {
							"value": "@variables('RUNID')",
							"type": "Expression"
						},
						"MESSAGE": {
							"value": "@concat('The job ', variables('JOBNAME'), ' has started')",
							"type": "Expression"
						},
						"EVENTCLASS": "/Load/Start",
						"TARGET": {
							"value": "@pipeline().parameters.TARGET",
							"type": "Expression"
						}
					}
				},
				"linkedServiceName": {
					"referenceName": "cgazbidbrlsv",
					"type": "LinkedServiceReference"
				}
			},
			{
				"name": "LoadFinish",
				"type": "DatabricksNotebook",
				"dependsOn": [
					{
						"activity": "DELETE_FLAGFILES",
						"dependencyConditions": [
							"Succeeded",
							"Failed"
						]
					}
				],
				"policy": {
					"timeout": "7.00:00:00",
					"retry": 0,
					"retryIntervalInSeconds": 30,
					"secureOutput": false,
					"secureInput": false
				},
				"typeProperties": {
					"notebookPath": "/Shared/GENERAL/LOG_ENTRY",
					"baseParameters": {
						"CUSTOMER": {
							"value": "@pipeline().parameters.CUSTOMER",
							"type": "Expression"
						},
						"SOURCE": {
							"value": "@pipeline().parameters.SOURCE",
							"type": "Expression"
						},
						"JOBNAME": {
							"value": "@variables('JOBNAME')",
							"type": "Expression"
						},
						"LEVELNAME": "INFO",
						"RUNID": {
							"value": "@variables('RUNID')",
							"type": "Expression"
						},
						"MESSAGE": {
							"value": "@concat('The job ', variables('JOBNAME'), ' has finished')",
							"type": "Expression"
						},
						"EVENTCLASS": "/Load/Finished",
						"TARGET": {
							"value": "@pipeline().parameters.TARGET",
							"type": "Expression"
						}
					}
				},
				"linkedServiceName": {
					"referenceName": "cgazbidbrlsv",
					"type": "LinkedServiceReference"
				}
			},
			{
				"name": "LoadFailed",
				"type": "DatabricksNotebook",
				"dependsOn": [
					{
						"activity": "LoadStart",
						"dependencyConditions": [
							"Failed"
						]
					},
					{
						"activity": "LOADDATETIME",
						"dependencyConditions": [
							"Failed"
						]
					},
					{
						"activity": "JOBNAME",
						"dependencyConditions": [
							"Failed"
						]
					},
					{
						"activity": "RUNID",
						"dependencyConditions": [
							"Failed"
						]
					},
					{
						"activity": "FLAGFOLDER",
						"dependencyConditions": [
							"Failed"
						]
					},
					{
						"activity": "IfFailed",
						"dependencyConditions": [
							"Failed"
						]
					},
					{
						"activity": "UntilRefreshComplete",
						"dependencyConditions": [
							"Failed"
						]
					},
					{
						"activity": "FilterCurrentRefresh",
						"dependencyConditions": [
							"Failed"
						]
					},
					{
						"activity": "GetRefreshStatus",
						"dependencyConditions": [
							"Failed"
						]
					},
					{
						"activity": "StartRefresh",
						"dependencyConditions": [
							"Failed"
						]
					},
					{
						"activity": "GetAASApiToken",
						"dependencyConditions": [
							"Failed"
						]
					}
				],
				"policy": {
					"timeout": "7.00:00:00",
					"retry": 0,
					"retryIntervalInSeconds": 30,
					"secureOutput": false,
					"secureInput": false
				},
				"typeProperties": {
					"notebookPath": "/Shared/GENERAL/LOG_ENTRY",
					"baseParameters": {
						"CUSTOMER": {
							"value": "@pipeline().parameters.CUSTOMER",
							"type": "Expression"
						},
						"SOURCE": {
							"value": "@pipeline().parameters.SOURCE",
							"type": "Expression"
						},
						"JOBNAME": {
							"value": "@variables('JOBNAME')",
							"type": "Expression"
						},
						"LEVELNAME": "ERROR",
						"RUNID": {
							"value": "@variables('RUNID')",
							"type": "Expression"
						},
						"MESSAGE": {
							"value": "@concat('The job ', variables('JOBNAME'), ' has failed')",
							"type": "Expression"
						},
						"EVENTCLASS": "/Load/Failed",
						"TARGET": {
							"value": "@pipeline().parameters.TARGET",
							"type": "Expression"
						}
					}
				},
				"linkedServiceName": {
					"referenceName": "cgazbidbrlsv",
					"type": "LinkedServiceReference"
				}
			},
			{
				"name": "RUNID",
				"type": "SetVariable",
				"typeProperties": {
					"variableName": "RUNID",
					"value": {
						"value": "@pipeline().RunId",
						"type": "Expression"
					}
				}
			},
			{
				"name": "JOBNAME",
				"type": "SetVariable",
				"dependsOn": [
					{
						"activity": "RUNID",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"typeProperties": {
					"variableName": "JOBNAME",
					"value": {
						"value": "@pipeline().Pipeline",
						"type": "Expression"
					}
				}
			},
			{
				"name": "LOADDATETIME",
				"type": "SetVariable",
				"dependsOn": [
					{
						"activity": "JOBNAME",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"typeProperties": {
					"variableName": "LOADDATETIME",
					"value": {
						"value": "@utcnow()",
						"type": "Expression"
					}
				}
			},
			{
				"name": "DELETE_FLAGFILES",
				"type": "Delete",
				"dependsOn": [
					{
						"activity": "FLAGFOLDER",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"policy": {
					"timeout": "7.00:00:00",
					"retry": 0,
					"retryIntervalInSeconds": 30,
					"secureOutput": false,
					"secureInput": false
				},
				"typeProperties": {
					"dataset": {
						"referenceName": "cgazbistaetl_flag",
						"type": "DatasetReference",
						"parameters": {
							"FLAGFOLDER": {
								"value": "@variables('FLAGFOLDER')",
								"type": "Expression"
							}
						}
					},
					"recursive": true,
					"enableLogging": false
				}
			},
			{
				"name": "FLAGFOLDER",
				"description": "This job is triggered by OPTI_CHECK, which monitors flag files in the ba-folder of the etl storage account. The flag files in this location need to be deleted at the end of this job.",
				"type": "SetVariable",
				"dependsOn": [
					{
						"activity": "IfFailed",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"typeProperties": {
					"variableName": "FLAGFOLDER",
					"value": {
						"value": "@pipeline().parameters.SOURCE",
						"type": "Expression"
					}
				}
			},
			{
				"name": "GetAASApiToken",
				"type": "WebActivity",
				"dependsOn": [
					{
						"activity": "LoadStart",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"policy": {
					"timeout": "7.00:00:00",
					"retry": 0,
					"retryIntervalInSeconds": 30,
					"secureOutput": false
				},
				"typeProperties": {
					"url": {
						"value": "@concat('https://login.microsoftonline.com/',pipeline().parameters.TENANTID,'/oauth2/token')",
						"type": "Expression"
					},
					"method": "POST",
					"headers": {
						"Content-Type": "application/x-www-form-urlencoded"
					},
					"body": {
						"value": "@concat('grant_type=client_credentials&resource=https://*.asazure.windows.net&client_id=',pipeline().parameters.CLIENTID,'&client_secret=',encodeUriComponent(pipeline().parameters.CLIENTSECRET))",
						"type": "Expression"
					}
				}
			},
			{
				"name": "UntilRefreshComplete",
				"type": "Until",
				"dependsOn": [
					{
						"activity": "FilterCurrentRefresh",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"typeProperties": {
					"expression": {
						"value": "@not(equals(activity('GetRefreshStatus').Status,'inProgress'))",
						"type": "Expression"
					},
					"activities": [
						{
							"name": "RetryRefresh",
							"type": "WebActivity",
							"dependsOn": [
								{
									"activity": "Wait",
									"dependencyConditions": [
										"Succeeded"
									]
								}
							],
							"policy": {
								"timeout": "7.00:00:00",
								"retry": 0,
								"retryIntervalInSeconds": 30,
								"secureOutput": false
							},
							"typeProperties": {
								"url": {
									"value": "@concat('https://',pipeline().parameters.REGION,'.asazure.windows.net/servers/',pipeline().parameters.SERVER,'/models/',pipeline().parameters.CUBE,'/refreshes/',activity('FilterCurrentRefresh').output.Value[0].refreshId)",
									"type": "Expression"
								},
								"method": "GET",
								"headers": {
									"Authorization": {
										"value": "@concat(string(activity('GetAASApiToken').output.token_type),' ',string(activity('GetAASApiToken').output.access_token))",
										"type": "Expression"
									}
								}
							}
						},
						{
							"name": "Wait",
							"type": "Wait",
							"typeProperties": {
								"waitTimeInSeconds": 30
							}
						}
					],
					"timeout": "7.00:00:00"
				}
			},
			{
				"name": "FilterCurrentRefresh",
				"type": "Filter",
				"dependsOn": [
					{
						"activity": "GetRefreshStatus",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"typeProperties": {
					"items": {
						"value": "@json(activity('GetRefreshStatus').output.Response)",
						"type": "Expression"
					},
					"condition": {
						"value": "@greaterOrEquals(item().startTime,addseconds(activity('StartRefresh').output.startTime,-30))",
						"type": "Expression"
					}
				}
			},
			{
				"name": "StartRefresh",
				"type": "WebActivity",
				"dependsOn": [
					{
						"activity": "GetAASApiToken",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"policy": {
					"timeout": "7.00:00:00",
					"retry": 0,
					"retryIntervalInSeconds": 30,
					"secureOutput": false
				},
				"typeProperties": {
					"url": {
						"value": "@concat('https://',pipeline().parameters.REGION,'.asazure.windows.net/servers/',pipeline().parameters.SERVER,'/models/',pipeline().parameters.CUBE,'/refreshes')",
						"type": "Expression"
					},
					"method": "POST",
					"headers": {
						"Authorization": {
							"value": "@concat(string(activity('GetAASApiToken').output.token_type),' ',string(activity('GetAASApiToken').output.access_token))",
							"type": "Expression"
						},
						"Content-Type": "application/json"
					},
					"body": {
						"Type": "Full",
						"CommitMode": "transactional",
						"MaxParallelism": 10,
						"RetryCount": 2
					}
				}
			},
			{
				"name": "IfFailed",
				"type": "IfCondition",
				"dependsOn": [
					{
						"activity": "UntilRefreshComplete",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"typeProperties": {
					"expression": {
						"value": "@equals(activity('GetRefreshStatus').status,'failed')",
						"type": "Expression"
					},
					"ifTrueActivities": [
						{
							"name": "ThrowError",
							"type": "WebActivity",
							"policy": {
								"timeout": "7.00:00:00",
								"retry": 0,
								"retryIntervalInSeconds": 30,
								"secureOutput": false
							},
							"typeProperties": {
								"url": {
									"value": "@string(activity('GetRefreshStatus').output)",
									"type": "Expression"
								},
								"method": "GET"
							}
						}
					]
				}
			},
			{
				"name": "GetRefreshStatus",
				"type": "WebActivity",
				"dependsOn": [
					{
						"activity": "StartRefresh",
						"dependencyConditions": [
							"Succeeded"
						]
					}
				],
				"policy": {
					"timeout": "7.00:00:00",
					"retry": 0,
					"retryIntervalInSeconds": 30,
					"secureOutput": false
				},
				"typeProperties": {
					"url": {
						"value": "@concat('https://',pipeline().parameters.Region,'.asazure.windows.net/servers/',pipeline().parameters.Server,'/models/',pipeline().parameters.CUBE,'/refreshes')",
						"type": "Expression"
					},
					"method": "GET",
					"headers": {
						"Authorization": {
							"value": "@concat(string(activity('GetAASApiToken').output.token_type),' ',string(activity('GetAASApiToken').output.access_token))",
							"type": "Expression"
						}
					},
					"body": {
						"Type": "Full",
						"CommitMode": "transactional",
						"MaxParallelism": 10,
						"RetryCount": 2
					}
				}
			}
		],
		"parameters": {
			"CUSTOMER": {
				"type": "String",
				"defaultValue": "Carglass NL"
			},
			"SOURCE": {
				"type": "String",
				"defaultValue": "ba"
			},
			"TARGET": {
				"type": "string",
				"defaultValue": "ia"
			},
			"TENANTID": {
				"type": "string",
				"defaultValue": "f465da48-e5ae-486b-9ffe-07885812bb2b"
			},
			"CLIENTID": {
				"type": "string",
				"defaultValue": "0dc2a6b1-2d19-4b03-82fe-5f46498a52d0"
			},
			"CLIENTSECRET": {
				"type": "string",
				"defaultValue": "?W[1*.yCvJPGBa4xRWhR3T9LL_7Eq29I"
			},
			"REGION": {
				"type": "string",
				"defaultValue": "westeurope"
			},
			"CUBE": {
				"type": "string",
				"defaultValue": "OPTI"
			},
			"SERVER": {
				"type": "string",
				"defaultValue": "cgazbiaasdev"
			}
		},
		"variables": {
			"RUNID": {
				"type": "String"
			},
			"JOBNAME": {
				"type": "String"
			},
			"LOADDATETIME": {
				"type": "String"
			},
			"FLAGFILE": {
				"type": "String"
			},
			"FLAGFOLDER": {
				"type": "String"
			}
		}
	},
	"type": "Microsoft.DataFactory/factories/pipelines"
}
```
