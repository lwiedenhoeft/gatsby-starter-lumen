---
title: Prometheus Operator - Service Monitor
date: "2019-12-18T22:40:32.169Z"
template: "post"
draft: false
slug: "prometheus-operator-service-monitor"
category: "Monitoring"
tags:
  - "Prometheus"
  - "Monitoring"
description: "In the following post, I will describe how you can add your own application metrics to the prom-operator monitoring."
socialImage: "/media/42-line-bible.jpg"
---
## Introduction

In the following post, I will describe how you can add your own application metrics to the prom-operator monitoring.

## The Service Monitor

The prometheus operator comes up with the Service Monitor concept:

![alt text][logo]

[logo]: https://478h5m1yrfsa3bbe262u7muv-wpengine.netdna-ssl.com/wp-content/uploads/2018/09/prometheus_operator_servicemonitor.png "Prometheus Operator Service Monitor Concept"

The "Service Monitor" is a custom resource. It tells Prometheus what k8s Service exposes metrics and where: service label selectors, its namespace, path, port, etc.

## Example

We have got an application in the "hello-world-1" namespace with label app: hello-world that exposes prometheus metrics on the  /actuator/metrics endpoint on the named port called web .

We are going to deploy ServiceMonitor for Prometheus to scrape the metrics every 15 seconds in monitoring namespace and with certain labels.

Below you can see the service monitor manifest.

```yaml
# hello-world-service-monitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: hello-world
  namespace: hello-world-1
  labels:
    release: prometheus-operator  # important!
spec:
  selector:
    matchLabels:
      app: hello-world
  endpoints:
  - port: web
    path: /actuator/prometheus
    interval: 5s
  namespaceSelector:
    matchNames:
      - hello-world-1
```

And here is the corresponding service manifest for the hello-world application.

```yaml
# hello-world-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-world
    tier: backend
  name: hello-world
  namespace: hello-world-1
spec:
  ports:
  - name: web
    port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello-world
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```