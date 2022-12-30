---
title: Configuration
tags:
  - starter-kit
  - configuration
description: Configure Rönd
meta_description: 
order: 2
---

# Rönd configuration

To correctly setup the container, you need to provide a few configurations. Some configurations are provided as environment variables, while others are provided via configuration files.

## Environment variables

| Name                         | Type      | Default value       | Required                   | Description                                                                                                                                                                                       |
|------------------------------|-----------|---------------------|----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `LOG_LEVEL`                  | `string`  | _info_              | -                          | One of "_info_", "_trace_", "_debug_", "_warning_", "_error_".                                                                                                                                    |
| `HTTP_PORT`                  | `string`  | **8080**            | -                          | Port to expose the API service.                                                                                                                                                                   |
| `STANDALONE`                 | `boolean` | `false`             | -                          | Defines whether the service is running as a sidecar or in standalone mode.                                                                                                                        |
| `TARGET_SERVICE_HOST`        | `string`  | -                   | if `STANDALONE` is `false` | The host of target service to redirect the traffic when authorization rules are passed.                                                                                                           | 
| `TARGET_SERVICE_OAS_PATH`    | `string`  | -                   | if `STANDALONE` is `false` | Endpoint path of sibling container to contact for retrieving the OAS definition (es. `/documentation/json`).                                                                                      |
| `API_PERMISSIONS_FILE_PATH`  | `string`  | -                   | -                          | File for manual configuration of the OAS. This replaces the automatic documentation fetch performed by the service towards `API_ERMISSIONS_FILE_PATH`. [See the example](#specify-api-permissions).|
| `OPA_MODULES_DIRECTORY`      | `string`  | -                   | ✅                          | Folder path where are provided the Rego files for OPA rules evaluation; these files will be used to evaluate policy. [See the example](policy-integration).                                           |
| `USER_PROPERTIES_HEADER_KEY` | `string`  | `miauserproperties` | -                          | Header name for the optional header that contains the user properties.                                                                                                                            |
| `USER_GROUPS_HEADER_KEY`     | `string`  | `miausergroups`     | -                          | Header name for the optional header that contains the user groups.                                                                                                                                |
| `CLIENT_TYPE_HEADER_KEY`     | `string`  | `client-type`       | -                          | Header name for the optional header that contains a client type identifier.                                                                                                                       |
| `MONGODB_URL`                | `string`  | -                   | -                          | URL to connect to a MongoDB instance (useful for RBAC data retrieval and `find_` Rego built-ins).                                                                                                 |
| `ROLES_COLLECTION_NAME`      | `string`  | -                   | if `MONGODB_URL` is set    | Name of the role collection.                                                                                                                                                                      |
| `BINDINGS_COLLECTION_NAME`   | `string`  | -                   | if `MONGODB_URL` is set    | Name of the bindings collection.                                                                                                                                                                  |
| `PATH_PREFIX_STANDALONE`     | `string`  | `/eval`             | -                          | When in `STANDALONE` mode, this variables configures the prefix for all validation APIs defined by the OAS.                                                                                       |
| `DELAY_SHUTDOWN_SECONDS`     | `int`     | **10**              | -                          | Seconds to delay forced server stop, useful for graceful shutdown.                                                                                                                                |
| `EXPOSE_METRICS` | `boolean` | `true` | - | Set to false if you don't want to collect and expose the Prometheus metrics |

## Configuration files

Rönd accepts two configuration files, one containing the OpenPolicy Agent Rego policies (required) and an optional file for OpenAPI Specification details.

### OpenPolicy Agent policies

The `.rego` file must be provided inside the directory specified with the `OPA_MODULES_DIRECTORY` environment variable. Right now is supported only a single file, so the first `.rego` file found will be used for policy evaluation. Any other `.rego` file will be ignored.

### OpenAPI Specification file

The OpenAPI Specification file is required only when the `TARGET_SERVICE_OAS_PATH` variable is not provided. This file specifies the API that are exposed by your service, and thus must be authorized with specific policies.

#### Specify API permissions

In order to define the Rego policies to be evaluated for the API to be authorized, you must define the Rönd custom attribute in your OAS schema.  
Depending on the version of Rönd you are using, you have define `x-rond` or `x-permission` attirbutes.

**For version 1.5 or newer**, Rönd lets you define the `x-rond` attribute in your OAS schema.
This attribute is an object that contains the following properties:
```json
"x-rond": {
    "requestFlow": {
        "policyName": "policy_to_be_executed_BEFORE_API_invocation",
        "generateQuery": true,
        "queryOptions": {
            "headerName": "x-query-header"
        }
    },
    "responseFlow": {
        "policyName": "policy_to_be_executed_AFTER_API_invocation"
    },
    "options": {
        "enableResourcePermissionsMapOptimization": false
    }
}
```

Here, `requestFlow` and `responseFlow` fields, define which policies have to be executed repectively before and after the API invocation.

{%
  include alert.html
  type="info"
  content="The `generateQuery` flag defines wether or not a query has to be generated and then forwarded to the application service through the header specified in the `headerName` field."
%}


**For older versions** the `x-permission` attribute should be used.  
This attribute is shaped as an object with the following properties:
```json
"x-permission": {
    "allow": "policy_to_be_executed_BEFORE_API_invocation", // Required.
    "resourceFilter": {
        "rowFilter": {
            "enabled": true,
            "headerKey": "acl_rows" // Identifier of the header sent to the requested service. The interpolated query will be injected Here. Defaults to `acl_rows`.
        }
    },
    "responseFilter": {
        "policy": "policy_to_be_executed_AFTER_API_invocation"
    },
    "options": {
        "enableResourcePermissionsMapOptimization": false
    },
}
```

{%
  include alert.html
  type="info"
  content="The `enableResourcePermissionsMapOptimization` flag enables the generation of an optimized map of user permissions. This is useful when performing RBAC logics.  
  More information available in the [Policy Integration page](policy-integration)"
%}

{%
  include alert.html
  type="warning"
  content="For Rönd versions **older than 1.5**, when defining the `resourceFilter` attribute, Rönd changes the behavior of the `allow` policy.  
  This allows you to write a policy that returns a query that is then forwarded to the application service using the header specified with the `headerKey` option.  
  For more details check the [Rows Filtering guide](/docs/policy-integration#rows-filtering)."
%}

{%
  include alert.html
  type="warning"
  content="Please note that any API that is not specified is immediately blocked."
%}

#### Request Flow

If you want the `greetings_read` policy to be evaluated when invoking the `GET /hello` API, your application service must define its API documentation as follows:

```json
{
    "paths": {
        "/hello": {
            "get": {
                "x-rond": {
                    "requestFlow": {
                        "policyName": "greetings_read"
                    }
                }
            }
        }
    }
}
```

#### Generate a query for the service

If you want to generate a query that is then forwarded to the application service, you can define the `requestFlow` option enabling the `generateQuery` field.  
The generated query is then forwarded to the application service using the header specified in the `headerName` field.  
For more details check the [Rows Filtering guide](/docs/policy-integration#rows-filtering).

```json
{
    "paths": {
        "/hello": {
            "get": {
                "x-rond": {
                    "requestFlow": {
                        "policyName": "greetings_read",
                        "generateQuery": true,
                        "queryOptions": {
                            "headerName": "x-acl-rows"
                        }
                    }
                }
            }
        }
    }
}
```

#### Response Flow

If you need to modify the response payload you can define the `responseFlow` field. 
With the configuration shown below, the `greetings_read` policy will be evaluated before contacting the application service.
When the application service sends its response, the `filter_response_example` policy will be evaluated and its result will be used as the new response body.

```json
{
    "paths": {
        "/hello": {
            "get": {
                "x-rond": {
                    "requestFlow": {
                        "policyName": "greetings_read",
                    },
                    "responseFlow": {
                        "policyName": "filter_response_example"
                    }
                }
            }
        }
    }
}
```


> Any API invocation to the path matching the one provided as `TARGET_SERVICE_OAS_PATH` with method `GET` will always be proxied to the target service, unless the given OpenAPI Specification provides the path with a custom policy configuration. In this case, the API will be proxied only if the policy evaluates successfully.

## Standalone mode

Rönd can run in standalone mode. To enable this mode, just set the `STANDALONE` environment variable to `true`. 

When the standalone mode is active, Rönd will expose the APIs defined in the provided OAS under the `PATH_PREFIX_STANDALONE` environment variable.
