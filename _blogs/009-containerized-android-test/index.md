+++
title = "Running containerized android tests in GCP using Pulumi and Selenoid"
description = "Leveraging Pulumi’s IaC feature to run Android test automation"
date = 2022-04-21T01:24:33+05:30

[taxonomies]
tags = ["docker", "selenoid", "android", "gcp", "pulumi", "ts", "appium", "iac"]

[extra]
toc = true
+++

![](https://cdn-images-1.medium.com/max/3840/1*qQ4VcDa22uq2a00xeOH7Mw.png)

In the previous blog we looked at running web automation tests using selenoid in [Docker](https://medium.com/p/871e0238e31f) and [Kubernetes](https://medium.com/p/624d955f6698). This time we are going to look at running containerized android tests in GCP using Pulumi and Selenoid.

[Pulumi](https://www.pulumi.com/) is a modern [Infrastructure as Code](https://www.pulumi.com/what-is/what-is-infrastructure-as-code/) platform that allows you to use familiar programming languages and tools to build, deploy, and manage cloud infrastructure. [Selenoid](https://aerokube.com/selenoid/) is a lightning fast Selenium protocol implementation running on browsers and android emulator in Docker containers.

> To get started with Pulumi and GCP, please refer to this [article](https://www.pulumi.com/docs/get-started/gcp/). It includes installation and environment setup.

![High level architecture](https://cdn-images-1.medium.com/max/2000/1*BMkYeNZF1zWDrxdxiDsLdA.png)

Totally there are 3 parts to this article,

1.  Environment setup in GCP,

2.  Running Selenoid containers, and

3.  Containerizing Android tests and executing it

## Environment setup in GCP

In order to run selenoid android containers we need a [Linux server](https://github.com/aerokube/selenoid/issues/687) with [nested virtualization](https://cloud.google.com/compute/docs/instances/nested-virtualization/overview) enabled. So we will use Pulumi to build infrastructure setup in GCP. One thing to keep in mind is that not every machine type in GCP supports nested virtualization; for more info refer to this [article](https://cloud.google.com/compute/docs/instances/nested-virtualization/overview#restrictions).

{{ gist(url="https://gist.github.com/madhank93/aaa95eb7e77ba6031e7d3da72afc70ea", class="gist") }}

[deploy.sh](https://gist.github.com/1d6d18fd27dd74292ac364a9a20ac5df) file contains scripts to install docker and docker compose in the GCP remote instance.

## Running Selenoid containers

Pulumi [docker provider](https://www.pulumi.com/registry/packages/docker/) packaged is used to run the containers in GCP instance.

{{ gist(url="https://gist.github.com/madhank93/58287b1a8607392f1b0c09b1da47f796", class="gist") }}

## Containerizing android tests and executing it

To demonstrate I have used Sauce Labs [swag app](https://github.com/saucelabs/sample-app-mobile/releases) using Java to automate the [login process](https://gist.github.com/madhank93/2fd8bfb0cee6cff3e77437ff738ec043#file-androidautomation-java). Containerized the test scripts using [docker](https://gist.github.com/madhank93/be0da07ede408c6ec9d0ee7465dc0882#file-dockerfile) multi stage build.

**Remote address**

```java
String APPIUM = "http://selenoid:4444/wd/hub";
```

The android automation test scripts should point to the above URL. If the test application is also deployed as part of a docker container.

```java
String APPIUM = "http://external-ip:4444/wd/hub";
```

If you want your setup to be in GCP and run tests from the local machine, all we need is to change the above URL to point to the **External IP** address.

**Specifying capabilities**

![Android Capabilities](https://cdn-images-1.medium.com/max/6060/1*rvJt9ff7SvrPGFMNzF0pug.png)

One thing to notice here is that rather than providing the local path of the APK file, I have specified the URL. Because the local path of the APK does not exist inside the container or else you have to [bind mount](https://docs.docker.com/storage/bind-mounts/) the APK file. Another approach is to [build an image](http://aerokube.com/images/latest/#_building_images) with APK preinstalled.

**Adding containers to Pulumi docker setup**

![Android demo test container](https://cdn-images-1.medium.com/max/3408/1*oqkYqo2m3LJMTQcUlvzp1Q.png)

Finally, the docker execution file of Pulumi will look like [this](https://gist.github.com/1d113a9043b06bc2e50a174ab01a5e3f). Since it is a demo application, I have pushed docker image to the [public registry](https://hub.docker.com/r/madhank93/android-demo); you should consider pushing it to private registry.

## Execution

Finally, we have reached to the execution part, once all the setup has been completed. All we need to do is run the below command.

```sh
pulumi up
```

Proceed with “**Yes**” for [pulumi preview](https://www.pulumi.com/docs/reference/cli/pulumi_preview/) updates, and you will have the infrastructure setup in 4 minutes [logs](https://gist.github.com/madhank93/b942c48567a37f9bbd788efcf4cc7bab#file-pulumi-logs-log). You can connect to **_http://external-ip:8080/_** to see the live execution. The Selenoid UI also provides the option to launch the session manually. Once the tests are completed, you can run the following command

```sh
pulumi destroy
```

The above command will destroy all the resources created as part of Pulumi (including instance, docker container and images, etc.)

The source code of this blog post can be found [here](https://github.com/madhank93/android-automation-gcp-pulumi).

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/running-containerized-android-tests-in-gcp-using-pulumi-and-selenoid-faf4c398cd6c)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/android-automation-gcp-pulumi](https://github.com/madhank93/android-automation-gcp-pulumi)

</center>

**References:**

[1] [https://www.pulumi.com/blog/](https://www.pulumi.com/blog/)

[2] [https://aerokube.com/selenoid/latest/](https://aerokube.com/selenoid/latest/)

[3] [https://github.com/pulumi/examples](https://github.com/pulumi/examples)

[4] [https://brunoscheufler.com/blog/2021-09-26-containers-as-code-with-pulumi-and-docker](https://brunoscheufler.com/blog/2021-09-26-containers-as-code-with-pulumi-and-docker)

[5] [https://stackoverflow.com/a/59022743](https://stackoverflow.com/a/59022743)
