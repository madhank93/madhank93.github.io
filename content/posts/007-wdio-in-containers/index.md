+++
title = "Running WebdriverIO tests in containers"
description = "A guide to running test automation scripts in container"
date = 2021-06-22T01:24:33+05:30

[taxonomies]
tags = ["wdio", "docker", "selenoid", "ts", "github-actions", "ci"]

[extra]
toc = true
+++

![](https://cdn-images-1.medium.com/max/2880/1*clLrNn1wKHiytehS2Mu3Fg.gif)

As we looked at how to run test automation scripts in selenoid using 3rd party add-on actions in my [previous blog](https://medium.com/testvagrant/github-actions-for-seleniod-test-scripts-df469062a08c). In this blog let’s dive into how to containerize [WebdriverIO](https://webdriver.io/) test automation scripts, and set up a [Selenoid](https://aerokube.com/selenoid/) environment using [Docker](https://www.docker.com/) to run in local and CI environments.

The main advantage of choosing docker over any 3rd party add-on is it enables users to package an application into containers — standardized executable components that combine application source code with all the dependencies required to run the code in any environment.

### Setup process

Let’s start this process by creating a [Dockerfile](https://docs.docker.com/engine/reference/builder/), which is a text document that contains all the commands to build an image, docker can build an image automatically by reading the instructions present in the docker file.

{{ gist(url="https://gist.github.com/madhank93/821651bc4c2adfa2e0b49047d33c29ba", class="gist") }}

The above Dockerfile will install the necessary packages and includes all the test automation source code.

Now let's create a [Docker Compose](https://docs.docker.com/compose/) file. All the instructions for running a container will be specified in this file in a YAML format.

For this example, we will use the [shared compose](https://docs.docker.com/compose/extends/) feature which lets you share the common configurations between the compose files.

{{ gist(url="https://gist.github.com/madhank93/35a31854fdb114c0884fceb82a31df91", class="gist") }}

The above docker-compose file will act as a base config file for both local and CI.

Now let’s create a compose file specific for the local environment, which enables the selenoid feature of VNC and video storage.

{{ gist(url="https://gist.github.com/madhank93/a5c8e4a91baeb16bca72549a86bfbef2", class="gist") }}

The next step is to create compose file specific to CI environments.

{{ gist(url="https://gist.github.com/madhank93/7b0f682b889d23dc3f211e25175cd625", class="gist") }}

Now let’s make necessary changes in wdio.conf.ts to support VNC and video storage which are specific to selenoid.

{{ gist(url="https://gist.github.com/madhank93/81a55d69a12a10f4a70edecac6a25eb6", class="gist") }}

And don’t forget to volume map the [browsers.json](https://gist.github.com/madhank93/a945f0d2fe622825fe29d15ecc25c7cc)\* to the container running selenoid.

To manage all the commands for creating and removing containers let’s create [Makefile](https://www.gnu.org/software/make/manual/make.html) and store all the commands in it.

{{ gist(url="https://gist.github.com/madhank93/4c96feb7171006fcc29f293ba8954237", class="gist") }}

Finally, let’s hook all these steps into Github Actions.

{{ gist(url="https://gist.github.com/madhank93/39b208e2d17b1e04d53e6ed9d7c94d32", class="gist") }}

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/testvagrant/running-webdriverio-tests-in-containers-871e0238e31f)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/wdio-docker-ci](https://github.com/madhank93/wdio-docker-ci)

[Allure report link](https://madhank93.github.io/wdio-docker-ci/5/)

</center>

**References:**

[1] [https://aerokube.com/selenoid/latest/](https://aerokube.com/selenoid/latest/)

[2] [https://aerokube.com/selenoid-ui/latest/](https://aerokube.com/selenoid-ui/latest/)
