# Deploy prometheus

[Prometheus](https://prometheus.io/), a [Cloud Native Computing Foundation](https://cncf.io/) project, is  is a popular open-source monitoring and alerting solution optimized for container environments. 

It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is observed to be true.

Follow the instructions in this document to deploy a self-managed prometheus in EKE cluster and the instructions are based on [Prometheus Community Kubernetes Helm Charts](https://github.com/prometheus-community/helm-charts)

If you are looking for a fully managed prometheus offering then please refer to [Amazon Managed Service for Prometheus](https://aws.amazon.com/prometheus/). 

## Prerequisites

- Kubernetes 1.22+
- Helm 3.9+

## Get Repository Info

```console
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## Install/Upgrade prometheus with default values 

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

## Install/Upgrade prometheus with custom values

- Create a `values.yaml` file with custom helm chart inputs. Refer to the `values.yaml` file in this repo for sample configurations. 

- Refer to the [official promethues chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus) for recent configurations. 

Run the following command to install prometheus with custom configurations

```console
helm upgrade -install [RELEASE_NAME] prometheus-community/prometheus --namespace [K8S_NAMESPACE] -f values.yaml --create-namespace --wait --debug
```

## Scraping Pod Metrics

This chart uses a default configuration that causes prometheus to scrape a variety of [kubernetes resource types](https://github.com/prometheus-community/helm-charts/blob/main/charts/prometheus/values.yaml#L614), provided they have the correct annotations. 

In order to get prometheus to scrape pods, you must add annotations to the the required pods as below:

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/path: /metrics
    prometheus.io/port: "8080"
```

You should adjust `prometheus.io/path` based on the URL that your pod serves metrics from. `prometheus.io/port` should be set to the port that your pod serves metrics from. Note that the values for `prometheus.io/scrape` and `prometheus.io/port` must be enclosed in double quotes.

## View/Query Pod Metrics

This chart creates a `prometheus-server` service with `ClusterIP` type which is accessible only inside the cluster. Change the [service type](https://github.com/prometheus-community/helm-charts/blob/main/charts/prometheus/values.yaml#L562) to `LoadBalancer` if you want to access prometheus outside cluster.

Implement [basic-auth](https://prometheus.io/docs/guides/basic-auth/) and IP restrictions if you are exposing prometheus outside the cluster.

Run the following `kubectl port-forward` command to connect to prometheus-server and go to `localhost:8080` in the browser.

```console
kubectl port-forward --namespace [K8S_NAMESPACE] svc/prometheus-server 8080:80
```

Query the required metrics in promethues UI

<img width="1920" alt="Screenshot 2023-01-26 at 14 53 24" src="https://user-images.githubusercontent.com/112865563/214868656-bb83c7c9-8f6d-49c8-87d6-7c4c74516cae.png">

<img width="1919" alt="Screenshot 2023-01-26 at 14 55 32" src="https://user-images.githubusercontent.com/112865563/214868685-3a920f58-557a-49dd-9015-1e06cb921e51.png">
