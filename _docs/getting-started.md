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

## Rönd capabilities

Rönd evaluated policy can be used to perform three different actions:

- Allow or block API invocations;
- Generate queries to filter data (currently supported syntax is MongoDB);
- Modify the response payload (only supported for JSON bodies).

## Managing Access Control System with Rönd
### RBAC

Rönd provides the means for implementing a full-fledged RBAC solution. Rönd uses MongoDB to store Roles and Bindings, and allows you to write policies on user roles permissions and bindings. To start using RBAC see [the guide here](docs/policy-integration#rbac-data-model).

### ACL
Rönd provides the means for implementing a full-fledged ACL solution. Rönd uses MongoDB to store User, Groups and Permissions. Rönd allows you to write policies on user roles and permissions. To start using ACL see [the guide here](docs/policy-integration#acl-data-model).


### ACL vs RBAC

ALC (Access Control List) and RBAC (Role-Based Access Control) are two popular access control systems used in computer security to ensure that only authorized individuals have access to resources. Here's a comparison between ALC and RBAC:

* Conceptual Difference: ALC is a permission-based access control system, which means it grants access based on a user's permission to access a particular resource. In contrast, RBAC is a role-based access control system, which means it grants access based on a user's role or job function within the organization.

* Complexity: ALC is a simpler access control system as compared to RBAC, as it only focuses on granting permissions to users on a per-resource basis. RBAC, on the other hand, is more complex, as it involves assigning roles to users and mapping those roles to permissions.

* Flexibility: ALC is less flexible compared to RBAC. As permissions are granted on a per-resource basis, it is challenging to manage large-scale systems with numerous resources and many users. In contrast, RBAC provides greater flexibility as it allows for the assignment of roles to users and the grouping of permissions based on job functions or departmental needs.

* Scalability: ALC is not scalable to a large number of users and resources, as managing permissions for every resource can be challenging. In contrast, RBAC is designed to handle larger systems with multiple users and resources, as it groups users by their roles and permissions by job function.

* Administration: ALC is more challenging to administer as permissions are managed on a per-resource basis. In contrast, RBAC is easier to administer as permissions are managed by roles, which can be assigned to groups of users.

Overall, RBAC is a more sophisticated and flexible access control system, ideal for larger organizations with many resources and users. ALC is a simpler system, suited for smaller organizations with fewer resources and users.

| Capability  | ACL | RBAC |
|-------------|-----|------|
| Simplicity  | ✅  |  ❌   |
| Flexibility | ❌  | ✅   |
| Scalability | ❌  | ✅   |
| Administration | ✅  | ❌   |
| -------------- |---- | ----- |

### References
* Wikipedia - [Role-based Access Control](https://en.wikipedia.org/wiki/Role-based_access_control)
* Wikipedia - [Access Control List](https://en.wikipedia.org/wiki/Access-control_list)
* Geeks For Geeks - [ACL](https://www.geeksforgeeks.org/access-lists-acl/)
* Imperva - [RBAC](https://www.imperva.com/learn/data-security/role-based-access-control-rbac/)