# Deploy grafana

[Grafana](https://grafana.com/) is an open-source observability and data visualization platform that allows you to query, visualize, alert on and understand your metrics no matter where they are stored.

Follow the instructions in this document to deploy a self-managed Grafana instance in EKE cluster and the instructions are based on [Grafana Community Kubernetes Helm Charts](https://github.com/grafana/helm-charts/tree/main/charts/grafana)

If you are looking for a fully managed Grafana solution, then please refer to [Amazon Managed Grafana](https://aws.amazon.com/grafana/) or [Grafana Cloud](https://grafana.com/products/cloud/).

## Prerequisites

- Kubernetes 1.22+
- Helm 3.9+


## Get Repository Info

```console
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

## Install/Upgrade grafana with default values 

```console
helm upgrade -install [RELEASE_NAME] grafana/grafana --namespace [K8S_NAMESPACE] --create-namespace --wait --debug
```

The above commands install the latest chart version and use the `--version` argument to install a specific version of the prometheus chart.

```console
helm upgrade -install [RELEASE_NAME] grafana/grafana --namespace [K8S_NAMESPACE] --version 18.0.0 --create-namespace --wait --debug
```

## Install/Upgrade prometheus with custom values

- Create a `values.yaml` file with custom helm chart inputs. Refer to the `values.yaml` file in this repo for sample configurations. 

- Refer to the [official grafana chart](https://github.com/grafana/helm-charts/blob/main/charts/grafana/values.yaml)) for recent configurations. 

Run the following command to install prometheus with custom configurations

```console
helm upgrade -install [RELEASE_NAME] grafana/grafana --namespace [K8S_NAMESPACE] -f values.yaml --create-namespace --wait --debug
```

## Access Grafana

This chart creates a `grafana` service with `ClusterIP` type which is accessible only inside the cluster. Change the [service type](https://github.com/grafana/helm-charts/blob/main/charts/grafana/values.yaml#L173) to `LoadBalancer` if you want to access grafana outside cluster.

Run the following `kubectl port-forward` command to connect to grafana and go to `localhost:3000` in the browser or use the loadbalancer DNS address

```console
kubectl port-forward --namespace [K8S_NAMESPACE] svc/grafana 3000:80
```

You can get the default username and password from the Kubernetes secret

```console
kubectl get secrets grafana --template='{{ range $key, $value := .data }}{{ printf "%s: %s\n" $key ($value | base64decode) }}{{ end }}'
```

Login to grafana with the default username and password

<img width="1920" alt="Screenshot 2023-01-26 at 17 01 01" src="https://user-images.githubusercontent.com/112865563/214902327-833852dc-45cb-4227-b775-e832d4774893.png">


## Configure prometheus Datasource

Follow the below steps to configure the prometheus data source and access the data stored in prometheus 

Goto Datasources -> Add data source -> Select Prometheus -> configure the datasource with prometheus endpoint -> Click "Save & Exit"

<img width="1049" alt="Screenshot 2023-01-26 at 17 03 37" src="https://user-images.githubusercontent.com/112865563/214902391-3d27bbfc-9416-4818-a784-61ffdbac2b05.png">

You should see a Successful "Data source is working" message if the prometheus endpoint is configured as expected.

<img width="1920" alt="Screenshot 2023-01-26 at 17 11 29" src="https://user-images.githubusercontent.com/112865563/214902736-be4fe1bf-dc69-48ff-8c92-5e073935fcd6.png">

You can now able to query the metrics in grafana and create dashboards/setup alerts.

<img width="1920" alt="Screenshot 2023-01-26 at 17 05 38" src="https://user-images.githubusercontent.com/112865563/214902492-a9bf84ef-6d08-4f9e-8077-210cfaaa6546.png">
