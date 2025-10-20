+++
title = "AWS services — Identity Access Management (IAM)"
description = "A high-level overview of the AWS service IAM"
date = 2023-08-16T01:25:33+05:30

[taxonomies]
tags = ["aws", "iam", "cloud", "identity-management"]

[extra]
toc = true
+++

![AWS — IAM service](https://cdn-images-1.medium.com/max/3840/1*v14tIyldZ_KXA-2riQo0qw.jpeg)

In this blog post, we are going to take a look at one of the most important and most used AWS services — [IAM](https://aws.amazon.com/iam/). It allows for securely managing identities and providing access to AWS services and resources.

![IAM service — high level](https://cdn-images-1.medium.com/max/3520/0*KA7HCzTnGiU0-ryL.png)

## How does it work?

With AWS — Identity and Access Management (IAM), you can specify who or what can access services and resources in AWS by using the following components.

![IAM workflow](https://cdn-images-1.medium.com/max/2000/1*S8__sG96zt94WCZBHiMjUQ.png)

### **1. [Users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html):**

Users represent individual entities (like people, applications, or services within AWS) that can interact with your AWS resources like EC2 instances or S3 buckets. Each user is assigned a unique username and access credentials (password or access keys) for authentication.

### 2. [Groups](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups.html):

Groups are collections of users (eg: Dev group or QA group). Assigning permissions to groups makes it easier than managing access for multiple users with similar roles or responsibilities. This simplifies the process of granting and revoking permissions across multiple users.

### 3. [Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html):

Roles are similar to users, but they are not associated with a specific identity. Instead, roles are assumed by resources, such as EC2 instances, Lambda functions, EKS cluster, or applications running on Azure. This enables these resources to access other AWS services securely without needing long-term credentials.

### 4. [Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html):

Permissions define what actions are allowed or denied to access AWS resources. By default, all the permissions are denied. Permissions are managed through policies, which are JSON documents that outline the permissions in a structured way. Policies can be attached to users, groups, and roles.

    Permission Name: ListBucketObjects
    Action: s3:ListBucket
    Resource: arn:aws:s3:::my-bucket

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:ListBucket",
          "Resource": "arn:aws:s3:::my-bucket"
        }
      ]
    }

### 5. [Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html):

A policy is a document that defines permissions in a granular manner. AWS provides managed policies that cover common use cases, and custom policies also can be created. Policies can be attached at various levels (user, group, role, or resource) to control access.

> #### One of the main advantage of using **_IAM services_** is helping us to grant [least privilege access](https://en.wikipedia.org/wiki/Principle_of_least_privilege) or granting only the permissions required to perform a task

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/aws-services-identity-access-management-iam-bcdf23bd0035)

</center>

**Reference**:

[1] [https://aws.amazon.com/iam/](https://aws.amazon.com/iam/)
