---
title: Getting Started
tags:
  - starter-kit
description: Getting started with Rönd
meta_description: 
order: 1
---

# Getting Started with Rönd

Rönd is responsible for handling authorization rules: it is useful to restrict access to specific resources and API.
While it supports expressive authorization rules possible on the request data, it also provides primitives for implementing your [RBAC solution](#Rönd-and-rbac).  
  
**We have prepared an easily configurable example to allow you to explore all the features of Rond. [You can find it inside the example repo](https://github.com/rond-authz/example)**

## Rönd Overview

Rönd is generally used as sidecar container. It intercepts all the incoming traffic, applies authorization rules and, if rules are passed, it forwards the traffic to your application service.

To define which APIs are exposed by your service, Rönd accepts an [OpenAPI 3 Specification](https://swagger.io/specification/) that can be provided either via file or via API. For OpenAPI 3 Specification via API, Rönd will contact your service to fetch the OAS: then, Rönd will use the OAS to configure itself.

For further details on how to configure Rönd, check out the [configuration page](/docs/configuration).

{%
  include alert.html
  type="info"
  content="Rönd can also be used in standalone mode. In this scenario, services must explicitly contact Rönd to perform authorization rule evaluation. Further information about this are available in the [configuration page](/docs/configuration)."
%}

## Rönd and RBAC

In addition to simple request-based authorization rules, Rönd provides the means for implementing a full-fledged RBAC solution. Rönd uses MongoDB to store Roles and Bindings, and allows you to write policies on user roles permissions and bindings.

## Rönd capabilities

Rönd evaluated policy can be used to perform three different actions:

- Allow or block API invocations;
- Generate queries to filter data (currently supported syntax is MongoDB);
- Modify the response payload (only supported for JSON bodies).
