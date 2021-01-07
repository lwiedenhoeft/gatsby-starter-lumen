---
title: Prometheus Operator Stack - Installing Prometheus in K8s with Helm 3
date: "2021-01-05T20:20:32.669Z"
template: "post"
draft: false
slug: "prometheus-operator-stack-with-helm"
category: "monitoring"
tags:
  - "Monitoring"
  - "Prometheus"
  - "Helm"
  - "Kubernetes"
  - "K8s"
description: "Back to the basics. A short introduction into the K8s operators concept and how-to install prometheus operator start."
socialImage: "/media/k8s-logo.svg"
---
## Introduction

Kubernetes monitoring time! I like monitoring (Yes i do!) and I really apprichiate when we can introduce a monitoring solution with ease. Here comes prometheus into play. Easy to install but hard to master. So lets start with a short introduction into the k8s operator concept followed by a prometheus operator installation how-to. Have fun!

## The K8s Operator

Kubernetes operators are extensions that can be used to create custom resource types (https://bit.ly/2KTSSXF). In addition to the standard Kubernetes resources such as Pods, DaemonSets, Deployments, etc., you can also create your own (so called custom) resources with the help of an operator. In this example, we are going to introduce Prometheus operator. (Lets call it PromOperator from now on :) ). Operators are of great use when you need to perform special manual tasks for your application to be able to run it correctly. Scaling complex apps, application version upgrades or even managing kernel modules for nodes in a computing cluster with specialised hardware. 
![operator_concept](/media/operator-k8s.png)

## The Idea

Building upon the definition above, we can describe the PromOperator as a resource inside K8s that allows simpler management of Prometheus instances (incl configuration and service discovery). This us to easily launch instances of Prometheus, to adjust their config, as well as to manage replicas, retention times, and persistence.
Furthermore, the PromOperator can automatically create monitoring target settings based on K8s labels. We just refer to services and pods we want to monitor in the PromOperator manifest, and the Operator will add the appropriate Prometheus configuration entries for the K8s auto-discovery.

## Installation

In this post, we use Helm 3 to install the definitions and resoures. You can avoid Helm by installing the operator via pure manifest files. If you want to go this way I can recommend this netways blog articel https://bit.ly/3ngHmCP

Please check if the kubectl and helm cli is installed.

### Install the helm repository
```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
-----------------
"prometheus-community" has been added to your repositories
```

### Test if installation was sucessfull
```bash
$ helm search repo prometheus-community
-----------------
NAME                                              	CHART VERSION	APP VERSION	DESCRIPTION
prometheus-community/alertmanager                 	0.3.0        	v0.21.0    	The Alertmanager handles alerts sent by client ...
prometheus-community/kube-prometheus-stack        	12.10.4      	0.44.0     	kube-prometheus-stack collects Kubernetes manif...
prometheus-community/prometheus                   	13.0.2       	2.22.1     	Prometheus is a monitoring system and time seri...
....
```

### Install the prometheus stack


> **_NOTE:_** Because of a bug (https://bit.ly/3nigQss), we have to install the chart version '9.3.4', if we want to deploy the stack to a docker-desktop one-node cluster. 

```bash
$ helm install prometheus prometheus-community/kube-prometheus-stack --version '9.3.4'
-----------------
NAME: prometheus
LAST DEPLOYED: Tue Jan  5 14:27:01 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace default get pods -l "release=prometheus"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```
Lets check if we were sucessfull! We should see two operator pods and one node-exporter pod per node.
```bash
$ kubectl --namespace default get pods -l "release=prometheus"
-----------------
NAME                                                   READY   STATUS    RESTARTS   AGE
prometheus-kube-prometheus-operator-676f756669-q47rb   2/2     Running   0          43m
prometheus-prometheus-node-exporter-rxj57              1/1     Running   0          43m
```

#### Prometheus
Great! Lets access Prometheus 
```bash
$ kubectl port-forward  $(kubectl get pod -n default --selector app=prometheus --output=jsonpath="{.items..metadata.name}"
) 9090
-----------------
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

![127.0.0.1:9090/targets](/media/prometheus.png)

#### Grafana
Awesome :) Now lets take a look at Grafana.

```bash
~ $ kubectl port-forward $(kubectl get pods -n default --selector app.kubernetes.io/name=grafana --output=jsonpath="{.items..metadata.name}") 3000
-----------------
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
```
![127.0.0.1:3000](/media/grafana.png)

## What comes next?
Check out my post about how to setup a service monitor!
https://www.larstogo.dev/posts/prometheus-operator-service-monitor

## Additional Links and sources

* https://github.com/prometheus-operator/prometheus-operator
* https://nws.netways.de/de/tutorials/2020/05/27/monitoring-kubernetes-mit-prometheus/
* https://medium.com/kubernetes-tutorials/simple-management-of-prometheus-monitoring-pipeline-with-the-prometheus-operator-b445da0e0d1a
