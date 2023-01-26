# Prometheus

[Prometheus](https://prometheus.io/), a [Cloud Native Computing Foundation](https://cncf.io/) project, is a systems and service monitoring system. It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is observed to be true.

Follow the instructions in this document to deploy prometheus in EKE cluster and it is based on [Prometheus Community Kubernetes Helm Charts](https://github.com/prometheus-community/helm-charts)

## Prerequisites

- Kubernetes 1.16+
- Helm 3.9+

## Get Repository Info

```console
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## Install/Upgrade prometheus with helm chart default values 

```console
helm upgrade -install [RELEASE_NAME] prometheus-community/prometheus --namespace [K8S_NAMESPACE] --create-namespace --wait --debug
```

By default this chart installs additional, dependent charts:

- [alertmanager](https://github.com/prometheus-community/helm-charts/tree/main/charts/alertmanager)
- [kube-state-metrics](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-state-metrics)
- [prometheus-node-exporter](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-node-exporter)
- [prometheus-pushgateway](https://github.com/walker-tom/helm-charts/tree/main/charts/prometheus-pushgateway)

Run the following command to install prometheus without any additional add-ons

```console
helm upgrade -install [RELEASE_NAME] prometheus-community/prometheus --set alertmanager.enabled=false --set kube-state-metrics.enabled=false --set prometheus-node-exporter.enabled=false --set prometheus-pushgateway.enabled=false --namespace [K8S_NAMESPACE] --create-namespace --wait --debug
```

The above commands install the latest chart version and use the `--version` argument to install a specific version of the prometheus chart.

```console
helm upgrade -install [RELEASE_NAME] prometheus-community/prometheus --namespace [K8S_NAMESPACE] --version 18.0.0 --create-namespace --wait --debug
```

## Install/Upgrade prometheus with custom helm chart values file

- Create a `values.yaml` file with custom helm chart inputs. Refer to the `values.yaml` file in this repo for sample configurations. 

- Refer to the [official charts](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus) for recent configurations. 

Run the following command to install prometheus with custom configurations

```console
helm upgrade -install [RELEASE_NAME] prometheus-community/prometheus --namespace [K8S_NAMESPACE] -f values.yaml --create-namespace --wait --debug
```