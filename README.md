# SentinelOne REST Collector IO
----
## About this Pack

This pack is built as a complete SOURCE + DESTINATION solution (identified by the IO suffix). Data collection and delivery happen entirely within the pack's context, eliminating the need to connect it to globally defined Sources and Destinations. 

This Pack is designed to collect, process, and output Microsoft SentinelOne data via the SentinelOne MGMT REST and GraphQL API's. It currently supports the following endpoints (documentation is behind a paywall, unfortunately):
* SentinelOne Alerts via MGMT REST API
* SentinelOne Alerts via GraphQL API
* SentinelOne Agent Inventory
* SentinelOne Asset Inventory
* SentinelOne Vulnerabilities via MGMT REST API
* SentinelOne Vulnerabilities via GraphQL API

The Pack includes OCSF and Splunk output processing. OCSF data is mapped to the following Classes:
* SentinelOne Alerts via MGMT REST API - [Detection Finding [2004] Class](https://schema.ocsf.io/1.4.0/classes/detection_finding)
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
The Pack contains Collectors that support both the "legacy" SentinelOne as well as the new Singularity Operations Center. If you've opted-in to Singularity Operations Center, use the following Collectors:
* SentinelOne Alerts - GraphQL API
* SentinelOne Vulnerabilities - GraphQL API
* SentinelOne Assets

If you have *not* opted in to Singularity Operations Center *or* you have existing configurations that depend on the "legacy" API, then use the following Collectors:
* SentinelOne Alerts - MGMT API
* SentinelOne Vulnerabilities - MGMT API
* SentinelOne Agents

In general, it should be fine to enable all six Collectors, but the GraphQL API tends to have more information than the MGMT API.

## Deployment
 This pack is configured by default to use the Worker Group's *Default Destination*.
* To use the *Default Destination*: No changes are required. The pack will route the data to the destination currently set as the Default on the Worker Group.
* To use a different Destination: You must update the pack's routes to specify your desired Destination.
* For immediate functionality without requiring Pack route filter expression modifications, every bundled Source within this pack adds a hidden field: `__packsource`. This field allows for seamless routing based on the Pack source.

### Configure the Collectors

* Obtain the following from your SentinelOne Administrator:
    * The SentinelOne Management hostname - this should be something like `https://<region>-<type>.sentinelone.net`. It's the host part of the URL used to login to SentinelOne. Replace `YOUR_S1_SITE` in the Collect URL field for each Collector with the correct value. 
    * An API token. SentinelOne has two methods for generating a token - via creating a Service User (*recommended*) or Ad-hoc by an account with the correct permissions. The Service User method allows you to set the token expiration to something longer than 30 days - the Ad-hoc method is hard-coded to a 30 day expiration. 
    * Update the Collect header field named `Authorization` in the following format: *'ApiToken YOUR_API_TOKEN'* (the value must be a valid JavaScript string)
* Perform a Run > Preview to verify that each Collector works correctly.
* Schedule each Collector - they all include a default cron schedule and those that require State Tracking have it enabled. Update as desired.

### Configure Output Format

Each data type can be configured to output data in either normalized JSON (default), OCSF, or Splunk (`_raw` + Splunk fields) format. Enable *only one* format for each of the following pipelines:
* `cribl_sentinelone_alerts`
* `cribl_sentinelone_alerts_graphql`
* `cribl_sentinelone_vulnerabilities`
* `cribl_sentinelone_vulnerabilities_graphql`
* `cribl_sentinelone_assets`
* `cribl_sentinelone_agents`

### Configure your Destination/Update Pack Routes
To ensure proper data routing, you must make a choice: retain the current setting to use the Default Destination defined by your Worker Group, or define a new Destination directly inside this pack and adjust the pack's route accordingly.

### Commit and Deploy
Once everything is configured, perform a Commit & Deploy to enable data collection.

### Variables

The Pack has the following variables:
* `sentinelone_default_splunk_index`: Default index for the Splunk output - defaults to `sentinelone`.

## Upgrades

Upgrading certain Cribl Packs using the same Pack ID can have unintended consequences. See [Upgrading an Existing Pack](https://docs.cribl.io/stream/packs#upgrading) for details.

## Release Notes

### Version 1.0.0
Initial release

## Contributing to the Pack

To contribute to the Pack, please connect with us on [Cribl Community Slack](https://cribl-community.slack.com/). You can suggest new features or offer to collaborate.

## License
This Pack uses the following license: [Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE).