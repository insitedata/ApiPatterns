# API Patterns
This page is a collection of "notes to self" on how to go about designing and implementing APIs. I'll reference the sites the information comes from.

# Resource, Request and Response Guidelines
## Headers
TODO

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
### Query Parameters
1. use case-sensitive keys `/profiles?firstName=joe&lastName=smith`
2. nice that the query parameter names can match the key name in a json payload. For keys can match json keys returned in a payload

## HTTP Return Status Codes
### Typcial Use of Codes
- **200** - OK - response body returned
- **201** - new resource was created. Response body not returned by default. However, query parameter could be used to "optionally" return the created resource if the client would like the representation returned.
- **400** - Bad Request - indicates something was wrong with the request. Return this for validation errors on data that is sent by the client and possibly fixable before re-try.
- **401** - Unauthorized - authorization is needed to access the resource. The request has not passed authorization.
- **403** - Forbidden - the request was understood but access is 
- **404** - Not Found - the  specific resource that was requested could not be found. 
TODO: discuss timeout 

### Special Return Status Situations
- When a search or find results in 0 results, return a 200, not a 404. In this case, the search was successful. However, the criteria simply didn't result in any matches. 

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
1. [5 Basic REST API Design Guidelines](https://blog.restcase.com/5-basic-rest-api-design-guidelines): URI resource guidelines
2. [Spring Data](https://docs.spring.io/spring-data/rest/docs/current/reference/html/#paging-and-sorting): paging and link guidelines

