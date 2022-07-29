# HTTP Status Code in Resource Timing

Proposal to add a new field `responseStatus` to PerformanceResourceTiming which holds an integer corresponding to HTTP status code returned when fetching the resource

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
    readonly attribute short integer responseStatus;
    ...
    ...
    [Default] object toJSON();
};
```

Sample usage:
```javascript
const entry_list = performance.getEntriesByType("resource");
console.log(entry_list[0].responseStatus);
```


## Response Status Code Values

The status code values would have a 1-1 mapping with the fetch [status](https://fetch.spec.whatwg.org/#concept-status) which is available on the [response](https://fetch.spec.whatwg.org/#concept-response-status) 

The status code would be `0` if the CORS(Cross Origin Resource Sharing[^1]) check[^2] fails.


## Potential Spec Changes

Fetch ([whatwg/fetch#1468](https://github.com/whatwg/fetch/pull/1468))
- In [fetch response handover](https://fetch.spec.whatwg.org/#fetch-finale) response'status is passed in as parameter to mark resource timing 

Resource Timing Level 2 ([w3c/resource-timing#335](https://github.com/w3c/resource-timing/pull/335))
- [4.3](https://w3c.github.io/resource-timing/#sec-performanceresourcetiming) : Adding new field to interface : responseStatus
- Getter steps for `responseStatus` returns response status.
- New parameter `responseStatus` in [Mark resource time](https://w3c.github.io/resource-timing/#dfn-mark-resource-timing) which would in turn pass it to [Setup the resource timing entry](https://w3c.github.io/resource-timing/#ref-for-dfn-setup-the-resource-timing-entry-1) where it would be used  to set the entry's responseStatus.
- [4.5](https://w3c.github.io/resource-timing/#sec-cross-origin-resources) : `responseStatus` would be `0` if CORS check[^2] fails

## Security/Privacy Considerations
- The status code is behind CORS check and hence the server has to opt in to make the information available.

### [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)

> 01.  What information might this feature expose to Web sites or other parties,
>      and for what purposes is that exposure necessary?

It exposes the HTTP status code returned when particular resource was fetched. It is only available when the CORS check passes. Knowing the status code can enable analysis by segregation of resources based on the returned status.  For instance analysis of 4xx and 5xx responses would be easier.

> 02.  Do features in your specification expose the minimum amount of information
>      necessary to enable their intended uses?

Yes

> 03.  How do the features in your specification deal with personal information,
>      personally-identifiable information (PII), or information derived from
>      them?

It does not deal with such information.

> 04.  How do the features in your specification deal with sensitive information?

It does not deal with sensitive information.

> 05.  Do the features in your specification introduce new state for an origin
>      that persists across browsing sessions?

No.

> 06.  Do the features in your specification expose information about the
>      underlying platform to origins?

No.

> 07.  Does this specification allow an origin to send data to the underlying
>      platform?

No.

> 08.  Do features in this specification enable access to device sensors?

No.

> 09.  Do features in this specification enable new script execution/loading
>      mechanisms?

No.

> 10.  Do features in this specification allow an origin to access other devices?

No.

> 11.  Do features in this specification allow an origin some measure of control over
>      a user agent's native UI?

No.

> 12.  What temporary identifiers do the features in this specification create or
>      expose to the web?

None.

> 13.  How does this specification distinguish between behavior in first-party and
>      third-party contexts?

No distinction.

> 14.  How do the features in this specification work in the context of a browser’s
>      Private Browsing or Incognito mode?

No difference.

> 15.  Does this specification have both "Security Considerations" and "Privacy
>      Considerations" sections?

No.

> 16.  Do features in your specification enable origins to downgrade default
>      security protections?

No.

> 17.  How does your feature handle non-"fully active" documents?

No difference.

> 18.  What should this questionnaire have asked?

None.

## Changelog
- Update 1 - Spec changes in fetch modified to set during HTTP Network Fetch
- Update 2 - Add Self-Review Questionnaire: Security and Privacy
- Update 3 - Replace TAO check with CORS check for status code
- Update 4 - Rename field to responseStatus
- Update 5 - Spec changes to pass in response status through Mark resource timing

[^1]: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
[^2]: https://fetch.spec.whatwg.org/#concept-cors-check
