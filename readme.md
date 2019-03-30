# JSON Payloads
This page describes the structure that JSON payloads will use for requests, responses and errors. in addition, it defines the approach for several general concepts such as sorting, paging and filtering. The approach utilized for sorting, paging and filter should be consistent across the API.

### Top Level Reserved Words in request, response and body payloads
Most of the major API vendors have developed a consistent approach for their APIs by creating a consistent JSON style guide. Some companies such as Google, have defined JSON top-level words so the services all have some consistency. Borrowing from Google and JSON-API, several reserved words should be used. Placing these words at the top level of the JSON (and moving domain data to the second level) allows for a standard while also allowing for the flexibility to add more words in the future.

- meta - metadata non-standard information about the request or response. It is optional implementation-specific.
- data - service or domain payload. Used with both requests (POST, PATCH, PUT) and responses. Payloads will either have data or errors. When both, errors take precedence. 
- errors - list of errors on a response
- page - contains discreet fields for paging, values used depends on the type of set being navigated. Typical keys are size, totalPages, number (meaning current page number), totalElements
- _links - link relations to self, related resources and paging hrefs. Use of this to provide pre-built links for clients to use to naviagate a result set or link to other resources is optional or may be used as needed.

#### Request Payloads
The examples below are applicable whenever a client performs a POST, PATCH or PUT method and includes a JSON message body.  

**Example 1**: Request contains a list of domain-related fields under the data property.
```
{
  "data": {
    "memberNumber": 99999,
    "taxId": 1234,
    "cifKey": "ci:cd"
  }
}
```
**Example 2**: Request contains two different  domain-related objects, each under a separate child key under data (data.memberParams, data.accountParams). 
```
{
  "data": {
    "memberParams": {
      "memberNumber": 99999,
      "taxId": 1234,
      "cifKey": "ci:cd"
    },
    "accountParams": 545689
  }
}
```

**Example 3**: Request includes a meta top-level attribute  where service-specific information can be passed. This information applies to the entire payload.
```
{
  "meta": {
    "clientContext": "UX594394"
  },
  "data": {
    "memberParams": {
      "memberNumber": 99999,
      "taxId": 1234,
      "cifKey": "ci:cd"
    },
    "accountParams": 545689
  }
}
```

#### Response Payloads

**Example 1**: Response includes some meta information for the client to use. In this case, application is providing some contextual information to the client and also indicating when application will go down. The information is meant to be service-specific.
```
{
  "meta": {
    "requestId": "U12345678",
    "minutesToDowntimeWindow": 20
  },
  "data": {
    "member": "MemberPlaceholder",
    "account": "AccountPlaceholder"
  }
}
```

**Example 2**: Response payload with pagination values for the client to use to construct paging. A summary pf pagination is covered in the Paging - Page Payload Reserved Word and related request parameters section.
```
{
  "data": [...
  ],
  "page": {
    "size": 5,
    "totalElements": 50,
    "totalPages": 10,
    "number": 0
  }
}
```

**Example 3**: Response payload with pagination values for the client to use to either construct paging and or follow links that have also been provided. A summary of _links usage is covered in section Links - Hypermedia links for paging and connection to related resources
```
{
  "data": [...
  ],
  "page": {
    "size": 5,
    "totalElements": 50,
    "totalPages": 10,
    "number": 1
  },
  "_links": {
    "self" : {
      "href" : "http://localhost:8080/persons/1234"
    },
    "next" : {
      "href" : "http://localhost:8080/persons?page=2&size=5" 
    },
    "prev" : {
      "href" : "http://localhost:8080/persons?page=0&size=5"
    }
  }
}
```

#### Error Payloads

Return the errors as a list use them under the top level errors key.

**TODO** provide example of top level error with children

**Example 1**: A single error is returned from the application. It is in the errors list:
```
{
  "errors": [
    {
      "errorCode": "FIELD_OUT_OF_RANGE_ERROR",
      "message": "FE-12345 memberId was out of range description property",
      "details": "ABCDE in errorCode property"
    }
  ]
}
```

**Example 2**: Error response example 2: list of errors. The second one provide field level details on it’s error. The “details” is optional but would allow us to provide discreet values to client if helpful (I could see using it on validation errors). Spring gives a lot of detail and this would be available for clients if we wish to expose it for some detailed errors.

```
{
  "errors": [
  {
    "errorCode": "FIELD_OUT_OF_RANGE_ERROR",
    "message": "FE-12345 memberId was out of range description property",
    "details": "someAddlInfo"
  },
  {
    "errorCode": "CROSS_EDIT_VALIDATION_ERROR",
    "message": "description-CE-23456 memberId must be specified when widgetId is blank",
    "details": "some other info"
  }
  ]
}
```

**Example 3**: Include a meta container on the response when we have something to return in it.
```
{
  "meta": {
    "CLIENT_CORRELATION_ID": "12345"
  },
  "errors": [
    {
      "errorCode": "FIELD_OUT_OF_RANGE_ERROR",
      "message": "FE-12345 memberId was out of range description property",
      "details": "someAddlInfo"
    }
  ]
}
```

#### Paging - Page Payload Reserved Word and related request parameters
GET methods that return a list of resources should have a consistent pattern of communicating paging information between the client application and microservice. Spring Data pattern has a good pattern for this need (see references). Use of pattern with this approach doesn't require an actual implementation of Spring Data; we're simply following the principles:

On GET methods, the page and size path parameters will convey the client's request `http://api.penfed.org/v1/widgets?page=2&size=20`
Payload responses will provide paging information under the top-level page attribute. Sub-attributes will include attributes such as size, totalPages, number and elements (whatever makes sense)

While use of "page" and "size" is the preferred approach that is consistent with Spring Data, some list services will not fit this model. For those situations, an additional option value, nextKey has been added.

On GET methods: `http://api.example.com/v1/widgets?nextKey=1234567`

**GET request query string using page and size**

`http://api.penfed.org/v1/widgets?page=2&size=20`

- page - requested page
- size - number of items to return for the page

Payload Response Key Value Attributes

- number - page number you are currently viewing
- size - size of a page
- totalElements - total elements available to retrieve (when applicable)
- totalPages - total number of pages (when applicable)

```
{
  "data": [...
  ],
  "page": {
    "number": 0,
    "size": 5,
    "totalElements": 50,
    "totalPages": 10
  }
}
```

**GET Request Query String Parameters using Next Key**

Parameters to use with a GET method `http://api.penfed.org/v1/widgets?nextKey=1234567`

- nextKey - id of next resource to retrieve in the list

**Payload Response Key Value Attributes**

nextKey - id of the next resource to retrieve in the list
```
{
  "data": [...
  ],
  "page": {
    "nextKey": "1234567"
  }
}
```

### Links - Hypermedia links for paging and connection to related resources
In a Hypermedia-oriented API, resources providing links make it easier for clients to navigate the application. Use of the "_links" keyword in the response payload is optional to be used as the situation calls for it. In some cases, links will provide paging hrefs for the clients to follow. In other cases, they will provide links to sub-resources or related resources. Whatever the situation, links will have the following characteristics:

Links will be grouped under the top level reserved word _links. This is consistent with Spring Data, which is following the HAL standard (see references links at the bottom of the page for more details)

For each link, a child object will be rendered that has one required attribute, href. The application can add additional attributes as needed. 

```
{
  "data": [...
  ],
  "page": {...
  },
  "_links": {
    "self" : {
      "href" : "http://localhost:8080/persons/1234"
    },
    "next" : {
      "href" : "http://localhost:8080/persons?page=2&size=5" 
    },
    "prev" : {
      "href" : "http://localhost:8080/persons?page=0&size=5"
    },
    "customerList" : {
      "href" : "http://localhost:8080/customers"
    }  
  }
}
```
### Response Codes
Response codes should be consistent with the REST principles. When researching the codes, start with the API Tutorial.
[Rest API Tutorial's HTTP Status Code List](https://www.restapitutorial.com/httpstatuscodes.html)
Some of the common response codes to use are described below:
 - **200** OK - typical success condition
 - **201** Created - Use when making the request to create new resources. No payload is returned but a Location header should be returned with a link to the newly created resource. 
 - **202** Accepted - The server accepted the request but hasn't processed it. This should be used in an asynchronous scenario.
 - **204** No Content - the request has been completed. Updated metadata can be included in the response headers but the service should not return an entity body
 - **400** Bad Request - client should change it before re-submitting
 - **401** Unauthorized - need to include authorization
 - **403** Forbidden - server understood the request and it is refusing to allow it
 - **404** Not Found - resource not found; note that 0 results in a search operation would yield a 200
 - **409** Conflict - can't be completed due to a state difference in the resource
 - **500** Internal Server Error - unexpected error that needs to be researched by the server team.
 - **503** Service Unavailable - typically, it could return this for a temporary condition but it could also return this during system change and a header that indicates when it would be available.
 - **504** Gateway Timeout - the current service received a timeout from a downstream system

### Naming Conventions for Known Data Types
List of naming conventions for different situations:
- xxxFlag: boolean suffix for a boolean
- xxxInd: boolean suffix for a boolean used as an indicator
- xxxCode: string intended to contain a code
- xxxDescription: free-form string used to describe the object is belongs to
- xxxDate - date-based field
- xxxTime - time-based field

### API Versioning and Backwards-compatibility
API Clients and providers should have a common set of expectations with respect to backwards-compatibility
- major API version is specified on the left side of the URI. Example: /v1/profiles/1234/addresses- backwards-compatibility is maintained at the major version level (i.e. v1)


**Backwards-compatibility changes to a major API version**
- definition of additional *optional* properties in a request
- definition of any additional properties in a response

**Breaking changes:**
- definition of additional *required* properties in a request
- change in a property's data type
- change in a property's format

#### Advice for Client Teams
- Code your unmarshalling logic to handle unexpected optional properties

#### Advice for Provider Teams
When a breaking change to a resource must be introduced and the major version of the service can not change, consider the following strategies:
- introduce a new resource with the change - this may not be elegant but it gets the job done in terms of both supporting existing clients and supporting the new requirement
- use a key value map element in the request or response and load it with the new elements. Since this would be defined as optional, it won't break existing clients. The client(s) that requires the new mandatory information can read it out of the map. With this approach, when a major version change is introduced, the items defined in a map are moved to a properly structure payload location.

TODO define optional  map for extensibility




### Transactional API Design Advice
#### Sequence timeouts between your service and downstream services 
The API being developed should publish its timeout and that timeout should be longer than the service it is calling. The intent is to minimize the possibility that your service returns a timeout when the transaction completes by the downstream resource.


### Pattern Conventions for Known Data Types
"code" fields
do not contain spaces
follow a consistent format. Pick a standard and stick with it. Some options are: 
 - JAVA_CONSTANTS_STYLE 
 - dash-delimited-style 
 - UpperCameCase
 - lowerCamelCase

### Swagger Naming Conventions
#### Definitions
use Upper Camel Case

### Careful with the Enums
discuss how they can cause issues. Make really sure the values won't change; otherwise use a valid values list

### Floating Point Numbers and Precision
TODO discuss use of string and format

### Reference Sample Currency Type



### References

Google JSON Style Guide

<https://google.github.io/styleguide/jsoncstyleguide.xml#params>

json:api

<http://jsonapi.org/format/>

Best Practices for Designing a Pragmatic RESTful API

<http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api>

Apigee – many links and blog engries on REST standards

<https://apigee.com>

Linking and Resource Expansion: REST API Tips

<https://stormpath.com/blog/linking-and-resource-expansion-rest-api-tips>

RESTful API Design Tips from Experience

<https://medium.com/studioarmix/learn-restful-api-design-ideals-c5ec915a430f>

Spring Data - paging and links reference

- <https://docs.spring.io/spring-data/rest/docs/current/reference/html/#_supported_http_methods> - discussion of page and size request parameter fields

- <https://docs.spring.io/spring-data/rest/docs/current/reference/html/#paging-and-sorting.prev-and-next-links> - discussion of previous and next links

- <https://docs.spring.io/spring-data/rest/docs/current/reference/html/#metadata> - additional examples of links to resources

HAL - Hypertext Application Language - details on link relations, Spring Data uses the HAL pattern for its links

- <http://stateless.co/hal_specification.html>

- <https://tools.ietf.org/html/draft-kelly-json-hal-08>

[Rest API Tutorial's HTTP Status Code List](https://www.restapitutorial.com/httpstatuscodes.html)



