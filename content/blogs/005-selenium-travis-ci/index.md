+++
title = "Selenium DevOps series: Run your scripts in Travis CI"
description = "A curated list of the working code example in Java"
date = 2020-04-26T01:24:33+05:30

[taxonomies]
tags = ["selenium", "snippets", "testing", "automation", "java"]

[extra]
toc = true
+++

# Selenium DevOps series: Run your scripts in Travis CI

Aren’t you bored of running your automation scripts locally?

![Source — unknown](https://cdn-images-1.medium.com/max/3600/1*IbnJC_qfjBAMMQLQ1scTgg.jpeg)_Source — unknown_

In this article, we are going to see how to run Selenium automation scripts in headless mode in the Travis CI environment. In three steps,

1. Setup Travis CI account

1. Add Travis YAML file

1. Push your code
   > Note: Testing your open source projects are free in Travis CI

## Setting up Travis CI account

1. Head to the URL — [https://travis-ci.org/](https://travis-ci.org/)

![Travis CI homepage](https://cdn-images-1.medium.com/max/2154/1*p4DpAsS-Ef_0HTvogr9f0A.png)_Travis CI homepage_

2. Click on Sign-up and Sign-in using your GitHub account

![sign-in page](https://cdn-images-1.medium.com/max/2000/1*yjyCrEmosXHm2B1ucBwzcA.png)_sign-in page_

3. Click on Authorize travis-ci

![authorizing travis ci](https://cdn-images-1.medium.com/max/2000/1*jCIuhn59xgZ5fGCyRMufPA.png)_authorizing travis ci_

4. Select the repository

![select repo](https://cdn-images-1.medium.com/max/2208/1*S-WF7VKEdtku7dA4w587Ww.png)_select repo_

5. Activate repository

![activate repo](https://cdn-images-1.medium.com/max/2528/1*1INM3fvmt0d5mff2bOM4jw.png)_activate repo_

6. Builds status for the repository

![build status](https://cdn-images-1.medium.com/max/2000/1*d08h-NH6OnCPT6Z07GINIw.png)_build status_

Now that our GitHub and Travis CI are integrated. Next step is we have to add a .travis.yml file in the same directory level

## Adding .travis.yml file

![.travis.yml](https://cdn-images-1.medium.com/max/2000/1*qmrP2OMaX9Nqc92BgemUKQ.png)_.travis.yml_

Adding .travis.yml file to your root folder level helps Travis CI to know how to build the project. Now let’s see what should go inside the file

    dist: trusty
    language: java
    jdk:
    - openjdk11

dist stands for distributions used to configure the version of Ubuntu

language used to configure programming language

jdk used to configure the JDK

Since we are going to run the scripts in the headless mode we don’t need to add [Xvfb](https://www.x.org/releases/X11R7.6/doc/man/man1/Xvfb.1.xhtml) — the virtual framebuffer (since from Chrome 59 and Chrome 60 for Windows)

## Pushing your code

Travis CI automatically recognizes that our project is built using Maven by identifying the presence of a pom.xml file. And automatically installs the dependencies.

{{ gist(url="https://gist.github.com/madhank93/46b4c21ae59142dd4cff550eed02664e", class="gist") }}

<!-- <iframe src="https://medium.com/media/a7e554ccc27ad0fe93f7fe2917d0315b" frameborder=0></iframe> -->

> In this i have used [webdrivermanager](https://github.com/bonigarcia/webdrivermanager) library as one of the dependencies in pom.xml file to manage binary drivers (e.g. _chromedriver_, _geckodriver_, etc.)

![settings](https://cdn-images-1.medium.com/max/3054/1*qR-i1V__qW9SEdncsgaAsg.png)_settings_

By default, Travis CI automatically starts build process when the code changes have been pushed or pull request has been created.

{% galleria() %}
{ "images":
[{"src": "https://cdn-images-1.medium.com/max/2734/1*XXM8rLAprBOpU3I26gx5CQ.png","title": "Clouds & Mountains", "description": "Just hanging out with each other."}] }
{% end %}

## Result

![](https://cdn-images-1.medium.com/max/2734/1*XXM8rLAprBOpU3I26gx5CQ.png)

The Build log shows us that our tests are passed !!!

- [madhank93/selenium-maven-travisci](https://github.com/madhank93/selenium-maven-travisci)
- [Travis CI](https://travis-ci.org/github/madhank93/selenium-maven-travisci)

**References:**

[1] [https://www.kenst.com/2018/10/how-to-run-your-tests-with-headless-chrome-ruby-version/](https://www.kenst.com/2018/10/how-to-run-your-tests-with-headless-chrome-ruby-version/)

[2] [https://docs.travis-ci.com/](https://docs.travis-ci.com/)
