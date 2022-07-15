---
title: Cheat Sheet
tags:
  - cheat-sheet
  - tutorial
description: Cheat Sheet to boost up your policy writing skills
order: 4
---

# Cheat sheet

This document describes a few examples of policies that can be written in Rego.

## Policies composition in OR

This simple policy will accept only requests with method `GET` or `HEAD`. It uses assignment to define new values `:=` and equality checks `==`.

```
my_policy {
    requestMethod := input.request.method
    requestMethod == "GET"
} {
    requestMethod := input.request.method
    requestMethod == "HEAD"
}
```

## Array iteration

Supposing that user properties have the `myList` which holds a list of strings, you can iterate the list and find whether an element is found with comparisons.

```
my_iteration_policy {
    aList := input.user.properties.myList
    aList[_] == "item_to_find"
}
```

### Iterate multiple lists

You can iterate over multiple lists using the same index by defining a custom iterator:

```
my_iteration_policy {
    aList := input.user.properties.myList
    anotherList := input.user.properties.myList2
    
    some i
    aList[i] == "item_to_find"
    anotherList[i] == "item_to_find_on_second_list"
}
```

## Query generation

If you want to create a query using the Query Generation feature of Rönd, you have to use the `data.resources` variable to obtain the generator and define your comparising you want to be matched in the final query.

```
my_query_generator {
    userBrand := input.user.properties.brandId
    generator := data.resources[_]

    generator.brandId == userBrand
}
```

The policy above generates a query that will filter for all the documents having `brandId` matching the `brandId` of the user.

## Debugging

You can use the `print` statement to debug your policies. Print statements are available only when `LOG_LEVEL` variable is set to `trace`.
