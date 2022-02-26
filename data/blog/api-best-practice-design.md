---
title: REST API design
date: 2014-06-01
tags: ['rest', 'architecture', 'api']
draft: false
summary: Best practices when designing a REST API, as commanded from an ivory tower.
---

Here follows an aggregation of some generic rules for generating a Web API to program against. Based upon a number of references, supplied at the bottom of this post.

## API design rules

### Use REST verbs

Use HTTP verbs which make sense from the perspective of the API consumer:

- `GET` will retrieve an resource
- `POST` will create a resource
- `PUT` will update a resource
- `PATCH` will partially update a resource
- `DELETE` will delete a resource

### Use nouns as endpoint names

Try and use nouns as endpoint names. For example:

- `GET /incidents` returns a list of incidents
- `GET /people` returns a list of people

If this is not plausible, verbs can be used in certain circumstances:

- `GET /search` returns a search endpoint

### Use plural names for endpoints

Keep the URL format consistent and always use a plural. Not having to deal with odd pluralization (person/people, mouse/mice) makes the life of the API consumer better and is easier for the API provider to implement (as most modern frameworks will natively handle /incidents and /incidents/12 under a common controller).

- `GET /incidents` returns a list of incidents
- `GET /incidents/201401011` returns an incident with the id '201401011'

### Relationships

If a relation can only exist within another resource, they should be accessed using a sub-action of the controller. For example:

- `GET /incidents/201401011/attachments` retrieves a list of attachments from incident '201401011'
- `GET /incidents/201401011/attachments/1` retrieves attachment '1' from incident '201401011'

Alternatively, if a relation can exist independently of the resource, it makes sense to just include an identifier for it within the output representation of the resource. The API consumer would then have to hit the relation's endpoint.

### Non-CRUD actions

If an operation does not fit inside a normal CRUD operation (e.g. Authorize SITREP) treat it like a sub-action and use RESTful principles to manipulate the action. Name the action using a verb. For example:

- `PUT /incidents/201401011/sitreps/1/authorize` authorizes sitrep '1' on incident '201401011'
- `DELETE /incidents/201401011/sitreps/1/authorize` unauthorizes sitrep '1' on incident '201401011'

### Documentation

All API actions should be documented using the standard .Net Xml documentation methods, as outlined in [this document](http://www.asp.net/web-api/overview/creating-web-apis/creating-api-help-pages).

### Versioning

The API will have major version numbers prepended to the URL. For example:

- `GET v1/incidents/201401011/sitreps/1/authorize` uses API major version '1'
- `GET v2/incidents/201401011/sitreps/1/authorize` uses API major version '2'

### Result filtering

Use a unique query parameter for each field that implements filtering. For example, when requesting a list of tickets from the /incidents endpoint, you may want to limit these to only those in the going state.

This is accomplished with a request like GET incidents?status=open. Here, state is a query parameter that implements a filter:

- `GET /incidents?state=going`

### Result sorting

Similar to filtering, a generic parameter sort can be used to describe sorting rules. Accommodate complex sorting requirements by letting the sort parameter take in list of comma separated fields, each with a possible unary negative to imply descending sort order. Let's look at some examples:

- `GET /incidents?sort=-status` - Retrieves a list of incidents in descending order of status
- `GET /tickets?sort=-status,created_at` - Retrieves a list of incidents in descending order of status. Within a specific priority, newer incidents are ordered first

### Aliases for common queries

To make the API experience more pleasant for the average consumer, consider packaging up sets of conditions into easily accessible RESTful paths. For example, a query to only show incidents which are at emergency warning could be packaged up as:

- `GET /incidents/emergency_warning`

### Limiting return fields from API

The API consumer doesn't always need the full representation of a resource. The ability select and chose returned fields goes a long way in letting the API consumer minimize network traffic and speed up their own usage of the API.

Use a fields query parameter that takes a comma separated list of fields to include. For example, the following request would retrieve just enough information to display a sorted listing of open tickets:

- `GET /incidents?fields=id,summary,updated_at&state=going&sort=-updated_at`

### Updates & creation should return a resource representation

A PUT, POST or PATCH call may make modifications to fields of the underlying resource that weren't part of the provided parameters (for example: created_at or updated_at timestamps). To prevent an API consumer from having to hit the API again for an updated representation, have the API return the updated (or created) representation as part of the response.

In case of a POST that resulted in a creation, use a HTTP 201 status code and include a Location header that points to the URL of the new resource.

### Envelopes

Don't use an envelope by default, but make it possible when needed. We can future proof the API by staying envelope free by default and enveloping only in exceptional cases.

There are 2 situations where an envelope is really needed - if the API needs to support cross domain requests over JSONP or if the client is incapable of working with HTTP headers.

These 2 scenarios are unlikely when we have access to HTTP Headers and will likely not use JSONP.

### Pagination

Pagination should not be part of an envelope. The correct way to include pagination details today is using the [Link header introduced by RFC 5988](http://tools.ietf.org/html/rfc5988#page-6). For example:

`Link: <https://api.contoso.com/users?page=3&per_page=100>; rel="next", <https://api.contoso.com/users?page=50&per_page=100>; rel="last"`

But this isn't a complete solution as many APIs do like to return the additional pagination information, like a count of the total number of available results. An API that requires sending a count can use a custom HTTP header like `X-Total-Count`.

### Errors

Just like an HTML error page shows a useful error message to a visitor, an API should provide a useful error message in a known consumable format. The representation of an error should be no different than the representation of any resource, just with its own set of fields.

The API should always return sensible HTTP status codes. API errors typically break down into 2 types: 400 series status codes for client issues & 500 series status codes for server issues. At a minimum, the API should standardize that all 400 series errors come with consumable JSON error representation. If possible (i.e. if load balancers & reverse proxies can create custom error bodies), this should extend to 500 series status codes.

A JSON error body should provide a few things for the developer - a useful error message, a unique error code (that can be looked up for more details in the docs) and possibly a detailed description. JSON output representation for something like this would look like:

```
{
  "code" : 1234,
  "message" : "Something bad happened :(",
  "description" : "More details about the error here"
}
```

Validation errors for PUT, PATCH and POST requests will need a field breakdown. This is best modelled by using a fixed top-level error code for validation failures and providing the detailed errors in an additional errors field, like so:

```
{
  "code" : 1024,
  "message" : "Validation Failed",
  "errors" : [
    {
      "code" : 5432,
      "field" : "first_name",
      "message" : "First name cannot have fancy characters"
    },
    {
        "code" : 5622,
        "field" : "password",
        "message" : "Password cannot be blank"
    }
  ]
}
```

### HTTP status codes

HTTP defines a bunch of meaningful status codes that can be returned from your API. These can be leveraged to help the API consumers route their responses accordingly. I've curated a short list of the ones that you definitely should be using:

- `200 OK` - Response to a successful GET, PUT, PATCH or DELETE. Can also be used for a POST that doesn't result in a creation.
- `201 Created` - Response to a POST that results in a creation. Should be combined with a Location header pointing to the location of the new resource
- `204 No Content` - Response to a successful request that won't be returning a body (like a DELETE request)
- `304 Not Modified` - Used when HTTP caching headers are in play
- `400 Bad Request` - The request is malformed, such as if the body does not parse
- `401 Unauthorized` - When no or invalid authentication details are provided. Also useful to trigger an auth popup if the API is used from a browser
- `403 Forbidden` - When authentication succeeded but authenticated user doesn't have access to the resource
- `404 Not Found` - When a non-existent resource is requested
- `405 Method Not Allowed` - When an HTTP method is being requested that isn't allowed for the authenticated user
- `410 Gone` - Indicates that the resource at this end point is no longer available. Useful as a blanket response for old API versions
- `415 Unsupported Media Type` - If incorrect content type was provided as part of the request
- `422 Unprocessable Entity` - Used for validation errors
- `429 Too Many Requests` - When a request is rejected due to rate limiting

## References

1. [Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
2. [Creating Help Pages for ASP.NET Web API](http://www.asp.net/web-api/overview/creating-web-apis/creating-api-help-pages)
3. [ASP.NET Web API: Using Namespaces to Version Web APIs](http://blogs.msdn.com/b/webdev/archive/2013/03/08/using-namespaces-to-version-web-apis.aspx)
4. [Web API Design Crafting Interfaces that Developers Love](http://info.apigee.com/Portals/62317/docs/web%20api.pdf)
