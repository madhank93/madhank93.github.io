+++
title = "A Deep Dive into the W3C WebDriver Specification"
description = "It’s time to look under the hood"
date = 2020-03-28T01:24:33+05:30

[taxonomies]
tags = ["selenium", "w3c", "webdriver"]

[extra]
toc = true
+++

![w3c-webdriver](https://cdn-images-1.medium.com/max/2390/1*szv6X0IYai76AwQx17DgsA.png)

Before getting into the topic lets first understand the difference between the terms WebDriver and Selenium.

When it comes to testing, these two terms WebDriver and Selenium are interchangeably used to refer the automating the web application.

[WebDriver ](https://w3c.github.io/webdriver/)is an HTTP based API to interact with a web browser. The standard is provided by W3C. WebDriver is a remote control interface that enables introspection and control of user agents. It provides a platform- and language-neutral wire protocol as a way for out-of-process programs to remotely instruct the behavior of web browsers.

[Selenium ](https://www.selenium.dev/documentation/en/)is a range of tools and libraries that enable and support the automation of web browsers. [Selenium WebDriver](https://www.selenium.dev/documentation/en/webdriver/understanding_the_components/) refers to both the language bindings and the implementations of the individual browser controlling code. This is commonly referred to as just _WebDriver_.

> Most of the browser vendors implements the [W3C WebDriver capabilities](https://w3c.github.io/webdriver/webdriver-spec.html) and protocol on Selenium 3.8.0 (JSON wire protocol is _OBSOLETE_ now) as a standalone server in a binary executable format.

- [Chrome](https://sites.google.com/a/chromium.org/chromedriver/downloads)

- [Firefox](https://github.com/mozilla/geckodriver/releases)

- [Opera](https://github.com/operasoftware/operachromiumdriver/releases)

- [EDGE](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/#downloads)

![selenium_architecture](https://cdn-images-1.medium.com/max/2440/1*de0VdIsqiluhVWEne3NF7A.jpeg) Selenium Architecture

> In this article, we are going to see **WebDriver API in action**, for that we need to download ChromeDriver and PostMan tool.

The first step is to [download](https://sites.google.com/a/chromium.org/chromedriver/downloads) the ChromeDriver executable. Download the appropriate driver version based on your OS.

To interact with the API we need a tool that allows us to make HTTP requests. For that we need to [download](https://www.postman.com/downloads/) the Postman tool. So that you can send and receive API requests.

# The test case that we are going to automate as follow,

1. Open the Chrome browser

2. Navigate to Google page

3. Find search text box

4. Send a search value

5. Find the search button

6. Click on the search button

7. Quit driver

## Let’s first start ChromeDriver in Terminal

Extract the file and run it using the command [./chromedriver](https://w3c.github.io/webdriver/#new-session). You will be given the port on which the WebDriver API is running.

```bash
$ ./chromedriver

Starting ChromeDriver 80.0.3987.106 (f68069574609230cf9b635cd784cfb1bf81bb53a-refs/branch heads/3987@{#882})
on port9515 Only local connections are allowed.
Please protect ports used by ChromeDriver and related test frameworks to prevent access by malicious code.
```

## 1. Open the Chrome browser

<p align="center">
    <a href='https://cdn-images-1.medium.com/max/2540/1*HX7JzMXo5R75qtkFTA4zhQ.png' target="_blank">
        <img alt="step 1" src="https://cdn-images-1.medium.com/max/2540/1*HX7JzMXo5R75qtkFTA4zhQ.png" width="65%">
    </a>
      &nbsp; &nbsp;
    <a href='https://cdn-images-1.medium.com/max/2000/1*I3cVPlrRTp4_n9pylkQnRw.png' target="_blank">
        <img alt="step 2" src="https://cdn-images-1.medium.com/max/2000/1*I3cVPlrRTp4_n9pylkQnRw.png" width="30%">
    </a>
</p>

<p align="center">
    <a href='https://cdn-images-1.medium.com/max/2000/1*O9gSQmtKsBafxdqHAQT0OQ.png' target="_blank">
        <img alt="step 3" src="https://cdn-images-1.medium.com/max/2000/1*O9gSQmtKsBafxdqHAQT0OQ.png" width="45%">
    </a>
    &nbsp; &nbsp;
    <a href='https://cdn-images-1.medium.com/max/2000/1*faSnhxAkGGE6RIsYbw5PyQ.png' target="_blank">
        <img alt="step 4" src="https://cdn-images-1.medium.com/max/2000/1*faSnhxAkGGE6RIsYbw5PyQ.png" width="45%">
    </a>
</p>

Now that the `chromedriver` started in the default port 9519. Let's open the browser. This is done by creating a new session. To create a new session using the WebDriver API, make an HTTP `POST` request to the [/session](https://w3c.github.io/webdriver/#new-session) endpoint. In addition, we need to define the type of browser. This information is sent in as a JSON object in the POST body. On success, the response includes a `sessionId`.

## 2. Navigate to Google page

![](https://cdn-images-1.medium.com/max/2000/1*tfiJOwHyAsNGBxgzgMqKUQ.png)

The next step is to open a URL in the browser. This is done with an HTTP `POST` request to [/session/<session_id>/url](https://w3c.github.io/webdriver/#navigate-to), with the POST body including the `URL` that will be opened

## 3. Find search text box

<img alt="step 1" src="https://cdn-images-1.medium.com/max/2076/1*We5Hhw6j_bIPqCNqO_EO2w.png" width="100%">

<p align="center">
    <img alt="step 2" src="https://cdn-images-1.medium.com/max/2000/1*YqaSJkKPYOsrpxR_YMIcsw.png" width="45%">
    &nbsp; &nbsp;
    <img alt="step 3" src="https://cdn-images-1.medium.com/max/2000/1*mTJcGR9zcyTUq9Z9eZKYwA.png" width="45%">
</p>

Now that we have opened the Google page, let's find the search text box. This is done with an HTTP `POST` request to [/session/<session_id>/element](https://w3c.github.io/webdriver/#find-element), with the POST body including the `location strategy` and `selector`

## 4. Send a search value

![](https://cdn-images-1.medium.com/max/2000/1*bLtuBqQlixM8buEikmwFYw.png)

After locating the search box, let's send the search value. This is done with an HTTP `POST` request to [/session/<session_id>/element/<element_id>/value](https://w3c.github.io/webdriver/#element-send-keys), with the POST body including the value in the `text` parameter

## 5. Find the search button

![](https://cdn-images-1.medium.com/max/2000/1*C-MRw9CryoicmxbjE8Kt9Q.png)

<p align="center">
    <img alt="step 2" src="https://cdn-images-1.medium.com/max/2000/1*bYRsHcSi-kgMYCpkZSRvbw.png" width="45%">
    &nbsp; &nbsp;
    <img alt="step 3" src="https://cdn-images-1.medium.com/max/2000/1*HTO1qzeTEKGG8hrk7YgtTw.png" width="45%">
</p>

Let's find the search text button. This is done with an HTTP `POST` request to [/session/<session_id>/element](https://w3c.github.io/webdriver/#find-element), with the POST body including the `location strategy` and `selector`

## 6. Click on search button

![](https://cdn-images-1.medium.com/max/2000/1*n-FjMiz5gfQZQDkjyGIgVQ.png)

Now let's click the search text button. This is done with an HTTP `POST` request to [/session/<session_id>/element/<element_id/click](https://w3c.github.io/webdriver/#element-click), with the POST body including the `empty dictionary`

## 7. Quit driver

![](https://cdn-images-1.medium.com/max/2000/1*EX3TgWgWfenwaCZIn9yQbA.png)

To quit the driver, send the HTTP `DELETE` request to [/session/<session_id>](https://w3c.github.io/webdriver/#delete-session)

<div align="center">* * * *</div>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/a-deep-dive-into-the-w3c-webdriver-specification-fcf0906048f9)

Thanks to [Peter Thomas](https://twitter.com/ptrthomas?lang=en) creator of [Karate DSL](https://github.com/intuit/karate)

You can find the [GitHub Source code](https://github.com/madhank93/automation_using_chromedriver_postman) with all of these above steps.

# References

[1] [https://www.selenium.dev/documentation/en/](https://www.selenium.dev/documentation/en/)

[2] [https://www.youtube.com/watch?v=IcCnzXTxFt0&feature=youtu.be](https://www.youtube.com/watch?v=IcCnzXTxFt0&feature=youtu.be)

[3] [https://www.slideshare.net/ptrthomas/a-deep-dive-into-the-w3c-webdriver-specification](https://www.slideshare.net/ptrthomas/a-deep-dive-into-the-w3c-webdriver-specification)

[4] [https://www.erranderr.com/blog/webdriver-ontology.html](<https://www.erranderr.com/blog/webdriver-ontology.html](https://www.erranderr.com/blog/webdriver-ontology.html)>)

[5] [https://sfconservancy.org/news/2018/may/31/seleniumW3C/](https://sfconservancy.org/news/2018/may/31/seleniumW3C/)

[6] [https://lists.w3.org/Archives/Public/public-browser-tools-testing/2016AprJun/0097.html](https://lists.w3.org/Archives/Public/public-browser-tools-testing/2016AprJun/0097.html)
