# Alertmanager Configuration
Alertmanager is used to process alerts sent from Prometheus. It will turn these alerts into notifications that can be sent to external systems. These are called [receivers](https://prometheus.io/docs/alerting/latest/configuration/#receiver). In order to ensure only the wanted alerts gets through, one can filter them when setting up [routes](https://prometheus.io/docs/alerting/latest/configuration/#route).

> NOTE: In order to sent alerts to Microsoft Teams, the [prometheus-msteams](https://code.bbraun.io/IT-WI-OpenShift/multi-cluster-configs/tree/main/base/prometheus-msteams) chart needs to be installed.

## Configuration
The customizable values are:
- If msTeams is enabled or not.
- Which channel in Teams that should be getting the alerts.
> NOTE that when msTeams is enabled, and prometheus-msteams is installed in the cluster, all notifications from AlertManager will be sent to the **prometheus-msteams** pod to be processed before forwarding it to the Teams channel.

Example of all configurables:
```yaml
alertConfig:
  enabled: true

cluster:
  name: ocp1
  env: dev

msTeams:
  enabled: false
  connector: openshift-to-teams # This needs to match the name that is used for prometheus-msteams
```

### How the alerts are being processed
```yaml
 routes:
{{ if .Values.msTeams.enabled }}                    # 1
  - matchers:
    - severity=~"alert|high|info|warning|critical"
    receiver: openshift-to-teams
  - matchers:
    - severity="Critical"
    receiver: "openshift-to-email"
{{ else }}                                          # 2
  - matchers:
    - severity=~"alert|high|info|warning|critical"
    receiver: openshift-to-email
{{- end }}
  - matchers:
    - severity="none"
    receiver: blackhole                             # 3
```
1. If we have enabled msTeams in our value file, we send _all_ alerts to the teams channel. Our email receiver will only be getting the _Critical_ alerts.
2. Else, if msTeams is disabled, we are forwarding all alerts to the email receiver.
3. We also send all the **none** alerts, e.g. [watchdog](https://runbooks.prometheus-operator.dev/runbooks/general/watchdog/#watchdog) to a "blackhole" that is not being received.

## Useful commands
In order to view the configuration on a running cluster via the CLI:
```bash
oc -n openshift-monitoring get secret alertmanager-main --template='{{ index .data "alertmanager.yaml" }}' |base64 -d 
```

[OpenShift Documentation on Alerting](https://docs.openshift.com/container-platform/latest/monitoring/managing-alerts.html)
