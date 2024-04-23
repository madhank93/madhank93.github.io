+++
title = "Monitoring Test automation: Gathering metrics — Part 1"
description = "Using Prometheus to gather metrics and visualizing it in Grafana"
date = 2022-10-19T01:24:33+05:30

[taxonomies]
tags = ["k8s", "prometheus", "grafana", "telegraf", "minikube", "monitoring"]

[extra]
toc = true
+++

![Monitoring stack](https://cdn-images-1.medium.com/max/3840/1*0sy6gcgqJ9tcjO8Jr1JRHg.png)

In this article we are going to see how to add a monitoring capability to my previous work [Scaling tests on Kubernetes](https://medium.com/testvagrant/scaling-tests-on-google-kubernetes-engine-with-cloud-build-624d955f6698). We will use [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) to transform selenosis metrics, [Prometheus](https://prometheus.io/) to collect metrics and visualize it using [Grafana](https://grafana.com/grafana/).

- [Prometheus](https://github.com/prometheus) is an open-source systems monitoring and alerting toolkit, stores its metrics as [time series](https://prometheus.io/docs/concepts/data_model/) data in Time series Database ([TSDB](https://en.wikipedia.org/wiki/Time_series_database))

- [Telegraf](https://github.com/influxdata/telegraf) is an open-source agent for collecting, processing, aggregating, and writing metrics

- [Grafana](https://github.com/grafana/grafana) is an open-source data-visualization platform

## Monitoring Setup

![Monitoring high level architecture](https://cdn-images-1.medium.com/max/6198/1*-_yQI0gJWspiWk16b3Fxyw.png)

> To follow along with this article, do checkout the [source code](https://github.com/madhank93/monitoring-test-automation) and
> use [minikube](https://minikube.sigs.k8s.io/docs/start/) or [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) or any self-hosted/managed k8s cluster.

### Install Prometheus to collect metrics

Recommended way of installing Prometheus in k8s cluster is using helm charts.

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack --values infra/prometheus/values.yml
```

Once the deployments are successful, port forward to access the Prometheus UI on the browser at [http://localhost:9090](http://localhost:9090)

```sh
kubectl port-forward service/prometheus-kube-prometheus-prometheus 9090:9090
```

### Setup Selenosis test infrastructure

Please refer to the detailed instructions of setting up test infrastructure in this [article](https://medium.com/testvagrant/scaling-tests-on-google-kubernetes-engine-with-cloud-build-624d955f6698). In short, execute the shell script **sh infra/selenosis/script.sh** from the root folder to complete the test infra setup.

Once the script has been successfully executed and pods are up & running, port forward to see the selenosis status at [http://localhost:4000/status](http://localhost:4000/status), usage of selenosis metrics is available in JSON format.

```sh
kubectl port-forward service/selenosis 4000:4444 -n selenosis
```

### Converting to Prometheus understandable metrics

Since Selenosis is a 3rd party application and doesn’t have **/metrics** endpoint exposed to scrape metrics. And also, Prometheus doesn’t understand metrics that are in JSON format, so we will use Telegraf [plugins](https://github.com/influxdata/telegraf/tree/master/plugins) to convert JSON into Prometheus understandable data format.

> And also there are a number of libraries and servers which help in [exporting existing metrics](https://prometheus.io/docs/instrumenting/exporters/) from third-party systems as Prometheus metrics.

![Metric conversion](https://cdn-images-1.medium.com/max/2000/1*afodzdmsJ54eShxxJmT9lw.png)

```toml
[[inputs.http]]
urls = ["http://selenosis:4444/status"]
data_format = "json"

[[outputs.prometheus_client]]
path = "/metrics"
listen = ":9273"
```

Execute the below commands to apply manifests files of telegraf to setup it up in k8s cluster.

```sh
kubectl apply -f infra/telegraf/
```

Now apply the following k8s manifest file so that Prometheus operator can pick up telegraf [service monitor](https://prometheus-operator.dev/docs/user-guides/getting-started/#installing-the-operator) _and add it to the scraping targets. _(Basically, operator takes cares most of creation/deletion/reload config of k8s app lifecycle, for more info refer [here](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/))

```sh
kubectl apply -f infra/prometheus/telegraf-svc-monitor.yml
```

### Querying Prometheus metrics using PromQL

Prometheus offers the [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/), using which we can select and aggregate time series data in real time.

![Prometheus metrics](https://cdn-images-1.medium.com/max/2098/1*LI_-fZbM4jsvwbITMQFVZA.png)

Selenosis exposes **_http_selenosis_active_** , **_http_selenosis_pending_** , **_http_selenosis_total_** and these metrics are of [gauge](https://prometheus.io/docs/tutorials/understanding_metric_types/#gauge) type (in which metrics can go up and down). And Prometheus supports [four types](https://prometheus.io/docs/tutorials/understanding_metric_types/#types-of-metrics) of metrics in total.

Start executing the tests to see selenosis metrics usage in prometheus by executing the following command in the terminal.

```sh
kubectl apply -f e2e-test.yml
```

### Visualizing metrics in Grafana

Grafana comes as part of the Prometheus helm installation, no additional setup is required and also it supports the PromQL out of the box. To access the Grafana UI, port forward the incoming request and access it on the browser at [http://localhost:3000/status](http://localhost:4000/status)

```sh
kubectl port-forward deployment/prometheus-grafana 3000:3000
```

![Metrics in Grafana](https://cdn-images-1.medium.com/max/3558/1*PR41-gOkxJNjAKtNq_RP_A.png)

From the above dashboard we track the usage metrics of Selenosis. When it started **_pending-0_** , **_active-0_** & **_total-10_** remains the same and as soon as the test started executing **_active_** count increased to **3**. Since no. of parallel execution is set to 3 in the testing framework. Once the tests are completed, the active count drops to zero.

This article shows a simple use-case of collecting and visualizing of Selenosis usage metrics. [Kube state metric](https://github.com/kubernetes/kube-state-metrics) comes as part of the Prometheus helm installation and it currently supports about [35+ resources](https://github.com/kubernetes/kube-state-metrics/tree/master/docs#exposed-metrics). So, start [building](https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/) your dashboards in Grafana.

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/monitoring-test-automation-gathering-metrics-part-1-3946d8050627)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/monitoring-test-automation](https://github.com/madhank93/monitoring-test-automation)

</center>

**References:**

[1] TechWorld with Nana — Prometheus series [Part 1](https://www.youtube.com/watch?v=QoDqxm7ybLc) [Part 2](https://www.youtube.com/watch?v=mLPg49b33sA&t=1s)

[2] [https://prometheus.io/docs/introduction/overview/](https://prometheus.io/docs/introduction/overview/)

[3] [https://docs.influxdata.com/telegraf/v1.23/](https://docs.influxdata.com/telegraf/v1.23/)

[4] [https://grafana.com/docs/](https://grafana.com/docs/)

[5] [https://prometheus-operator.dev/](https://prometheus-operator.dev/)

[6] [https://aerokube.com/selenoid/latest/#\_advanced_features](https://aerokube.com/selenoid/latest/#_advanced_features)
