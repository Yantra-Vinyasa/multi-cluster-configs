# LokiStack

The _LokiStack_ object is deployed using a pre-defined set of sizes.

| Description         | 1x.extra-small | 1x.small      | 1x.medium   |
|---------------------|----------------|---------------|-------------|
| Data transfer       | Demo use only. | 500GB/day     | 2TB/day     |
| Queries per second  | Demo use only. | 25-50 QPS     | 25-75 QPS   |
|                     | at 200ms       | at 200ms      |             |
| Replication factor  | None           | 2             | 3           |
| Total CPU requests  | 5 vCPUs        | 36 vCPUs      | 54 vCPUs    |
| Total Memory requests | 7,5Gi         | 63Gi          | 139Gi       |
| Total Disk requests | 150Gi          | 300Gi         | 450Gi       |

> NOTE: 1x.extra-small is only intended for demo purposes

Read more [here](https://docs.openshift.com/container-platform/4.13/logging/cluster-logging-loki.html).

## Configuration
Some configuration is possible on both a global as well as on a tenant level.

* **Ingestion:**
  - IngestionLimits defines the limits applied on ingested log streams.

* **Queries:**
  - QueryLimits defines the limit applied on querying log streams.

* **Retention:**
  - Retention defines how long logs are kept in storage.


More information regarding configurable fields can be found in the [upstream documentation](https://grafana.com/docs/loki/latest/operations/request-validation-rate-limits/).