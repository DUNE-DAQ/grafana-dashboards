# Grafana DAQ dashboard repository

This repository collects graphana dashboards created to support DAQ operations and testing.

| folder          | description                              |
| --              | ---                                      |
| overiew         | DAQ system overview dashboards           |
| timing          | Timing system, HSI and timing in general |
| data collection | Event building, data streaming, ...      |


## Helm

### Release Process

* Ensure the `Chart.yaml` has the correct `appVersion`.
  * It should match the version tagged in the repo
* Increment the `version` of the chart in `Chart.yaml`
  * It should be fine for this to match the the `appVersion`.

### Grafana
The [official grafana helm chart](https://artifacthub.io/packages/helm/grafana/grafana) will need the following values set:

```yaml
---
sidecar:
  dashboards:
    enabled: true
    provider:
      foldersFromFilesStructure: true
    label: grafana_dashboard
    labelValue: 1
networkPolicy:
  enabled: true
```

This can be done with a `helm -n monitoring install grafana grafana/grafana -f values.yaml` and veified with `helm -n monitoring get values grafana`

### Helm Deployment Process

This repo can be packaged as a chart via `helm package .`

The resulting helm chart can be installed via : `helm -n monitoring install dune-grafana-dashboards /path/to/package`.

If you want a non-default folder you will need to set an override value of:

```yaml
---
dashboards:
  folder: my_folder_name
```
