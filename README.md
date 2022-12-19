# Grafana DAQ dashboard repository

This repository collects graphana dashboards created to support DAQ operations and testing.

| folder          | description                              |
| --              | ---                                      |
| overiew         | DAQ system overview dashboards           |
| timing          | Timing system, HSI and timing in general |
| data collection | Event building, data streaming, ...      |


## Helm

To build a helm chart of this repo for kubernetes:

* Ensure the `Chart.yaml` has the correct `appVersion`.  It should match the version tagged in the repo
* Increment the `version` of the chart in `Chart.yaml`
* Run `helm package .`
* Deploy the new package with `helm`

If your grafana instance is set to import `ConfigMap` entries with `grafana_dashboard == 1`, restarting grafana should refresh the dashboards.
