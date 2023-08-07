---
layout: post
title:  "Create a professional web service - Part IV - HATEOAS"
author: Jordi Dalmau Heras
date:   2023-07-10 17:48:24 +0200
categories: development
---
Learn how to build a profesional web service with JavaScript, NodeJS and Express.

## HATEOAS

Hypermedia as the Engine of Application State (HATEOAS) is a category within the REST architectural style that involves replacing parts of the information represented by links. This allows the application that downloads the information to handle the structure of this information without prior knowledge and treat it later.

HATEOAS can be used to provide intelligence to the client, enabling it to make other calls from a single call.

There is no absolute standard for how to represent hypermedia controls.

To help explain the specific properties of a web-style system and identify where HATEOAS fits, we can use the [Leonard Richardson](http://www.crummy.com/) four level [model of restful maturity](https://martinfowler.com/articles/richardsonMaturityModel.html).

### Level 0, HTTP as a transport system for remote interactions

The starting point for the model is using HTTP as a way to communicate remotely, but without using fully the features of the web. Instead, it uses HTTP like a tunnel to create its own remote interaction mechanism, often relying on Remote Procedure Invocation.

For example, a level 0 API uses verbs in the URLs (a bad practice for REST):

- /addContainer
- /deleteContainer
- /getClients

### Level 1, resources

The second level for the model considers the elements of the API as resources. Each one has its own endpoint, and the URLs do not end with a slash '/'. The slash is used only to set a hierarchy between resources.

For example, the endpoints at this level use resource names instead of verbs or something else:

- /container
- /client
- /register

### Level 2, verbs

The third level for the model is when the API uses the [verbs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods), [parameters of the URL and the query parameters](https://en.wikipedia.org/wiki/URL), [HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) for the results, and [headers](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields), used as metadata with different pourpose.

For example, the endpoints of this level are like:

- [GET]/container?clientId=1234
- [DELETE]/container/1

### Level 3, hypermedia controls

This level is divided into two aspects, content negotiation and resource link discovery.

**Content negotiation** allows the client to specify the format of the data in which it wants the result. It is requested with the Accept header in the request. For example, a REST service client that wants the data in JSON format must provide an Accept: application/json header and if it wants it in XML format an Accept: application/xml header. In case of sending data in the request body, the format of the data provided is specified with the Content-Type header. In case the service does not support the provided data type or is not capable of providing the requested format, it returns the status code 415 indicating unsupported data type format.

**Resource link or HATEOAS** is the principle that applies links in the data of the entities that allows to navigate between them and discover the available actions, a client of a REST service that implements HATEOAS does not need to know the URLs to interact with the different actions, these are returned in the response data as metadata.

For example, a response with HATEOAS can be like:

```bash
#!/usr/bin/env bash
curl -v http://localhost:8080/product/1

{
  "productId": 1,
  "productName": "product 1",
  "links": [
    {
        "href": "1/stocks",
        "rel": "products",
        "type" : "GET"
    }
  ]
}
```

## Swagger and HATEOAS

Swagger allows you to document a REST service and includes support for documenting a service that complies with the HATEOAS hypermedia principle. Swagger provides several annotations that are included in the service interface. When processing these annotations, it generates a schema of the service interface with OpenAPI, which is then used to generate documentation including the endpoints, arguments, verbs, response codes, and model data.

Swagger also allows you to call services and obtain the curl command to make the request from the command line.

## Advantages and problems

With HATEOAS, clients do not construct URLs of resources to make requests; instead, they obtain the URLs from the response data. At the same time, the response specifies the possible actions, so the client does not need to implement logic to determine if an operation is possible. The application also does not need to construct URLs with string interpolation to include an entity identifier; the full link is returned in the data. This approach allows clients to avoid implementing this logic and reduces the risk of the server and client's possible operations logic getting out of sync.

On the downside, HATEOAS adds complexity to the API, which affects both the API developer and the API consumer. Additional work needs to be done to add the appropriate links in each response based on the state of the entity. This makes building the API more complex compared to an API that does not implement HATEOAS.

API clients also face added complexity in understanding the semantics of each binding and processing each response to get the bindings. While the benefits of HATEOAS may outweigh this added complexity, it must be taken into account.

If the API is public, there is a possibility that some clients might misuse it and not utilize hypermedia, rendering HATEOAS useless in such cases

## Useful links

- <https://martinfowler.com/articles/richardsonMaturityModel.html>
- <https://picodotdev.github.io/blog-bitix/2021/07/los-niveles-de-madurez-rest-ejemplo-con-hateoas-y-documentacion-con-swagger-de-un-servicio-con-spring-boot/>
- <https://www.npmjs.com/package/express-hateoas-links>
