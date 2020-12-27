---
title: Logstash filter - How to test your configuration
date: "2020-12-30T20:20:32.669Z"
template: "post"
draft: false
slug: "logstash-filter-test"
category: "cicd"
tags:
  - "Loogging"
  - "Monitoring"
  - "Development"
description: "For testing the configuration before deploying it, it is useful to test your configuration on your local machine."
socialImage: "/media/42-line-bible.jpg"
---
## Introduction

For testing the configuration before deploying it, it is useful to test your configuration on your local machine.

## Start

Clone the following repository (https://github.com/lwiedenhoeft/logstash-filter-development) and start the container

```bash
$ docker-compose up
```    

## Change Logstash pipeline

You can change and test your logstash pipeline on the fly, because started Logstash with the ` --config.reload.automatic` flag.

```yml
input {
  http {
    port => 8080 # default: 8080
  }
}
 
## Add your filters / logstash plugins configuration here
filter {
}
 
output {
    stdout {}
}
```

## Run the test

Append your test message to the example.json and send it to logstash:

```bash
$ curl -XPUT 'http://127.0.0.1:8080/example' --data "@example.json"
```    

Check and validate the output of the Logstash Docker Container.
## Additional Links

* https://www.elastic.co/logstash
* https://behave.readthedocs.io/en/latest/
