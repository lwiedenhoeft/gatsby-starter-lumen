---
title: Python Testcontainers - Python port for testcontainers-java
date: "2020-12-23T20:20:32.669Z"
template: "post"
draft: false
slug: "python-testcontainers"
category: "cicd"
tags:
  - "Tests"
  - "Testcontainers"
  - "CICD"
  - "Python"
description: "Testcontainers-python provides capabilities to spin up docker containers (such as a database, Selenium web browser, or any other container) for testing."
socialImage: "/media/42-line-bible.jpg"
---
## Introduction

The intention of this post is to test the use of 
[testcontainers-python](https://testcontainers-python.readthedocs.io/en/latest/) for integration tests in python code. 
This post tries to demonstrate:

1. Integration of testcontainers-python with [behave](https://behave.readthedocs.io/en/latest/).
2. The start of a generic container - in this example RabbitMQ.
3. Await the app running in the container to finish initializing.
4. Reach out to the app running inside the test container.


## Running

To run the code and the linked integration tests, create a virtual environment.
```bash
    $ python3 -m venv ./.venv 
    $ source .venv/bin/activate 
    $ pip install -r requirements.txt
```    
We assume that docker has been installed and is running.

### simple_http_server.py

`simple_http_server.py` is the application we want to test. It is a simplistic web server that delivers a 
'Hello, world!' string in reply to `GET` requests and pushes the content of the request to RabbitMQ when a `POST` request is received. 

To test the application, start a docker container running RabbitMQ:
```bash
    $ docker run -d --hostname my-rabbit -p 15672:15672 -p 5672:5672 --name some-rabbit rabbitmq:3.8.3-management
```    
Run the webserver specifying a port to receive requests and the RabbitMQport to send messages to:
```bash
    $ source .venv/bin/activate
    (.venv) $ python3 simple_http_server.py 8082 5672
```    
Communicate with the webserver via cURL:
```bash
    $ curl http://localhost:8082
      Hello, world!
    $
    $ curl -v http://localhost:8082 -d'[{"hello":"world"}]'
      *   Trying ::1...
      * TCP_NODELAY set
      * Connection failed
      * connect to ::1 port 8082 failed: Connection refused
      *   Trying 127.0.0.1...
      * TCP_NODELAY set
      * Connected to localhost (127.0.0.1) port 8082 (#0)
      > POST / HTTP/1.1
      > Host: localhost:8082
      > User-Agent: curl/7.64.1
      > Accept: */*
      > Content-Length: 19
      > Content-Type: application/x-www-form-urlencoded
      > 
      * upload completely sent off: 19 out of 19 bytes
      * HTTP 1.0, assume close after body
      < HTTP/1.0 201 Created
      < Server: BaseHTTP/0.6 Python/3.7.7
      < Date: Fri, 27 Mar 2020 13:33:16 GMT
      < 
      * Closing connection 0
```
    
Open http://localhost:15672/#/queues/%2F/test_queue with the credentials guest:guest in order to view the queue and display 
the 'hello world' message added to the queue.

### Running the integration tests

Activate the virtual environment and start `behave`:
```bash
    $ source .venv/bin/activate
    (.venv) $ behave

    Pulling image rabbitmq:3.8.3-management
    ⠇
    Container started:  00c3b92042
    Feature: Testcontainers POC: demonstrates use of Testcontainers for integration testing # features/everything.feature:1

      Scenario: When a GET request is made to the simple webserver, a successful 'Hello World' response is returned  # features/everything.feature:3
        When a GET request is made to the simple webserver                                                           # features/steps/steps.py:12 0.004s
        Then the webserver will respond with a HTTP status of "200"                                                  # features/steps/steps.py:27 0.000s
        And the response body will contain "Hello, world!"                                                           # features/steps/steps.py:32 0.000s

      Scenario: When a POST request is made to the simple webserve, the request body is added to the RabbitMQ queue  # features/everything.feature:8
        When a POST request is made to the simple webserver                                                          # features/steps/steps.py:17 0.033s
        Then the webserver will respond with a HTTP status of "201"                                                  # features/steps/steps.py:27 0.000s
        And the response body will be empty                                                                          # features/steps/steps.py:37 0.000s
        And the request body is added to the RabbitMQ queue                                                          # features/steps/steps.py:42 0.027s

    1 feature passed, 0 failed, 0 skipped
    2 scenarios passed, 0 failed, 0 skipped
    7 steps passed, 0 failed, 0 skipped, 0 undefined
    Took 0m0.065s
    (.venv) $
```
## Additional Links

* https://testcontainers-python.readthedocs.io/en/latest/
* https://behave.readthedocs.io/en/latest/
* based on https://github.com/spt-development/testcontainers-python-poc
