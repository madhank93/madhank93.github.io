+++
title = "Production-ready EKS cluster setup — Part II"
description = "Using Terraform IaC tool to provision the AWS EKS cluster"
date = 2023-08-20T01:25:33+05:30

[taxonomies]
tags = ["eks", "k8s", "aws", "terraform", "iac"]

[extra]
toc = true
+++

![Banner](https://cdn-images-1.medium.com/max/3840/1*hNt_R_7aZHJr9bXZU8Tc0g.png)

As a continuation of my [previous blog](https://medium.com/@madhankumaravelu93/production-ready-eks-cluster-setup-part-i-49a4eba171cc) series on production-level EKS cluster setup, in this, we are going to see how to create an [AWS EKS](https://aws.amazon.com/eks/) cluster using [Terraform](https://www.terraform.io/).

Terraform is an open-source infrastructure tool to provision and manage infrastructure in any cloud. In this article, we’ll make use of the following features of TF

- [Modules](https://developer.hashicorp.com/terraform/language/modules) — Are a collection of `.tf` files and/or `.tf.json` files kept together in a directory. It is the main way to package and reuse resource configurations. For this article, we’ll make use of [VPC](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest) and [EKS](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest) created and maintained by the [community](https://github.com/terraform-aws-modules).

- [Workspaces](https://developer.hashicorp.com/terraform/language/state/workspaces) — Each [state file](https://developer.hashicorp.com/terraform/language/state/purpose) (`.tfstate`) stored in the backend belongs to a workspace. Some [backend allows](https://developer.hashicorp.com/terraform/language/state/workspaces#backends-supporting-multiple-workspaces) the creation of multiple named workspaces in this case we are using [S3](https://developer.hashicorp.com/terraform/language/settings/backends/s3). Initially, it creates the default workspace. And this approach might not be suitable for some cases like deployments requiring separate credentials and access controls. Use cases and limitations can be [referred to here](https://developer.hashicorp.com/terraform/cli/workspaces#use-cases).

![TF — workspace flow](https://cdn-images-1.medium.com/max/2090/1*6gsqQOrWch_mmqp_CnHL7A.png)

## Code walk-through

### 1. Providers

Terraform relies on plugins called providers to interact with cloud providers, SaaS providers, and other APIs.

```t
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.7.0"
    }
    tls = {
      source  = "hashicorp/tls"
      version = "4.0.4"
    }
  }
  required_version = "~> 1.3"

  # State storage
  backend "s3" {
    bucket = "my-eks-cluster-01"
    region = "us-east-1"
    key    = "terraform.tfstate"
  }
}
```

In the above snippet, we are making use of [TLS](https://registry.terraform.io/providers/hashicorp/tls/latest/docs) and [AWS](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) providers to interact with the resources supported by AWS. Authenticating to AWS cloud provider can be referred to [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication-and-configuration).

### 2. Modules and Resources

[Resources](https://developer.hashicorp.com/terraform/language/resources) are the key component in Terraform, resource block describes one or more infrastructure objects, such as virtual networks, compute instances, or higher-level components such as DNS records.

```t
# Creating VPC configs
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"

  cidr = "10.0.0.0/16"
  azs  = slice(data.aws_availability_zones.available.names, 0, 3)

  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]

  enable_nat_gateway     = true
  single_nat_gateway     = true
  one_nat_gateway_per_az = false

  enable_dns_hostnames = true
  enable_dns_support   = true

  public_subnet_tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
    "kubernetes.io/role/elb"                      = 1
  }

  private_subnet_tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb"             = 1
  }
}

# IAM Role for EKS Cluster Group
resource "aws_iam_role" "eks_cluster_role" {
  name = "${local.cluster_name}-eks-cluster-role"
    assume_role_policy = <<POLICY
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Service": "eks.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
POLICY
}

resource "aws_iam_role_policy_attachment" "AmazonEKSClusterPolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.eks_cluster_role.name
}

# IAM Role for EKS Node Group
resource "aws_iam_role" "eks_nodegroup_role" {
  name = "${local.cluster_name}-eks-nodegroup-role"

  assume_role_policy = jsonencode({
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
    Version = "2012-10-17"
  })
}

resource "aws_iam_role_policy_attachment" "eks-AmazonEKSWorkerNodePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
  role       = aws_iam_role.eks_nodegroup_role.name
}

resource "aws_iam_role_policy_attachment" "eks-AmazonEKS_CNI_Policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
  role       = aws_iam_role.eks_nodegroup_role.name
}

resource "aws_iam_role_policy_attachment" "eks-AmazonEC2ContainerRegistryReadOnly" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  role       = aws_iam_role.eks_nodegroup_role.name
}

# Creating EKS cluster with managedd node groups
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.15.3"

  cluster_name    = local.cluster_name
  cluster_version = "1.27"

  vpc_id                         = module.vpc.vpc_id
  subnet_ids                     = module.vpc.private_subnets
  cluster_endpoint_public_access = true

  cluster_iam_role_arn = aws_iam_role.eks_cluster_role.arn

  eks_managed_node_group_defaults = {
    ami_type     = "AL2_x86_64"
    iam_role_arn = aws_iam_role.eks_nodegroup_role.iam_role_arn
  }

  eks_managed_node_groups = {
    one = {
      name = "node-group-1"

      instance_types = ["t3.small"]

      min_size     = 1
      max_size     = 3
      desired_size = 2
    }

    two = {
      name = "node-group-2"

      instance_types = ["t3.small"]

      min_size     = 1
      max_size     = 2
      desired_size = 1
    }
  }
}

resource "aws_iam_openid_connect_provider" "eks" {
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = [data.tls_certificate.eks.certificates[0].sha1_fingerprint]
  url             = module.eks.cluster_oidc_issuer_url
}
```

In the above snippet, we made use of the aws_iam_role and aws_iam_role_policy_attachment resources to create an IAM role and attach it to the policy that has been created.

### 3. Outputs

Output values print the information about the infrastructure on the command line, and can expose information for other Terraform configurations to use. It is similar to return values in programming languages.

```t
output "cluster_endpoint" {
  description = "Endpoint for EKS control plane"
  value       = module.eks.cluster_endpoint
}

output "cluster_security_group_id" {
  description = "Security group ids attached to the cluster control plane"
  value       = module.eks.cluster_security_group_id
}

output "region" {
  description = "AWS region"
  value       = var.region
}

output "cluster_name" {
  description = "Kubernetes Cluster Name"
  value       = module.eks.cluster_name
}
```

## Executing

Firstly create the workspace using `terraform workspace new dev` the command from the CLI. Then apply the terraform changes using `terraform apply --auto-approve` the command to create the resources.

![](https://cdn-images-1.medium.com/max/3830/1*AkGeMlDYFHqMdNjmtcMvSA.png)

![](https://cdn-images-1.medium.com/max/2000/1*3r-omqFmn-WVaJK_U8UQ9g.png)

![Terraform s3 backend, output, and EKS cluster](https://cdn-images-1.medium.com/max/3840/1*G9TJl_Aqnx4aguCur9f3sw.png)

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/production-ready-eks-cluster-setup-part-ii-f702542cde7c)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/learn-tf/aws/provision-eks-cluster](https://github.com/madhank93/learn-tf/tree/main/aws/provision-eks-cluster)

</center>

**Reference**:

[1] [https://terraform.io/](https://terraform.io/)

[2] [https://registry.terraform.io/](https://registry.terraform.io/)

[3] [https://github.com/terraform-aws-modules](https://github.com/terraform-aws-modules)
