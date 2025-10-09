# Cribl SentinelOne REST Collector Pack
----
## About this Pack

This Pack is designed to collect, process, and output Microsoft SentinelOne data via the SentinelOne MGMT REST and GraphQL API's. It currently supports the following endpoints (documentation is behind a paywall, unfortunately):
* SentinelOne Alerts via MGTM REST API
* SentinelOne Alerts via GraphQL API
* SentinelOne Agent Inventory
* SentinelOne Asset Inventory
* SentinelOne Vulnerabilities via MGMT REST API
* SentinelOne Vulnerabilities via GraphQL API

The Pack includes OCSF and Splunk output processing. OCSF data is mapped to the following Classes:
* SentinelOne Alerts via MGTM REST API - [Detection Finding [2004] Class](https://schema.ocsf.io/1.4.0/classes/detection_finding)
* SentinelOne Alerts via GraphQL API - [Detection Finding [2004] Class](https://schema.ocsf.io/1.4.0/classes/detection_finding)
* SentinelOne Agent Inventory - [Device Inventory Info [5001] Class](https://schema.ocsf.io/1.4.0/classes/inventory_info)
* SentinelOne Asset Inventory - [Device Inventory Info [5001] Class](https://schema.ocsf.io/1.4.0/classes/inventory_info)
* SentinelOne Vulnerabilities via MGMT REST API - [Vulnerability Finding [2002] Class](https://schema.ocsf.io/1.4.0/classes/vulnerability_finding)
* SentinelOne Vulnerabilities via GraphQL API - [Vulnerability Finding [2002] Class](https://schema.ocsf.io/1.4.0/classes/vulnerability_finding)

Splunk data is mapped to the following sourcetypes - where applicable, these are the sourcetypes used by the [SentinelOne App For Splunk](https://splunkbase.splunk.com/app/5433):
* SentinelOne Alerts via MGTM REST API: `sourcetype=sentinelone:api:alerts`
* SentinelOne Alerts via GraphQL API: `sourcetype=sentinelone:graphql:alerts`
* SentinelOne Agent Inventory:`sourcetype=sentinelone:api:agents`
* SentinelOne Asset Inventory: `sourcetype=sentinelone:api:assets`
* SentinelOne Vulnerabilities via MGMT REST API: `sourcetype=sentinelone:channel:application_management:risks`
* SentinelOne Vulnerabilities via GraphQL API: `sourcetype=sentinelone:graphql:vulnerabilities`


## Which Collectors Do I Use?
The Pack contains Collectors that support both the "legacy" SentinelOne as well as the new Singularity Operations Center. If you've opted-in to Singularity Operations Center, then use the following Collectors:
* SentinelOne Alerts - GraphQL API
* SentinelOne Vulnerabilities - GraphQL API
* SentinelOne Assets

If you have *not* opted in to Singularity Operations Center *or* you have existing configurations that depend on the "legacy" API, then use the following Collectors:
* SentinelOne Alerts - MGMT API
* SentinelOne Vulnerabilities - MGMT API
* SentinelOne Agents

In general, it should be fine to enable all six Collectors, but the GraphQL API tends to have more information than the MGMT API.

## Deployment
After installing the Pack, you must perform the following:

* Add the Event Breaker Ruleset included in Appendix A below to your Stream instance under Processing > Knowledge > Event Breaker Rules. You should perform a Commit/Deploy before proceeding.
* Obtain the following from a SentinelOne Administrator:
    *  The SentinelOne Management hostname - this should be something like `https://<region>-<type>.sentinelone.net`. It's the host part of the URL used to login to SentinelOne
    *  An API token. SentinelOne has two methods for generating a token - via creating a Service User or Ad-hoc by an account with the correct permissions. The Service User method allows you to set the token expiration to something longer than 30 days - the Ad-hoc method is hard-coded to a 30 day expiration. 

* Add the appropriate Collector Sources in Appendix B to your Stream instance via Data > Sources > Collectors > REST. 
* For each Collector, replace the following:
  * In Collect URL, replace `your_management_url`
  * In Collect headers->Authorization, replace `your_api_token`
* Perform a Run > Preview to verify that each Collector works correctly.
* Schedule each Collector and enable State Tracking (with default configuration) for the Alerts and Vulnerabilities Collectors *only*. Suggested schedules for each are:
   *  Alerts: Every 5 minutes with default State Tracking
   *  Vulnerabilities: Every 15 minutes with default State Tracking
   *  Assets/Agents: Once/day - these endpoints are configured to retrieve all available Assets/Agents so keep that in mind.
* Connect your SentinelOne Rest Collectors to the Pack. On the global Routes page, add a new route, specify a filter expression (if using the default Collector names, than something like `__inputId.includes('in_sentinelone')` will work) and choose the `cribl-sentinelone-rest` Pack in the Pipeline dropdown.

The following are the in-Pack configurable items - review/update them as needed. 

### Outputs

Each data type can be configured to output data in either OCSF or normalized JSON (Splunk) format. Enable *only one* format for each of the following pipelines:
* `cribl_sentinelone_alerts`
* `cribl_sentinelone_alerts_graphql`
* `cribl_sentinelone_vulnerabilities`
* `cribl_sentinelone_vulnerabilities_graphql`
* `cribl_sentinelone_assets`
* `cribl_sentinelone_agents`

### Variables

The Pack has the following variables:
* `sentinelone_default_splunk_index`: Default index for the Splunk output - defaults to `sentinelone`.

## Release Notes

### Version 0.1.0 - 2025-08-15

External Beta
* Contains the pipelines for processing the six SentinelOne data types
* Supports either OCSF or Splunk output formats

## Contributing to the Pack
To contribute to the Pack, please connect with us on [Cribl Community Slack](https://cribl-community.slack.com/). You can suggest new features or offer to collaborate.

## License
This Pack uses the following license: [Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE).


## Appendix A
Consolidated SentinelOne API Event Breaker

```
{
  "id": "SentinelOne API Ruleset",
  "minRawLength": 256,
  "description": "SentinelOne-API",
  "rules": [
    {
      "condition": "_raw.includes('node') && (_raw.include('alerts') || _raw.includes('vulnerabilities'))",
      "type": "json_array",
      "timestampAnchorRegex": "/detectedAt/",
      "timestamp": {
        "type": "auto",
        "length": 150
      },
      "timestampTimezone": "UTC",
      "timestampEarliest": "-420weeks",
      "timestampLatest": "+1week",
      "maxEventBytes": 134217728,
      "disabled": false,
      "parserEnabled": false,
      "shouldUseDataRaw": false,
      "jsonExtractAll": false,
      "eventBreakerRegex": "/[\\n\\r]+(?!\\s)/",
      "name": "SentinelOne GraphQL API",
      "jsonArrayField": "edges"
    },
    {
      "condition": "true",
      "type": "json_array",
      "timestampAnchorRegex": "/updatedAt|lastScanDate/",
      "timestamp": {
        "type": "auto",
        "length": 150
      },
      "timestampTimezone": "local",
      "timestampEarliest": "-420weeks",
      "timestampLatest": "+1week",
      "maxEventBytes": 134217728,
      "disabled": false,
      "parserEnabled": false,
      "shouldUseDataRaw": false,
      "jsonExtractAll": false,
      "eventBreakerRegex": "/[\\n\\r]+(?!\\s)/",
      "name": "SentinelOne API",
      "jsonArrayField": "data"
    }
  ]
}
```

## Appendix B
SentinelOne Graph Collector Sources JSON

### SentinelOne Alerts - MGMT API
```
{
  "type": "collection",
  "ttl": "4h",
  "ignoreGroupJobsLimit": false,
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {
    "cronSchedule": "*/5 * * * *",
    "maxConcurrentRuns": 1,
    "skippable": true,
    "run": {
      "rescheduleDroppedTasks": true,
      "maxTaskReschedule": 1,
      "logLevel": "info",
      "jobTimeout": "0",
      "mode": "run",
      "timeRangeType": "relative",
      "timeWarning": {},
      "expression": "true",
      "minTaskSize": "1MB",
      "maxTaskSize": "10MB",
      "stateTracking": {},
      "timestampTimezone": "UTC",
      "earliest": "-6m@m",
      "latest": "-1m@m"
    },
    "enabled": false
  },
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "none",
        "itemList": [
          "cloud-detection/alerts"
        ]
      },
      "collectMethod": "get",
      "pagination": {
        "type": "response_body",
        "maxPages": 0,
        "attribute": [
          "pagination.nextCursor"
        ],
        "lastPageExpr": "!pagination.nextCursor.includes('a')"
      },
      "authentication": "none",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": true,
      "decodeUrl": true,
      "rejectUnauthorized": false,
      "captureHeaders": false,
      "safeHeaders": [],
      "retryRules": {
        "type": "backoff",
        "interval": 1000,
        "limit": 5,
        "multiplier": 2,
        "maxIntervalMs": 20000,
        "codes": [
          429,
          503
        ],
        "enableHeader": true,
        "retryConnectTimeout": false,
        "retryConnectReset": false,
        "retryHeaderName": "retry-after"
      },
      "__scheduling": {
        "stateTracking": {}
      },
      "collectUrl": "'https://your_management_url/web/api/v2.1/cloud-detection/alerts'\r\n",
      "clientSecretParamValue": "",
      "collectRequestParams": [
        {
          "name": "createdAt__gte",
          "value": "C.Time.strftime(earliest ? earliest : Date.now()/1000 - 30*24*60*60,\"%Y-%m-%dT%H:%M:%S.%fZ\") "
        },
        {
          "name": "createdAt__lte",
          "value": "C.Time.strftime(latest ? latest : Date.now()/1000 - 60,\"%Y-%m-%dT%H:%M:%S.%fZ\") "
        },
        {
          "name": "limit",
          "value": "1000"
        }
      ],
      "collectRequestHeaders": [
        {
          "name": "Authorization",
          "value": "`ApiToken your_api_token`"
        }
      ]
    },
    "destructive": false,
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "breakerRulesets": [
      "SentinelOne API Ruleset"
    ],
    "metadata": [
      {
        "name": "ctype",
        "value": "__collectible.id.split('/').pop()"
      }
    ]
  },
  "savedState": {},
  "description": "Use for non-Singularity Operations Center Alerts",
  "id": "in_sentinelone_alerts"
}
```
### SentinelOne Alerts - GraphQL API
```
{
  "type": "collection",
  "ttl": "4h",
  "ignoreGroupJobsLimit": false,
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {
    "cronSchedule": "*/5 * * * *",
    "maxConcurrentRuns": 1,
    "skippable": true,
    "run": {
      "rescheduleDroppedTasks": true,
      "maxTaskReschedule": 1,
      "logLevel": "info",
      "jobTimeout": "0",
      "mode": "run",
      "timeRangeType": "relative",
      "timeWarning": {},
      "expression": "true",
      "minTaskSize": "1MB",
      "maxTaskSize": "10MB",
      "stateTracking": {},
      "timestampTimezone": "UTC",
      "earliest": "-6m@m",
      "latest": "-1m@m"
    },
    "enabled": false
  },
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "none",
        "itemList": [
          "cloud-detection/alerts"
        ]
      },
      "collectMethod": "post",
      "pagination": {
        "type": "none",
        "maxPages": 0,
        "attribute": [
          "pagination.nextCursor"
        ],
        "lastPageExpr": "!pagination.nextCursor.includes('a')"
      },
      "authentication": "none",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": true,
      "decodeUrl": true,
      "rejectUnauthorized": false,
      "captureHeaders": false,
      "safeHeaders": [],
      "retryRules": {
        "type": "backoff",
        "interval": 1000,
        "limit": 5,
        "multiplier": 2,
        "maxIntervalMs": 20000,
        "codes": [
          429,
          503
        ],
        "enableHeader": true,
        "retryConnectTimeout": false,
        "retryConnectReset": false,
        "retryHeaderName": "retry-after"
      },
      "__scheduling": {
        "stateTracking": {}
      },
      "collectUrl": "'https://your_management_url/web/api/v2.1/unifiedalerts/graphql'\r\n",
      "clientSecretParamValue": "",
      "collectRequestParams": [
        {
          "name": "query",
          "value": "`query criblAlerts {    alerts (first: 1000, filters: [{fieldId: \"updatedAt\", dateTimeRange: {end: ` + `${latest ? latest : Math.round(Date.now()/1000)}` + `, start: ` +  `${earliest ? earliest : Math.round(Date.now()/1000 - 30*24*60*60)}` + `, startInclusive: true}}]){      edges { node { analystVerdict analytics { category name typeValue uid } asset { agentUuid agentVersion assetTypeClassifier category connectivityToConsole id lastLoggedInUser name osType osVersion pendingReboot policy subcategory } assignee { email fullName userId } attackSurfaces classification confidenceLevel createdAt dataSources description detectedAt detectionSource { product vendor } detectionTime { asset { agentVersion consoleIpAddress domain ipV4 ipV6 lastLoggedInUser osName osRevision osType policy subscriptionTime } attacker { host ip } cloud { accountId cloudProvider image instanceId instanceSize location network tags } kubernetes { clusterName containerId containerImageName containerLabels containerName controllerLabels controllerName controllerType namespaceLabels namespaceName nodeLabels nodeName podLabels podName } scope { accountId accountName groupName siteName } targetUser { domain emailAddress name } } externalId firstSeenAt id lastSeenAt name noteExists process { cmdLine file { certSubject md5 name path sha1 sha256 } parentName } realTime { scope { account { id name } group { id name } site { id name } } } result severity status storylineId ticketId updatedAt } } } } `"
        }
      ],
      "collectRequestHeaders": [
        {
          "name": "Authorization",
          "value": "`ApiToken your_api_token`"
        }
      ],
      "collectBody": "`query criblAlerts {    alerts {      edges {        node {          id          asset{            agentUuid            assetTypeClassifier            category            id            lastLoggedInUser            name            osType            osVersion            subcategory          }          createdAt          detectedAt          attackSurfaces          classification          confidenceLevel          dataSources          description          detectionSource{            product          }          name          status          analystVerdict          assignee {            fullName            email          }        }      },      totalCount    }  }`"
    },
    "destructive": false,
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "breakerRulesets": [
      "SentinelOne API Ruleset"
    ],
    "metadata": [
      {
        "name": "ctype",
        "value": "__collectible.id.split('/').pop()"
      }
    ]
  },
  "savedState": {},
  "description": "Use for Singularity Operations Center Alerts",
  "id": "in_sentinelone_alerts_graphql"
}
```
### SentinelOne Vulnerabilities - MGMT API
```
{
  "type": "collection",
  "ttl": "4h",
  "ignoreGroupJobsLimit": false,
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {
    "cronSchedule": "*/5 * * * *",
    "maxConcurrentRuns": 1,
    "skippable": true,
    "run": {
      "rescheduleDroppedTasks": true,
      "maxTaskReschedule": 1,
      "logLevel": "info",
      "jobTimeout": "0",
      "mode": "run",
      "timeRangeType": "relative",
      "timeWarning": {},
      "expression": "true",
      "minTaskSize": "1MB",
      "maxTaskSize": "10MB",
      "stateTracking": {},
      "timestampTimezone": "UTC",
      "earliest": "-6m@m",
      "latest": "-1m@m"
    },
    "enabled": false
  },
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "none",
        "itemList": [
          "cloud-detection/alerts"
        ]
      },
      "collectMethod": "get",
      "pagination": {
        "type": "response_body",
        "maxPages": 0,
        "attribute": [
          "pagination.nextCursor"
        ],
        "lastPageExpr": "!pagination.nextCursor.includes('a')"
      },
      "authentication": "none",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": true,
      "decodeUrl": true,
      "rejectUnauthorized": false,
      "captureHeaders": false,
      "safeHeaders": [],
      "retryRules": {
        "type": "backoff",
        "interval": 1000,
        "limit": 5,
        "multiplier": 2,
        "maxIntervalMs": 20000,
        "codes": [
          429,
          503
        ],
        "enableHeader": true,
        "retryConnectTimeout": false,
        "retryConnectReset": false,
        "retryHeaderName": "retry-after"
      },
      "__scheduling": {
        "stateTracking": {}
      },
      "collectUrl": "'https://your_management_url/web/api/v2.1/application-management/risks'\r\n",
      "clientSecretParamValue": "",
      "collectRequestParams": [
        {
          "name": "limit",
          "value": "1000"
        },
        {
          "name": "detectionDate__gte",
          "value": "C.Time.strftime(earliest ? earliest : Date.now()/1000 - 30*24*60*60,\"%Y-%m-%dT%H:%M:%S.%fZ\") "
        },
        {
          "name": "detectionDate__lte",
          "value": "C.Time.strftime(latest ? latest : Date.now()/1000 - 60,\"%Y-%m-%dT%H:%M:%S.%fZ\") "
        }
      ],
      "collectRequestHeaders": [
        {
          "name": "Authorization",
          "value": "`ApiToken your_api_token`"
        }
      ]
    },
    "destructive": false,
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "breakerRulesets": [
      "SentinelOne API Ruleset"
    ],
    "metadata": [
      {
        "name": "ctype",
        "value": "__collectible.id.split('/').pop()"
      }
    ]
  },
  "savedState": {},
  "description": "Use for non-Singularity Operations Center Vulnerabilities",
  "id": "in_sentinelone_vulnerabilities"
}
```
### SentinelOne Vulnerabilities - GraphQL API
```
{
  "type": "collection",
  "ttl": "4h",
  "ignoreGroupJobsLimit": false,
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {
    "cronSchedule": "*/5 * * * *",
    "maxConcurrentRuns": 1,
    "skippable": true,
    "run": {
      "rescheduleDroppedTasks": true,
      "maxTaskReschedule": 1,
      "logLevel": "info",
      "jobTimeout": "0",
      "mode": "run",
      "timeRangeType": "relative",
      "timeWarning": {},
      "expression": "true",
      "minTaskSize": "1MB",
      "maxTaskSize": "10MB",
      "stateTracking": {},
      "timestampTimezone": "UTC",
      "earliest": "-6m@m",
      "latest": "-1m@m"
    },
    "enabled": false
  },
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "none",
        "itemList": [
          "cloud-detection/alerts"
        ]
      },
      "collectMethod": "post",
      "pagination": {
        "type": "none",
        "maxPages": 0,
        "attribute": [
          "pagination.nextCursor"
        ],
        "lastPageExpr": "!pagination.nextCursor.includes('a')"
      },
      "authentication": "none",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": true,
      "decodeUrl": true,
      "rejectUnauthorized": false,
      "captureHeaders": false,
      "safeHeaders": [],
      "retryRules": {
        "type": "backoff",
        "interval": 1000,
        "limit": 5,
        "multiplier": 2,
        "maxIntervalMs": 20000,
        "codes": [
          429,
          503
        ],
        "enableHeader": true,
        "retryConnectTimeout": false,
        "retryConnectReset": false,
        "retryHeaderName": "retry-after"
      },
      "__scheduling": {
        "stateTracking": {}
      },
      "collectUrl": "'https://your_management_url/web/api/v2.1/xspm/findings/vulnerabilities/graphql'\r\n",
      "clientSecretParamValue": "",
      "collectRequestParams": [
        {
          "name": "query",
          "value": "`query criblVulberabilities { vulnerabilities ( filters: [{fieldId: \"detectedAt\", dateTimeRange: {end: ` + `${latest ? latest : Math.round(Date.now()/1000)}` + `, start: ` +  `${earliest ? earliest : Math.round(Date.now()/1000 - 120*24*60*60)}` + `, startInclusive: true}}]){ edges { node { analystVerdict asset { category cloudInfo { accountId accountName providerName region resourceId resourceLink } criticality domain id kubernetesInfo { cluster clusterId namespace } name osType privileged subcategory type } assignee { deleted email fullName id } cve { epssScore exploitMaturity exploitedInTheWild id publishedDate remediationLevel reportConfidence score score } detectedAt exclusionPolicyId id name product scope { account { id name } group { id name } site { id name } } severity software { fixVersion name type vendor version } status vendor } } }}`"
        }
      ],
      "collectRequestHeaders": [
        {
          "name": "Authorization",
          "value": "`ApiToken your_api_token`"
        }
      ],
      "username": "MTk0ODg6YXdzOnVzLWVhc3QtMToxNDUwMTI5MjQ4NzY1NDg1NTg1",
      "password": "hEqmqaa3o8...nZFK8zDjU=",
      "collectBody": "`query criblAlerts {    alerts {      edges {        node {          id          asset{            agentUuid            assetTypeClassifier            category            id            lastLoggedInUser            name            osType            osVersion            subcategory          }          createdAt          detectedAt          attackSurfaces          classification          confidenceLevel          dataSources          description          detectionSource{            product          }          name          status          analystVerdict          assignee {            fullName            email          }        }      },      totalCount    }  }`"
    },
    "destructive": false,
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "breakerRulesets": [
      "SentinelOne API Ruleset"
    ],
    "metadata": [
      {
        "name": "ctype",
        "value": "__collectible.id.split('/').pop()"
      }
    ]
  },
  "savedState": {},
  "description": "Use for Singularity Operations Center Vulnerabilities",
  "id": "in_sentinelone_vulnerabilities_graphql"
}
```
### SentinelOne Assets
```
{
  "type": "collection",
  "ttl": "4h",
  "ignoreGroupJobsLimit": false,
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {
    "cronSchedule": "*/5 * * * *",
    "maxConcurrentRuns": 1,
    "skippable": true,
    "run": {
      "rescheduleDroppedTasks": true,
      "maxTaskReschedule": 1,
      "logLevel": "info",
      "jobTimeout": "0",
      "mode": "run",
      "timeRangeType": "relative",
      "timeWarning": {},
      "expression": "true",
      "minTaskSize": "1MB",
      "maxTaskSize": "10MB",
      "stateTracking": {},
      "timestampTimezone": "UTC",
      "earliest": "-6m@m",
      "latest": "-1m@m"
    },
    "enabled": false
  },
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "none",
        "itemList": [
          "cloud-detection/alerts"
        ]
      },
      "collectMethod": "get",
      "pagination": {
        "type": "response_body",
        "maxPages": 0,
        "attribute": [
          "pagination.nextCursor"
        ],
        "lastPageExpr": "!pagination.nextCursor.includes('a')"
      },
      "authentication": "none",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": true,
      "decodeUrl": true,
      "rejectUnauthorized": false,
      "captureHeaders": false,
      "safeHeaders": [],
      "retryRules": {
        "type": "backoff",
        "interval": 1000,
        "limit": 5,
        "multiplier": 2,
        "maxIntervalMs": 20000,
        "codes": [
          429,
          503
        ],
        "enableHeader": true,
        "retryConnectTimeout": false,
        "retryConnectReset": false,
        "retryHeaderName": "retry-after"
      },
      "__scheduling": {
        "stateTracking": {}
      },
      "collectUrl": "'https://your_management_url/web/api/v2.1/xdr/assets/surface/endpoint'\r\n",
      "clientSecretParamValue": "",
      "collectRequestParams": [],
      "collectRequestHeaders": [
        {
          "name": "Authorization",
          "value": "`ApiToken your_api_token`"
        }
      ]
    },
    "destructive": false,
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "breakerRulesets": [
      "SentinelOne API Ruleset"
    ],
    "metadata": [
      {
        "name": "ctype",
        "value": "__collectible.id.split('/').pop()"
      }
    ]
  },
  "savedState": {},
  "description": "Use for Singularity Operations Center Assets",
  "id": "in_sentinelone_assets_surface"
}
```

### SentinelOne Agents
```
{
  "type": "collection",
  "ttl": "4h",
  "ignoreGroupJobsLimit": false,
  "removeFields": [],
  "resumeOnBoot": false,
  "schedule": {
    "cronSchedule": "*/5 * * * *",
    "maxConcurrentRuns": 1,
    "skippable": true,
    "run": {
      "rescheduleDroppedTasks": true,
      "maxTaskReschedule": 1,
      "logLevel": "info",
      "jobTimeout": "0",
      "mode": "run",
      "timeRangeType": "relative",
      "timeWarning": {},
      "expression": "true",
      "minTaskSize": "1MB",
      "maxTaskSize": "10MB",
      "stateTracking": {},
      "timestampTimezone": "UTC",
      "earliest": "-6m@m",
      "latest": "-1m@m"
    },
    "enabled": false
  },
  "streamtags": [],
  "workerAffinity": false,
  "collector": {
    "conf": {
      "discovery": {
        "discoverType": "none",
        "itemList": [
          "cloud-detection/alerts"
        ]
      },
      "collectMethod": "get",
      "pagination": {
        "type": "response_body",
        "maxPages": 0,
        "attribute": [
          "nextCursor"
        ]
      },
      "authentication": "none",
      "timeout": 0,
      "useRoundRobinDns": false,
      "disableTimeFilter": true,
      "decodeUrl": true,
      "rejectUnauthorized": false,
      "captureHeaders": false,
      "safeHeaders": [],
      "retryRules": {
        "type": "backoff",
        "interval": 1000,
        "limit": 5,
        "multiplier": 2,
        "maxIntervalMs": 20000,
        "codes": [
          429,
          503
        ],
        "enableHeader": true,
        "retryConnectTimeout": false,
        "retryConnectReset": false,
        "retryHeaderName": "retry-after"
      },
      "__scheduling": {
        "stateTracking": {}
      },
      "collectUrl": "'https://your_management_url/web/api/v2.1/agents'\r\n",
      "clientSecretParamValue": "",
      "collectRequestParams": [],
      "collectRequestHeaders": [
        {
          "name": "Authorization",
          "value": "`ApiToken your_api_token`"
        }
      ]
    },
    "destructive": false,
    "type": "rest"
  },
  "input": {
    "type": "collection",
    "staleChannelFlushMs": 10000,
    "sendToRoutes": true,
    "preprocess": {
      "disabled": true
    },
    "throttleRatePerSec": "0",
    "breakerRulesets": [
      "SentinelOne API Ruleset"
    ],
    "metadata": [
      {
        "name": "ctype",
        "value": "__collectible.id.split('/').pop()"
      }
    ]
  },
  "savedState": {},
  "description": "Use for non-Singularity Operations Center Agent Inventory",
  "id": "in_sentinelone_agents"
}
```