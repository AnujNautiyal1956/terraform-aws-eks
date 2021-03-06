# Amazon EKS (Elastic Kubernetes Service)
[Amazon EKS](https://aws.amazon.com/eks/) is a fully managed Kubernetes service. Customers trust EKS to run their most sensitive and mission critical applications because of its security, reliability, and scalability.

* This module will create an EKS cluster on AWS. It will have a control plane and you can register multiple heterogeneous node groups as data plane.
* This module will give you a utility bash script to set up a kubernetes configuration file to access the EKS cluster.
* This module has several sub-modules to deploy kubernetes controllers and utilities using helm provider.

## Examples
- [Amazon EKS Autoscaling](https://github.com/Young-ook/terraform-aws-eks/blob/main/examples/autoscaling)
- [Amazon EKS on AWS Fargate](https://github.com/Young-ook/terraform-aws-eks/blob/main/examples/fargate)
- [Amazon EKS on AWS Graviton](https://github.com/Young-ook/terraform-aws-eks/blob/main/examples/arm64)
- [Amazon EKS with Spot Instances](https://github.com/Young-ook/terraform-aws-eks/blob/main/examples/spot)
- [Amazon ECR](https://github.com/Young-ook/terraform-aws-eks/blob/main/examples/ecr)

## Quickstart
### Setup
```hcl
module "eks" {
  source  = "Young-ook/eks/aws"
  name    = "eks"
  tags    = { env = "test" }
}
```
Run terraform:
```
$ terraform init
$ terraform apply
```
## Generate kubernetes config
This terraform module provides users with a shell script that extracts the kubeconfig file of the EKS cluster. When users run the terraform init command in their workspace, the script is downloaded with the terraform module from the terraform registry. User can see how to run this script in terraform output after terraform apply command completes successfully. Using this script, users can easily obtain a kubeconfig file. So, they can use this kubeconfig file for access to the EKS cluster (with Spinnaker). The original script is here [update-kubeconfig.sh](https://github.com/Young-ook/terraform-aws-eks/blob/main/script/update-kubeconfig.sh) and users can check out the details of the script.

**[Important]** Before you run this script you must configure your local environment to have proper permission to get the credentials from EKS cluster on your AWS account whatever you are using aws-cli or aws-vault.

## IAM Role for Service Account
After then you will see the created EKS cluster and node groups and IAM role. For more information about configuration of service account mapping for IAM role in Kubernetes, please check out the [IAM Role for Service Accounts](https://github.com/Young-ook/terraform-aws-eks/blob/main/modules/iam-role-for-serviceaccount/README.md)

# Known Issues
## Unauthorized
You might get an error message when this module tries to create a `aws-auth` configuration map for a new EKS cluster. When prompted, re-apply the terraform configuration. Here is an example error message:
```
module.eks.kubernetes_config_map.aws-auth[0]: Creating...

Error: Unauthorized

  on .terraform/modules/eks/main.tf line 341, in resource "kubernetes_config_map" "aws-auth":
 341: resource "kubernetes_config_map" "aws-auth" {
```

## Configmap already exist
If you are trying to replace a managed nodegroup with a (self-managed) nodegroup, you may get an error message as this module tries to generate the `aws-auth` config map. This is because the managed nodegroup resource does not delete the `aws-auth` configmap when it is removed, but the self-managed nodegroup needs the `aws-auth` configmap for node registration, which causes a conflict. When prompted, delete exsiting `aws-auth` configmap using kubectl and retry the terraform apply command. Here is an example error message:
```
module.eks.kubernetes_config_map.aws-auth[0]: Creating...

Error: configmaps "aws-auth" already exists

  on .terraform/modules/eks/main.tf line 343, in resource "kubernetes_config_map" "aws-auth":
 343: resource "kubernetes_config_map" "aws-auth" {
```
