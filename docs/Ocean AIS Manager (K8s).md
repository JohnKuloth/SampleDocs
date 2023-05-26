
title: Ocean AIS Manager (K8s)
created at: Mon Apr 11 2022 17:59:32 GMT+0000 (Coordinated Universal Time)
updated at: Fri Sep 23 2022 15:40:16 GMT+0000 (Coordinated Universal Time)
---

# Ocean AIS Manager (K8s)

This document aims to set up a document template for the Engineering teams in order to ask for the deployment of new apps by the SRE team.

After the deployment in the production environment of the new component, this document will be the exploitation note of the SRE team.

All the sections are **mandatory**.

* * *

# TLDR - Monitoring links

| Environment |  Component | Links |
| ----------- | ---------- | ----- |
|             |            |       |
|             |            |       |
|             |            |       |
|             |            |       |
|             |            |       |

* * *

# What is it ?

The AIS manager have several mission:

-   Maintain vessel master data
-   Record AIS positions in tsdb
-   Transmit information de-duplicated to solution about vessel and positions
-   Transmit information de-duplicated to DSS through Bg Query about route, vessel and positions

The repository contain 3 workers running with the same code but depending on environment variable, they will run differently.

They have direct dependencies with a rabbitMQ instance, a postgresql database and another postgresql database with a timescaledb extension.

![aisManagerContainer.png](media_Ocean%20AIS%20Manager%20(K8s)/aisManagerContainer.png)

![aisManagerComponent.png](media_Ocean%20AIS%20Manager%20(K8s)/aisManagerComponent.png)

* * *

# Who ?

| Name             | Mail                                                              | Tags     |
| ---------------- | ----------------------------------------------------------------- | -------- |
| @Hugo Lecuyer    | [hugo.lecuyer@gshippeo.com](mailto:hugo.lecuyer@gshippeo.com)     | @hugo    |
| @Omar Ghalawinji | [omar.ghalawinji@shippeo.com](mailto:omar.ghalawinji@shippeo.com) |          |
| @Tristan Fransen | [tristan.fransen@shippeo.com](mailto:tristan.fransen@shippeo.com) | @Tristan |
| @Fabien Sarcel   | [fabien.sarcel@shippeo.com](mailto:fabien.sarcel@shippeo.com)     | @Fabien  |

* * *

# Repository

<https://github.com/shippeo/dba.ocean-ais-manager>

* * *

# Use Cases

Cf functional documentation in the [README.md](https://github.com/shippeo/dba.ocean-ais-manager#readme)

* * *

# Hosting

[To complete by the SRE team : where the app is deployed and the different environment. This section can be replaced by a link redirecting to a dedicated documentation.]

* * *

# Deployment

[Prod]

When a pull request is merged on `main` in the [repository](https://github.com/shippeo/dba.ocean-ais-manager), the CI (github action) will build a new image and push it to the Harbor repository. In order to deploy the new image, you need to make a pull request on [argoCD](https://github.com/shippeo/shippeo.argocd-shippeo) repository and modify this [file](https://github.com/shippeo/shippeo.argocd-shippeo/blob/main/ocean-data-processing/prod-values.yaml).

Change the tag name for `stable/ocean-ais-manager` image. Once the PR reviewed and merged, you can access [argoCD application](https://argo.internal.shippeo.cloud/applications/shippeo-ocean-data-processing-prod-gke-prod-core-main-eu?resource=) to follow the deployment.

Beware that three workers depends on the same image: ocean-ais-manager-vessel, ocean-ais-manager-route, ocean-ais-manager-position.

* * *

# Configuration

\[Describe all the configuration needed by your component to run (token, environment variables, etc). This configuration needs to be configured in the `shippeo-deployments` repository. You can use an external link to the dedicated config files.

Passwords, credentials or certificate **must not be **disclosed here. Create or use an existing Bitwarden collection to share this information (include the name of the collection here).]

<https://github.com/shippeo/shippeo.argocd-shippeo/tree/main/ocean-data-processing>

This configuration needs to be configured in the [`argocd-shippeo`](https://github.com/shippeo/shippeo.argocd-shippeo/tree/main/ocean-data-processing) repository.

| ** RMQ_QUEUE: shippeo.queue.spire.position**                                                                                                                                                                                                                                                                                                                                                                                                  |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| _It's the queue that the worker will consume. It can be also _```shippeo.queue.spire.``_vessel_```_ or _`shippeo.queue.spire.route`                                                                                                                                                                                                                                                                                                           |
| **MESSAGE_TYPE: position**                                                                                                                                                                                                                                                                                                                                                                                                                    |
| _It's the message format expected by worker. It can be also _`_vessel_`_ or _`route`_. It has to be coherent with the queue consumed_                                                                                                                                                                                                                                                                                                         |
| **LOG_LEVEL: DEBUG**                                                                                                                                                                                                                                                                                                                                                                                                                          |
| _The log level. Usually set to INFO but can be set to DEBUG for a short time or for an env without to much activity_                                                                                                                                                                                                                                                                                                                          |
| **ASYNCIO_LOG_LEVEL: DEBUG**                                                                                                                                                                                                                                                                                                                                                                                                                  |
| _Idem_                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| **RMQ_URL:amqp://user:password@<https://rabbitmq.core.dev.shippeo>**                                                                                                                                                                                                                                                                                                                                                                          |
| _Credential can be found: _[_dev_](https://sec-exch.shippeo.io/#/organizations/f91d9c71-bcbf-46c1-af92-7808c2240a65/vault?collectionId=10469865-ac26-458a-9a93-22c792f60c4e)_, _[_qa_](https://sec-exch.shippeo.io/#/organizations/f91d9c71-bcbf-46c1-af92-7808c2240a65/vault?collectionId=1f9d6973-071f-4cd7-b856-beaaf8c28b2d)_, prod (TBD): Must be set as a _[_sealsecret_](https://slite.com/api/public/notes/rq7aSAfURaP-F9/redirect) . |
| **TSDB_URL=postgres://ais_manager:{password}@timescaledb-dev-shippeodatalake.aivencloud.com:21071**                                                                                                                                                                                                                                                                                                                                           |
| _Credential can be found: _[_dev_](https://sec-exch.shippeo.io/#/vault?collectionId=2a18a261-a66a-4887-acc3-bd06d154621b)_, qa, prod (TBD): Must be set as a _[_sealsecret_](https://slite.com/api/public/notes/rq7aSAfURaP-F9/redirect) .                                                                                                                                                                                                    |
| DB_URL=postgres://geofencing:{password}@pgbouncer.core.dev.shippeo.com:5432/geofencing                                                                                                                                                                                                                                                                                                                                                        |
| _Credential can be found: _[_dev_](https://sec-exch.shippeo.io/#/vault?collectionId=276157d0-679c-4e82-b06e-25200ed97557)_, qa, prod (TBD): Must be set as a _[_sealsecret_](https://shippeo.slite.com/app/docs/rq7aSAfURaP-F9?source=search).                                                                                                                                                                                                |
| METRIC_URL=vmagent-main.monitoring.svc                                                                                                                                                                                                                                                                                                                                                                                                        |
| _Destination url for metrics send on influx format_                                                                                                                                                                                                                                                                                                                                                                                           |
| METRIC_PORT="8089"                                                                                                                                                                                                                                                                                                                                                                                                                            |
| _Port for metrics send on influx format_                                                                                                                                                                                                                                                                                                                                                                                                      |
| METRIC_DEBUG="false"                                                                                                                                                                                                                                                                                                                                                                                                                          |
| _If set to True, metrics are not send but appear in the logs instead_                                                                                                                                                                                                                                                                                                                                                                         |
| METRIC_PREFIX=oceanaismanager                                                                                                                                                                                                                                                                                                                                                                                                                 |
| _Prefix for all mettrics sended_                                                                                                                                                                                                                                                                                                                                                                                                              |
| ENV=prod                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| _Necessary to know for which env we send metrics_                                                                                                                                                                                                                                                                                                                                                                                             |
| PARALLELISM_MAX_SIZE="2"                                                                                                                                                                                                                                                                                                                                                                                                                      |
| _Fix the max num of connection that can be ask for db and the number of messages that can be prefetch by rmq_                                                                                                                                                                                                                                                                                                                                 |

* * *

# Logs

[To complete by the SRE team, describe here the namespace and how to access the logs.]

* * *

# Monitoring

[Describe the metrics you put in place to observe your component]

| Name                                 | Description                                                                                                             | Trigger                                                | Metric used                                                                                                                                      | Trigger from → to                                                                                                                                                         |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [name of the alert]                  | Count the number of calls to [your-component-endpoint]                                                                  | Execution count above [VALUE] for [DURATION]]          | [key of the metric]                                                                                                                              | Grafana → Emergency                                                                                                                                                       |
| Low positions record rate            | Ocean positions record rate dropped under 15k line for the past 15 min.                                                 | Number of position created under 15000 for 15 min      | count_over_time(oceanaismanager_execution_duration_seconds{outcome="created", env="prod", worker_type="position"}[15m])                          | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Emergency  |
| High positions record rate           | Ocean positions record rate increased above 5 million positions a day.                                                  | Number of position created above 5000000 for 1 day     | count_over_time(oceanaismanager_execution_duration_seconds{outcome="created", env="prod", worker_type="position"}[1d])                           | [argocd-shippeo ](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30)→ GCP Alerts |
| Error positions with retry (requeue) | Ais manager for positions generate errors that are requeue. It's probably linked to connections trouble with databases. | Number of errors with requeue above 10 for 5 min       | count_over_time(oceanaismanager_execution_duration_seconds{env="prod", worker_type="position", rmq_message_outcome="error_requeue"}[5m])         | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Alert      |
| Error positions with no retry        | Ais manager for positions generate errors (no retry).                                                                   | Number of errors without requeue above 10 for 5 min    | count_over_time(oceanaismanager_execution_duration_seconds{env="prod", worker_type="position", rmq_message_outcome="error_no_requeue"}[5m])      | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Alert      |
| Low vessels record rate              | Ocean vessel record rate dropped under 30 updates/creation over 15 minutes                                              | Number of vessel created/updated under 30 for 15 min   | sum(count_over_time(oceanaismanager_execution_duration_seconds{worker_type="vessel", data_outcome=~"created\|updated\|updated_bq"}[15m]))        | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Emergency  |
| High vessels record rate             | Ocean vessel record rate increased above 20 000 updates/creation a day.                                                 | Number of vessel created/updated above 20000 for 1 day | sum(count_over_time(oceanaismanager_execution_duration_seconds{worker_type="vessel", data_outcome=~"created\|updated\|updated_bq"}[1d])) > 20000 | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Alert      |
| Error vessel with retry (requeue)    | Ais manager for vessel generate errors that are requeue. It's probably linked to connections trouble with databases.    | Number of errors with requeue above 10 for 5 min       | count_over_time(oceanaismanager_execution_duration_seconds{env="prod", worker_type="vessel", rmq_message_outcome="error_requeue"}[5m])           | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Alert      |
| Error vessel with no retry           | Ais manager for positions generate errors (no retry).                                                                   | Number of errors without requeue above 10 for 5 min    | count_over_time(oceanaismanager_execution_duration_seconds{env="prod", worker_type="vessel", rmq_message_outcome="error_no_requeue"}[5m])        | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Alert      |
| Low routes record rate               | Ocean route record rate dropped under 2000 records over 15 minutes.                                                     | Number of route records under 2000 for 15 min          | count_over_time(oceanaismanager_execution_duration_seconds{worker_type="route", outcome="updated"}[15m])                                         | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Emergency  |
| High routes record rate              | Ocean route record rate increased above 1 million records a day.                                                        | Number of route records above 1000000 for 1 day        | count_over_time(oceanaismanager_execution_duration_seconds{worker_type="route", outcome=~"updated"}[1d])                                         | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Alert      |
| Error route with retry (requeue)     | Ais manager for route generate errors that are requeue. It's probably linked to connections trouble with databases.     | Number of errors with requeue above 10 for 5 min       | count_over_time(oceanaismanager_execution_duration_seconds{env="prod", worker_type="route", rmq_message_outcome="error_requeue"}[5m])            | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Alert      |
| Error route with no retry            | Ais manager for positions generate errors (no retry)                                                                    | Number of errors without requeue above 10 for 5 min    | count_over_time(oceanaismanager_execution_duration_seconds{env="prod", worker_type="route", rmq_message_outcome="error_no_requeue"}[5m])         | [argocd-shippeo](https://github.com/shippeo/shippeo.argocd-shippeo/blob/1684b161002fcdfb68927f6dbd5b58e9a03821ce/ocean-data-processing/prod-values.yaml#L30) → Alert      |
| shippeo.queue.spire.vessel           | Messages are pilling in the queue.                                                                                      | Number of messages in queue above 15 for 10 min        | <https://panoptes.shippeo.com/d/yCMlQSWVz/aandp-ocean-rmq-alerts?orgId=1&fullscreen&panelId=2>                                                   | Graphana (panoptes) → Emergency                                                                                                                                           |
| shippeo.queue.spire.route            | Messages are pilling in the queue.                                                                                      | Number of messages in queue above 15 for 10 min        | <https://panoptes.shippeo.com/d/yCMlQSWVz/aandp-ocean-rmq-alerts?orgId=1&fullscreen&panelId=3>                                                   | Graphana (panoptes) → Emergency                                                                                                                                           |
| shippeo.queue.spire.position         | Messages are pilling in the queue.                                                                                      | Number of messages in queue above 15 for 10 min        | <https://panoptes.shippeo.com/d/yCMlQSWVz/aandp-ocean-rmq-alerts?orgId=1&fullscreen&panelId=4>                                                   | Graphana (panoptes) → Emergency                                                                                                                                           |

          