# Content-Type in Resource Timing

Proposal to add a new field `contentType` to PerformanceResourceTiming which holds a string corresponding to the content type of the fetched resource

## Use cases

Servers and CDNs sometimes "swap out" file formats transparently (URL doesn't change) based on User Agent string or Client Hints[^1] when responding to a request for a resource. Having access to the content type in those cases would enable knowing exactly what format was being used.
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

The content type value would reflect the [Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) header on the response.

It would be an empty string if the CORS[^2] check fails

Content-type is only reflected if it is one among a set of predefined allowed values. It has to be a mimetype that is identified in https://mimesniff.spec.whatwg.org/#mime-type-groups and is documented in Mimesniff spec.

For instance if we have a Content-Type : `image/png34232` returned from the server, the mimetype would match with `image/png`, the suffix is discarded and only the matched mimetype `image/png` would be exposed.


## Potential Spec Changes

Fetch
- Extracting content type from response's Header list.
- Set content-type to empty string if mode is `navigate` and not same origin.
- Pass in content-type as parameter to `Mark resource timing`

Resource Timing Level 2
- [4.3](https://w3c.github.io/resource-timing/#sec-performanceresourcetiming) : Adding new field to interface : contentType
- Mark resource timing takes new parameter `mimeType` and passes it to Setup resource timing entry
- In Setup resource timing entry, value of `contentType` is set to matched value if the MIMEtype's [essence](https://mimesniff.spec.whatwg.org/#mime-type-essence)'s prefix matches one of [MIME type groups](https://mimesniff.spec.whatwg.org/#mime-type-groups), and otherwise to the empty string
- [4.5](https://w3c.github.io/resource-timing/#sec-cross-origin-resources) : `contentType` would be an empty string if CORS[^2] check fails


[^1]: [Client Hints](https://developer.mozilla.org/en-US/docs/Web/HTTP/Client_hints)
[^2]: [Cross-Origin-Resource-Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS), [CORS check](https://fetch.spec.whatwg.org/#concept-cors-check)

## Changelog
- Update 1 - Updated to only reflect the value of Content-type header instead of relying on sniffing