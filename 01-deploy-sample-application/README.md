# deploy-sample-application

Follow the instructions after the EKS cluster configurations are completed. We will deploy a few core kubernetes objects and provision an external loadbalancer to access the application outside the EKS cluster.

## Prerequisites

- An active EKS cluster and kubectl is configured to the correct EKS cluster
- Clone this repo to your local and set the current working directory to the cloned repo

## Create a Namespace

In Kubernetes namespaces provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Run the below command to create namespace

    kubectl create namespace eks-sample-app

## Create a Kubernetes deployment

A Kubernetes Deployment tells Kubernetes how to create or modify instances of the pods that hold a containerized application. Deployments can help to efficiently scale the number of replica pods, enable the rollout of updated code in a controlled manner, or roll back to an earlier deployment version if necessary. To learn more, see Deployments in the Kubernetes documentation.

Apply the deployment manifest to your cluster.

    kubectl apply -f 01-deployment.yaml

Review the deployment configurations.

    kubectl describe deployments.apps --namespace eks-sample-app eks-sample-deployment


## Create a service

A service allows you to access all replicas through a single IP address or name. For more information, see Service in the Kubernetes documentation.

There are different types of Service objects, and the one we want to use for testing is called LoadBalancer, which means an external load balancer. Amazon EKS has support for the LoadBalancer type using the class Elastic Load Balancer (ELB). EKS will automatically provision and de-provision a ELB when we create and destroy service objects.

Apply the service manifest to your cluster.

	kubectl apply -f 02-service.yaml

View all resources that exist in the eks-sample-app namespace.

	kubectl get all --namespace eks-sample-app

<img width="1039" alt="Screenshot 2023-01-15 at 11 07 31" src="https://user-images.githubusercontent.com/112865563/212537162-b61bdc66-7f48-4b25-9c56-f977de65edcd.png">

You can see AWS automatically provisioned an external loadbalancer for the service type loadbalancer and you can access the application outside the cluster with the DNS name available under the EXTERNAL-IP field.

<img width="1046" alt="Screenshot 2023-01-15 at 11 07 37" src="https://user-images.githubusercontent.com/112865563/212537170-abd8ca8a-9e9b-438d-9bed-56f8a7ffd857.png">


## Deploy a new application version

In kubernetes you can easily deploy a new version of an existing deployment by updating the image details.

Apply `03-update-deployment.yaml` deployment manifest to your cluster.

    kubectl apply -f 03-update-deployment.yaml

Kubernetes performs a rolling update by default to minimize the downtime during upgrades and create a replica set and pods.
Review the deployment configurations and verify the image details

    kubectl describe deployments.apps --namespace eks-sample-app   eks-sample-deployment

Once you're finished with the sample application, you can remove the sample namespace, service, and deployment with the following command.

    kubectl delete namespace eks-sample-app
