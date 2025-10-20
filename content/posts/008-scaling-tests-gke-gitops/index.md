+++
title = "Scaling tests on Google Kubernetes Engine with Cloud Build"
description = "A guide to running test automation scripts in distributed env using gitops approach"
date = 2021-10-12T01:24:33+05:30

[taxonomies]
tags = ["k8s", "gke", "gitops", "gcp", "ci", "cloud-build"]

[extra]
toc = true
+++

![Source — google & Built using — canva](https://cdn-images-1.medium.com/max/4480/1*tP8OkC__HFt0ctvKVm3nvw.png)

As we looked at how to run automation scripts in selenoid using [docker](https://www.docker.com/) and [docker-compose](https://docs.docker.com/compose/) in my [previous blog](https://medium.com/testvagrant/running-webdriverio-tests-in-containers-871e0238e31f). In this blog let’s dive into how to run [WebdriverIO](https://webdriver.io/) test automation scripts in scalable mode using [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine) and [Cloud Build](https://cloud.google.com/build).

[Kubernetes](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/) is an open-source system for automating deployment, scaling, and management of containerized applications. **_Google Kubernetes Engine_** — provides a simple way to automatically deploy, scale, and manage Kubernetes. And **_Cloud Build_** — it is a serverless CI/CD platform to build, test, and deploy.

According to the documentation of [Aerokube](https://aerokube.com/), selenoid is [not recommended](https://aerokube.com/selenoid/latest/#_selenoid_in_kubernetes) for the Kubernetes platform. And [Moon](https://aerokube.com/moon/) is pay per use model. So in this article, we are going to explore [Selenosis](https://github.com/alcounit/selenosis) — a scalable, stateless selenium hub for the Kubernetes cluster.

## Selenosis setup walkthrough

### 1. Creating a namespace

[Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) are Kubernetes objects which help in organising resources and partition a single Kubernetes cluster into multiple virtual clusters. It provides a degree of isolation from the other parts of the cluster.

    apiVersion: v1
    kind: Namespace
    metadata:
      name: selenosis

### 2. Creating service for selenosis

[Service](https://kubernetes.io/docs/concepts/services-networking/service/) is an abstract way to expose an application running on a set of Pods as a network service. It creates a permanent IP address, the lifecycle of pod and service are not connected. Even if the pods crash and are recreated, the service IP remains the same.

{{ gist(url="https://gist.github.com/madhank93/c1b7ef110e0c492980e14643de308843", class="gist") }}

### 3. Creating selenosis deployment

[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) describes the desired state of a pod or a replica set, then gradually updates the environment (for example, creating or deleting replicas) until the current state matches the desired state specified in the deployment file. In general, we don’t work directly with pods, we will create deployments. It is mainly for stateless apps.

{{ gist(url="https://gist.github.com/madhank93/b9a526cdc7063e88f7db7b7c790d9ab5", class="gist") }}

### 4. Selenosis HPA

[Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) automatically scales the number of Pods in a replication controller, deployment, replica set or stateful set based on observed CPU utilisation

{{ gist(url="https://gist.github.com/madhank93/58b9e436052ee8f5486f449589411716", class="gist") }}

### 5. Other configs

- Below is the command to convert the [browser](https://gist.github.com/madhank93/4957d448ac1814587392705e377da908#file-browsers-yaml) yaml file into ConfigMap

  kubectl create cm selenosis-config --from-file=browsers.yaml=selenosis-deploy/browsers.yaml -n selenosis

- Use [core-dns.yaml](https://gist.github.com/madhank93/3d6f62ffd72c5a3b31bb13970ebd9bfa#file-06-coredns-yaml) to turn off the DNS caching for the selenosis namespace

- Use [pre-pull-image.yaml](https://gist.github.com/madhank93/9f7329dd203a23a176c0d84b212ce5c7#file-07-pre-pull-images-yml) to pre pull the chrome and firefox browser images

- To debug use [selenoid-ui.yaml](https://gist.github.com/madhank93/e92e5729b10a9dc695da3b464f5c4754#file-04-selenoid-ui-yaml) \*\*\*which enables the UI feature to debug

## Containerizing test scripts

[Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) used to run a finite task(which is a suitable k8s object to use for running test scripts). It will create one or more Pods and performs a given task. Once the task is completed successfully, the pod will be exited.

{{ gist(url="https://gist.github.com/madhank93/7478c960d7ba916537eab94d4c79ec79", class="gist") }}

## Creating a project in GCP

Create a new project in [GCP](https://cloud.google.com/), go to Cloud Build, enable the api. And few other settings as shown in the below picture (to follow along with this tutorial you need a cloud billing account, a [\*free tier](https://cloud.google.com/free/docs/gcp-free-tier)\* is sufficient). And make sure you have the required permissions in the IAM & Admin section.

![](https://cdn-images-1.medium.com/max/5744/1*9DYaVACXkkGlkbADj9c1mA.png)

![1. Cloud build settings, and 2. IAM & Admin configurations](https://cdn-images-1.medium.com/max/5740/1*Zx80UpKCM1U39eDE5FHSKw.png)

## Setting up Cloud build

Go to Cloud Build > Triggers section, click on Create Trigger and add the details as follow. So whenever the code has been pushed to GitHub, cloud build flow will be triggered.

![](https://cdn-images-1.medium.com/max/3248/1*M5yfmgYvTirnI2rGs-KDBg.png)

![](https://cdn-images-1.medium.com/max/2312/1*Sa50IIagaW4QOu6L_1M-ew.png)

![Cloud build setup](https://cdn-images-1.medium.com/max/2572/1*NwPfm264-JF_UJWTaRdE1w.png)

And finally creating cloudbuild.yml file to execute the process of building docker image from source code (e2e), pushing it to a container registry, creating a GKE cluster and finally starting e2e tests.

![Cloud build flow](https://cdn-images-1.medium.com/max/2000/1*FPjkChOGGmgtvrQyfv1mIg.png)

{{ gist(url="https://gist.github.com/madhank93/08945a9d867d106ee143e5bf1147f790", class="gist") }}

## Execution

According to the cloud build settings, whenever the code has been pushed to Github repositories, execution will begin.

![](https://cdn-images-1.medium.com/max/5764/1*e9qUIU1TH6uQZHuepPuFDQ.png)

![](https://cdn-images-1.medium.com/max/5756/1*PLsmmNbVmcjwoPMIdaqQpA.png)

![1. Cloud build logs, 2. GKE workloads and 3. Chrome/Firefox browsers pods](https://cdn-images-1.medium.com/max/5760/1*UUSVseBqlIF_fOotzKbgjA.png)

## Result

Running all the 25 spec/tests files in chrome and firefox browsers parallelly, and completing it all within well under 04:41 mins. More or less time remains constant since all the tests are running in parallel no matter how many tests scripts has been authored, maximum time will take to complete the whole process will the slowest of all the test.

![Test execution results](https://cdn-images-1.medium.com/max/5744/1*dSPlXLrStA2RjMDdh6zTjQ.png)

⚠️❗⚠️ Finally make sure to delete the **_GKE cluster_** and **_Container images_** to avoid any charges.

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/testvagrant/scaling-tests-on-google-kubernetes-engine-with-cloud-build-624d955f6698)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/wdio-gke-cloud-build](https://github.com/madhank93/wdio-gke-cloud-build)

</center>

**References:**

[1] [https://github.com/alcounit/selenosis](https://github.com/alcounit/selenosis)

[2] [https://github.com/alcounit/selenosis-deploy](https://github.com/alcounit/selenosis-deploy)

[3] [https://cloud.google.com/kubernetes-engine/docs/tutorials/gitops-cloud-build](https://cloud.google.com/kubernetes-engine/docs/tutorials/gitops-cloud-build)

[4] [https://codefresh.io/kubernetes-tutorial/single-use-daemonset-pattern-pre-pulling-images-kubernetes/](https://codefresh.io/kubernetes-tutorial/single-use-daemonset-pattern-pre-pulling-images-kubernetes/)
