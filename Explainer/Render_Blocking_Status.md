# Render Blocking Status

Proposal to add a new field `render blocking status` to PerformanceResourceTiming which holds a string highlighting the status of stylesheets and scripts.

## Use cases

RUM providers could then easily determine the render blocking resources without having to rely on complex heurestics to determine the same.
This would enable analysis for below scenaarios and possibly more.
- Determine resources downloaded before FCP but were not render blocking? (and hence, could've been delayed)
- Determine resources that were render blocking, but weren't discovered early enough? (and hence, could've benefited from being preloaded)
(Reference : https://github.com/w3c/resource-timing/issues/262)


## API Changes and Example Code

The PerformanceResourceTiming Interface in <a href="https://w3c.github.io/resource-timing/#sec-performanceresourcetiming">resource-timing</a> would be updated to 
```bash
[Exposed=(Window,Worker)]
interface PerformanceResourceTiming : PerformanceEntry {
    ...
    ...
    readonly attribute DOMString renderBlockingStatus;
    ...
    ...
    [Default] object toJSON();
};
```

Sample usage:
```javascript
const entry_list = performance.getEntriesByType("resource");
console.log(entry_list[0].renderBlockingStatus);
```


## Render Blocking Status Values

The following values are proposed for the resource blocking status

`blocking` - A potentially render blocking resource

`non-blocking` - Non blocking resource

We primarily reuse the value of the [render blocking](https://fetch.spec.whatwg.org/#request-render-blocking) boolean associated with fetch [request](https://fetch.spec.whatwg.org/#concept-request)


## Potential Spec Changes

Fetch ([whatwg/fetch#1449](https://github.com/whatwg/fetch/pull/1449))
- New boolean field render-blocking added to [fetch timing info](https://fetch.spec.whatwg.org/#fetch-timing-info)
- [Fetching](https://fetch.spec.whatwg.org/#fetching) is modified to set [fetch timing info](https://fetch.spec.whatwg.org/#fetch-timing-info)'s render-blocking field using request's [render-blocking](https://fetch.spec.whatwg.org/#request-render-blocking)

Resource Timing Level 2 ([w3c/resource-timing#327](https://github.com/w3c/resource-timing/pull/327))
- [4.3](https://w3c.github.io/resource-timing/#sec-performanceresourcetiming) : Adding new field to interface : renderBlockingStatus
- Getter steps for `renderBlockingStatus` return `"blocking"` if [timing-info](https://w3c.github.io/resource-timing/#dfn-timing-info)'s newly added render-blocking field is `true`, else `"non-blocking"`

## Changelog
- Update 1 - Spec changes modified to reuse `Request`'s `render-blocking` boolean.
