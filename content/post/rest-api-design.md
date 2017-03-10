+++
menu = ""
description = ""
categories = []
tags = []
banner = ""
date = "2017-03-03T15:47:54+08:00"
title = "rest api design"
draft = true
images = []
+++

*note from Google and Microsoft design guide*

## Core idea behind REST
Its core principle is to define named resources that can be manipulated using a small number of methods.

## Design Flow
The Design Guide suggests taking the following steps when designing resource- oriented APIs (more details are covered in specific sections below):

* Determine what types of resources an API provides.
* Determine the relationships between resources.
* Decide the resource name schemes based on types and relationships.
* Decide the resource schemas.
* Attach minimum set of methods to resources.

## HTTP verbs

Method  | Description                                                                                                                | Is Idempotent
------- | -------------------------------------------------------------------------------------------------------------------------- | -------------
GET     | Return the current value of an object                                                                                      | True
PUT     | Replace an object, or create a named object, when applicable                                                               | True
DELETE  | Delete an object                                                                                                           | True
POST    | Create a new object based on the data provided, or submit a command                                                        | False
HEAD    | Return metadata of an object for a GET response. Resources that support the GET method MAY support the HEAD method as well | True
PATCH   | Apply a partial update to an object                                                                                        | False
OPTIONS | Get information about a request; see below for details.                                                                    | True

*ps: Services that allow callers to specify key values on create SHOULD support UPSERT semantics, and those that do MUST support creating resources using PATCH.*

## API URL

```http
GET https://api.contoso.com/v1.0/people/123/addresses
```

Note the structure of url.
  * It includes a version
  * It use plural for collection name
  * resources can be nested

## Query
**Filter**
```http
GET https://library.googleapis.com/v1/shelves?filter=price lt 10.00
```
**Sort**
```http
GET https://api.contoso.com/v1.0/people?orderBy=name desc,hireDate asc
```
**pagination**
```http
GET https://api.contoso.com/v1.0/people?page=1&&page_size=10

{
  "value":[...],
  "next_page": 2
}
```
**Partial**
```http
GET https://library.googleapis.com/v1/shelves?fields=name
```
**View**
```http
GET https://library.googleapis.com/v1/shelves/shelf1/books?view=BASIC
```
