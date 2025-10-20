+++
title = "Production-ready EKS cluster setup — Part III"
description = "Using the Pulumi IaC tool to provision the AWS EKS cluster"
date = 2023-08-22T01:25:33+05:30

[taxonomies]
tags = ["eks", "k8s", "aws", "pulumi", "ts", "iac"]

[extra]
toc = true
+++

![Banner](https://cdn-images-1.medium.com/max/3840/1*WaJ5Ibd4WbRHvXCB3jRASA.jpeg)

As a continuation to my blog series of [Part-I](https://medium.com/@madhankumaravelu93/production-ready-eks-cluster-setup-part-i-49a4eba171cc), and [Part-II](https://medium.com/@madhankumaravelu93/production-ready-eks-cluster-setup-part-ii-f702542cde7c) on production-level EKS cluster setup, in this, we are going to see how to create an [AWS EKS](https://aws.amazon.com/eks/) cluster using [Pulumi](https://www.pulumi.com/) as the IaC and [Typescript](https://www.typescriptlang.org/) as the programming language.

[Pulumi](https://www.pulumi.com/) is a modern [Infrastructure as Code](https://www.pulumi.com/what-is/what-is-infrastructure-as-code/) platform that allows you to use familiar programming languages and tools to build, deploy, and manage cloud infrastructure.

In a similar fashion, we will break this article into smaller parts for better understanding. Let’s dive directly into the coding part.

### 1. Starting with the configuration

```ts
// Grab some values from the Pulumi configuration (or use default values)
const config = new pulumi.Config();
const minClusterSize = config.getNumber("minClusterSize") || 3;
const maxClusterSize = config.getNumber("maxClusterSize") || 6;
const desiredClusterSize = config.getNumber("desiredClusterSize") || 3;
const eksNodeInstanceType = config.get("eksNodeInstanceType") || "t3.medium";
const vpcNetworkCidr = config.get("vpcNetworkCidr") || "10.0.0.0/16";
```

In many cases, a single project will have a different configuration based on the environments we choose to run. For example, Dev instance configs will differ from Prod instances. Such key-value pairs config can be maintained at Pulumi.<stack-name>.yml instead of hard-coding it.

### 2. Creating a VPC

```ts
// Creating VPC
const eksVpc = new aws.ec2.Vpc("eks-vpc", {
  tags: {
    name: "my-eks-vpc",
  },
  cidrBlock: vpcNetworkCidr,
});

// Subnet cidr block range
const privateSubnetCidrBlock = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"];
const publicSubnetCidrBlock = [
  "10.0.101.0/24",
  "10.0.102.0/24",
  "10.0.103.0/24",
];

// Helper function to create subnets
const createSubnetAndGetID = (
  cidrRange: string,
  index: number,
  isPublic: boolean
) =>
  new aws.ec2.Subnet(`${isPublic ? "public" : "private"}-subnet-${index}`, {
    vpcId: eksVpc.id,
    cidrBlock: cidrRange,
    mapPublicIpOnLaunch: isPublic,
    availabilityZone: index / 2 == 0 ? "us-west-2a" : "us-west-2b",
    tags: {
      name: "my-eks-subnets",
    },
  }).id;

// Create private subnet
const privateSubnet = privateSubnetCidrBlock.map((subnet, index) =>
  createSubnetAndGetID(subnet, index, false)
);

// Create public subnet
const publicSubnet = publicSubnetCidrBlock.map((subnet, index) =>
  createSubnetAndGetID(subnet, index, true)
);
```

The above code snippet will take care of the VPC cluster creation along with the private and public subnets spanned across different availability zones.

### 3. IAM role for EKS cluster

```ts
// IAM role for EKS cluster
const eksClusterRole = new aws.iam.Role("eks-cluster-role", {
  assumeRolePolicy: JSON.stringify({
    Version: "2012-10-17",
    Statement: [
      {
        Action: "sts:AssumeRole",
        Effect: "Allow",
        Principal: {
          Service: "eks.amazonaws.com",
        },
      },
    ],
  }),
});

// Attach the AmazonEKSClusterPolicy to the role
new aws.iam.RolePolicyAttachment("cluster-role-policy-attachment", {
  role: eksClusterRole.name,
  policyArn: "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy",
});
```

Creating an IAM role and associating it with the policy AmazonEKSClusterPolicy , since Kubernetes clusters managed by Amazon EKS need access to other AWS services.

### 4. EKS cluster

```ts
// Creating EKS cluster
const eksCluster = new aws.eks.Cluster("eks-cluster", {
  roleArn: eksClusterRole.arn,
  vpcConfig: {
    subnetIds: [...privateSubnet, ...publicSubnet],
  },
  version: "1.27",
  tags: {
    name: "my-eks-cluster",
  },
});
```

Creating an EKS cluster in the pre-configured subnets and attaching the eksClusterRole cluster role to it.

### 5. IAM role for Worker Nodes

```ts
// Create an IAM role for EKS worker nodes
const workerNodeRole = new aws.iam.Role("worker-node-role", {
  assumeRolePolicy: JSON.stringify({
    Version: "2012-10-17",
    Statement: [
      {
        Action: "sts:AssumeRole",
        Principal: {
          Service: "ec2.amazonaws.com",
        },
        Effect: "Allow",
      },
    ],
  }),
});

// Attach the EKS Worker Node, CNI and container registry policy
new aws.iam.RolePolicyAttachment("worker-node-policy-attachment", {
  role: workerNodeRole.name,
  policyArn: "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy",
});

new aws.iam.RolePolicyAttachment("cni-policy-attachment", {
  role: workerNodeRole.name,
  policyArn: "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy",
});

new aws.iam.RolePolicyAttachment("container-registry-policy-attachment", {
  role: workerNodeRole.name,
  policyArn: "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly",
});
```

Creating an IAM role for the worker node group and associating it with the following policiesAmazonEC2ContianerRegistryReadOnly, AmazonEKSWorkerNodePolicy, and AmazonEKS_CNI_Policy since the EC2 worker nodes need access to the EKS control plane and pull container images from the AWS registry.

### 6. Worker node group

```ts
// Create a Node Group
new aws.eks.NodeGroup("worker-node-group", {
  clusterName: eksCluster.name,
  nodeRoleArn: workerNodeRole.arn,
  subnetIds: [...privateSubnet, ...publicSubnet],
  scalingConfig: {
    desiredSize: desiredClusterSize,
    maxSize: maxClusterSize,
    minSize: minClusterSize,
  },
  instanceTypes: [eksNodeInstanceType],
});
```

The above code will create the AWS-managed node group and it will take care of attaching the nodes to the EKS control plane with auto-scaling capability.

### 7. Exporting cluster details

```ts
// Export the cluster name, ARN and kubeconfig
export const clusterName = eksCluster.name;
export const clusterArn = eksCluster.arn;
export const clusterKubeConfig = kubeconfig;
export const vpcId = eksVpc.id;
```

**export** is used to display the outputs of your stack to the standard output.

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/production-ready-eks-cluster-setup-part-iii-c629648f9cd0)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[Github madhank93/learn-pulumi/aws/provision-eks-cluster](https://github.com/madhank93/learn-pulumi/tree/main/aws/provision-eks-cluster)

</center>

**Reference:**

[1] [https://www.pulumi.com/docs/clouds/aws/](https://www.pulumi.com/docs/clouds/aws/)

[2] [https://www.pulumi.com/registry/packages/aws/installation-configuration/](https://www.pulumi.com/registry/packages/aws/installation-configuration/)

[3] [https://www.pulumi.com/registry/packages/aws/api-docs/](https://www.pulumi.com/registry/packages/aws/api-docs/)
