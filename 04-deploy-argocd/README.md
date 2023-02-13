# Argo CD

[Argo CD](https://argo-cd.readthedocs.io/en/stable/) is a declarative, GitOps continuous delivery tool for Kubernetes. This repository contains the instructions to deploy argo cd in the EKS cluster and configurations to manage applications in Argo CD.

## Prerequisites

- Kubernetes 1.22+
- kubectl


## Install Argo CD

Follow the [official documentation](https://argo-cd.readthedocs.io/en/stable/getting_started/) to deploy Argo CD to your cluster.

Use the [manifests](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml) file to quickly install Argo CD or use the [helm chart](https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd) to customize the configurations.

Run the command `kubectl get pods -n argocd` to verify the installation and ensure all the pods are in `Running` state.

```console
❯❯ kubectl get pods -n argocd
NAME                                               READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                    1/1     Running   0          136m
argocd-applicationset-controller-bdbc5976d-rsz4p   1/1     Running   0          136m
argocd-dex-server-7c8974cfc9-zq894                 1/1     Running   0          136m
argocd-notifications-controller-56dbd4976-4kdjn    1/1     Running   0          136m
argocd-redis-6bdcf5f74-wdx5v                       1/1     Running   0          136m
argocd-repo-server-5bcc9567f8-5rjfc                1/1     Running   0          136m
argocd-server-5ccfbc6db6-dz8c5                     1/1     Running   0          136m

```

Kubectl port-forwarding can be used to connect to the Argo CD API server without exposing the service.

```console
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

The API server can then be accessed using https://localhost:8080

You can use the loadbalancer address if the service is exposed outside the cluster.

Login to the Argo CD dashboard, The default username is `admin` and run the below command to retrieve the default password.

```console
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

## Deploy Applications

In Argo CD you can deploy the applications using the Helm charts or manifest files stored in a git repository. For this example, we will deploy Prometheus and grafana to the EKS cluster and use the official helm charts. 

You can install Helm charts through the UI, or in the [declarative GitOps way](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/). We recommend you follow the declarative GitOps way to deploy an argo application.

Run the below command to deploy grafana and prometheus applications via Argo CD and explore the deployed configurations in argo CD UI


```console
kubectl apply -f ./applications/grafana/grafana.yaml
kubectl apply -f ./applications/prometheus/prometheus.yaml
```

Change the `targetRevision` version `6.50.5` in `grafana.yaml` file and apply the changes. Argo CD will automatically identify the changes in the configurations and roll out the new version.

## Demo

https://user-images.githubusercontent.com/112865563/215147384-92f62a74-b411-42e9-859a-5896ad870707.mp4
