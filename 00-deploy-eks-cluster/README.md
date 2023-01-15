# deploy-eks-cluster

This example is based on eksctl which is a simple CLI tool for creating and managing clusters on EKS.EKS Clusters can be deployed and managed with a number of solutions including Terraform, Cloudformation,AWS Console and AWS CLI.

## Prerequisites

- An active AWS account
- VPC - eksctl creates a new vpc named eksctl-my-demo-cluster-cluster/VPC in the target region
- IAM permissions – The IAM security principal that you're using must have permissions to work with Amazon EKS IAM roles and service-linked roles, AWS CloudFormation, and a VPC and related resources.
- Install kubectl,eksctl and AWS CLI in your local machine or in the CICD setup

### IAM Setup

- For this setup, create an IAM policy name AmazonEKSAdminPolicy with the policy details in `01-AmazonEKSAdminPolicy.json` and attach the policy to the principal

### Creating an EKS cluster

The eksctl tool uses CloudFormation under the hood, creating one stack for the EKS master control plane and another stack for the worker nodes.

Run the command below to create a new cluster in the `us-west-2` region; expect this to take around 20 minutes and refer to eksctl documentation for all available options to customize the cluster configurations.

	eksctl create cluster \
    --version 1.23 \
    --region us-west-2 \
    --node-type t3.medium \
    --nodes 3 \
    --nodes-min 1 \
    --nodes-max 4 \
 	--name my-demo-cluster 

As an alternative, you can also use YAML, as sort of DSL (domain specific language) script for creating Kubernetes clusters with EKS.

Run the below command to create the cluster (expect this to take around 20 minutes):
		
    eksctl create cluster -f 02-demo-cluster.yaml






