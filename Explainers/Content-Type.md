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

User agents are allowed to truncate the Content-type values. For instance if we have a Content-Type : `image/png34232` returned from the server, user agents may choose to only return `image/png`.


## Potential Spec Changes

Fetch
- Add content type field to response body info
- In fetch reponse handover 3.3.6, set response body info's content type to empty string if mode is `navigate` and no cross origin redirects happened or if response is an opaque response.

Resource Timing Level 2
- [4.3](https://w3c.github.io/resource-timing/#sec-performanceresourcetiming) : Adding new field to interface : contentType
- The `contentType` getter steps are to return this's resource info's content type's essence. The user agent may truncate the value in an implementation-defined way.
- [4.5](https://w3c.github.io/resource-timing/#sec-cross-origin-resources) : `contentType` would be an empty string if [Cross-Origin-Resource-Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) [check](https://fetch.spec.whatwg.org/#concept-cors-check) fails


## Changelog
- Update 1 - Updated to only reflect the value of Content-type header instead of relying on sniffing
- Update 2 - Allow user agents to truncate values
