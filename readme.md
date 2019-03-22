# API Patterns
This page is a collection of "notes to self" on how to go about designing and implementing APIs. I'll reference the sites the information comes from.

## Resources (URIs)
1. use nouns and not verbs
`/profiles/1234 vs. /get-profile/1234 `
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

## HTTP Return Status Codes
### Typcial Codes in Use
- 200 - OK - response body returned
- 201 - new resource was created. Response body not returned by default. However, query parameter could be used to "optionally" return the created resource if the client would like the representation returned.
- 400 - Bad Request - indicates something was wrong with the request. Return this for validation errors on data that is sent by the client and possibly fixable before re-try.
- 401 - Unauthorized - authorization is needed to access the resource. The request has not passed authorization.
- 403 - Forbidden - the request was understood but access is 
- 404 - Not Found - the  specific resource that was requested could not be found. 

### Special Return Code Situations
- When a search or find results in 0 results, return a 200, not a 404. In this case, the search was successful. However, the criteria simply didn't result in any matches. 

## References
1. [5 Basic REST API Design Guidelines](https://blog.restcase.com/5-basic-rest-api-design-guidelines): URI resource guidelines
2. [Spring Data](https://docs.spring.io/spring-data/rest/docs/current/reference/html/#paging-and-sorting): paging and link guidelines
