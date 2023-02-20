---
title: Policy Integration
tags:
  - starter-kit
  - configuration
description: How to write a policy integrated with Rönd features
meta_description: 
order: 3
---

# How to write a policy

Since policies are executed with [Open Policy Agent](https://www.openpolicyagent.org), they must be written in the [Rego language](https://www.openpolicyagent.org/docs/latest/policy-reference/).

The whole set of Rego capabilities is supported. In addition to that, Rönd also provides a set of custom built-ins.

## Rego input

Rönd provides policies a special `input` object.  
This object contains information concerning the request and is shaped as follows:

```
{
  "request": {
    "method":  String,
    "path":    String,
    "headers": Object {
      String: Array[String]
    },
    "pathParams": Object,
    "query":   Object {
      String: Array[String]
    },
    "body": Object
  },
  "response": {
    "body": Object
  },
  "user": {
    "properties": Object{
      // this object contains the user properties specified in the header provided with the `USER_PROPERTIES_HEADER_KEY`
    },
    "groups": Array[String], // list of strings provided in the `USER_GROUPS_HEADER_KEY`
    "bindings": Array[Binding], // only provided if MongoDB configuration is set
    "roles": Array[Role], // only provided if MongoDB configuration is set
    "resourcePermissionsMap": Object{} // Created only for APIs with `option.enableResourcePermissionsMapOptimization` enabled
  },
  "clientType": String
}
```

{%
  include alert.html
  type="info"
  content="The request body in the input object is only provided if the request method is either `POST`, `PUT`, `DELETE`  or `PATCH` and the request Content Type is `application/json`."
%}

### Regarding `resourcePermissionsMap`

Some policies, especially the ones that needs to perform time-consuming tasks on the response body, may take a while to be processed. Since iterating over user bindings, roles, and permissions may be time-consuming you can use the `resourcePermissionsMap` to verify user permissions in constant time.

The `resourcePermissionsMap` is a map containing a set of key/value pairs. The key is composed as `permissionId:resourceType:resourceId`, and the value is always set to `true`.

{%
  include alert.html
  type="info"
  content="Since creating the map still requires some computations to be performed over the user bindings, you may not perceive any optimization benefit from using the feature, if the Rego policy is already fast. Enable the feature only when necessary to avoid useless computations to be performed."
%}

### Rego package

In order to execute policy a valuation, a Rego package file is required. You have to provide it inside the directory specified with the `OPA_MODULES_DIRECTORY` environment variable.  
Since the package **must** be named `policies`, the Rego file must start as follows:

```go
package policies

// Write your policy here ...
```

## Rego Policies 101

Let's look to an example of a policy that checks the presence of a specific header in the `request` field.  
The policy contains a *permission* named `api_key`, and the required header that we want to check is `x-api-key`.

To do this, we can follow two similar approaches:  
We can look for the `x-api-key` header to be contained in the `input.request.headers` map:

```rego
package policies

default api_key = false
api_key {
  count(input.request.headers["X-Api-Key"]) != 0
}
```

In alternative, Rönd provides a set of [built-in functions](/docs/policy-integration#custom-built-ins) like the [`get_header`](/docs/policy-integration#get_header) one.

For a more complete set of available functions please check out the [Rönd cheat-sheet](/docs/cheat-sheet).  
You can find out other real examples on our [examples repository on GitHub](https://github.com/rond-authz/example). 


### Rows Filtering

Sometimes you may need to filter out some results (e.g. basing on user privileges).  
When defining a policy with `requestFlow`, Rönd can automatically generate a query for your DBMS coming from the evaluation of the permissions of a user.

To do this, just set `"generateQuery": true` inside the `requestFlow` object in your OAS schema.

Rönd will now provide you the iterable `data.resources[_]` to build up the query. 
You can use it to perform any kind of comparison between it and anything you want from the input or maybe a constant value.  
We remind you that the query will be built from the field name of the `resource` object accessed in the permission.

This query is then passed to the requested service through the header specified by the `headerName` field in your [OAS Schema](/docs/configuration#openapi-specification-file).

{%
  include alert.html
  type="warning"
  content="To build your query remember to assign to `data.resources[_]` iterable the same properties you have defined in the data model you need to be queried."
%}

{%
  include alert.html
  type="warning"
  content="For Rönd versions **older than 1.5**, the `headerKey` field of your OAS Schema should be used."
%}

{%
  include alert.html
  type="warning"
  content="Policy for row filtering does not support the `default` declaration.  
  E.g: `default allow = true`. Using it will prevent any filter to be generated."
%}

Let's see it in action:
``` json
"x-rond": {
   "requestFlow": {
      "policyName": "generate_query_policy_name",
      "generateQuery": true,
      "queryOptions": {
         "headerName": "x-query-header"
      }
   }
}
```

``` rego
package policies

generate_query_policy_name {
   input.request.method == "GET"
   resource := data.resources[_]
   resource._id == input.user.userId
   resource.description == "this is the user description"
}

generate_query_policy_name {
   input.request.method == "GET"
   resource := data.resources[_]
   resource.managerId == input.user.userId
   resource.name == input.request.path[1]
}
```

In the example above, given a valid rows filtering configuration, the policy requires that the requested resource details can be retrieved by the user and by its manager.  
This way, given the following input to the permission evaluator:
```json
[
   {
      "input": {
         "method": "GET",
         "request": { "path": ["resource", "654321"] },
         "user": { "userId": "123456" }
      }
   }
]
```

The request is accepted and in the header specified by `headerName` we can find the following object representing a MongoDB query:
```json
[
   {
      "$or":[
         {
            "$and":[
               {
                  "_id":{
                     "$eq":"123456"
                  }
               },
               {
                  "description":{
                     "$eq":"this is the user description"
                  }
               }
            ]
         },
         {
            "$and":[
               {
                  "managerId":{
                     "$eq":"123456"
                  }
               },
               {
                  "_id":{
                     "$eq":"654321"
                  }
               }
            ]
         }
      ]
   }
]
```

Let's inspect another use case.  
This time we want to allow the request only if the user is between 20 and 30 years old.

```rego
package policies

check_user_age {
   input.request.method == "GET"
   resource := data.resources[_]
   resource._id == input.userId
   resource.age >= 20
   resource.age <= 30
}
```

Given the following input to the permission evaluator:
```json
[
   {
      "input": {
         "method": "GET",
         "userId": 12345
      }
   }
]
```

The request is accepted and in the header specified by `headerName` we can find the following object representing a MongoDB query:
```json
[
   {
      "$and":[
         {
            "_id":{
               "$eq":12345
            }
         },
         {
            "age":{
               "$gte":20
            }
         },
         {
            "age":{
               "$lte":30
            }
         }
      ]
   }
]
```


### Response Filtering

Sometimes you need to filter out or manipulate some fields in the response before sending it to the client This case is known as `Response Filtering`.  
Rönd allows you to manipulate the response body directly in a policy.

Unlike Request policies where policies returns a boolean value, Response Filtering policies should return a set that represents the new response object.  
For more details about sets, check out the [OPA documentation](https://www.openpolicyagent.org/docs/latest/policy-language/#generating-sets).

The policy MUST respect the syntax:

``` rego
policy_name [return_value] {
   somePolicyContent == true
   return_value := "the body of the response"
}
```

And MUST return the new value of the modified data:

```rego
filter_response_example [result] {
   body := input.request.body
   result := object.remove(body, ["someField"])
}
```

{%
  include alert.html
  type="warning"
  content="Response filtering is applied only if the response Content Type is `application/json`.  
  If any other Content Type is found (based on the `Content-Type` header value), an error will be sent to the caller."
%}

## Custom Built-ins

### `get_header`

Since headers keys are transformed in canonical format (i.e. "x-api-key" become "X-Api-Key") by Go, you can use the `get_header` built-in function to allow accessing them in case-insensitive mode.

```
output := get_header(headerKey: String, headers: Map[String]Array<String>) 
```

The returned output is the first header value present in the `headers` map at key `headerKey`. If `headerKey` doesn't exist, the returned output is an empty string.

Without the built-in, you have to access headers in canonical form:

```go
package policies

default api_key = false

api_key {
  count(input.request.headers["X-Api-Key"]) != 0
}
```

However, with the `get_header` function, you can write the header name as you prefer:

```go
package policies

default api_key = false
api_key {
  get_header("x-api-key", input.request.headers) != ""
}
```

### `find_one` 

The `find_one` built-in function can be used to fetch data from a MongoDB collection.  
It accepts the collection name and a custom query, and returns the document that matches the query using the `FindOne` MongoDB API.

Example usage to fetch a rider from the `riders` collection by using the rider identifier provided in the request path parameters:

```rego
riderId := input.request.pathParams.riderId
rider := find_one("riders", { "riderId": riderId })
rider.available == true
```

### `find_many`

The `find_many` built-in function can be used to fetch multiple data from a MongoDB collection.  
It accepts the collection name and a custom query, and returns the documents that match the query using the `Find` MongoDB API.

Example usage to fetch a rider from the `riders` collection by using the rider identifier provided in the request path parameters:

```rego
riders := find_many("riders", { "available": true })
rider := riders[_]
rider.riderId == input.request.pathParams.riderId
```


## RBAC Data Model

In order for Rönd to perform checks over users permissions, you need to configure the `MONGODB_URL`, `ROLES_COLLECTION_NAME` and `BINDINGS_COLLECTION_NAME` variables.

When these variables are configured, Rönd will fetch users permissions data from the specified sources.  
These permissions will be then provided with the two iterables `input.user.roles[_]` and `input.user.bindings[_]`.  
You can now use these two iterables to build up your policy according to your needs.

{%
  include alert.html
  type="warning"
  content="If `MONGODB_URL` variable is set, then the envs `ROLES_COLLECTION_NAME` and `BINDINGS_COLLECTION_NAME` are required."
%}

{%
  include alert.html
  type="info"
  content="A _Subject_ may represent both a user or another application. Its identifier is retrieved by Rönd from the Mia-Platform standard header `miauserid`  

  _Groups_ are retrieved by Rönd from the Mia-Platform standard header `miausergroups`)"
%}

##### Roles

This collection contains all the roles needed in an organization with their specific permissions.
The collection fields are:

- **roleId** (string, required): the **_unique_** id of the roles
- **name** (string,required): name of the role
- **description** (string): description of the role
- **permissions** (string array, required): list of permissions ids for the role

```json
[
   {
      "roleId": "roleUniqueIdentifier",
      "name": "TL",
      "description": "company tech leader",
      "permissions": [
         "console.project.view",
         "console.environment.deploy",
         "console.company.billing.view"
      ]
   }
]
```

##### Bindings

A Binding represents an association between a set of Subjects (or groups), a set of Roles and (optionally) a Resource.
This collection contains all the bindings between users or groups of users and a resource with a list of roles. The fields for this collections are:

- **bindingId** (string, required): **_unique_** id of the binding
- **groups** (string array): list of user group identifiers
- **subject** (string array): list of user ids
- **roles** (string array): list of role ids
- **permissions**: (strings) list of permission ids
- **resource**: (object) with properties `type` and `id`

```json
[
   {   
      "bindingId": "bindingUniqueIdentifier",
      "groups": [
         "team1"
      ],
      "subjects": [
         "bob"
      ],
      "roles": [
         "TLRoleId"
      ],
      "permissions": [
         "canDoStuff",
         "canDoActions",
      ],
      "resource": {
         "resourceId": "project1",
         "resourceType": "project"
      }
   }
]
```

### RBAC Policies for permission evaluation

Let's check, in the following example, if the user have the permission to read some data:  
```rego
package policies

has_read_permission {
   userRoles := input.user.roles[_]
   userRoles.permissions[_] == "can_read"
}
```

## RBAC Based ACL Data Model

You can use a the [binding association collection](#bindings) to rapresent a ACL data model, in this case you could only use the following fields:

- **bindingId** (string, required): **_unique_** id of the binding
- **subject** (string array): list of user ids, _usually contains only one subject_
- **permissions**: (strings) list of permissions or grants (i.e. `canRead`, `canWrite`, ....)
- **resource**: (object) with properties `type` and `id`.

```json
[
   {   
      "bindingId": "bindingUniqueIdentifier",
      "subjects": [
         "bob"
      ],
      "permissions": [
         "canDoStuff",
         "canDoActions",
      ],
      "resource": {
         "resourceId": "project1",
         "resourceType": "project"
      }
   }
]
```

### RBAC Based ACL Policies for permission evaluation

Let's check, in the following example, if the user have the permission to read some data:  
```rego
package policies

has_read_permission {
   user := input.user
   user.permissions[_] == "canRead"
}

has_read_permission_for_given_company {
   companyId := input.request.query["companyId"][0]
   user := input.user[_].resource == { resourceType: "company", resourceId: companyId }
   user.permissions[_] == "canRead"
}
```

## ACL Data Model

```json
[
   {   
      "_id": "bindingUniqueIdentifier",
      "userId": "bob",
      "permissions": [
         "canDoStuff",
         "canDoActions",
      ],
      "resource": {
         "resourceId": "project1",
         "resourceType": "project"
      }
   }
]
```

### ACL Policies for permission evaluation

Let's check, in the following example, if the user have the permission to read some data:  
```rego
package policies

has_read_permission {
   user := find_one("users", { "userId": input.user.id })
   user.permissions[_] == "canDoStuff"
}
```