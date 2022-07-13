# HTTP Status Code in Resource Timing

Proposal to add a new field `responseStatusCode` to PerformanceResourceTiming which holds an integer corresponding to HTTP status code returned when fetching the resource

## Use cases

Currently there is no straightforward way to tell if a resource failed loading for developers using the Resource Timing API. 
https://github.com/w3c/resource-timing/pull/19 helps in the case of errored responses, but doesn't provide indication for that for requests that got an error response.
- RUM customers wanting to analyze 4xx and 5xx responses for their own sites would benefit
(Reference : https://github.com/w3c/resource-timing/issues/90, https://github.com/w3c/navigation-timing/issues/126#issuecomment-632429543)


## API Changes and Example Code

The PerformanceResourceTiming Interface in <a href="https://w3c.github.io/resource-timing/#sec-performanceresourcetiming">resource-timing</a> would be updated to 
```bash
[Exposed=(Window,Worker)]
interface PerformanceResourceTiming : PerformanceEntry {
    ...
    ...
    readonly attribute short integer responseStatusCode;
    ...
    ...
    [Default] object toJSON();
};
```

Sample usage:
```javascript
const entry_list = performance.getEntriesByType("resource");
console.log(entry_list[0].responseStatusCode);
```


## Response Status Code Values

The status code values would have a 1-1 mapping with the fetch [status](https://fetch.spec.whatwg.org/#concept-status) which is available on the [response](https://fetch.spec.whatwg.org/#concept-response-status) 

It would be `0` if the TAO check fails


## Potential Spec Changes

Fetch ([whatwg/fetch#1468](https://github.com/whatwg/fetch/pull/1468))
- New integer field response-status added to [response body info](https://fetch.spec.whatwg.org/#response-body-info), default 0
- [HTTP Network Feature](https://fetch.spec.whatwg.org/#http-network-fetch) in step 9.5, set response's body info's response-status to HTTP status code after step 3 in while loop

Resource Timing Level 2 ([w3c/resource-timing#335](https://github.com/w3c/resource-timing/pull/335))
- [4.3](https://w3c.github.io/resource-timing/#sec-performanceresourcetiming) : Adding new field to interface : responseStatusCode
- Getter steps for `responseStatusCode` returns [resource info](https://w3c.github.io/resource-timing/#dfn-resource-info)'s response status.
- [4.5](https://w3c.github.io/resource-timing/#sec-cross-origin-resources) : `responseStatusCode` would be `0` if TAO check fails

## Security/Privacy Considerations
- The status code is behind TAO check and hence the server has to opt in to make the information available.

## Changelog
- Update 1 - Spec changes in fetch modified to set during HTTP Network Fetch
