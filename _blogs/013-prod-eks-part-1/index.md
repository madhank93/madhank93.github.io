+++
title = "Production ready EKS cluster setup — Part I"
description = "Using AWS CloudFormation infrastructure as a tool to provision the EKS cluster"
date = 2023-08-11T01:25:33+05:30

[taxonomies]
tags = ["eks", "aws", "cloud-formation", "iac", "k8s"]

[extra]
toc = true
+++

![AWS EKS — CloudFormation](https://cdn-images-1.medium.com/max/3840/1*ohvTASQBpHSRzpUQdffJEg.png)

In this article, we are going to see how to create a Production ready EKS cluster by following the steps using the CloudFormation template and AWS dashboard.

![Overall steps](https://cdn-images-1.medium.com/max/2000/1*Mdp6f6Lyckfgf5LDA3tb6A.png)

### [1. Setting up networking components](https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html)

When it comes to networking setup Amazon EKS service has specific [requirements and considerations](https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html) for the VPC and subnets in the cluster being deployed. And AWS also [maintains](https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html) a CloudFormation [template](https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml) that can help us create VPC and subnets.

Head on to CloudFormation section, choose to Create stack option, and add the following AWS S3 URL template it has the option of creating subnets in both public and private.

    https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml

After uploading the template you’ll have the option to customize the CIDR blocks. And I’m naming this stack has the `eks-vpc-stack`.

![VPC parameter config](https://cdn-images-1.medium.com/max/2810/1*6-0xIfJAUpO1hjIqZV-4yg.png)

### [2. Creating an IAM role](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html)

AWS-managed Kubernetes EKS service needs to access other AWS services on your behalf to manage the resources. For that, we need to create an IAM role with access to [AmazonEKSClusterPolicy](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/AmazonEKSClusterPolicy.html) and I’m naming it as `eks-cluster-role`

![IAM role](https://cdn-images-1.medium.com/max/2662/1*ZBLIYJaZF6udwYbKks7AUw.png)

### 3. Creating an EKS cluster

Upon creating the EKS cluster make sure to select the previously created VPC & IAM role as follows and complete the setup.

![](https://cdn-images-1.medium.com/max/2032/1*Zkhn-zWFqXVsuDp_PxNU8Q.png)

![EKS setup](https://cdn-images-1.medium.com/max/2000/1*J3UXDgM4bFz8MV91OeNxtg.png)

Once the setup is completed, update the kube config details using AWS CLI. You can now access the namespace but not the Kubernetes nodes since we haven’t configured the worker nodes yet.

![AWS kubeconfig setup](https://cdn-images-1.medium.com/max/3208/1*5TltVWPbtBmgPkNpfXIMyA.png)

### [4. Creating self-managed worker nodes](https://docs.aws.amazon.com/eks/latest/userguide/launch-workers.html)

Go to CloudFormation stack and add the following template to it.

    https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2022-12-23/amazon-eks-nodegroup.yaml

In the parameter section, provide the `ClusterName` matching the EKS cluster name we’ve created and selecting the appropriate VPC, Subnets, Security Group, and SSH key pair.

### 5. Joining Worker nodes to the Control plane

To attach the worker nodes to the control plane, modify the following yml file key `rolearn` with the arn value of `NodeInstanceRole` (created as part of the Worker node CloudFormation template). And apply the k8s file to attach the worker nodes to the control plane.

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: aws-auth
      namespace: kube-system
    data:
      mapRoles: |
        - rolearn: arn:aws:iam::942487838076:role/eks-worker-node-NodeInstanceRole-1B9EVIY7CEYD8
          username: system:node:{{EC2PrivateDNSName}}
          groups:
            - system:bootstrappers
            - system:nodes

Once the above k8s file is applied successfully; we can able to get all the nodes by running the below command.

![EKS — nodes](https://cdn-images-1.medium.com/max/3206/1*Y1PJS3RKlp5X213FW6rkIA.png)

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/production-ready-eks-cluster-setup-part-i-49a4eba171cc)

</center>

**References:**

[1] [https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
