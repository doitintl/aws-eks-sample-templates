# deploy-Elasticsearch-Filebeat-Kibana-stack

This repository contains sample code to deploy a self-managed elastic search cluster in EKS with kibana and filebat.

The required resources are created and managed via [Elastic Cloud on Kubernetes (ECK)](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-overview.html) which is a Kubernetes Operator to orchestrate Elastic applications (Elasticsearch, Kibana, APM Server, Enterprise Search, Beats, Elastic Agent, and Elastic Maps Server) on Kubernetes.

The ECK operator relies on a set of Custom Resource Definitions (CRD) to declaratively define how each application is deployed. ECK simplifies deploying the whole Elastic stack on Kubernetes, giving us tools to automate and streamline critical operations.

It focuses on streamlining all those critical operations such as, Managing and monitoring multiple clusters, Upgrading to new stack versions with ease, Scaling cluster capacity up and down, Changing cluster configuration, Dynamically scaling local storage (includes Elastic Local Volume, a local storage driver), Scheduling backups etc.

The sample configurations are based on elastic search version 8.6.1 and. Refer to the official documentation for version details.

## Prerequisites

- [EKS cluster with Kubernetes 1.22+](https://github.com/doitintl/aws-eks-devops-best-practices/tree/main/00-create-eks-cluster)
- [EKS cluster with Amazon EBS CSI Driver](https://docs.aws.amazon.com/eks/latest/userguide/managing-ebs-csi.html) (Note: The add-on is deployed as part of cluster creation if you have used [create-eks-cluster](https://github.com/doitintl/aws-eks-devops-best-practices/tree/main/00-create-eks-cluster)]
- Attach `AmazonEBSCSIDriverPolicy` to the worker node role
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Install ECK Operator

To deploy Elasticsearch on Kubernetes, first we need to install the ECK operator in the Kubernetes cluster.

There are two main ways to install the ECK in a Kubernetes cluster, 1) Install ECK using the YAML manifests, and 2) Install ECK using the Helm chart. This installation is based on YAML manifests.

Run the following command to install the custom resource definitions for ECK operator version 2.6.1

    kubectl create -f https://download.elastic.co/downloads/eck/2.6.1/crds.yaml

Install the operator with its RBAC rules:

    kubectl apply -f https://download.elastic.co/downloads/eck/2.6.1/operator.yaml


Verify the ECK operator installtion and ensure the workload is running as expected

```
❯❯ kubectl get crd
NAME                                                   CREATED AT
agents.agent.k8s.elastic.co                            2023-02-10T16:19:25Z
apmservers.apm.k8s.elastic.co                          2023-02-10T16:19:26Z
beats.beat.k8s.elastic.co                              2023-02-10T16:19:27Z
elasticmapsservers.maps.k8s.elastic.co                 2023-02-10T16:19:28Z
elasticsearchautoscalers.autoscaling.k8s.elastic.co    2023-02-10T16:19:29Z
elasticsearches.elasticsearch.k8s.elastic.co           2023-02-10T16:19:30Z
enterprisesearches.enterprisesearch.k8s.elastic.co     2023-02-10T16:19:31Z
kibanas.kibana.k8s.elastic.co                          2023-02-10T16:19:32Z

❯❯ kubectl get all -n elastic-system
NAME                     READY   STATUS    RESTARTS   AGE
pod/elastic-operator-0   1/1     Running   0          74s

NAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/elastic-webhook-server   ClusterIP   10.100.206.147   <none>        443/TCP   76s

NAME                                READY   AGE
statefulset.apps/elastic-operator   1/1     78s

#Monitor the operator pod logs
❯❯ kubectl logs -f -n elastic-system pod/elastic-operator-0
```

## Deploy Elasticsearch Cluster

Now that ECK is running in the Kubernetes cluster, let's deploy an elastic search cluster with 1 Master node and 2 Data node pods in the `default` namespace.

Refer to `elasticsearch.yaml` for [elatsicsample configurations and [ECK documentation](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-orchestrating-elastic-stack-applications.html) for all the available configuration options. Customize the configurations based on your requirements.


    kubectl apply -f elasticsearch.yaml


Verify the installation

```
❯❯ kubectl get statefulset,pods,sc,pv,pvc

NAME                               READY   AGE
statefulset.apps/demo-es-data      2/2     94s
statefulset.apps/demo-es-masters   1/1     94s

NAME                    READY   STATUS    RESTARTS   AGE
pod/demo-es-data-0      1/1     Running   0          95s
pod/demo-es-data-1      1/1     Running   0          95s
pod/demo-es-masters-0   1/1     Running   0          95s

NAME                                        PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  2d16h

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                          STORAGECLASS   REASON   AGE
persistentvolume/pvc-7f3961d0-e227-4e63-a4bb-48178b33d37e   10Gi       RWO            Delete           Bound    default/elasticsearch-data-demo-es-data-1      gp2                     93s
persistentvolume/pvc-b553429d-fe34-4c7b-bcf0-eb8262144408   10Gi       RWO            Delete           Bound    default/elasticsearch-data-demo-es-data-0      gp2                     93s
persistentvolume/pvc-d37144db-fba3-48a8-a866-b882be011d3a   10Gi       RWO            Delete           Bound    default/elasticsearch-data-demo-es-masters-0   gp2                     93s

NAME                                                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/elasticsearch-data-demo-es-data-0      Bound    pvc-b553429d-fe34-4c7b-bcf0-eb8262144408   10Gi       RWO            gp2            98s
persistentvolumeclaim/elasticsearch-data-demo-es-data-1      Bound    pvc-7f3961d0-e227-4e63-a4bb-48178b33d37e   10Gi       RWO            gp2            98s
persistentvolumeclaim/elasticsearch-data-demo-es-masters-0   Bound    pvc-d37144db-fba3-48a8-a866-b882be011d3a   10Gi       RWO            gp2            98s

```

Elasticsearch svc `demo-es-http` is created as a ClusterIP service and accessible only within the cluster. Change the service type to [loadbalancer](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-services.html#k8s-allow-public-access) if required.

A default user named elastic is automatically created with the password stored in a Kubernetes secret:


    ELASTIC_USER_PASSWORD=$(kubectl get secret demo-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')


Test the elasticsearch connection from your local work station

    kubectl port-forward service/demo-es-http 9200


Disabling certificate verification using the -k flag is not recommended and should be used for testing purposes only

    curl -u "elastic:$ELASTIC_USER_PASSWORD" -k "https://localhost:9200"

Sample output:

```
{
  "name" : "demo-es-masters-0",
  "cluster_name" : "demo",
  "cluster_uuid" : "9js8hJuAQhmdXv7p1C-YJw",
  "version" : {
    "number" : "8.6.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "180c9830da956993e59e2cd70eb32b5e383ea42c",
    "build_date" : "2023-01-24T21:35:11.506992272Z",
    "build_snapshot" : false,
    "lucene_version" : "9.4.2",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## Deploy Kibana Cluster

The next step is to deploy kibana, a free and open user interface that lets you visualize your Elasticsearch data.

Refer to `kibana.yaml` for sample configurations and refer to [eck documents](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-kibana-es.html) for all available configuration options.

    kubectl apply -f kibana.yaml

Verify the installation 

```
❯❯ kubectl get all -l "common.k8s.elastic.co/type=kibana"

NAME                           READY   STATUS    RESTARTS   AGE
pod/demo-kb-799d67ffff-7tftp   1/1     Running   0          2m49s

NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)          AGE
service/demo-kb-http   LoadBalancer   10.100.235.170   a8cad8cba32374f788b1744cb79c46d5-2001568967.us-west-2.elb.amazonaws.com   5601:30645/TCP   2m54s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/demo-kb   1/1     1            1           2m54s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/demo-kb-799d67ffff   1         1         1       2m53s
```

Access the kibana application via the network loadbalancer and use the default elasticsearch username and password. (Ex: https://a8cad8cba32374f788b1744cb79c46d5-2001568967.us-west-2.elb.amazonaws.com:5601)

![Screenshot 2023-02-13 at 11 26 07](https://user-images.githubusercontent.com/112865563/218456889-44b9a760-cd5b-4a78-ab27-3fc91072eca2.jpeg)

![Screenshot 2023-02-13 at 11 27 29](https://user-images.githubusercontent.com/112865563/218456816-c55e8e56-d7c1-4579-87b5-e10ef22582e4.jpeg)

## Deploy Filebeat

Filebeat is a lightweight shipper for forwarding and centralizing log data.It is installed as a daemonset in the kubernetes cluster and collects the logs generated by the containers. The collected logs are shipped to elasticsearch and indexed. You can then query the logs via kibana or elasticsearch API.

Refer to `filebeat.yaml` for sample configurations and refer to [eck documents](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-beat-configuration.html) for all available configuration options.

    kubectl apply -f filebeat.yaml

Verify the installation

```
❯❯ kubectl get pods -l "beat.k8s.elastic.co/name=demo"

NAME                       READY   STATUS    RESTARTS   AGE
demo-beat-filebeat-4xvs2   1/1     Running   0          2m53s
demo-beat-filebeat-7s8f5   1/1     Running   0          2m53s
Chimbus-MBP:05-deploy-ELK-stack chimbu$ kubectl get all -l "beat.k8s.elastic.co/name=demo"
NAME                           READY   STATUS    RESTARTS   AGE
pod/demo-beat-filebeat-4xvs2   1/1     Running   0          3m2s
pod/demo-beat-filebeat-7s8f5   1/1     Running   0          3m2s

NAME                                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/demo-beat-filebeat   2         2         2       2            2           <none>          3m5s

```

Login to kibana and create a data view to explore the collected data. Also deploy a sample application and explore the log data.

![Screenshot 2023-02-13 at 11 27 52](https://user-images.githubusercontent.com/112865563/218457017-a8cf3742-9a61-4123-8cf3-973ba17b1a0c.jpeg)

![Screenshot 2023-02-13 at 11 42 33](https://user-images.githubusercontent.com/112865563/218457038-6970def8-a1ea-4b31-8aaf-53cc260fe0a8.jpeg)

![Screenshot 2023-02-13 at 11 25 22](https://user-images.githubusercontent.com/112865563/218457121-387c3594-e416-490a-8bf3-5ac9fe5be808.jpeg)

![Screenshot 2023-02-13 at 11 25 42](https://user-images.githubusercontent.com/112865563/218457163-e8eddb76-2e69-4d4c-926f-cfef60c05332.jpeg)
