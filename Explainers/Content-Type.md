# Content-Type in Resource Timing

Proposal to add a new field `contentType` to PerformanceResourceTiming which holds a string corresponding to the content type of the fetched resource

## Use cases

Servers and CDNs sometimes "swap out" file formats transparently (URL doesn't change) based on User Agent string, `Accept` headers or [Client Hints](https://developer.mozilla.org/en-US/docs/Web/HTTP/Client_hints) when responding to a request for a resource. By providing access to the content type in those cases it would be possible for developers to know exactly which format the server has declared.
 - Useful to understand format adoption for a website
 - From analytics perspective, this could be helpful in segregating user experiences based on the image formats the client decoded.

(Reference : https://github.com/w3c/resource-timing/issues/203#issuecomment-889427414, https://github.com/w3c/resource-timing/issues/203#issuecomment-1109990279)

## API Changes and Example Code

The PerformanceResourceTiming Interface in <a href="https://w3c.github.io/resource-timing/#sec-performanceresourcetiming">resource-timing</a> would be updated to 
```bash
[Exposed=(Window,Worker)]
interface PerformanceResourceTiming : PerformanceEntry {
    ...
    ...
    readonly attribute DOMString contentType;
    ...
    ...
    [Default] object toJSON();
};
```

Sample usage:
```javascript
const entry_list = performance.getEntriesByType("resource");
console.log(entry_list[0].contentType);
```


## Content type Values

The content type value would reflect the [Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) header on the response. It does NOT return the sniffed value.

It would be an empty string if the [Cross-Origin-Resource-Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) [check](https://fetch.spec.whatwg.org/#concept-cors-check) fails


## Potential Spec Changes

Fetch (https://github.com/whatwg/fetch/pull/1481)
- Add content type field to response body info
- In fetch reponse handover 3.3.6,  if request’s mode is not "navigate" or response’s has-cross-origin-redirects is false, set response body info's content type to extracted content type from it's headers list

Resource Timing (https://github.com/w3c/resource-timing/pull/341)
- [4.3](https://w3c.github.io/resource-timing/#sec-performanceresourcetiming) : Adding new field to interface : contentType
- The `contentType` getter steps are to return this's resource info's content type after serializing if its not null and empty string otherwise


## Security/Privacy Considerations
- The content-type is behind CORS check and hence the server has to opt in to make the information available.

### [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)

> 01.  What information might this feature expose to Web sites or other parties,
>      and for what purposes is that exposure necessary?

It exposes the Content-type header value set by the server when the resource was fetched. It is only available when the CORS check passes.

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
- Update 1 - Updated to only reflect the value of Content-type header instead of relying on sniffing
- Update 2 - Allow user agents to truncate values
- Update 3 - Add spec PR links
- Update 4 - Return content-type as is without truncating (discussion : https://github.com/whatwg/fetch/pull/1481#pullrequestreview-1175803751)
- Update 5 - Add Self-Review Questionnaire: Security and Privacy
