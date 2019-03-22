# API Patterns
This page is a collection of "notes to self" on how to go about designing and implementing APIs. I'll reference the sites the information comes from. As this expands, I'll add swagger examples and provide those files in the repo.

# Resource, Request and Response Guidelines
## Headers
TODO: discuss caching, etags, correlation ids, 

## Resources (URIs)
1. use nouns and not verbs
`/profiles/1234` vs. `/get-profile/1234 `
1. use Plural resources `/profiles`
1. delimit complex names using spinal case (hyphens) in all lower case words, not underscores or a camel case variation.
```
/profiles/1234/first-name   : good and readable
/profiles/1234/firstname    : not as good but still readable
/profiles/1234/first_name   : not good
```
## Query Parameters
1. use case-sensitive keys `/profiles?firstName=joe&lastName=smith`
2. nice that the query parameter names can match the key name in a json payload. For keys can match json keys returned in a payload
1. an array of values for a parameter should be passed as a comma-delimited list. This is the default approach in swagger [see items collection format](https://www.restapitutorial.com/httpstatuscodes.html) . Example: `colors=brown,blue,green`. Other formats are acceptable too and this varies a bit depending on the API. So use CSV format as the default but recognize that other formats may be better for consistency within an API. See Swagger link for more details.

## HTTP Return Status Codes
### Typical Use of Codes
- **200** - OK - response body returned
- **201** - new resource was created. Response body not returned by default.
- **400** - Bad Request - indicates something was wrong with the request. Return this for validation errors on data that is sent by the client and possibly fixable before re-try.
- **401** - Unauthorized - authorization is needed to access the resource. The request has not passed authorization.
- **403** - Forbidden - the request was understood but access is 
- **404** - Not Found - the  specific resource that was requested could not be found. 
- **408** - Request timeout - the request was not processed completely before a timeout occurred. The request "may have" succeeded. However, the client can not be sure. Idempotent requests can be retried. 

### Special HTTP Status Situations
- When a search or find results in 0 results, return a 200, not a 404. In this case, the search was successful. However, the criteria simply didn't result in any matches. 
- while a 201 does not typically return a payload, one can optionally return the representation of the resource if directed when a known query parameter is provided in the request. Example: `POST /profiles?returnResource=true`

# Paging
TODO - refer to Spring Data reference 
Query parameters: `...\widgets?page=1&size=5`
Payload: size, totalElements, totalPages, number

# Payload Guidelines
### Use a json wrapper around typical requests and responses
- Normal request and response payloads use a wrapper
```
{"data": "some complex object"}
```
- Error response contains a list of error objects

# Swagger 2.0 Guidelines
### Currency and Decimals
All decimal objects that require precision should be passed as strings, not as numbers. Numbers can be interpretted differently by different engines. Passing values as strings is safer.
An alternative is to multiple the value by the number of decimal places the value contains.
Swagger definitions:
~~~~
NumberTwoDecimals:
type: string
pattern: '^((-?[0-9]+)|(-?([0-9]+)?[.][0-9]{2}))$'
maxLength: 32
description: >
  decimal value formatted as a string that requires two decimal places
  when a decimal point is provided
~~~~
# Runtime Recommendations
- Timeout coordination - when a chain of resources process a request, coordinate the timeouts so that the very back-end resource has the shortest timeout. Each layer above as a slightly shorter timeout. If timeouts are not coordinated and an API higher in the calling chain times out before the service it calls, a transaction may process while the client receives an error.

# References
1. [REST API Tutorial](https://www.restapitutorial.com/httpstatuscodes.html): HTTP status codes
2. [5 Basic REST API Design Guidelines](https://blog.restcase.com/5-basic-rest-api-design-guidelines): URI resource guidelines
3. [Yet Another RESTful API Standard (YARAS)](https://github.com/darrin/yaras/blob/master/restful-standards.md): good, common sense API advice
4. [Spring Data](https://docs.spring.io/spring-data/rest/docs/current/reference/html/#paging-and-sorting): paging and link guidelines
5. [OpenAPI (Swagger) 2.0 Specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#items-object), [Developer Docs](https://swagger.io/docs/specification/2-0/basic-structure/)
6. [OpenAPI (Swagger) 3.0.2 Specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md), [Developer Docs](https://swagger.io/docs/specification/basic-structure/)
7. [RAML 1.0 Specification](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md)
